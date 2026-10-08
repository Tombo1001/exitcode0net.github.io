# Honeygain Docker Compose Setup: Run It on a Home Server

> Deploy Honeygain in a Docker container with Docker Compose. Earn passive income by sharing unused bandwidth from a home server or always-on machine.

Source: https://exitcode0.net/posts/honeygain-docker-compose-setup/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-03-04
Updated: 2026-10-08
Tags: docker, docker-compose, honeygain



[*](https://r.honeygain.me/TOM30548 "https://r.honeygain.me/TOM30548")*
So [Honeygain](https://r.honeygain.me/TOM30548 "https://r.honeygain.me/TOM30548") has finally arrived as a Docker container and this article will give you everything you need to build your own docker-compose YAML file for faster deployments.

You can see the docs provided by the Honeygain devs on the matter here:

[https://honeygain.zendesk.com/hc/en-us/articles/360018979919-How-to-run-Honeygain-on-Docker-Linux-](https://honeygain.zendesk.com/hc/en-us/articles/360018979919-How-to-run-Honeygain-on-Docker-Linux- "https://honeygain.zendesk.com/hc/en-us/articles/360018979919-How-to-run-Honeygain-on-Docker-Linux-")

However, they do not provide a nice way to deploy time and again from a docker-compose file, scroll down for a template! At the time of writing, you are permitted to run the service on two devices per public IP. Unfortunately, the docker image doesn’t currently support the content delivery feature

Running the Honeygain docker image (without docker-compose)
-----------------------------------------------------------

If oyu just want to run the container here are the steps to do so, provided that you have a docker environment ready to go:

1. Pull the Docker image

```
docker pull honeygain/honeygain
```

2. Open Honeygain Terms of Use. If you agree with our Terms of Use, please continue

```
docker run honeygain/honeygain -tou-get 
```

3. Start the Honeygain Docker container

```
docker run honeygain/honeygain -tou-accept -email ACCOUNT_EMAIL -pass ACCOUNT_PASSWORD -device DEVICE_NAME
```

Replace ACCOUNT\_EMAIL with your Honeygain account email  
Replace  ACCOUNT\_PASSWORD with your Honeygain account password  
Replace DEVICE\_NAME with a name that you would like to give to your Docker container. This name will be visible on the Dashboard.  
**NOTE: Use different DEVICE\_NAME for every container that you create.**

Honeygain docker-compose YAML example
-------------------------------------

Now here is how an example of a docker-compose file for the Honeygain container. **Please note that you will still need to change your device name between deployments**.

```
version: '3'

services:
  honeygain:
    container_name: honeygain
    image: honeygain/honeygain
    command: -tou-accept -email my.name@email.com -pass mylamepassword -device sweethoney01
    restart: unless-stopped
```

**Change the email, password and device name to suit!** And be careful not to publish your compose file with real credentials to any public code repository.

And here is how to run your copose file in detached mode (fromt he direcotry where you YAML file is stored):

```
docker-compose run -d
```



---
Markdown version of https://exitcode0.net/posts/honeygain-docker-compose-setup/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
