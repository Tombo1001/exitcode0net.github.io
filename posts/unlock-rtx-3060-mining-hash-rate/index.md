# Unlock RTX 3060 mining hash rate

> How the 470.05 developer driver unlocked the RTX 3060 mining hash rate in 2021, taking NiceHash from 22 MH/s to 42 MH/s. Kept for historical reference.

Source: https://exitcode0.net/posts/unlock-rtx-3060-mining-hash-rate/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-03-21
Updated: 2026-10-08
Tags: 470.05, nicehash, rtx-3060


> **Note:** The developer driver 470.05 referenced in this article is no longer distributed by NVIDIA. NVIDIA subsequently re-introduced and then re-patched the hash rate limiter across multiple driver generations. This article is kept for historical reference. If you still have the card, it is far more useful today running local LLMs. See [running Ollama in a Proxmox LXC with NVIDIA GPU passthrough](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/).

There has been lots of news coverage around the recent mistake made by Nvidia with their RTX 3060 driver allowing uninhibited mining hash rates: a great round-up from [The Verge](https://www.theverge.com/2021/3/16/22333544/nvidia-rtx-3060-ethereum-mining-rate-limit-unlock-driver "https://www.theverge.com/2021/3/16/22333544/nvidia-rtx-3060-ethereum-mining-rate-limit-unlock-driver"). A developer driver inadvertently included code used for internal development which removes the hash rate limiter on RTX 3060 in some configurations. The driver has been removed, but there are lots of copies in circulation.

There are still some limitations, you must have a monitor attached and can only run a single RTX 3060 at a time. However, it is only a matter of time before motivated miners get over the remaining hurdles.

![unlock RTX 3060 mining hash rate - gigabyte RTX 3060](https://i0.wp.com/www.gigabyte.com/FileUpload/Global/KeyFeature/1728/innergigabyteimages/kf-img.png?resize=500%2C286&ssl=1)GeForce RTX 3060 GAMING OC 12G
How to unlock RTX 3060 mining hash rate
---------------------------------------

**My setup:**

* A single **RTX 3060** connected to a monitor
* Running Nicehash on Windows 10

I am just a gamer looking to recoup some of the extortionate price I paid for a graphics card which I spent weeks trying to buy 😊 I use NiceHash, because it is convenient and profitable for me, I am not a dedicated miner. If you want to get started with NiceHash, here is my referral link: **[https://www.nicehash.com/?refby=b5363bcf-2c11-4482-82d2-d224cd1895ab](https://www.nicehash.com/?refby=b5363bcf-2c11-4482-82d2-d224cd1895ab "https://www.nicehash.com/?refby=b5363bcf-2c11-4482-82d2-d224cd1895ab")**.

NiceHash covers the impact that the driver limitation has on the RTX 3060’s hashing rate here: [https://www.nicehash.com/blog/post/nvidia-rtx-3060-mining-hashrate](https://www.nicehash.com/blog/post/nvidia-rtx-3060-mining-hashrate "https://www.nicehash.com/blog/post/nvidia-rtx-3060-mining-hashrate"). On the Gigabyte OC card that I have, I was seeing ~22MH/s before installing the development driver. Whilst this still profitable, it is close to 50% of what the card is capable of.

### Installing Nvidia development driver 470.05

I was early on the news as was able to get a copy of the driver from a forum user who had repackaged it – noting that the driver didn’t work for MSI cards. The download was hosted on Mega, which always gives me a reason to worry, so I ran the download through virus total first and it came back clean. But it really is the case that when you start installing drivers from alternative sources, you are putting yourself at considerable risk.

Whilst I can’t confirm how long this download will remain available, I wish you the best of luck in obtaining it:

[https://drive.google.com/file/d/12jIy0zkJlikzfUevEmsQRWj8-IP91K-9/view?usp=sharing](https://drive.google.com/file/d/12jIy0zkJlikzfUevEmsQRWj8-IP91K-9/view?usp=sharing "https://drive.google.com/file/d/12jIy0zkJlikzfUevEmsQRWj8-IP91K-9/view?usp=sharing")

I would recommend running a PowerShell hash check on the file to make sure that it matches below – this will tell you if it has been co-opted:

```
PS C:\Drivers> **Get-FileHash .\470.05.zip**

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
SHA256          C97C25360E76ED1252B18E583B93F6446EB3801136F38E9214B770C1A4053A16       C:\Drivers\470.05.zip
```

It installs much like installing any other Nvidia driver, run the setup file and reboot when you are done. After a reboot, open up the Nvidia control panel and check your driver version:

![unlock RTX 3060 mining hash rate - dev driver installed](https://i0.wp.com/exitcode0.net/wp-content/uploads/2021/03/image-1.png?resize=334%2C149&ssl=1)RTX 3060 running driver 470.05
Now that I am running this beta driver, here is some output from NiceHash running the daggerhashimoto algorithm. I am seeing between 39 and 42 MH/s; this is without any overclocking tweaks to the GPU.

![unlock RTX 3060 mining hash rate - 470.05 driver](https://i1.wp.com/exitcode0.net/wp-content/uploads/2021/03/image.png?resize=774%2C279&ssl=1)Effecive hashrate between 39 and 42 MH/s – Driver version 470.05
Big disclaimer… You are responsible for your own hardware and system integrity. This is a very bleeding edge subject, so your mileage may vary considerably. Happy mining!



---
Markdown version of https://exitcode0.net/posts/unlock-rtx-3060-mining-hash-rate/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
