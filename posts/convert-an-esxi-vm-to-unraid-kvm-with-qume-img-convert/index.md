# Convert an ESXi VM to UNRAID KVM with qume-img convert

> Convert an ESXi VMDK virtual machine to UNRAID KVM with qemu-img convert, keeping the Windows drivers and boot configuration intact.

Source: https://exitcode0.net/posts/convert-an-esxi-vm-to-unraid-kvm-with-qume-img-convert/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-02-03
Updated: 2026-10-08
Tags: convert-vmdk, esxi, kvm, unraid



Here’s a quick guide on how to convert ESXi VMs to and UNRAID KVM virtual machine – it’s a simple process but a minefield to navigate on searches and forums.

I currently have a windows 10 VM running on an ESXi server (vmdk) which I want to migrate to my UNRAID server which uses KVM to host VMs. The vmdk file is unfortunately thick provisioned, so I have had to do some disk juggling, due to my limited amount of available of space on my array.

Step 1 – Copy the VMDK files to the UNRAID server
-------------------------------------------------

There are a number of ways to do this, but I chose to mount an UNRAID share as an NFS datastore in ESXi and copy the VM folder across (note: copy not move). Remember to power off the VM before doing this. If you need help mounting NFS shares, just leave a comment below.

Step 2 – Convert ESXi vmdk file to an img file using ‘qemu-img convert’
-----------------------------------------------------------------------

Now that we have a copy of the vmdk files on our UNRAID server, we can use the **qemu-img** tool to convert the files to a format that UNRAID can use to boot the VM – a .img file. Get console access to your UNRAID server and use the following command format:

```
qemu-img convert -p -f vmdk -O raw /mnt/user/<the location of your vmdk file> /mnt/user/<the location of your new file>.img
```

* -p shows you the progress of the conversion. The larger the vmdk, the longer it will take
* -f the input file type
* -O the output file type
* /mnt/user/ is where UNRAID stores its shares.

My VM was thick provisioned, so my output file was the same size as the vmdk. If your vmdk is thin provisioned, this process will swell the .img file – effectively making it thick provisioned – so make sure that you have enough available disk space for the conversion. I didn’t have this space available so I have to use the [unassigned devices plugin](https://forums.unraid.net/topic/44104-unassigned-devices-managing-disk-drives-and-remote-shares-outside-of-the-unraid-array/ "https://forums.unraid.net/topic/44104-unassigned-devices-managing-disk-drives-and-remote-shares-outside-of-the-unraid-array/") and mount a USB hard drive to complete the conversion. Unassigned drives are mounted in /mnt/disks/.

Step 3 – Create a KVM VM with the converted .img file
-----------------------------------------------------

Now we can create a VM using our newly created .img file; here are the settings I used for my converted Windows VM:

The CPU, memory, graphics card and other passthrough devices have no impact on the machine booting up. The VM will likely need to reconfigure itself to use the new hardware setup and you might need to install the virtIO network drivers. 

If you are having problems, then it is best to keep the VM config simple and add devices as you solve problems.

### Physical machine to UNRAID VM?

Much of the same principals of this guide can be applied to moving a physical windows machine over to an UNRAID, KVM virtual machine. I will be covering this in future posts – using Clonezilla to build a .img file of a physical Windows machine.



---
Markdown version of https://exitcode0.net/posts/convert-an-esxi-vm-to-unraid-kvm-with-qume-img-convert/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
