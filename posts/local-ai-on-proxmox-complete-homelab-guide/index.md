# Local AI on Proxmox: The Complete Homelab Guide (2026)

> How to run local LLMs, voice and image generation on a Proxmox homelab: GPU choice, LXC vs VM, Ollama, Open WebUI, Tailscale access, agents and backups.

Source: https://exitcode0.net/posts/local-ai-on-proxmox-complete-homelab-guide/
Author: Tom Cocking (https://tomcocking.com)
Published: 2026-11-10
Updated: 2026-11-10
Tags: local-ai, proxmox, ollama, lxc, nvidia, tailscale, home-assistant, claude-code, self-hosted


Everything AI-related on this site runs on one Proxmox host in a cupboard. A GPU in an LXC container serves Ollama, a web front end sits on a VM next to it, Home Assistant talks to local speech models for voice, and my coding agents work from an always-on VM that I reach over Tailscale. None of it costs per token, none of it leaves the house, and the whole stack has survived every host reboot since I built it.

This post is the map. Each section says what the piece is for, what I run, and links to the post where I actually built it. If you're starting from nothing, read it top to bottom. If you're adding one piece, jump to that section.

## Why run AI locally at all

Three reasons, in the order they mattered to me.

**Cost.** A long agent run against a frontier API bills by the token and the meter keeps running while it thinks. A local model on a card you already own costs electricity. For the boring work, summarising, drafting, classifying, rewriting, a local 8B model is good enough and it's free.

**Privacy.** Home Assistant voice commands, camera descriptions, documents, the inside of my own repos. I'd rather that stayed on the LAN.

**Availability.** No rate limits, no outages, no model being deprecated under you. The model you pulled last year still runs exactly the same next year.

The trade-off is capability. Local models are a tier or two behind the frontier for hard reasoning. I still use Claude for real coding work, which is why the [dev VM setup](https://exitcode0.net/posts/dumb-terminal-dev-vm-tailscale-claude-code/) exists. Local AI is the complement, not the replacement, and the [Ollama post](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/) says the same.

## Hardware: what actually matters

VRAM is the only number that matters for which models you can run. Everything else affects speed.

| Model size | Typical VRAM needed (Q4) | Fits on |
|---|---|---|
| 3B (llama3.2) | about 3GB | anything with 4GB+ |
| 7B to 8B (llama3.1, mistral) | 5 to 6GB | 8GB cards |
| 13B to 14B | 9 to 10GB | 12GB cards |
| 27B to 32B | 18 to 20GB | 24GB cards |
| 70B | 40GB+ | two cards or CPU offload |

A used 12GB card is the sweet spot for a homelab. It runs every 8B model with room for context, handles 13B, and costs less than a month of API bills for a heavy user. If you still have a [mining-era RTX 3060](https://exitcode0.net/posts/more-rtx-3060-nicehash-overclocking/) in a drawer, that's a 12GB card and it's exactly what this stack wants.

### What a 3060 actually does

This is the card in my Ollama container: an RTX 3060 with 12 GB, driver 610.43.02, on a Proxmox VE 8.4 host. I measured these on 11 October 2026 with Ollama 0.33.3, one 200-token answer from the same prompt at temperature 0 with thinking off, after a warm-up run. The server's default context was 32,768 tokens.

| Model | Generation | Prompt processing | VRAM in use |
|---|---|---|---|
| gemma4:e2b | 128 tokens/s | 1,289 tokens/s | 3.1 GB |
| phi-4-mini 3.8B (Q8_0) | 68 tokens/s | 1,448 tokens/s | 6.5 GB |
| qwen3.5:9b | 53 tokens/s | 448 tokens/s | 7.0 GB |
| Qwen3.5 9B DeepSeek-V4-Flash (Q6_K) | 42 tokens/s | 87 tokens/s | 8.3 GB |
| gemma4:12b | 38 tokens/s | 437 tokens/s | 8.4 GB |

The VRAM column is the card's total memory in use read from `nvidia-smi` straight after each run, so it includes the context. Anything over about 30 tokens a second feels instant in chat. All five fit in 12 GB. Only one is loaded at a time on my server.

What those models can do for real work is in two posts: [which model works as a Home Assistant conversation agent](https://exitcode0.net/posts/home-assistant-ollama-conversation-agent-local-llm/) and [Claude Code against a local model](https://exitcode0.net/posts/claude-code-local-ollama-model-anthropic-api/). The short version is that the best model for one job was not the best for the other.

Two other things to check before you buy or repurpose a card:

- **Does the host have an iGPU or a second GPU?** If not, handing the only card to a VM leaves the host with no display. That decision alone decides LXC versus VM for me (next section).
- **Power and slot.** Proxmox boxes tend to be small. Check the PSU has the PCIe power connectors and the case has the length.

## Proxmox layout: LXC or VM for the GPU

This is the first real decision and the [Ollama post](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/) goes into the reasoning. The short version:

| | LXC container | VM with PCI passthrough |
|---|---|---|
| Driver lives on | the Proxmox host | inside the VM |
| Host keeps the GPU | yes (shared via device nodes) | no (card is handed over) |
| Overhead | near zero | small, but real |
| Isolation | kernel shared with host | full |
| Multiple consumers | several containers can share one card | one VM owns it |
| Setup pain | cgroup rules and bind mounts | IOMMU, vfio, UEFI fiddling |

I run Ollama in an unprivileged LXC. The driver is installed once on the host, the container gets the device nodes and the CUDA libraries bind-mounted in, and the host still has a display. The full container config is in the Ollama post and it's the thing I'd hand to an agent first.

Go the VM route if you need Windows, a desktop, or you want hard isolation. Everything in the rest of this guide works either way, because Ollama is just a service on a LAN IP.

## The model server: Ollama

Ollama is the piece everything else talks to. It pulls models and serves a REST API on port 11434. By default it unloads an idle model after five minutes so the card isn't tied up.

The build is here: [Running Ollama in a Proxmox LXC with NVIDIA GPU Passthrough](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/). It covers the host driver install, the container config with the cgroup rules and bind mounts, the `OLLAMA_HOST=0.0.0.0` override so the LAN can reach it, and a troubleshooting section for the four ways it fails.

My server overrides four defaults, set as systemd environment variables in the container:

| Variable | My value | Why |
|---|---|---|
| `OLLAMA_HOST` | `0.0.0.0:11434` | So other machines on the LAN can reach it |
| `OLLAMA_CONTEXT_LENGTH` | `32768` | Agents send a lot before you type anything |
| `OLLAMA_MAX_LOADED_MODELS` | `1` | One model at a time fits the card |
| `OLLAMA_KEEP_ALIVE` | `-1` | Keep the model in VRAM, so there's no reload delay |

That last pair has a cost. With one model resident forever, the first request after switching models waited about 20 seconds for a reload in my Home Assistant test.

The two commands you'll use most:

```bash
# from the Proxmox host, into the container
pct exec 106 -- ollama pull llama3.1
pct exec 106 -- nvidia-smi --query-gpu=name,memory.used,memory.total --format=csv
```

If you want to try models before committing a GPU to a container, [LM Studio on a desktop](https://exitcode0.net/posts/testing-llama3-with-lmstudio/) is the zero-setup way to find out whether an 8B model is good enough for what you want. I did that with [Llama 3](https://exitcode0.net/posts/testing-llama3-with-lmstudio/) and [Llama 3.1](https://exitcode0.net/posts/testing-llama3_1-with-gpt4all/) before building the container.

## A front end: Open WebUI

The API is fine for scripts and agents. For chatting, you want Open WebUI. It's a ChatGPT-style interface that points at your Ollama server, keeps conversation history and handles several users. It doesn't need the GPU itself, so it can live on any machine on the network. Mine runs in Docker on one of the VMs.

```yaml
services:
  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    ports:
      - "3000:8080"
    environment:
      - OLLAMA_BASE_URL=http://<ollama-ip>:11434
    volumes:
      - open-webui:/app/backend/data
    mem_limit: 2g
    restart: unless-stopped

volumes:
  open-webui:
```

Browse to port 3000 and create an account. The first account becomes the admin, so do that before anyone else on the network does.

I tested version 0.12.0 against the Ollama container. All five models I'd pulled showed up in the model picker with no further setup, and it sat at about 0.7 to 1.2 GB of RAM while idle or answering. The first reply after loading a model is slow: a 9B answer took 38 seconds in my first test, most of it the model loading.

![Open WebUI showing a conversation with qwen3.5:9b about backing up a Proxmox LXC container, with a three item task list generated under the reply](https://exitcode0.net/images/open-webui-ollama-chat.jpg)

The same Ollama server feeds Home Assistant and the coding agents below, so you pick models once. I've also run Open WebUI with a Tailscale container in the same compose project, which gives it its own tailnet name. When I restarted mine after a few months, the Tailscale container couldn't log back in, so it's LAN only for now. Plain LAN access is the simpler default.

## Reaching it from anywhere: Tailscale

Nothing in this stack is exposed to the internet. The Ollama container has a fixed LAN address and listens on all interfaces, so anything on the home network can use it. For everywhere else it joins the tailnet with the standard Tailscale install inside the container. The [Ollama post](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/) has every step: the TUN device lines in the container config, the install, a key file you shred after use, and `tailscale up --hostname=ollama-lxc`. That's all it needs. After that the API is reachable from a phone, a laptop or a hotel at `http://ollama-lxc:11434`, with no port forwarding and no change to the Proxmox host.

The same pattern covers the other pieces: [Home Assistant with TLS over Tailscale](https://exitcode0.net/posts/homeassistant-tls-with-tailscale/) for the voice assistant, and the [always-on dev VM](https://exitcode0.net/posts/dumb-terminal-dev-vm-tailscale-claude-code/) that I work from. You need the HTTPS for Home Assistant voice anyway, because browsers won't give a page the microphone over plain HTTP, as I found out in the [local voice post](https://exitcode0.net/posts/home-assistant-local-voice-whisper-piper-docker/).

One tailnet means one set of names. At home, Open WebUI, Home Assistant and the coding agents use the LAN address. Away from home, the tailnet name does the same job.

## Agents: the part that made this worth building

Local models on their own are a chat box. The step change was pointing coding agents at them and at the rest of the homelab.

- [A persistent dev VM over Tailscale](https://exitcode0.net/posts/dumb-terminal-dev-vm-tailscale-claude-code/) runs T3 Code and Claude Code around the clock, so the laptop is just a window onto it. That's where the agent work happens.
- [Giving Claude Code a real browser](https://exitcode0.net/posts/claude-code-browser-docker-playwright-mcp/) puts a headful Chromium in a container on that VM, so an agent can shop, scrape and screenshot sites that block plain fetches.
- [Giving Claude Code eyes inside Docker containers](https://exitcode0.net/posts/claude-code-wsl2-docker-compose-logs/) is the same idea for logs: SSH access so the agent reads container output instead of you pasting it.
- [Pi Agent's extension model](https://exitcode0.net/posts/understanding-pi-agent-extension-model/) and the [dependency audit extension I built for it](https://exitcode0.net/posts/pi-agent-dep-audit-supply-chain/) cover the locally hosted agent that runs against Ollama rather than a frontier API.

- [Claude Code against a local Ollama model](https://exitcode0.net/posts/claude-code-local-ollama-model-anthropic-api/) is the other direction: the same agent, pointed at the card in this box instead of the API. On small tasks it managed a bug fix in about a minute. It also reported success when its tests couldn't run, so I check.

The frontier model does the hard reasoning. The local model does the volume. The homelab is what lets both run unattended.

## Voice: Home Assistant without the cloud

Home Assistant's voice pipeline needs speech-to-text and text-to-speech, and both can run locally in Docker:

- [Wyoming Whisper](https://exitcode0.net/posts/wyoming-whisper-docker-compose/) for speech-to-text.
- [Wyoming Piper](https://exitcode0.net/posts/wyoming-piper-docker-compose/) for text-to-speech.

Both are small enough to run on CPU, and [wiring them into Assist](https://exitcode0.net/posts/home-assistant-local-voice-whisper-piper-docker/) is one compose file. Add Home Assistant's Ollama integration pointed at the container above and the conversation agent is local too. I tested that on three models and wrote up [which one worked](https://exitcode0.net/posts/home-assistant-ollama-conversation-agent-local-llm/). Keep Home Assistant's own matcher in front of it, so the common commands never wait for the model.

That's the full "Alexa, but it's mine" stack.

## Images: Stable Diffusion

The one piece I run on demand rather than always-on. [Stable Diffusion in April 2024](https://exitcode0.net/posts/generating-artwork-with-stable-diffusion-april-2024/) covers what local image generation was good for and where it fell short. It shares the GPU with Ollama only if Ollama lets go of the card. With my server keeping a model loaded forever, I'd unload it first (`ollama stop <model>`) and check `nvidia-smi` before starting an image run.

## Backups and moving house

The Ollama container is just an LXC, so it backs up with `vzdump` like anything else. Model files are big (a 300GB rootfs in my case) so exclude them from nightly backups and re-pull instead; the config is what matters.

If the container needs to move to a different host, [migrating between standalone Proxmox nodes](https://exitcode0.net/posts/migrate-vm-between-standalone-proxmox-nodes/) covers streaming the backup over SSH without building a cluster. Containers restore with `pct restore` rather than `qmrestore`, and the GPU bind mounts need the driver already installed on the destination host.

## What I run today

| Piece | Where | Post |
|---|---|---|
| Ollama + GPU | unprivileged LXC 106 on the Proxmox host, RTX 3060 12 GB, 300 GB disk | [Ollama in a Proxmox LXC](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/) |
| Open WebUI | Docker Compose on a VM, LAN only | section above |
| Tailscale | in the Ollama container, the dev VM and Home Assistant | [Ollama in a Proxmox LXC](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/), [HA TLS with Tailscale](https://exitcode0.net/posts/homeassistant-tls-with-tailscale/) |
| Coding agents | always-on dev VM, plus [Claude Code on the local model](https://exitcode0.net/posts/claude-code-local-ollama-model-anthropic-api/) | [Dumb terminal dev VM](https://exitcode0.net/posts/dumb-terminal-dev-vm-tailscale-claude-code/) |
| Agent browser | Chromium container on the dev VM | [Claude Code browser](https://exitcode0.net/posts/claude-code-browser-docker-playwright-mcp/) |
| Voice | Whisper and Piper in Docker | [Whisper](https://exitcode0.net/posts/wyoming-whisper-docker-compose/), [Piper](https://exitcode0.net/posts/wyoming-piper-docker-compose/), [wiring them into Assist](https://exitcode0.net/posts/home-assistant-local-voice-whisper-piper-docker/) |
| Conversation agent | Ollama model, Home Assistant matcher first | [Which model works](https://exitcode0.net/posts/home-assistant-ollama-conversation-agent-local-llm/) |
| Images | Stable Diffusion, on demand | [Stable Diffusion](https://exitcode0.net/posts/generating-artwork-with-stable-diffusion-april-2024/) |

## What's coming

Posts I haven't written yet in this cluster:

1. Best GPU for Ollama in a UK homelab (what to buy used, and why 12GB)
2. Proxmox VM vs LXC for Ollama, with numbers
3. Updating NVIDIA drivers on the host without breaking LXC passthrough
4. Best Ollama models for a 12GB card
5. Open WebUI in its own LXC

Each one will link back here, and this page will be updated as they land.

## Or just hand this page to your agent

This page is the map rather than a build, so the handoff is for planning:

```text
Read https://exitcode0.net/posts/local-ai-on-proxmox-complete-homelab-guide/ and the linked Ollama post, then propose a plan for running local AI on my Proxmox host. Before proposing anything, check what GPU is installed (nvidia-smi or lspci on the host), whether the host has a second GPU or iGPU, how much VRAM there is, and which container IDs and storage pools are free. Recommend LXC or VM based on what you find and explain why. Do not change anything on the host.
```

The page gives the agent the architecture and the decision points. Only you know what card is in the box and what else the host is already doing.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/local-ai-on-proxmox-complete-homelab-guide/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
