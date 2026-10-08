# Backup Nextcloud with RCLONE

> Back up a self-hosted Nextcloud instance to UNRAID NAS using Rclone WebDAV integration. Automated encrypted cloud-to-local sync with selective folder targeting.

Source: https://exitcode0.net/posts/backup-nextcloud-with-rclone/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-02-27
Updated: 2026-10-08
Tags: backups, nextcloud, rclone, unraid



I’ve made previous posts about backing cloud services with RCLONE – [Backup Google Photos with Rclone](https://exitcode0.net/posts/backup-google-photos-with-rclone/ "https://exitcode0.net/posts/backup-google-photos-with-rclone/") – and this post will detail how to backup a self-hosted cloud service, Nextcloud. 

[](https://nextcloud.com/install/# "https://nextcloud.com/install/#")[Get started with Nextcloud](https://nextcloud.com/install/# "https://nextcloud.com/install/#")
There are a lot of benefits to running your own private ‘cloud’ storage system, but for me, my top concern is privacy and Nextcloud gives me the ability to take accountability for my own data. With this in mind, I am conscious of my Nextcloud server going offline or suffering data loss, so I want a way to create 1 or more backups of my datacentre hosted, Nextcloud data on my home NAS. 

My home NAS is running Linux, UNRAID to be precise. It would be possible to run the Nextcloud client on a windows or linux desktop to sync all of the files to a machine. However, RCLONE allows you to copy the data from Nextcloud over a webdav connection.

How to configure RCLONE Nextcloud backups
-----------------------------------------

Let’s start by running the RCLONE configuration tool:

```
rclone config
```

Then use the following options:

```
n) New remote
--
name: MyNextcloudBackup
--
type: webdav
--
url: https://<your nextcloud server url>/remote.php/webdav/
--
vendor: nextcloud
--
user: <your nextcloud username>
--
pass: <your nextcloud user password>
--
bearer_token: Remote config
```

This will encrypt the password so that it is not stored in plaintext.  
Now we can run the copy to the UNRAID server:

```
rclone copy -P MyNextcloudBackup:<folder name> /mnt/user/<share and path on your UNRAID server>
```

If you leave <folder> blank then it will download all folder in your nextcloud account.  
-P Will show you the progress of the copy job.

You can take this one step further and run this on a schedule using the user scripts plugin: [https://forums.unraid.net/topic/48286-plugin-ca-user-scripts/](https://forums.unraid.net/topic/48286-plugin-ca-user-scripts/ "https://forums.unraid.net/topic/48286-plugin-ca-user-scripts/")

---

Other useful posts:
-------------------

* Backup Google Photos with Rclone – [https://exitcode0.net/posts/backup-google-photos-with-rclone/](https://exitcode0.net/posts/backup-google-photos-with-rclone/ "https://exitcode0.net/posts/backup-google-photos-with-rclone/")
* Kali Linux – How to upgrade python 2.7 to python 3.7 – [https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ "https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/")
* Switching to a Linux laptop – https://exitcode0.net/switching-to-a-linux-laptop/

I’m working hard to bring as much useful content to this blog as I can. You can support this site by sharing pages or posts and I will be forever indebted to those who do!



---
Markdown version of https://exitcode0.net/posts/backup-nextcloud-with-rclone/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
