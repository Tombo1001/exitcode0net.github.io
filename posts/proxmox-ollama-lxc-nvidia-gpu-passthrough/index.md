# Running Ollama in a Proxmox LXC with NVIDIA GPU Passthrough

> Set up Ollama in a Proxmox LXC with NVIDIA GPU passthrough, Tailscale access and Open WebUI. Run local LLMs with GPU acceleration on your own hardware.

Source: https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/
Author: Tom Cocking (https://tomcocking.com)
Published: 2026-03-22
Updated: 2026-10-08
Tags: proxmox, ollama, nvidia, lxc, tailscale, llm, self-hosted, homelab, linux


Running large language models locally is genuinely useful — no API costs, no rate limits, and your data stays on your own hardware. The catch is getting GPU acceleration working inside a Proxmox LXC container, which involves a few non-obvious steps around driver installation and cgroup device passthrough.

### Why LXC and not a VM?
 
VM GPU passthrough wasn't an option here — no iGPU meant the host would have had no display output once the card was handed off. LXC was the practical solution, and it turns out to be a good one anyway: containers share the host kernel directly, so the GPU stays bound to the host's NVIDIA driver and the container accesses it via bind-mounted device nodes and cgroup permissions. On top of that, LXCs are lighter weight than VMs, with less overhead and near-instant startup times. For a dedicated service like Ollama, it's a solid fit.

This guide documents the full process: NVIDIA driver setup on the PVE host, creating and configuring the LXC, installing Ollama with GPU support, and optionally exposing it over a Tailscale network. It's been tested on Proxmox VE 8 (Debian Bookworm) with a consumer NVIDIA GPU. We will push the boat out and include Tailscale to make the Ollama API available within our Tailnet (optional).

> **Updated October 2026:** Proxmox VE 9 (Debian 13 Trixie) has been out for a while. I haven't rebuilt this container on it yet, but nothing below is version-specific apart from the Debian template name, so swap `debian-12-standard` for the `debian-13-standard` template if you're on VE 9. The cgroup2 device rules and the bind mounts are the same on both. Also added: a model size versus VRAM table, troubleshooting as proper headings, and the agent handoff block at the end.

If you'd rather try a few models on a desktop before committing a GPU to a container, I did exactly that in [testing Llama 3 with LM Studio](https://exitcode0.net/posts/testing-llama3-with-lmstudio/). And if the container ends up on the wrong host later, [migrating between standalone Proxmox nodes](https://exitcode0.net/posts/migrate-vm-between-standalone-proxmox-nodes/) covers moving it without building a cluster.

---

## Prerequisites

### Disable Secure Boot

Proprietary NVIDIA drivers won't load with Secure Boot enabled. Head into your UEFI firmware settings and disable it before going any further.

> **Note:** "Non-free" in Debian just means proprietary or closed-source — not that you have to pay for it. NVIDIA drivers are free to use.

### Install NVIDIA Drivers on the PVE Host

The drivers need to be installed on the **host**, not just inside the container. The container will bind-mount the necessary device nodes and libraries directly from the host.

```bash
# Add non-free and non-free-firmware repositories
sed -i 's/main contrib/main contrib non-free non-free-firmware/' /etc/apt/sources.list

apt update
apt install -y pve-headers nvidia-driver
```

Then reboot the host:

```bash
reboot
```

> **Heads up:** Any VMs with `onboot: 1` will restart automatically when PVE comes back up. Brief downtime is unavoidable.

After the reboot, verify the driver loaded correctly:

```bash
nvidia-smi
```

You should see your GPU listed with driver version and memory info. If this doesn't work, don't proceed — the passthrough won't function without working host drivers.

---

## Create the LXC Container

Download a Debian 12 template if you don't already have one, then create the container. Adjust storage pool name, resource allocation, and container ID to suit your setup.

```bash
# Download Debian 12 template
pveam update
pveam download local debian-12-standard_12.7-1_amd64.tar.zst

# Create the container
pct create 106 local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \
  --hostname ollama \
  --storage local-lvm \
  --rootfs local-lvm:300 \
  --memory 8192 \
  --swap 512 \
  --cores 4 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --features nesting=1 \
  --unprivileged 1 \
  --onboot 1
```

A few things worth noting here:

- **`--unprivileged 1`** — keep the container unprivileged where possible; the cgroup rules below handle GPU access
- **`--features nesting=1`** — required for systemd to work correctly inside the container
- **`300G rootfs`** — LLM model weights are large; `llama3` alone is around 4–5GB, so space adds up quickly

---

## Configure GPU Passthrough

This is the critical part. Edit the container config file at `/etc/pve/lxc/106.conf` (replace `106` with your container ID) and append the following:

```
# NVIDIA GPU passthrough
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 235:* rwm
lxc.cgroup2.devices.allow: c 226:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-caps/nvidia-cap1 dev/nvidia-caps/nvidia-cap1 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-caps/nvidia-cap2 dev/nvidia-caps/nvidia-cap2 none bind,optional,create=file
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/libcuda.so.1 usr/lib/x86_64-linux-gnu/libcuda.so.1 none bind,optional,create=file
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/libcuda.so usr/lib/x86_64-linux-gnu/libcuda.so none bind,optional,create=file
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/libnvidia-ml.so.1 usr/lib/x86_64-linux-gnu/libnvidia-ml.so.1 none bind,optional,create=file
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/libnvidia-ptxjitcompiler.so.1 usr/lib/x86_64-linux-gnu/libnvidia-ptxjitcompiler.so.1 none bind,optional,create=file
lxc.mount.entry: /usr/bin/nvidia-smi usr/bin/nvidia-smi none bind,optional,create=file
```

The cgroup rules grant the container access to the NVIDIA device nodes (major numbers 195, 235, 226). The mount entries bind the GPU device files and the CUDA libraries from the host into the container. The `optional` flag means the container will still start even if a device node doesn't exist — useful if your GPU doesn't expose all of these.

### Configure TUN Device (Tailscale)

If you plan to run Tailscale inside the container, append this as well:

```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

### Start the Container

```bash
pct start 106
```

---

## Install Ollama

Ollama provides a convenience install script that handles everything:

```bash
pct exec 106 -- bash -c 'curl -fsSL https://ollama.com/install.sh | sh'
```

By default Ollama only binds to `127.0.0.1`, which means it's only accessible from inside the container itself. To reach it from your LAN or other services, override this with a systemd drop-in:

```bash
pct exec 106 -- mkdir -p /etc/systemd/system/ollama.service.d
pct exec 106 -- bash -c 'cat > /etc/systemd/system/ollama.service.d/override.conf <<EOF
[Service]
Environment="OLLAMA_HOST=0.0.0.0"
EOF'

pct exec 106 -- systemctl daemon-reload
pct exec 106 -- systemctl restart ollama
pct exec 106 -- systemctl enable ollama
```

Verify GPU access is working from inside the container:

```bash
pct exec 106 -- nvidia-smi
```

If this returns your GPU info, the passthrough is working correctly.

---

## Install Tailscale (Optional)

If you want to access Ollama securely from anywhere on your Tailnet, install Tailscale inside the container. This is separate from any Tailscale installation on the host. I use the same approach for Home Assistant, and [Home Assistant TLS with Tailscale](https://exitcode0.net/posts/homeassistant-tls-with-tailscale/) goes into the Tailscale side in more detail, including HTTPS certificates.

```bash
pct exec 106 -- bash -c 'curl -fsSL https://tailscale.com/install.sh | sh'
```

### Authenticate Securely

Rather than typing an auth key directly into a shell command (which leaves it in your history), write it to a file, use it, then shred it:

Generate a reusable auth key at: https://login.tailscale.com/admin/settings/keys

```bash
# Create a secure directory for the key
pct exec 106 -- mkdir -p /etc/tailscale
pct exec 106 -- chmod 700 /etc/tailscale

# Write the key to a file
pct exec 106 -- bash -c 'echo "tskey-auth-YOURKEY" > /etc/tailscale/authkey'
pct exec 106 -- chmod 600 /etc/tailscale/authkey

# Enable tailscaled and authenticate
pct exec 106 -- systemctl enable --now tailscaled
pct exec 106 -- tailscale up --authkey=file:/etc/tailscale/authkey --hostname=ollama-lxc

# Shred the key file — Tailscale state persists in /var/lib/tailscale/
pct exec 106 -- shred -u /etc/tailscale/authkey
```

> **Why shred?** `shred -u` overwrites the file before deleting it, preventing recovery. Once authenticated, Tailscale persists its state in `/var/lib/tailscale/` — the auth key file is not needed again unless you fully re-authenticate.

Verify it connected:

```bash
pct exec 106 -- tailscale status
```

The container should appear at the top of the list as the local node.

---

## Working with Models

All of these can be run from the PVE host via `pct exec`, or from inside the container shell (`pct exec 106 -- bash`):

```bash
# Pull models
ollama pull llama3.2
ollama pull mistral
ollama pull codellama

# List installed models
ollama list

# Remove a model
ollama rm llama3.2

# Interactive chat session
ollama run llama3.2
```

### Which models fit on your GPU

Ollama's default downloads are 4-bit quantised, and the rule of thumb is that the model needs a little more VRAM than its file size. Roughly:

| Model size | Typical VRAM needed (Q4) | Fits on |
|---|---|---|
| 3B (llama3.2) | about 3GB | anything with 4GB+ |
| 7B to 8B (llama3.1, mistral) | 5 to 6GB | 8GB cards |
| 13B to 14B | 9 to 10GB | 12GB cards |
| 27B to 32B | 18 to 20GB | 24GB cards |
| 70B | 40GB+ | two cards or CPU offload |

If a model is bigger than the card, Ollama splits it between GPU and system RAM and it still runs, just much slower. `/api/ps` (below) tells you how much ended up on the GPU.

---

## API Reference

Ollama exposes a REST API on port `11434`. Once the container has a LAN IP (check with `pct exec 106 -- ip addr`), you can use it from any machine on your network.

### List installed models

```bash
curl http://<your-container-ip>:11434/api/tags
```

### Chat (streaming)

```bash
curl http://<your-container-ip>:11434/api/chat \
  -d '{
    "model": "llama3.2",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

### Generate (single prompt, no streaming)

```bash
curl http://<your-container-ip>:11434/api/generate \
  -d '{
    "model": "llama3.2",
    "prompt": "Explain what a TCP handshake is",
    "stream": false
  }'
```

### Check loaded models and VRAM usage

```bash
curl http://<your-container-ip>:11434/api/ps
```

> Models are automatically unloaded after 5 minutes of inactivity. An empty response from `/api/ps` is normal when idle — it doesn't mean something is broken.

### Check GPU stats from the PVE host

```bash
pct exec 106 -- nvidia-smi --query-gpu=name,memory.total,memory.used,memory.free,utilization.gpu --format=csv
```

---

## Open WebUI (Optional GUI)

The API is useful for scripting and integrations, but for general use a web interface is much more convenient. [Open WebUI](https://github.com/open-webui/open-webui) is a polished ChatGPT-style frontend that connects to your Ollama instance.

The simplest deployment is a Docker Compose stack on a separate VM or machine on your LAN, pointing `OLLAMA_BASE_URL` at `http://<your-container-ip>:11434`. If you're running Tailscale everywhere, you can also point it at the Tailnet address for cross-site access. If you work from a [persistent dev VM over Tailscale](https://exitcode0.net/posts/dumb-terminal-dev-vm-tailscale-claude-code/), the same address works from there too.

Open WebUI is out of scope for this post but worth looking at if you want something more usable than raw `curl` commands.

---

## Full /etc/pve/lxc/106.conf Reference

For reference, here's a complete working config:

```
arch: amd64
cores: 4
features: nesting=1
hostname: ollama
memory: 8192
net0: name=eth0,bridge=vmbr0,ip=dhcp,type=veth
onboot: 1
ostype: debian
rootfs: local-lvm:vm-106-disk-0,size=300G
swap: 512
lxc.cgroup2.devices.allow: c 195:* rwm
lxc.cgroup2.devices.allow: c 235:* rwm
lxc.cgroup2.devices.allow: c 226:* rwm
lxc.mount.entry: /dev/nvidia0 dev/nvidia0 none bind,optional,create=file
lxc.mount.entry: /dev/nvidiactl dev/nvidiactl none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-modeset dev/nvidia-modeset none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm dev/nvidia-uvm none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-uvm-tools dev/nvidia-uvm-tools none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-caps/nvidia-cap1 dev/nvidia-caps/nvidia-cap1 none bind,optional,create=file
lxc.mount.entry: /dev/nvidia-caps/nvidia-cap2 dev/nvidia-caps/nvidia-cap2 none bind,optional,create=file
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/libcuda.so.1 usr/lib/x86_64-linux-gnu/libcuda.so.1 none bind,optional,create=file
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/libcuda.so usr/lib/x86_64-linux-gnu/libcuda.so none bind,optional,create=file
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/libnvidia-ml.so.1 usr/lib/x86_64-linux-gnu/libnvidia-ml.so.1 none bind,optional,create=file
lxc.mount.entry: /usr/lib/x86_64-linux-gnu/libnvidia-ptxjitcompiler.so.1 usr/lib/x86_64-linux-gnu/libnvidia-ptxjitcompiler.so.1 none bind,optional,create=file
lxc.mount.entry: /usr/bin/nvidia-smi usr/bin/nvidia-smi none bind,optional,create=file
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

---

## Container Management Quick Reference

```bash
# Start / stop / status
pct start 106
pct shutdown 106
pct status 106

# Drop into a shell
pct exec 106 -- bash

# Watch Ollama logs
pct exec 106 -- journalctl -u ollama -f

# Tailscale status
pct exec 106 -- tailscale status
```

---

## Troubleshooting

### `nvidia-smi` fails inside the container

Check that the host drivers are working first (`nvidia-smi` on the PVE host). If the host is fine, double-check the cgroup device rules and mount entries in the container config — a typo there will silently fail at container start.

### Ollama isn't accessible from the LAN

Make sure the `OLLAMA_HOST=0.0.0.0` override is in place and the service restarted after adding it. Confirm with `pct exec 106 -- ss -tlnp | grep 11434`.

### Container won't start after editing the config

Run `pct start 106` from the shell and watch for errors. Missing device nodes referenced in `lxc.mount.entry` lines will cause startup failures if you haven't used `optional` — check each path exists on the host.

### Models are slow or not using the GPU

Check `/api/ps` while a model is loaded — it will show whether it's running on GPU or CPU. If CPU-only, the CUDA libraries probably aren't getting through to the container correctly.

### `nvidia-smi` works but Ollama still runs on CPU

The device nodes are there but the CUDA libraries aren't. Check the `libcuda.so.1` and `libnvidia-ml.so.1` bind mounts point at files that exist on the host (`ls -l /usr/lib/x86_64-linux-gnu/libcuda*` on PVE), then restart the container. Driver upgrades on the host can move these.

---

That's the whole setup, and it has been running unattended since I built it. The container survives host reboots, Tailscale comes back on its own, and the only maintenance has been `ollama pull` when a model I care about gets an update.

## Or just hand this page to your agent

If you'd rather not type any of this, paste the following into Claude Code or whichever agent you use on the Proxmox host:

```text
Read https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/ and set up an Ollama LXC with NVIDIA GPU passthrough on this Proxmox host. Before touching anything, run nvidia-smi on the host, pct list, and pvesm status, and confirm with me the container ID, storage pool, rootfs size, memory and cores to use, and whether I want the Tailscale section. Do not reboot the host without asking.
```

The page gives the agent the host driver steps, the container config and the verification commands. Only you know which container ID is free, which storage pool has 300GB spare and whether a host reboot is acceptable right now.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
