# How to pass through a drive to a UNRAID VM

> Pass a physical drive straight to an UNRAID VM for raw storage access and keep an existing Windows install by attaching its SSD to a KVM VM.

Source: https://exitcode0.net/posts/how-to-pass-through-a-drive-to-a-unraid-vm/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-12-12
Updated: 2026-10-08
Tags: libvirt, unraid, vm



I recently migrated my ‘physical’ Windows 10 desktop to a ‘virtual’ machine running under UNRAID (libvert). This is a quick guide on how I was able to use my existing installation on a SSD and pass it through to a VM.

Why passthrough a drive to a VM?
--------------------------------

In some cases, it might be beneficial to start from scratch when creating a Windows 10 VM on UNRAID, creating a new virtual disk and having a fresh install. I chose to take an alternative path because I was contempt with my Windows install as it was. I had my Steam/Epic Games library downloaded onto one of the two disks already in the system and a number of applications installed and configured just the way I like them.

In short, being able to preserve my disks in their current state would save me hours of unnecessary reconfiguration.

So why not create an image of those disks and place them on the UNRAID array or cache? Well, I plan to have multiple VMs on my UNRAID server (other Physical to Virtual conversions), a number of docker containers, applications and uses for the array which will all be IO heavy operations. Giving my Windows VM an independent disk subsystem will ensure that it never encounters Disk IO bottlenecks. For me, this is a desirable feature.

*My Windows 10 VM, with it’s two physical disks.*
The UNRAID prerequisites
------------------------

To pass through a drive to a UNRAID VM, there are a number of prerequisites that must be fulfilled. The following plugin will be needed:

Dlandon’s [Unassigned Devices](https://forums.unraid.net/topic/92462-unassigned-devices-managing-disk-drives-and-remote-shares-outside-of-the-unraid-array/ "https://forums.unraid.net/topic/92462-unassigned-devices-managing-disk-drives-and-remote-shares-outside-of-the-unraid-array/") – this is a great plugin that allows a drive that is not a part of the array or cache to be used in a number of ways. It makes it possible to mount, share, or pass through an unassigned device (disk).

*My two Windows disks marked for ‘pass thru’*
Once you have enabled ‘pass thru’ for your disk, it is worth taking a note of the disk IDs, these will be needed for the following stages:

* INTEL\_SSDSC2BF240A4L\_CVDA411205W12403GN
* WDC\_WD5000LPLX-75ZNTT0\_WXP1EC54W1JD

UNRAID VM Configuration
-----------------------

Now we can look to create our Windows 10 VM, using our passed through Disks (Intel SSD boot drive and Western Digital HDD data drive). Here is a brief list of my settings:

* Machine: Q35-4.2
* BIOS: OVMF
* Hyper-V: No
* Primary Disk: Manual, /dev/disk/by-id/<disk ID which we noted in the previous step>, sata
* 2nd Disk: Manual, /dev/disk/by-id/<disk ID which we noted in the previous step>, sata
* *Any other devices, CPU cores and Memory allocations*.

*My Windows 10 VM configuration.*
**NOTE:** My physical Windows installation was configured with UEFI (The C: drive was a GPT disk). If you were using legacy/BIOS, to boot into windows (Your C: drive is a BMR disk) you will need to set the UNRAID VM’s BIOS to SeaBIOS.

The Result
----------

A Windows 10 VM with ‘physical’ disks and in my case a physical GPU too! My Windows instance booted up, reconfigured to account for the change in available memory and CPU, and rebooted again once this was complete.

I did need to mount the virtIO ISO file and install chipset and network drivers – this also makes Windows observe a VM stop command as a shutdown signal, much like VMware tools.

---

More UNRAID Nuggets
-------------------

* [https://exitcode0.net/posts/convert-an-esxi-vm-to-unraid-kvm-with-qume-img-convert/](https://exitcode0.net/posts/convert-an-esxi-vm-to-unraid-kvm-with-qume-img-convert/ "https://exitcode0.net/posts/convert-an-esxi-vm-to-unraid-kvm-with-qume-img-convert/") – CONVERT AN ESXI VM TO UNRAID KVM WITH QUME-IMG CONVERT



---
Markdown version of https://exitcode0.net/posts/how-to-pass-through-a-drive-to-a-unraid-vm/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
