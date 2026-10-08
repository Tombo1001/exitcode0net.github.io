# How to make a VirtualBox VM the same resolution as host

> Match a VirtualBox VM's resolution to the host display with the VBoxSVGA controller, fullscreen scaling and dynamic resolution adjustment.

Source: https://exitcode0.net/posts/how-to-make-a-virtualbox-vm-the-same-resolution-as-host/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-01-16
Updated: 2026-10-08
Tags: resolution, virtualbox



Yet another simple problem/resolution. If you are looking to make your [VirtualBox](https://www.virtualbox.org/ "https://www.virtualbox.org/") VM’s resolution match that of your host making full-screen mode, truly full screen, look no further, here is the answer.

Once again we are looking at an issue caused by a default setting. You can set your VM to full-screen mode, but it not likely to rescale to the native resolution of your display.

The setting which you need to change requires you to have the VM powered off. Enter the settings for the VM > Display > Screen Tab > Graphics Controller: **VBoxSVGA**.

Once you have changed it to VBoxSVGA, you can boot your VM and it will automatically fill the available resolution within the bounds of the virtual box window. Subsequently, if you switch to full screen mode, the VM will change the resolution to match the host’s display.

*Change the graphics controller to VBoxSVGA*

---

Other VirtualBox tips and How-To posts
--------------------------------------

Clipboard and Shared Folders on Kali Linux with VirtualBox – [https://exitcode0.net/clipboard-and-shared-folders-on-kali-linux-with-virtualbox/](https://exitcode0.net/posts/clipboard-and-shared-folders-on-kali-linux-with-virtualbox/)



---
Markdown version of https://exitcode0.net/posts/how-to-make-a-virtualbox-vm-the-same-resolution-as-host/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
