# Running Uptime Kuma With Docker Compose

> Deploy Uptime Kuma website and API monitoring with Docker Compose. Self-hosted uptime tracker and status page for infrastructure monitoring in a home lab.

Source: https://exitcode0.net/posts/running-uptime-kuma-with-docker-compose/
Author: Tom Cocking (https://tomcocking.com)
Published: 2023-05-01
Updated: 2026-10-08
Tags: docker, uptime-kuma, monitoring, docker-compose, self-hosted


## Introduction

Uptime-Kuma is an open-source, self-hosted website monitoring tool. It can check the uptime and response time of websites, APIs, and other services at regular intervals and alert you if any of them go down. Uptime-Kuma can be installed on your own server and customized to suit your needs. In this article, we will explore how to set up and run Uptime-Kuma using Docker Compose.

## The Docker Compose File

The following is the Docker Compose file for the Uptime-Kuma service:

```
version: '3.3'

services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    volumes:
      - ./data:/app/data
    ports:
      - 3001:3001
    restart: always
```

This Docker Compose file defines a single service named "uptime-kuma". The service is built from the "louislam/uptime-kuma" Docker image, which is the latest version available. The container name is set to "uptime-kuma" as well.

The "volumes" section maps the "/app/data" directory inside the container to the "./data" directory on the host machine. This allows us to persist data across container restarts. Uptime-Kuma stores all its configuration and monitoring data in this directory, so it's important to keep it intact.

The "ports" section maps port 3001 on the host machine to port 3001 in the container. This allows us to access the Uptime-Kuma web interface from our local machine. By default, Uptime-Kuma runs on port 3001, but you can change it if necessary.

Finally, the "restart" section ensures that the container will always be restarted if it exits. This is useful for ensuring that the Uptime-Kuma service is always available, especially if it crashes or is shut down unexpectedly.

## Running Uptime-Kuma with Docker Compose

To run Uptime-Kuma with Docker Compose, first, make sure you have Docker and Docker Compose installed on your machine. Then, create a new directory for your Uptime-Kuma project and save the above Docker Compose file as "docker-compose.yml" in that directory. 

Next, run the following command from the same directory:

```
docker-compose up -d
```

This command will start the Uptime-Kuma container in the background and detach from it. You can then access the Uptime-Kuma web interface by going to http://localhost:3001 in your web browser.

## Conclusion

In this article, we explored how to set up and run the Uptime-Kuma website monitoring tool using Docker Compose. We looked at the different sections of the Docker Compose file and explained how they work together to create a functional Uptime-Kuma service. By running Uptime-Kuma with Docker Compose, you can easily deploy and manage the tool on your own server, without having to worry about dependencies or configuration.

---
Markdown version of https://exitcode0.net/posts/running-uptime-kuma-with-docker-compose/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
