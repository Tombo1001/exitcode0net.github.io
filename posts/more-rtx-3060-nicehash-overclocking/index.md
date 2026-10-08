# RTX 3060 NiceHash Overclocking and the 470.05 Hash Rate Unlock (2021 Archive)

> The RTX 3060 mining settings that reached 48 MH/s in 2021: the 470.05 driver unlock, MSI Afterburner clocks and fan curve. Archived, with what to do with the card now.

Source: https://exitcode0.net/posts/more-rtx-3060-nicehash-overclocking/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-04-07
Updated: 2026-10-08
Tags: crypto, nicehash, overclocking, rtx-3060


> **Note:** NVIDIA has since patched the RTX 3060 hash rate limiter across multiple driver versions. The developer driver 470.05 used in this article is no longer available from NVIDIA. Crypto mining on consumer GPUs is generally no longer economically viable. If you still have the card, it is far more useful today running local LLMs. See [running Ollama in a Proxmox LXC with NVIDIA GPU passthrough](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/).

![RTX 3060 Nicehash overclocking - 25% improvement](https://i1.wp.com/exitcode0.net/wp-content/uploads/2021/04/rtx-nicehash.png?resize=232%2C232&ssl=1)
> **Updated October 2026:** this page now holds all three of my 2021 RTX 3060 mining posts in one place: the 470.05 driver unlock, the first overclock that reached 44 MH/s, and the later settings that reached 48 MH/s. The old URLs redirect here. I've removed the third-party driver download link; the driver was pulled by NVIDIA and installing one from a file share was never a good idea.

## The 470.05 driver unlock (March 2021)

In March 2021 NVIDIA accidentally shipped a developer driver, 470.05, without the hash rate limiter that the RTX 3060 launched with. The Verge had a [good round-up](https://www.theverge.com/2021/3/16/22333544/nvidia-rtx-3060-ethereum-mining-rate-limit-unlock-driver) at the time. The driver was pulled within days but copies circulated. It only worked with a single RTX 3060 and a monitor attached. On my Gigabyte RTX 3060 Gaming OC 12G, NiceHash's DaggerHashimoto went from about 22 MH/s on the stock driver to 39 to 42 MH/s on 470.05 with no other changes. NVIDIA re-patched the limiter in later drivers, then removed it altogether once mining stopped mattering, so none of this applies to current drivers.

## First overclock: 44 MH/s (April 2021)

With 470.05 in place I followed the usual mining recipe in MSI Afterburner: lower the core clock, lower the power limit, raise the memory clock. Settings on a V1 (non-LHR) card:

- Power limit: 65%
- Core clock: -400 MHz
- Memory clock: +800 MHz
- Fan speed: auto

That took a consistent 38 to 41 MH/s up to 44+ MH/s, and NiceHash reported board power dropping from about 140 W to 110 W. The rest of this post is the second round of tuning that got to 48 MH/s.

## Second round: 48 MH/s

My previous settings to acheive 44 MH/s used the following sentiment:

* Lower the core clock rate
* Lower the power limit(%)
* **Increase**the memory clock rate

But I have since made some overclocking improvements and some additional changes to improve the longevity of the card. So much so that my overall uplift from stock (using the 470.05 driver) is sitting at around **25%**! I am now able to reach **48MH/s** with the DaggerHashimoto algorithm.

![RTX 3060 Nicehash overclocking - 25% improvement](https://i0.wp.com/exitcode0.net/wp-content/uploads/2021/04/image-4.png?resize=587%2C70&ssl=1)Behold, 48 MH/s on an RTX 3060 (the card which NVidia does not want you to mine on)
### RTX 3060 overclocking with MSI Afterburner

Your millage may vary with your individual card, but my improved settings to reach higher quickminer hash rates are as follows:

* **Power limit: 75%**
* **Core Clock: -500 MHz** – as low as it will go
* **Memory Clock + 1300 MHz**
* **Fan Speed: Auto** – with a more aggressive fan curve, see below…

![RTX 3060 Nicehash overclocking - improved hash rate and fan curve](https://i2.wp.com/exitcode0.net/wp-content/uploads/2021/04/image-5-1024x680.png?resize=774%2C514&ssl=1)The MSI Afterburner settings to achieve 48MH/s on a RTX 3060
Keeping the GPU safe
--------------------

![RTX 3060 Nicehash overclocking - more aggressive fan curve for low VRAM temperatures.](https://i0.wp.com/exitcode0.net/wp-content/uploads/2021/04/image-6.png?resize=398%2C613&ssl=1)A more aggressive fan curve to cool VRAM
I had been warned by a kind comment on my last post that whilst my GPU’s core clock temp might be low, the overclocked VRAM temp might be extremely high and limiting the performance of the card. What’s more it would not be conducive to a long living graphics card.

I use my profiles in MSI Afterburner to switch back to stock settings when I want to play some video games – I am not a dedicated miner. So killing my card prematurely is not something I want to proceed in doing. I have chose to keep my fan on auto, to spare my ears, but applied a more aggressive fan curve to help keep the card temps down.

My particular Gigabyte card does not have a dedicated VRRAM temperature sensor – or at least it is not recognized by [GPU-Z](https://www.techpowerup.com/gpuz/ "https://www.techpowerup.com/gpuz/") or [HWinfo](https://www.hwinfo.com/ "https://www.hwinfo.com/"). Therefore, my only hope is to keep the average temperate of the card under control.

**(If you are going to follow this step, remember to tick the box to ‘Enable user defined software automatic fan control’)**

## What to do with an RTX 3060 in 2026

Mining on a consumer GPU stopped making money a long time ago. The card itself is still excellent for one job: it has 12GB of VRAM, which is the sweet spot for running local language models. Mine now serves Ollama from a Proxmox container and that is the post to read next: [running Ollama in a Proxmox LXC with NVIDIA GPU passthrough](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/). If you kept your 3060 for the same reasons I kept mine, the Afterburner profile you want today is "stock".


---
Markdown version of https://exitcode0.net/posts/more-rtx-3060-nicehash-overclocking/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
