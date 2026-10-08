# Wyoming Piper Docker Compose: Local Text-to-Speech for Home Assistant

> Deploy Wyoming Piper text-to-speech in Docker for Home Assistant, giving local voice output with a choice of languages and voice models.

Source: https://exitcode0.net/posts/wyoming-piper-docker-compose/
Author: Tom Cocking (https://tomcocking.com)
Published: 2023-05-16
Updated: 2026-10-08
Tags: docker, wyoming, piper, tts, home-assistant, voice-assistant


Wyoming Piper is the text-to-speech half of a fully local Home Assistant voice assistant, and the whole thing is one Docker Compose service. Below is the compose file I run, the voice model choice, and how to point Home Assistant at it. The speech-to-text half is [Wyoming Whisper](https://exitcode0.net/posts/wyoming-whisper-docker-compose/). This post walks through setting up Wyoming Piper using Docker Compose. Piper is a fast, local neural text to speech system originally optimised for the Raspberry Pi 4. It supports many languages, and voice samples: https://rhasspy.github.io/piper-samples.

Wyoming Piper is a speech recognition and natural language understanding system that can be used for voice control in various applications. It uses the Rhasspy framework and provides support for different languages and voices.

## Prerequisites

Before you begin, make sure you have Docker and Docker Compose installed on your system. You can find installation instructions for your operating system on the [Docker website](https://docs.docker.com/get-docker/) and [Docker Compose website](https://docs.docker.com/compose/install/).

## Docker Compose File

Create a new file called `docker-compose.yml` and open it in a text editor. Copy the following content into the file:

```yaml
version: "3"
services:
  wyoming-piper:
    image: rhasspy/wyoming-piper
    ports:
      - "10200:10200"
    volumes:
      - "./piper-data:/data"
    command: [ "--voice", "en-gb-southern_english_female-low" ]
    restart: unless-stopped
```

Let's go through the different sections of this Docker Compose file.

### Version

The `version` section specifies the version of the Docker Compose file format. In this case, we're using version "3".

### Services

The `services` section defines the services that make up your application. In our case, we have a single service called `wyoming-piper`.

### Wyoming Piper Service

Under the `wyoming-piper` service, we have the following configurations:

- `image`: Specifies the Docker image to use for the service. In this case, we're using the `rhasspy/wyoming-piper` image.
- `ports`: Maps the container's port `10200` to the host's port `10200`. This allows us to access Wyoming Piper's web interface from our local machine.
- `volumes`: Mounts the `./piper-data` directory on the host to the `/data` directory inside the container. This is used to persist Wyoming Piper's data.
- `command`: Specifies the command-line arguments to pass to the container. In this example, we're using the English (GB) Southern English Female (Low) voice sample.
- `restart`: Sets the restart policy for the container. In this case, the container will be automatically restarted unless explicitly stopped.

## Starting Wyoming Piper

To start Wyoming Piper, open a terminal or command prompt, navigate to the directory where you saved the `docker-compose.yml` file, and run the following command:

```
docker-compose up -d
```

The `-d` flag runs the containers in the background (detached mode).

Wait for Docker Compose to download the necessary Docker images and start the Wyoming Piper container. You can check the progress in the terminal output.

Once the container is up and running, you can access Wyoming Piper's web interface by opening a web browser and navigating to `http://localhost:10200`.

## Conclusion

In this blog post, we've walked you through setting up Wyoming Piper using Docker Compose. Docker Compose allows you to manage the different components of Wyoming Piper in a unified and reproducible way. You can customize the configurations in the `docker-compose.yml`

At this point we have all the components needed for a fully local voice assistant stack, deployed with docker compose. It is now possible to follow the remainder of the Home Assistant docuemntation in configuring your assistant: https://www.home-assistant.io/docs/assist/voice_remote_local_assistant.

**Relevant and supporting posts:**
- [How to Use a Docker Compose File for Wyoming Whisper](https://exitcode0.net/posts/wyoming-whisper-docker-compose/)
- [Homeassistant Enable MagicDNS and HTTPS Certificates in Tailscale](https://exitcode0.net/posts/homeassistant-tls-with-tailscale/)



---
Markdown version of https://exitcode0.net/posts/wyoming-piper-docker-compose/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
