# RTX 3060 Nicehash mining overclock settings

> RTX 3060 NiceHash overclock settings from 2021 that reached 44+ MH/s: core clock reduction and memory overclock in MSI Afterburner. Kept for reference.

Source: https://exitcode0.net/posts/rtx-3060-nicehash-mining-overclock-settings/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-04-02
Updated: 2026-10-08
Tags: crypto, msi-afterburner, nicehash, rtx-3060


> **Note:** NVIDIA has since patched the RTX 3060 hash rate limiter across multiple driver versions. The developer driver 470.05 used in this article is no longer available from NVIDIA. Crypto mining on consumer GPUs is generally no longer economically viable. If you still have the card, it is far more useful today running local LLMs. See [running Ollama in a Proxmox LXC with NVIDIA GPU passthrough](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/).

I recently managed to buy a Gigabyte RTX 3060 without lining the pockets of a scaler, lucky me! I recently discussed how I was able to circumvent the Nvidia crypto mining limitations: [Unlock RTX 3060 mining hash rate](https://exitcode0.net/posts/unlock-rtx-3060-mining-hash-rate/) (that post also includes access to the driver). This doubled the RTX 3060 Nicehash mining rate using the ‘unlocked’ Nvidia beta driver – going from ~22MH/s up to ~40MH/s. Impressive!

Now I am going to share my RTX 3060 overclock settings, which enabled me to achieve a further 10% uplift in Nicehash performance. But first I must disclaimer that you are responsible for your own hardware and any changes you make are at your own risk. What’s more, modern silicone is a lottery, you might not get these exact mining hash rates, or you might get more; keep an eye on your thermals and protect your card and safety of your system accordingly!

\*\*Important Note - this post is based on a V1 (non LHR RTX 3060)\*\*

![](\_\_GHOST\_URL\_\_/content/images/2021/11/rtx-nicehash.png)

## RTX 3060 mining overclock settings
Before the overclock I was seeing between 38 and 41.5 MH/s when Nicehash was running the DaggerHashimoto algorithm (on the 470.05 driver). After some tweaks in MSI Afterburner, I was able to consistently strike 44+MH/s, a nice 10% increase.

My settings are mainly based on those recommended for the RTX 3060Ti here: https://miningchamber.com/gpu-mining/rtx-3060-ti-mining-settings. The synopsis is as follows:

Lower the core clock rate
Lower the power limit(%)
Increase the memory clock rate

![](\_\_GHOST\_URL\_\_/content/images/2021/11/Screenshot-2021-11-22-153315.png)

\*\*My RTX 3060 MSI afterburner settings:\*\*
- Power limit: 65%
- Core Clock: -400 MHz
- Memory Clock + 800 MHz
- Fan Speed: Auto

\*\*EDIT\*\*: I have since managed even greater hash rates, which I have covered in a new post: [More RTX 3060 Nicehash overclocking](https://exitcode0.net/posts/more-rtx-3060-nicehash-overclocking/).

![My current RTX 3060 Overclock for 47MH/s](\_\_GHOST\_URL\_\_/content/images/2021/11/Capture.PNG)

I would suggest that these settings rest on the side of caution and are stretching my card too hard. I am happy that the card is stable with these settings and is in fact running a little cooler at the same or lower fan speed than at stock. What’s more, Nicehash suggests that my total wattage has dropped from 140w to \*\*110w\*\* – lowering the ROI for the card.

I am using the profile switching function in MSI Afterburner to make it easy to flick back to stock settings and play some video games when I want to. I’d recommend this for anyone else mining on their card to record the outrageous cost of buying it.

Have you overclocked your RTX 3060? Share your afterburner settings and hashrate in the comments below… Let’s see who won the silicon lottery!



---
Markdown version of https://exitcode0.net/posts/rtx-3060-nicehash-mining-overclock-settings/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
