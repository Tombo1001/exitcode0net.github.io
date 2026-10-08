# Wyoming Whisper Docker Compose: Local Speech-to-Text for Home Assistant

> Run Wyoming Whisper speech-to-text in Docker Compose for Home Assistant. Tested compose file, model speeds measured on a Ryzen 5 5600 and a test script.

Source: https://exitcode0.net/posts/wyoming-whisper-docker-compose/
Author: Tom Cocking (https://tomcocking.com)
Published: 2023-05-11
Updated: 2026-10-08
Tags: docker, wyoming, whisper, stt, home-assistant, voice-assistant, docker-compose

> **Updated October 2026:** I re-tested this on 8 October 2026 against the current `rhasspy/wyoming-whisper` image. The compose file no longer needs a `version` line, the default model is now `small-int8` instead of `medium-int8`, and there's a table of measured model speeds, a test script and a troubleshooting section. The Home Assistant steps come from my original post. I haven't re-run them on a current Home Assistant release.

Wyoming Whisper is the speech-to-text half of a fully local Home Assistant voice assistant. It wraps faster-whisper, a fast implementation of OpenAI's Whisper models, and Home Assistant talks to it over the Wyoming protocol. The text-to-speech half is [Wyoming Piper](https://exitcode0.net/posts/wyoming-piper-docker-compose/).

I run Home Assistant in a container, so I don't use the Whisper add-on. If you're on Home Assistant OS, the add-on is the easier route. Everything here is for people running plain Docker.

## Prerequisites

You need Docker with the Compose plugin. `docker compose version` should print a version. I tested on Compose 2.40.3. If you only have the old `docker-compose` command, upgrade it, because the commands below use `docker compose`.

## Step 1: create the Docker Compose file

Make a directory and save this as `compose.yaml` inside it:

```yaml
services:
  wyoming-whisper:
    image: rhasspy/wyoming-whisper
    ports:
      - "10300:10300"
    volumes:
      - ./whisper-data:/data
    command: ["--model", "small-int8", "--language", "en"]
    restart: unless-stopped
```

What each part does:

- `image` is the official `rhasspy/wyoming-whisper` image. Its entrypoint already sets the listen address and data directory, so you only pass the model and language.
- `ports` publishes Wyoming's port 10300. Home Assistant connects here.
- `volumes` keeps downloaded models in `./whisper-data`, so they survive the container being recreated.
- `command` picks the model and the language. Leave out `--language` and the default is auto-detection. If you only ever speak English, setting it saves the guesswork.
- `restart: unless-stopped` brings the container back after a reboot.

### Whisper models and how fast they are

The model downloads on first start, so the first start is slow and later ones aren't. Wait for `Ready` in the logs before you add the integration.

I timed each int8 model on a six-core Ryzen 5 5600 with no GPU. The clip was the same 1.8 seconds of Piper saying "Wyoming Piper is working." Times are for a model that's already loaded.

| Model | On disk | RAM | Time | What it heard |
|---|---|---|---|---|
| tiny-int8 | 44 MiB | 226 MiB | 0.3s | "Wildman Piper is working." |
| base-int8 | 79 MiB | 544 MiB | 0.6s | "Wyoming Piper is working." |
| small-int8 | 246 MiB | 742 MiB | 1.3s | "Wyoming Piper is working." |
| medium-int8 | 752 MiB | 1.9 GiB | 4.7s | "Wyoming Piper is working." |

Tiny heard "Wyoming" as "Wildman", which tells you where the floor is. Medium got it right but took 4.7 seconds to transcribe a sentence that takes under two seconds to say. With a voice assistant you wait that long after every command, so I'd start with `small-int8` on a mini PC and `base-int8` on a Raspberry Pi 4. I haven't timed a Pi, so that one is a guess.

It's one clip in one synthetic voice, so read the last column as an anecdote and the timings as the useful part. The original version of this post also listed full-precision tiny, base, small and medium builds. They're bigger and I haven't benchmarked them.

## Step 2: start the container

```bash
docker compose up -d
docker compose logs -f wyoming-whisper
```

Wait for `Ready`, then press Ctrl+C to leave the logs. `docker compose ps` should show the container as healthy.

### Test it without Home Assistant

Wyoming is a plain TCP protocol, so there's no web page on port 10300 and a browser will show nothing. To check the port is open:

```bash
nc -zv localhost 10300
```

That only proves something is listening. To prove it can transcribe, I wrote a small Python script that speaks the protocol using nothing but the standard library. It can ask a server what it offers, have Piper say a sentence, and send that audio to Whisper. Save this as `wyoming_check.py`:

```python
#!/usr/bin/env python3
"""Minimal Wyoming client for testing Piper and Whisper. Standard library only.

  python3 wyoming_check.py describe HOST PORT
  python3 wyoming_check.py say      HOST PORT VOICE "text to speak"   (saves say.raw)
  python3 wyoming_check.py hear     HOST PORT                         (transcribes say.raw)
"""
import json, socket, sys, time

def send(sock, kind, data=None, payload=b""):
    body = json.dumps(data).encode() if data else b""
    header = {"type": kind}
    if body:
        header["data_length"] = len(body)
    if payload:
        header["payload_length"] = len(payload)
    sock.sendall(json.dumps(header).encode() + b"\n" + body + payload)

def read(f):
    line = f.readline()
    if not line:
        return None, b""
    event = json.loads(line)
    data = event.get("data") or {}
    if event.get("data_length"):
        data = {**data, **json.loads(f.read(event["data_length"]))}
    payload = f.read(event["payload_length"]) if event.get("payload_length") else b""
    return {"type": event["type"], "data": data}, payload

mode, host, port = sys.argv[1], sys.argv[2], int(sys.argv[3])
sock = socket.create_connection((host, port), timeout=120)
f = sock.makefile("rb")

if mode == "describe":
    send(sock, "describe")
    info, _ = read(f)
    for kind in ("tts", "asr"):
        for program in info["data"].get(kind) or []:
            items = program.get("models") or program.get("voices") or []
            print(f"{kind}: {program['name']}, {len(items)} available, e.g. {[i['name'] for i in items[:3]]}")

elif mode == "say":
    start = time.time()
    send(sock, "synthesize", {"text": sys.argv[5], "voice": {"name": sys.argv[4]}})
    pcm, rate = bytearray(), None
    while True:
        event, payload = read(f)
        if event is None or event["type"] == "audio-stop":
            break
        if event["type"] == "error":
            sys.exit(f"error: {event['data']}")
        if event["type"] == "audio-start":
            rate = event["data"]["rate"]
        if event["type"] == "audio-chunk":
            pcm += payload
    open("say.raw", "wb").write(pcm)
    open("say.rate", "w").write(str(rate))
    print(f"{len(pcm) / 2 / rate:.1f}s of audio at {rate} Hz in {time.time() - start:.1f}s")

elif mode == "hear":
    pcm, rate = open("say.raw", "rb").read(), int(open("say.rate").read())
    fmt = {"rate": rate, "width": 2, "channels": 1}
    start = time.time()
    send(sock, "transcribe", {"language": "en"})
    send(sock, "audio-start", fmt)
    for i in range(0, len(pcm), 2048):
        send(sock, "audio-chunk", fmt, pcm[i:i + 2048])
    send(sock, "audio-stop")
    while True:
        event, _ = read(f)
        if event is None or event["type"] == "error":
            sys.exit(f"error: {event}")
        if event["type"] == "transcript":
            print(f"heard {event['data']['text'].strip()!r} in {time.time() - start:.1f}s")
            break
```

With this container and the one from the [Piper post](https://exitcode0.net/posts/wyoming-piper-docker-compose/) both running:

```bash
python3 wyoming_check.py describe localhost 10300
python3 wyoming_check.py say localhost 10200 en_GB-alba-medium "Wyoming Piper is working."
python3 wyoming_check.py hear localhost 10300
```

This is what mine printed:

```text
asr: faster-whisper, 1 available, e.g. ['small-int8']
2.0s of audio at 22050 Hz in 0.9s
heard 'Wyoming Piper is working.' in 1.5s
```

If the last line prints your sentence back, speech-to-text and text-to-speech both work, and anything still broken is on the Home Assistant side.

## Step 3: integrate Wyoming Whisper with Home Assistant

Now that the server is up, add it in Home Assistant under `Settings > Devices & Services > Add Integration`. Search for Wyoming Protocol and enter the IP address of the machine running Docker plus port 10300. Don't use `localhost`, because inside the Home Assistant container that means Home Assistant itself.

![Adding the Wyoming Whisper integration to Home Assistant](https://exitcode0.net/images/wyoming-HA-integration.png)

Then open `Settings > Voice assistants`, create or edit an assistant, and choose `faster-whisper` as its speech-to-text engine. The [Home Assistant Wyoming docs](https://www.home-assistant.io/integrations/wyoming/) cover the rest.

## Troubleshooting

### Home Assistant can't connect

Run `nc -zv <docker-host-ip> 10300` from the machine running Home Assistant. If that fails, the problem is the network or a firewall, not Whisper. If it succeeds and Home Assistant still can't connect, check you entered the host's IP and not `localhost`.

### It takes ages to answer

Drop to a smaller model using the table above, then run `docker compose up -d` again to recreate the container with the new `--model`. This image runs on the CPU by default, so the model size is the main lever.

### It mishears short commands

Try the next model up. The tiny model turned "Wyoming" into "Wildman" in my test.

### The data folder is owned by root

Docker created the files in `whisper-data` as root on my machine, so deleting them needs sudo:

```bash
sudo rm -rf whisper-data
```

### Can I use a GPU?

The server has a `--device cuda` option, but I couldn't find a published GPU image on Docker Hub. I looked for `latest-gpu` and `gpu` tags and found neither, so you'd build from the repository's GPU Dockerfile. I haven't, because the machine I tested on has no GPU.

## Which model I'd actually run

The 2023 version of this post defaulted to `medium-int8`. On a six-core desktop CPU that takes 4.7 seconds to transcribe a two-second sentence, which is a long time to stand in the kitchen waiting for a light to come on. `small-int8` heard exactly the same thing in 1.3 seconds.

If you're setting up HTTPS so the browser microphone works with Assist, my [Home Assistant HTTPS with Tailscale post](https://exitcode0.net/posts/homeassistant-tls-with-tailscale/) covers the certificates.

## Or just hand this page to your agent

If you'd rather not type any of this, paste the following into your coding agent:

```text
Read https://exitcode0.net/posts/wyoming-whisper-docker-compose/ and set up Wyoming Whisper in Docker Compose on this machine. Before changing anything, run docker compose version and docker ps, check whether port 10300 is already in use, and show me the compose file you plan to write. Ask me which Whisper model and language I want, and tell me how much RAM and what CPU this machine has before you suggest a model. Don't touch my Home Assistant config. Stop once docker compose ps shows the container healthy and tell me the host IP and port to enter in Home Assistant.
```

The page gives the agent the compose file, the model timings and the test script. Only you know how much RAM the box has, which language you speak to it and whether port 10300 is free.

Allow ten minutes, most of it model download.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/wyoming-whisper-docker-compose/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
