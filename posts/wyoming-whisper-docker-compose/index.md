# Wyoming Whisper Docker Compose: Local Speech-to-Text for Home Assistant

> Run Whisper speech-to-text in Docker for a local Home Assistant voice assistant, with a lightweight Wyoming Whisper model for fast recognition.

Source: https://exitcode0.net/posts/wyoming-whisper-docker-compose/
Author: Tom Cocking (https://tomcocking.com)
Published: 2023-05-11
Updated: 2026-10-08
Tags: docker, wyoming, whisper, stt, home-assistant, voice-assistant


Wyoming Whisper is the speech-to-text half of a fully local Home Assistant voice assistant, and it runs as a single Docker Compose service on a Pi or any small box. Below is the compose file, the model choice, and how to point Home Assistant at it. The text-to-speech half is [Wyoming Piper](https://exitcode0.net/posts/wyoming-piper-docker-compose/). Wyoming Whisper is an open-source, lightweight voice assistant designed to run on a Raspberry Pi or other low-powered device. The impetus for this compose defined container is to intergate with a Home Assistant 2023.5 container and ultimate have a fully local voice assistant. Whisper will provide our speech-to-text service and the Wyoming protocol is how it will be integrated with Home Assistant.

**NOTE:** For your reference, I am using Home Assistant in a container which is why I have not simply setup the wyoming wishper addons.

## Prerequisites

Before we get started, make sure you have Docker and Docker Compose installed on your system. If you need help with installation, please refer to the official Docker documentation.

## Step 1: Create a Docker Compose File

Create a file called `docker-compose.yml` in a directory of your choosing and add the following content to it:

```yaml
version: '3'
services:
  wyoming-whisper:
    image: rhasspy/wyoming-whisper
    ports:
      - "10300:10300"
    volumes:
      - ./whisper-data:/data
    command: [ "--model", "medium-int8", "--language", "en" ]
    restart: unless-stopped
```

This file defines a single service named `wyoming-whisper`. Let's go over what each line does:

- `version: '3'`: specifies the version of Docker Compose file format being used.
- `image: rhasspy/wyoming-whisper`: specifies the Docker image to use for the service. In this case, we are using the `rhasspy/wyoming-whisper` image.
- `ports: - "10300:10300"`: specifies the port mapping between the container and host. The `10300:10300` mapping means that the container's port `10300` is mapped to the host's port `10300`.
- `volumes: - ./whisper-data:/data`: specifies the volume mapping between the container and host. The `./whisper-data:/data` mapping means that the `./whisper-data` directory on the host is mapped to the `/data` directory in the container.
- `command: [ "--model", "medium-int8", "--language", "en" ]`: specifies the command to run when the container starts. In this case, we are running Wyoming Whisper with the `medium-int8` model and English language.
- `restart: unless-stopped`: specifies the restart policy for the container. In this case, the container will automatically restart unless it is explicitly stopped by the user.

### Whispher models

You might be trying to run this container on a lightweight device such as a Raspberry Pi, in which case it would be wise to sellect a lightweight model for improved performance. Here are the model options, sorted from least to most accurate (fastest to slowest):

- tiny-int8 (43 MB)
- tiny (152 MB)
- base-int8 (80 MB)
- base (291 MB)
- small-int8 (255 MB)
- small (968 MB)
- medium-int8 (786 MB)
- medium (3.1 GB)

The model file is downloaded on first run of the container.

## Step 2: Start the Container

To start the container, navigate to the directory where your `docker-compose.yml` file is located and run the following command:

```bash
docker-compose up -d
```

The `-d` flag runs the container in the background.

## Step 3: Integrate Wyoming Whisper with Home Assistant

Now that the server is up and running we can add a new integration in our Home Assistant settings. This can be done under: `Settings > Devices & Services > Add Integration`.

![Adding the Wyoming Wishper integration to Home Assistant](https://exitcode0.net/images/wyoming-HA-integration.png)

## Conclusion

That's it! You now know how to use a Docker Compose file to deploy and configure Wyoming Whisper. Docker Compose is a powerful tool that simplifies the deployment and management of complex applications. If you are looking to prepare your HA install for a fully local voice assistant and you need to setup HTTPS, consider reading the previous post - linked below - on how this can be done with Tailscale MagicDNS and HTTPS certificates.

---
Markdown version of https://exitcode0.net/posts/wyoming-whisper-docker-compose/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
