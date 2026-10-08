# More RTX 3060 Nicehash overclocking

> RTX 3060 NiceHash overclocking settings that reached 48 MH/s in 2021: MSI Afterburner core reduction, memory boost and a fan curve for VRAM temperatures.

Source: https://exitcode0.net/posts/more-rtx-3060-nicehash-overclocking/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-04-07
Updated: 2026-10-08
Tags: crypto, nicehash, overclocking, rtx-3060


> **Note:** NVIDIA has since patched the RTX 3060 hash rate limiter across multiple driver versions. The developer driver 470.05 used in this article is no longer available from NVIDIA. Crypto mining on consumer GPUs is generally no longer economically viable. If you still have the card, it is far more useful today running local LLMs. See [running Ollama in a Proxmox LXC with NVIDIA GPU passthrough](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/).

![RTX 3060 Nicehash overclocking - 25% improvement](https://i1.wp.com/exitcode0.net/wp-content/uploads/2021/04/rtx-nicehash.png?resize=232%2C232&ssl=1)
This is part 2 from my previous post on RTX 3060 Nicehash overclocking settings. I don’t want to edit the previous article because the content still stands to be accurate for the hash rate I achieved. However, I have since learned even more about the card and managed to improve my Nicehash quick miner hash rate by a further 10%!

If you want to see the first/part1 post, you can do so here: [RTX 3060 Nicehash mining overclock settings](https://exitcode0.net/posts/rtx-3060-nicehash-mining-overclock-settings/ "https://exitcode0.net/posts/rtx-3060-nicehash-mining-overclock-settings/")  
And if you are looking for my post (and the download) on unlocking the hash rate with the 470.05 driver, that’s here: [Unlock RTX 3060 mining hash rate](https://exitcode0.net/posts/unlock-rtx-3060-mining-hash-rate/).

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



---
Markdown version of https://exitcode0.net/posts/more-rtx-3060-nicehash-overclocking/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
