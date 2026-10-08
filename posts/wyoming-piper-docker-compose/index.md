# Wyoming Piper Docker Compose: Local Text-to-Speech for Home Assistant

> Run Wyoming Piper text-to-speech in Docker Compose for Home Assistant. Tested compose file, voice naming, 174 voices and corrections to my 2023 post.

Source: https://exitcode0.net/posts/wyoming-piper-docker-compose/
Author: Tom Cocking (https://tomcocking.com)
Published: 2023-05-16
Updated: 2026-10-08
Tags: docker, wyoming, piper, tts, home-assistant, voice-assistant, docker-compose

> **Updated October 2026:** I re-tested this on 8 October 2026 against the current `rhasspy/wyoming-piper` image and corrected three things I got wrong in 2023. Piper is text-to-speech, not speech recognition. Port 10200 isn't a web interface. And voices are now named like `en_GB-alba-medium`. The compose file also no longer needs a `version` line. The Home Assistant steps are unchanged from the original and I haven't re-run them on a current release.

Wyoming Piper is the text-to-speech half of a fully local Home Assistant voice assistant, and it runs as one Docker Compose service. Piper is a fast neural text-to-speech system that was first tuned for the Raspberry Pi 4. The Wyoming wrapper lets Home Assistant use it over the network. It supports many languages and voices, and you can hear them on the [Piper samples page](https://rhasspy.github.io/piper-samples). The speech-to-text half is [Wyoming Whisper](https://exitcode0.net/posts/wyoming-whisper-docker-compose/).

## Prerequisites

You need Docker with the Compose plugin. `docker compose version` should print a version. I tested on Compose 2.40.3. Installation instructions for your OS are on the [Docker website](https://docs.docker.com/get-docker/).

## Docker Compose file

Save this as `compose.yaml` in a new directory:

```yaml
services:
  wyoming-piper:
    image: rhasspy/wyoming-piper
    ports:
      - "10200:10200"
    volumes:
      - ./piper-data:/data
    command: ["--voice", "en_GB-alba-medium"]
    restart: unless-stopped
```

What each part does:

- `image` is the official `rhasspy/wyoming-piper` image. Its entrypoint already sets `--uri tcp://0.0.0.0:10200` and `--data-dir /data`, so you only pass the voice.
- `ports` publishes Wyoming's port 10200. Home Assistant connects here.
- `volumes` keeps downloaded voices in `./piper-data`. A medium voice is about 63 MB.
- `command` sets the default voice. My 2023 post used `en-gb-southern_english_female-low`. That still works, and I've switched to `en_GB-alba-medium` because medium voices sound better.
- `restart: unless-stopped` brings the container back after a reboot.

## Starting Wyoming Piper

```bash
docker compose up -d
docker compose logs -f wyoming-piper
```

The first start downloads the voice from Hugging Face, which took a few seconds here. Wait for `Ready`, then press Ctrl+C to leave the logs. `docker compose ps` should show the container as healthy.

There's no web interface to open. Port 10200 speaks the Wyoming protocol over TCP, and `curl http://localhost:10200` returns nothing. I checked, and it's the opposite of what my original post said. Newer images do have an opt-in web UI for managing custom voices, switched on with `--web-server --web-server-host 0.0.0.0`. I haven't used it.

### Test it without Home Assistant

Save the test script from the [Whisper post](https://exitcode0.net/posts/wyoming-whisper-docker-compose/#test-it-without-home-assistant) as `wyoming_check.py`, then run:

```bash
python3 wyoming_check.py describe localhost 10200
python3 wyoming_check.py say localhost 10200 en_GB-alba-medium "Wyoming Piper is working."
```

Mine printed:

```text
tts: piper, 174 available, e.g. ['ar_JO-kareem-low', 'ar_JO-kareem-medium', 'bg_BG-dimitar-medium']
2.0s of audio at 22050 Hz in 0.9s
```

The script saves the audio as `say.raw`, which the Whisper container can transcribe. That round trip is the quickest way to prove both halves work before Home Assistant is involved.

## Choosing a voice

Voice names follow the pattern `language_REGION-name-quality`, for example `en_GB-alba-medium`. The image still accepts the 2023 hyphenated names and maps them across, but the underscore form is the current naming.

Quality is low, medium or high. In my tests low voices produced 16,000 Hz audio and medium voices 22,050 Hz.

The image advertises 174 voices, and you don't have to download them in advance. Ask for one that isn't in `./piper-data` and Piper fetches it on the first request. When I asked for `en_US-lessac-medium`, that first request took 3.6 seconds while it downloaded. Later requests took under a second.

Idle, the container used about 230 MiB of RAM. To change the default voice, edit `--voice` and run `docker compose up -d` again.

## Adding it to Home Assistant

In Home Assistant go to `Settings > Devices & Services > Add Integration`, search for Wyoming Protocol and enter the IP address of the machine running Docker plus port 10200. Use the host's IP, not `localhost`, because inside the Home Assistant container that means Home Assistant itself. Then pick `piper` as the text-to-speech engine in `Settings > Voice assistants`.

Piper and Whisper together are everything the local voice pipeline needs, and with the small Whisper model they used about 1 GiB of RAM between them on my test machine. The remainder of the setup is in Home Assistant's own guide to a [local voice assistant](https://www.home-assistant.io/voice_control/voice_remote_local_assistant/).

## Troubleshooting

### Home Assistant can't connect

Run `nc -zv <docker-host-ip> 10200` from the machine running Home Assistant. If that fails, the problem is the network or a firewall. If it succeeds, check you entered the host's IP and not `localhost`.

### The browser shows nothing at port 10200

That's expected. It's a TCP protocol service, not a web server.

### The first reply is slow

If you picked a voice that hasn't been downloaded yet, the first request waits for the download. Mine took 3.6 seconds. Later ones were fast.

### The voice sounds thin

Low-quality voices are 16 kHz. Switch to the medium version of the same voice.

### The data folder is owned by root

Docker created the files in `piper-data` as root on my machine, so deleting them needs sudo.

### Can I use a GPU?

There's a `--use-cuda` option that needs a GPU-enabled onnxruntime. I haven't tried it, because the machine I tested on has no GPU, and Piper is fast enough on a CPU that I wouldn't bother.

## Both halves agree

`en_GB-alba-medium` says "Wyoming Piper is working." in about two seconds, and `small-int8` Whisper hears the same words back. If something breaks later, look at the Home Assistant wiring before you blame either container.

**Relevant and supporting posts:**

- [Wyoming Whisper Docker Compose](https://exitcode0.net/posts/wyoming-whisper-docker-compose/)
- [Home Assistant HTTPS with Tailscale](https://exitcode0.net/posts/homeassistant-tls-with-tailscale/)

## Or just hand this page to your agent

If you'd rather not type any of this, paste the following into your coding agent:

```text
Read https://exitcode0.net/posts/wyoming-piper-docker-compose/ and set up Wyoming Piper in Docker Compose on this machine. Before changing anything, run docker compose version and docker ps, check whether port 10200 is already in use, and show me the compose file you plan to write. Ask me which language and voice I want, and don't pick a voice for me. Don't touch my Home Assistant config. Stop once docker compose ps shows the container healthy and tell me the host IP and port to enter in Home Assistant.
```

The page gives the agent the compose file, the voice naming rules and the test script. Only you know which language you want, which voice sounds right to you and whether port 10200 is free.

Choosing a voice is the fun part, so budget a few minutes at the samples page.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/wyoming-piper-docker-compose/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
