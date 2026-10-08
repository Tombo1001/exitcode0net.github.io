# How to Expand an Ubuntu LVM Logical Volume to Use All Free Space on the PV

> Expand an Ubuntu LVM logical volume to use all free space on the physical volume with lvextend and resize2fs, when the root partition fills up.

Source: https://exitcode0.net/posts/how-to-expand-an-lvm-lv-to-use-all-space-on-the-pv/
Author: Tom Cocking (https://tomcocking.com)
Published: 2024-09-04
Updated: 2026-10-08
Tags: linux, lvm, ubuntu


> **Updated October 2026:** added the one-line quick answer, the "I grew the virtual disk first" path (growpart and pvresize), a troubleshooting section and the agent handoff block. The original walkthrough below is unchanged.

## Quick answer

If your Ubuntu root volume is full and `vgdisplay` shows free space in the volume group, this one command gives the whole lot to the root LV and grows the filesystem in the same step:

```bash
sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

`-l +100%FREE` means "all the free extents", and `-r` runs the filesystem resize for you (ext4 or XFS). Swap the path for your own LV, which `lvdisplay` shows as `LV Path`. The rest of this post explains what's going on and covers the case where the free space isn't showing up yet.

**2024 Update** - I found myself stuck in this situation again, my Ubuntu root partition was completely full to the brim... I'm scrambling to remove apt cache and pruning all docker images, but then I run `vgdisplay` only to realise that my LVM LV isn't even using the full amount of physical disk space (this is a VM and I have just expanded the VMDK file). Thankfully I wrote this blog post 4 years ago as a how-to. Well now I am going to give this post a refresh and this includes some extra details which I needed during the panic of a full disk.

This title has far too many acronyms so I will start by clarifying the problem that this post aims to solve. I have a Ubuntu server instance running LVM ([Logical Volume Manager](https://wiki.ubuntu.com/Lvm "https://wiki.ubuntu.com/Lvm")). My primary LV (Logical Volume) was ‘full’ but was only using 4GB of a 29GB PV (Physical Volume). So if you are suffering the same issue, here are the commands you need to fix this problem.

Let's break down the acronyms and understand why your Logical Volume (LV) might not be using all the available space on your Physical Volume (PV).

- LVM (Logical Volume Manager): Think of LVM as a flexible way to manage your hard drive space in Linux. It sits on top of your physical disks and lets you carve them up into logical units.
- PV (Physical Volume): This is the actual hard drive or partition that LVM is managing. It's the raw storage space.
- LV (Logical Volume): This is a chunk of space that LVM carves out from the PV. You can think of it as a virtual partition. It's what your operating system sees and interacts with.

### Why not use all the PV space when creating a LV?

When you first set up your LVM, you might not allocate all the space on your PV to your LV. This is often intentional, leaving room for:

- Future Expansion: You might want to expand your LV later without having to add more physical disks.
- Creating More LVs: You might want to create additional LVs on the same PV for different purposes (e.g., one for the operating system, one for data).

**Key Takeaway**: Just because you have free space on your PV doesn't mean your LV automatically uses it. You need to explicitly tell LVM to extend the LV and then resize the filesystem to make use of that extra space.

## Ok Let's fix this...

### Free up some space:

It might be possible to free up some space without the need to start expanding disks and volumes.

We can check free space on our partitions by look at the `df -h` output:

| Filesystem             | 1K-blocks | Used    | Available | Use%     | Mounted on |
| ---------------------- | --------- | ------- | --------- | -------- | ---------- |
| /dev/mapper/i-hate-lvm | 5166504   | 5150120 | **0**     | **100%** | /          |

`df -h` Shows the space available in mounted partitions
This is fictitious data, but this is reflective of the fact that the root filesystem is full. This caused me a **LOT** of headaches and I needed to free up some free space on that partition before I could run any of the upcoming commands to expand the LV. Ways to free up space include:

```bash
sudo apt clean
sudo apt autoremove -y
```

If you are running snap, you can use the following command to remove older versions of a snap application:

```bash
snap remove "$snapname" --revision="$revision"
```

## Finding the available PV space

Now that we have a tiny bit of space on ‘/’ we cab start to work on expanding the available space in the LV. First, we need to find out the available space in the PV which is where the LV ‘lives’.

```bash
vgdisplay
```

the value we are looking for specifically is:

```bash
Free  PE / Size       2319 / 9.06 GiB
```

As per the example above, we have just over 9GB available. I want to go ahead and give all of that space to my one and only LV.

### Free PE / Size shows 0 but you grew the disk

If you expanded the virtual disk in Proxmox, VMware or VirtualBox and `vgdisplay` still shows no free extents, the new space hasn't reached LVM yet. The partition and the physical volume both need growing first. On a default Ubuntu install the PV is partition 3 of the main disk:

```bash
sudo growpart /dev/sda 3      # grow the partition into the new disk space
sudo pvresize /dev/sda3       # tell LVM the PV got bigger
sudo vgdisplay                # Free PE / Size should now be non-zero
```

`growpart` comes from the `cloud-guest-utils` package. Check the disk and partition names with `lsblk` before running either command, because on an NVMe disk it'll be `/dev/nvme0n1` and `/dev/nvme0n1p3`.

## Extending LV with lvextend

First we need to find the LV path:

```bash
lvdisplay
```

In my hypothetical example this is:

```bash
    --- Logical volume ---
    LV Path                /dev/i-hate-lvm
```

So now we can extend the LV, consuming the available disk space, this is done with the `lvextend` command:

```bash
sudo lvextend -l +2319 /dev/i-hate-lvm
```

Adjust the `2319` value and the `/dev/` path to suit your required level of expansion and the LVM volume. If you just want everything, `-l +100%FREE` saves you reading the number off `vgdisplay`, and adding `-r` does the filesystem resize in the next section for you:

```bash
sudo lvextend -r -l +100%FREE /dev/i-hate-lvm
```

## Extending an LV filesystem with resize2fs

Finally, now that the LV is larger, we need to expand the filesystem on the volume to make the newly acquired space available for use. Fortunately this another really simple command:

```bash
resize2fs /dev/mapper/i-hate-lvm
```

Pay attention to the fact that I am running this command against `/dev/mapper/<volume-name>`.

If the filesystem is XFS rather than ext4 (`df -T` tells you), `resize2fs` will refuse. Use `sudo xfs_growfs /` with the mount point instead.

Run the `df -h` command and again and you should now have ALL OF THE FREE SPACE!

---

Congratulations! You've successfully navigated the sometimes-confusing world of LVM and expanded your logical volume. By using lvextend to claim the available space on your physical volume and then resize2fs to make that space usable by your filesystem, you've given your Ubuntu system some much-needed breathing room. No more frantic cache cleaning or image pruning – for now, at least! Remember to keep an eye on your disk usage and repeat this process as needed to keep your system running smoothly.

Consider giving this post a bookmark or a share to help out a fellow LVM sufferer in a moment of need.

## Troubleshooting

**`lvextend` says "No free extents" or "Insufficient free space"**
The volume group has nothing left to give. Either the disk really is full, or you grew the virtual disk and skipped `growpart` and `pvresize` above.

**`resize2fs` says "Bad magic number in super-block"**
The filesystem isn't ext4. Run `df -T` and use `xfs_growfs` for XFS.

**`growpart` says "NOCHANGE"**
The partition already fills the disk, so the missing space is somewhere else. Check `lsblk` for unallocated space and `pvs` for a PV that isn't in the volume group.

**`df -h` still shows the old size after `lvextend`**
You extended the LV but not the filesystem. Run `resize2fs` (or use `lvextend -r` next time).

---

## More useful Ubuntu wrangling posts:

- How to upgrade Ubuntu from 18.04 to 20.04 – [https://exitcode0.net/how-to-upgrade-ubuntu-from-18-04-to-20-04/](https://exitcode0.net/posts/how-to-upgrade-ubuntu-from-18-04-to-20-04/)
- Setup the Nextcloud Snap Package with HTTPS and a separate data path – [https://exitcode0.net/setup-the-nextcloud-snap-package-with-https-and-a-separate-data-path/](https://exitcode0.net/posts/setup-the-nextcloud-snap-package-with-https-and-a-separate-data-path/)

## Or just hand this page to your agent

If you'd rather not type any of this, paste the following into Claude Code or whichever agent you use:

```text
Read https://exitcode0.net/posts/how-to-expand-an-lvm-lv-to-use-all-space-on-the-pv/ and expand my root logical volume to use all free space in its volume group. Before running anything, run lsblk, df -T, vgdisplay and lvdisplay and show me the output, and confirm the disk, partition, VG name, LV path and filesystem type with me. Do not run growpart or pvresize unless vgdisplay shows no free extents and I confirm the disk was recently enlarged.
```

The page gives the agent the commands and the order to run them in. Only you know which disk was grown, what the volume group is called and whether there's anything on that disk you'd rather it didn't touch.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/how-to-expand-an-lvm-lv-to-use-all-space-on-the-pv/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
