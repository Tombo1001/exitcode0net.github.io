# How to Migrate VMs Between Standalone Proxmox Nodes Without Clustering

> A complete guide to migrating VMs between standalone Proxmox hosts without clustering, using vzdump streaming over SSH

Source: https://exitcode0.net/posts/migrate-vm-between-standalone-proxmox-nodes/
Author: Tom Cocking (https://tomcocking.com)
Published: 2026-02-15
Updated: 2026-10-08
Tags: proxmox, vm-migration, lvm, vzdump, ssh


Moving virtual machines between Proxmox hosts is straightforward when you have a cluster configured, but what if you're running standalone nodes? This guide walks through the process of migrating VMs between two independent Proxmox servers on the same network, without needing to set up clustering.

**[IMAGE: Diagram showing two Proxmox servers on a flat LAN with an arrow indicating VM migration between them]**

I recently needed to migrate several VMs from an older Proxmox host to a newer one with faster storage. Both systems were on the same network but not clustered, and I wanted to avoid the overhead of downloading backup files through the GUI. The solution? Streaming backups directly over SSH using Proxmox's built-in `vzdump` tool.

## The Scenario

Two standalone Proxmox hosts on the same LAN (not clustered):
- **Source host** (let's call it Node-A) - the old server with VMs to migrate
- **Target host** (let's call it Node-B) - new server with an LVM volume group ready for storage

The VMs are powered off and need to be transferred efficiently without consuming disk space on the source host.

## Prerequisites and Preparation

Before starting the migration, there are a few setup steps that will make the process much smoother.

### Verify Storage on the Target Host

LVM volumes don't appear in `df -h` output, so use `vgs` instead to check your volume groups and available space:

```bash
vgs
```

You should see your volume groups listed with their total size and free space.

**[IMAGE: Screenshot showing vgs output with volume group details and free space]**

### Register Storage in Proxmox

Make sure your target storage is registered in Proxmox. Check with:

```bash
pvesm status
```

If your LVM volume group isn't listed, add it through the GUI:
1. Navigate to **Datacenter → Storage → Add → LVM**
2. Select your volume group
3. Enable it for disk images and containers

### Set Up Passwordless SSH

The vzdump streaming method relies on SSH, so you need key-based authentication to prevent password prompts from interrupting the transfer.

On the source host (Node-A):

```bash
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519
ssh-copy-id root@<target-host-ip>
```

Test the connection and accept the host key fingerprint:

```bash
ssh root@<target-host-ip>
# Confirm you can connect, then exit
```

### Install tmux for Session Persistence

VM migrations can take hours depending on disk size and network speed. If your SSH connection drops mid-transfer, you'll lose everything. Install tmux to keep processes running:

```bash
apt install tmux
```

Start a tmux session before beginning any long-running transfers:

```bash
tmux
```

If you get disconnected, reconnect with:

```bash
tmux attach
```

**[IMAGE: Terminal screenshot showing tmux session with a migration in progress]**

## Step 1: Create a Staging Area

You need temporary storage on the target host to receive the backup file before restoring it. Create a logical volume for staging:

```bash
lvcreate -L 120G -n staging <vg-name>
mkfs.ext4 /dev/<vg-name>/staging
mkdir /mnt/staging
mount /dev/<vg-name>/staging /mnt/staging
```

### How Much Space Do You Need?

The `vzdump` tool only backs up **used data**, not the full allocated disk size, and compresses it with zstd. A 500GB disk with 60GB of actual data might compress down to 30-40GB.

To estimate the required space, check actual disk usage on the source host:

```bash
# Find the disk path
pvesm path <storage>:<vmid>/<disk-name>

# Check real usage
du -h /path/to/vm-disk
```

### Resizing the Staging Area

If you realize mid-transfer that you need more space:

```bash
umount /mnt/staging
lvresize -L 200G <vg-name>/staging
e2fsck -f /dev/<vg-name>/staging
resize2fs /dev/<vg-name>/staging
mount /dev/<vg-name>/staging /mnt/staging
```

> **Pro tip:** If the staging area and final VM storage share the same volume group, you'll need space for both simultaneously. Consider using a separate VG for staging to avoid contention.

## Step 2: Stream the Backup Over SSH

This is where the magic happens. Instead of creating a backup on the source host and then transferring it, we stream it directly to the target host.

On the source host (inside your tmux session):

```bash
tmux
vzdump <VMID> --mode stop --compress zstd --stdout | \
  ssh root@<target-host-ip> 'cat > /mnt/staging/vzdump-qemu-<VMID>.vma.zst'
```

**[IMAGE: Diagram showing data flow from vzdump → SSH pipe → target host staging area]**

This command:
- Stops the VM (if not already stopped)
- Creates a compressed backup stream
- Pipes it through SSH directly to the target host
- Saves it to the staging area

### Monitor Transfer Progress

On the target host, watch the file grow:

```bash
watch ls -lh /mnt/staging/
```

The file size should steadily increase. When it stops growing and the vzdump process on the source completes, the transfer is done.

## Step 3: Restore the VM

Once the backup file is fully transferred, restore it on the target host:

```bash
qmrestore /mnt/staging/vzdump-qemu-<VMID>.vma.zst <VMID> --storage <target-storage>
```

If the VM ID is already in use on the target host, choose a different number.

### Verify the Configuration

Check the restored VM's config file:

```bash
cat /etc/pve/qemu-server/<VMID>.conf
```

Pay special attention to the network bridge name (e.g., `vmbr0`). Make sure it matches your target host's network configuration. If the bridge names differ between hosts, update the config file accordingly.

### Start and Test

Power on the VM:

```bash
qm start <VMID>
```

Verify it boots correctly and has network connectivity.

**[IMAGE: Screenshot of Proxmox web GUI showing the migrated VM running successfully]**

## Step 4: Clean Up

Once you've confirmed the VM works on the target host:

**Remove the staging area:**

```bash
umount /mnt/staging
lvremove <vg-name>/staging
```

**Delete the original VM from the source host:**

```bash
qm destroy <VMID> --purge
```

This removes the VM and all associated disks.

**Optional - Remove SSH key:**

If you don't want to keep passwordless SSH access, edit `~/.ssh/authorized_keys` on the target host and remove the source host's public key.

## Troubleshooting Common Issues

### "VM is locked (backup)"

A previous failed backup left a lock file. Clear it with:

```bash
qm unlock <VMID>
```

### "VM is paused – cannot shutdown"

The VM is in a bad state. Force stop it:

```bash
qm stop <VMID>
qm unlock <VMID>
```

### "trying to get global lock – waiting..."

Stale vzdump processes may be blocking operations. Find and kill them:

```bash
ps aux | grep vzdump
kill -9 <PID>
```

### SSH Connection Resets During Transfer

This is why we use tmux. If your session drops but the process is still running:

On the source host:
```bash
ps aux | grep vzdump
```

On the target host:
```bash
watch ls -lh /mnt/staging/
```

If the file isn't growing and vzdump is dead, the SSH pipe broke. Kill any stale processes, start a new tmux session, and restart the transfer.

**[IMAGE: Terminal showing ps aux output with a running vzdump process]**

## GUI Alternative (If You Prefer Point-and-Click)

While there's no built-in GUI migration tool for non-clustered nodes, you can use this workaround:

1. **On source host GUI:** VM → Backup tab → "Backup Now" → Download the `.vma.zst` file
2. **On target host GUI:** Storage (that accepts backups) → Upload → select your storage as target
3. Right-click the uploaded backup → Restore

This works, but for large VMs it's clunky since you're downloading and re-uploading through your browser. The CLI streaming method is far more efficient.

## Why Not Just Set Up a Cluster?

Good question! Proxmox clustering is great for production environments where you want high availability, live migration, and centralized management. But it has downsides:

- Clustering requires reliable low-latency networking
- All nodes share the same configuration database
- Issues on one node can affect the cluster
- Breaking a cluster can be messy

For home labs or environments where you don't need HA, keeping nodes standalone is simpler. You can still migrate VMs when needed using the method described here. The same vzdump-over-SSH approach works for containers too, which matters if something like the [Ollama LXC with GPU passthrough](https://exitcode0.net/posts/proxmox-ollama-lxc-nvidia-gpu-passthrough/) lives on a host you're retiring. Restore those with `pct restore` instead of `qmrestore`.

## Conclusion

Migrating VMs between standalone Proxmox hosts doesn't require clustering or complex setups. With passwordless SSH, a staging area, and the `vzdump` streaming method, you can efficiently move virtual machines between servers without consuming extra disk space on the source.

The key takeaways:
- Always work inside `tmux` for long transfers
- Stream backups over SSH instead of creating local files first
- Size your staging area based on actual disk usage, not allocation
- Verify network bridge names after restoration

This approach has saved me countless hours compared to downloading and re-uploading backup files through the GUI. Whether you're decommissioning old hardware, load balancing, or just reorganizing your infrastructure, this method gets the job done reliably.

Have you migrated VMs between Proxmox hosts? Any tips or tricks I missed? Leave a comment below!


---
Markdown version of https://exitcode0.net/posts/migrate-vm-between-standalone-proxmox-nodes/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
