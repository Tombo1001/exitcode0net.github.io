# Honeygain Docker Compose Setup: Run It on a Home Server

> Run Honeygain in Docker with Compose. Tested compose file that keeps your login out of the YAML, the real CLI flags, and what docker inspect still shows.

Source: https://exitcode0.net/posts/honeygain-docker-compose-setup/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-03-04
Updated: 2026-10-08
Tags: docker, docker-compose, honeygain

> **Updated October 2026:** I re-tested the container on 8 October 2026. It's now Honeygain CLI 0.9.0, every flag my original post used still exists, and the compose file below keeps your login out of the YAML. I also fixed the last command, which was wrong in the original. I didn't log in with a real account for the test, so I haven't re-checked what appears on the dashboard. This post contains my Honeygain referral link, which earns me a bonus if you sign up through it.

[Honeygain](https://r.honeygain.me/TOM30548) runs as a Docker container, and this post gives you a docker-compose file you can redeploy in seconds. Honeygain's own instructions are here:

[How to run Honeygain on Docker (Linux)](https://honeygain.zendesk.com/hc/en-us/articles/360018979919-How-to-run-Honeygain-on-Docker-Linux-)

Their page doesn't give you a compose file, so scroll down for one. When I first wrote this in 2021 you could run the service on two devices per public IP, and the image didn't support the content delivery feature. Honeygain's terms change, so check their current rules before you rely on either of those.

## Running the Honeygain docker image (without docker-compose)

If you just want the container running, here are the steps, assuming you already have Docker.

1. Pull the Docker image:

```bash
docker pull honeygain/honeygain
```

2. Read the Honeygain Terms of Use, and carry on only if you agree with them:

```bash
docker run honeygain/honeygain -tou-get
```

3. Start the container:

```bash
docker run honeygain/honeygain -tou-accept -email ACCOUNT_EMAIL -pass ACCOUNT_PASSWORD -device DEVICE_NAME
```

Replace `ACCOUNT_EMAIL` with your Honeygain account email, `ACCOUNT_PASSWORD` with your account password, and `DEVICE_NAME` with the name you want on the dashboard. Use a different device name for every container you create.

Those are the only flags the image accepts. `docker run honeygain/honeygain -h` lists exactly six: `-device`, `-email`, `-pass`, `-tou-accept`, `-tou-get` and `-version`. There are no environment variables to put credentials in.

## Honeygain docker-compose YAML example

The version in my 2021 post had your email and password written straight into the compose file, which is how credentials end up in a Git repository. This version reads them from a `.env` file that sits next to it.

Create `.env`:

```bash
HONEYGAIN_EMAIL=you@example.com
HONEYGAIN_PASSWORD=change-me
HONEYGAIN_DEVICE=homelab-01
```

Lock it down so only you can read it:

```bash
chmod 600 .env
```

Then create `compose.yaml`:

```yaml
services:
  honeygain:
    container_name: honeygain
    image: honeygain/honeygain
    command: -tou-accept -email ${HONEYGAIN_EMAIL} -pass ${HONEYGAIN_PASSWORD} -device ${HONEYGAIN_DEVICE}
    restart: unless-stopped
```

`docker compose config` shows the file with the three variables filled in, which is a quick way to check you've spelled them right. If this directory is in Git, add `.env` to `.gitignore` before you commit anything.

Change the device name in `.env` for every machine you deploy to.

## Start it

From the directory with `compose.yaml` and `.env`:

```bash
docker compose up -d
docker compose logs -f honeygain
```

The device name you chose should appear on your Honeygain dashboard. If the container keeps restarting, read the logs first.

The original version of this post ended with `docker-compose run -d`, which was wrong. I tried it. `run` starts a separate one-off container rather than the service, and `docker compose down` doesn't remove that container afterwards. Use `up -d`.

## The password is still visible

Moving the login into `.env` keeps it out of your YAML. It doesn't hide it from the machine. The image only takes the password as a command line argument, so anyone who can run `docker inspect honeygain` on the host sees it in plain text. I checked:

```text
[-tou-accept -email you@example.com -pass change-me -device homelab-01]
```

Use a password for your Honeygain account that you don't use anywhere else, and keep the Docker socket to people you trust.

## Why I wrote this up

The 2021 post was a template with a warning not to publish it with real credentials, and I'd rather the template didn't invite that. Moving the login into `.env` took two minutes, and the container receives exactly the same command line as before.

## Or just hand this page to your agent

If you'd rather not type it all, paste the following into your coding agent:

```text
Read https://exitcode0.net/posts/honeygain-docker-compose-setup/ and set up Honeygain in Docker Compose on this machine. Don't ask me to paste my Honeygain email or password into this chat. Create compose.yaml and a .env file containing placeholder values, tell me to fill in my real login myself, and set .env to chmod 600. Check whether this directory is a Git repository and add .env to .gitignore if it is. Use docker compose up -d and not docker compose run. Ask me what device name to use before you start the container.
```

The page gives the agent the compose file, the flags and the `.env` layout. Only you know your account login, which device name is free and whether Honeygain's current terms let you run it where you want to.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/honeygain-docker-compose-setup/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
