# Passing Through an NVMe Drive to an ESXi Virtual Machine

> Pass through NVMe drives to ESXi virtual machines via RDM raw device mapping. Enable VM direct storage access to physical NVMe disks for specialised workloads.

Source: https://exitcode0.net/posts/esxi-nvme-disk-passthrough/
Author: Tom Cocking (https://tomcocking.com)
Published: 2023-05-04
Updated: 2026-10-08
Tags: esxi, nvme, passthrough, vmware, hardware


> **Note:** Broadcom completed its acquisition of VMware in November 2023 and discontinued the free ESXi hypervisor tier. The steps in this article remain technically valid for licensed VMware vSphere. If you're running a home lab, [Proxmox VE](https://www.proxmox.com/en/proxmox-virtual-environment/overview) is a capable free alternative worth considering.

ESXi is a powerful hypervisor that allows for the virtualization of multiple operating systems on a single physical machine. However, there are times when you may need to pass through a physical device, such as an NVMe drive, to a virtual machine. In this blog post, we will explore the steps required to pass through an NVMe drive to an ESXi virtual machine.

## Prerequisites

Before proceeding, ensure that you have the following:

- `root` access to an ESXi server
- An NVMe drive that you want to pass through to a virtual machine
- Knowledge of the datastore and existing virtual machine name that you want to use

## Step 1: Enable SSH on your ESXi Server

The first step is to enable SSH on your ESXi server. This can be done by logging into the ESXi web client and navigating to "Manage" > "Settings" > "Security & Users" > "Services." From here, you can enable SSH and start the SSH service.

## Step 2: SSH into Your ESXi Server

Once SSH is enabled, you can SSH into your ESXi server by opening a terminal and running the following command:

```
ssh root@192.168.1.10
```

Replace `192.168.1.10` with the IP address of your ESXi server.

## Step 3: Create a Storage Directory

Next, create a directory where you can store the virtual disk file. This can be done with the following command:

```
mkdir /vmfs/volumes/datastore_name/existing_vm_name/storage
```

Replace `datastore_name` with the name of the datastore that you want to use and `existing_vm_name` with the name of the existing virtual machine that you want to use.

## Step 4: List Available Disks

List the available disks on your ESXi server with the following command:

```
ls -l /vmfs/devices/disks
```

Make a note of the name of the NVMe disk that you want to pass through to the virtual machine. It should begin with `t10.NVMe`.

## Step 5: Create the Virtual Disk File

Create the virtual disk file with the following command:

```
vmkfstools -z /vmfs/devices/disks/t10.NVMe_____YOUR_DISK_NAME_____ "/vmfs/volumes/datastore_name/existing_vm_name/storage/disk_RDM_1.vmdk"
```

Replace `YOUR_DISK_NAME` with the name of the NVMe disk that you want to pass through (found in step 4), `datastore_name` with the name of the datastore that you want to use, and `existing_vm_name` with the name of the existing virtual machine that you want to use.

## Step 6: Edit the Existing VM Settings

Back in the ESXi web interface, edit the settings of the existing virtual machine that you want to pass the NVMe drive through to. Navigate to `Edit Settings`, click `Add Other Device` and select `NVMe Controller`.

## Step 7: Add the Virtual Hard Disk

Add the virtual hard disk to the virtual machine by selecting "Existing Hard Disk" and browsing to the virtual disk file that you created in Step 5. Following along with our example, the file would be found at `datastore_name/existing_vm_name/storage/disk_RDM_1.vmdk` in the Datastore browser.

## Step 8: Configure the Virtual Disk

Expand the hard disk options of the newly added disk and configure it with the following settings:

- Set Controller Location to: NVMe Controller 0, NVMe (0:0).
- Set Disk Compatibility to: Virtual
- Set Disk Mode to: Independent - Persistent
- Set Disk Compatibility Mode back to: Physical

## Cleanup and Conclusion

Now you can save and close the VM settings, before powering on an letting the VM boot. In my case, I passed through my old laptop NVMe disk so that I could access and backup some old file. Lastly, don't forget to resecure your ESXi server, by disabling SSH.

---
Markdown version of https://exitcode0.net/posts/esxi-nvme-disk-passthrough/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
