# [SOLVED] - Running Docker on a Raspberry Pi 4

> Fix Docker install errors on a Raspberry Pi 4 running Raspbian Buster. Resolves the kernel cgroup issue that stops containers starting on ARM boards.

Source: https://exitcode0.net/posts/solved-running-docker-on-a-raspberry-pi-4/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-07-10
Updated: 2026-10-08
Tags: 4gb, docker, linux, pi-4, raspberry-pi, raspbian



**The Raspberry Pi 4 has now been released offering up to 4GB of RAM! All of the horsepower required for an excellent lower power, docker host.**

![Pi4 Running Docker](https://exitcode0.net/images/legacy/2019-07-Pi4-Docker.png)
However, there are currently issues undergoing work which prevent docker from running on the only Rasbian image currently available for the Pi 4 – ‘[Rasbian Buster](https://www.raspberrypi.org/downloads/raspbian/ "https://www.raspberrypi.org/downloads/raspbian/")‘. Details of these issues can been found here on the GitHub thread – [https://github.com/docker/for-linux/issues/709](https://github.com/docker/for-linux/issues/709 "https://github.com/docker/for-linux/issues/709")

---

### Current Working Solution

Fear not, for there is a simple way to fool your docker installation and successfully getting it to run on the Pi 4.

First, let make sure that your raspbian install is up to date:

```
sudo apt-get update

sudo apt-get upgrade
```

Now let’s install docker:

```
curl -SL get.docker.com | sed 's/9)/10)/' | sh
```

Once complete the installer will advise that you should add the Pi user to the docker group if you wish to run docker commands from that account:

```
usermod -aG docker pi
```

**And you’re done! Docker will now be running on your Raspberry Pi 4.**

---

[](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url "https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url")If this posted helped you, consider throwing a penny in the tip jar to support this site.

---



---
Markdown version of https://exitcode0.net/posts/solved-running-docker-on-a-raspberry-pi-4/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
