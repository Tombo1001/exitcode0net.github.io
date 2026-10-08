# Unraid USB Controller Passthrough - VM hot swap USB

> Pass a USB controller through to an UNRAID VM for hot-swappable USB devices, using an Inateck PCIe card and libvirt device configuration.

Source: https://exitcode0.net/posts/unraid-usb-controller-passthrough-vm-hot-swap-usb/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-03-03
Updated: 2026-10-08
Tags: libvirt, unraid, usb-hot-swap



![Unraid USB Controller Passthrough - Fresco FL1100](https://images-na.ssl-images-amazon.com/images/I/81koU3VBFOL._AC_SL1500_.jpg)
For the longest time, I have been running VMs on my Unraid server and had the need to passthrough USB devices. I will be the first to admit that I have too many computers and a shrink might classify it as an addition. Coupled with my quest for a clutter-free desk, I have been faced with the issue of swapping USB devices like a keyboard and mouse, from one machine to another Unraid virtual machine (vm).

Solving the physical component of changing the device from one machine to another is easy. Just acquire a USB switcher/KVM such as this:

[![](https://images-na.ssl-images-amazon.com/images/I/51LZkoc-DUL._SL1001_.jpg)](https://www.amazon.co.uk/UGREEN-Sharing-Keyboard-Scanner-Printer-Black/dp/B01N6GD9JO/ref=sr_1_3?dchild=1&keywords=usb+kvm&qid=1614788907&sr=8-3 "https://www.amazon.co.uk/UGREEN-Sharing-Keyboard-Scanner-Printer-Black/dp/B01N6GD9JO/ref=sr_1_3?dchild=1&keywords=usb+kvm&qid=1614788907&sr=8-3")A simple USB switch / KVM
There are hundreds of these and they are relatively cheap. However, the real problem is that when you switch the keyboard and mouse into the Unraid server (in an attempt to ‘plug’ them into a VM), they are assigned to the Unraid host, not a VM.

In the past I have developed a way to use a Pi with a button attached to send [virsh](https://linux.die.net/man/1/virsh "https://linux.die.net/man/1/virsh") commands to Unraid, connecting the USB device to a particular VM. This worked well, but required me to push 2 buttons; one to physically switch the USB devices, followed by another to run the [virsh](https://linux.die.net/man/1/virsh "https://linux.die.net/man/1/virsh") command from a Pi – only mildly irritating. The main irritation was the limit to the number of times you could ‘attach’ a USB device to a VM before its virtual hub became full. So I tried removing the device before attaching it (all using virsh) and ultimately ran out of skill and/or patients.

What I really needed was a USB controller to be permanently pass through to the VM. Then, whenever a device was plugged into that USB controller, it would be connecting to the VM and never the Unraid host. And so then next issue (which we are about to solve), which USB controller and how to pass it through?

Inateck Superspeed 4 Ports PCI-E to USB 3.0
-------------------------------------------

I found the ‘Inateck Superspeed 4 Ports PCI-E to USB 3.0’ on [Amazon](https://www.amazon.co.uk/gp/product/B00B6ZCNGM/ "https://www.amazon.co.uk/gp/product/B00B6ZCNGM/") and took a bit of an educated guess on whether I would be able to pass it through. The supported card/controllers list is very thin on the ground.

Importantly, it used the Fresco FL1100 series chipset, which to the best of my reading, can be passed through to a VM. It is also popular with crypto mining for this reason.

How to pass through the USB controller
--------------------------------------

**Prerequisites:**

* A working VM
* CPU and Bios that supports VT-d (and has it enabled)
* USB pcie card

### Find the device identifier

With the USB card plugged in, we must first identify it. **Sadly, I cannot confirm that this card will show up in its** own IOMMU group as it did for me, so your mileage may vary. My card shows up in its own IOMMU group and I can see its ID: 1b73:1100.

*Fresco FL1100 USB Controller*
### Edt Boot config

Now we need to tell Unraid to ignore/backlist the device by its ID, so that it will be available for passthrough. Go to the Main page, and click on your flash device to start editing the Syslinux boot config:

Under ‘*kernel /bzimage*‘ paste the following line, replacing the ID with yours appropriately:

```
  append initrd=/bzroot pci-stub.ids=1b73:1100
```

**Save** this, then it is time to **reboot**.

### Edit the VM

Now we can edit the VM (with it powered off) and add the pcie USB card:

**SUCCESS!**

You should now have true USB hot swap in your UNRAID VM. I have had this working reliably for a number of weeks at this point, the passthrough config survives a complete system reboot and UNRAID upgrades.



---
Markdown version of https://exitcode0.net/posts/unraid-usb-controller-passthrough-vm-hot-swap-usb/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
