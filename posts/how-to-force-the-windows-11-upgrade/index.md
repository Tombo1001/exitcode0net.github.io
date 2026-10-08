# How to Force the Windows 11 Upgrade Now Windows 10 Is Out of Support

> Force the Windows 11 upgrade with Microsoft's Installation Assistant, check compatibility first, and what the Windows 10 end of support means for the machines left behind.

Source: https://exitcode0.net/posts/how-to-force-the-windows-11-upgrade/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-10-06
Updated: 2026-10-08
Tags: windows-11, windows-10, windows-update



> **Updated October 2026:** Windows 10 reached end of support on 14 October 2025. Machines still on it get no security updates unless enrolled in Extended Security Updates, so forcing the upgrade is now the sensible default rather than a way to jump a queue. The Installation Assistant steps below still work. My two older posts on forcing the Windows 10 2004 and 20H2 feature updates have been folded into the end of this page, since those updates no longer matter.

Windows 11 became available on the 5th October 2021 and for those people with compatible machines, this guide will help you jump the queue and force the Windows 11 upgrade. 

**Before you start…**

* Take a backup all all important data onto an external storage device/location.
* set aside at least 1 hour for the upgrade – maybe longer, depending on your internet connection speed.
* Make sure that your Windows 10 installation is up to date with no pending updates.

Compatibility
-------------

You might have seen a number of articles and news stories floating around about Windows 11 dropping support for a large number of computers, particularly older systems. The quickest way to test your system’s compatibility with Windows 11 is using the Microsoft **WindowsPCHealthCheckSetup** tool: [https://aka.ms/GetPCHealthCheckApp](https://aka.ms/GetPCHealthCheckApp "https://aka.ms/GetPCHealthCheckApp")

![](https://i1.wp.com/exitcode0.net/wp-content/uploads/2021/10/win11-2.png?resize=774%2C758&ssl=1)Windows Health Check Tool
Once you have the application installed and running, press the ‘**Check now**‘ button to test Windows 11 compatibility.

Downloading the Windows 11 installer
------------------------------------

To get started with the upgrade, first download the installer:

![](https://i2.wp.com/exitcode0.net/wp-content/uploads/2021/10/win11-1.2-1.png?resize=774%2C301&ssl=1)[https://www.microsoft.com/en-us/software-download/windows11](https://www.microsoft.com/en-us/software-download/windows11 "https://www.microsoft.com/en-us/software-download/windows11")
![](https://i0.wp.com/exitcode0.net/wp-content/uploads/2021/10/win11-8.png?resize=370%2C229&ssl=1)
Once the installer is running, it will start to download Windows 11 and begin the in-place upgrade. This will not perform a clean install of Windows and retain most of your settings and keep all of your files and applications.

![](https://i1.wp.com/exitcode0.net/wp-content/uploads/2021/10/win11-5.png?resize=774%2C549&ssl=1)
Once the installer reaches 100% on stage 3/3, your computer will ask to restart or automatically restart after a 15 minutes countdown. This reboot might take longer than usual as the upgrade is applied. On completion, expect to see a Windows 11 welcome screen.

**And finally, your system will now be running Windows 11!**

![](https://i0.wp.com/exitcode0.net/wp-content/uploads/2021/10/win11-7.png?resize=682%2C628&ssl=1)

## If the PC Health Check says no

The usual blockers are TPM 2.0 disabled in firmware, Secure Boot off, or a CPU below Microsoft's supported list. The first two are fixed in the UEFI settings and are worth checking before writing the machine off. An unsupported CPU is a different story. The upgrade can be forced with registry edits, but Microsoft doesn't guarantee updates on those installs, so for a machine that's just a browser and a terminal I'd put Linux on it instead. That's roughly what I did with my own laptop in [turning it into a dumb terminal for a Tailscale dev VM](https://exitcode0.net/posts/dumb-terminal-dev-vm-tailscale-claude-code/).

## Windows 10 feature updates (historical)

Before Windows 11, the same trick applied to Windows 10's twice-yearly feature updates. Microsoft staggered the over-the-air rollout, and if you were tired of hitting "check for updates", the Windows 10 download page offered an Update Assistant that did the in-place upgrade immediately. I used it for the 2004 update in June 2020 (mostly to get WSL2) and the 20H2 update in January 2021. One tip from both: suspend BitLocker or any third-party full-disk encryption before running it. None of that is needed now. If a machine is still on Windows 10, the only feature update that matters is the one to Windows 11 above.

## Or just hand this page to your agent

```text
Read https://exitcode0.net/posts/how-to-force-the-windows-11-upgrade/ and help me upgrade this PC to Windows 11. First run the PC Health Check (or check TPM, Secure Boot and the CPU model yourself) and show me the results. Confirm there's a backup before starting, and do not change firmware settings or registry keys without asking.
```

The page gives the agent the order of operations. Only you know whether the backup exists and whether this machine is worth upgrading.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/how-to-force-the-windows-11-upgrade/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
