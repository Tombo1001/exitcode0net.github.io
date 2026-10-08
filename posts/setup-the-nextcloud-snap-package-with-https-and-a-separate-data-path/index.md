# Setup the Nextcloud Snap Package with HTTPS and a separate data path

> Install the Nextcloud snap on Ubuntu with Let's Encrypt HTTPS and a separate data mount on an external drive, with automatic certificate renewal.

Source: https://exitcode0.net/posts/setup-the-nextcloud-snap-package-with-https-and-a-separate-data-path/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-03-29
Updated: 2026-10-08
Tags: linux, nextcloud, snap, snapcraft, ubuntu



[](https://snapcraft.io/nextcloud "https://snapcraft.io/nextcloud") [https://snapcraft.io/nextcloud](https://snapcraft.io/nextcloud "https://snapcraft.io/nextcloud")
I have been using the Nextcloud snap package for a long time, but some recent system changes in my colocation facility mean that it is time for a big upgrade. Rather than migrate my existing Nextcloud instance, I have decided to build a new install and here are my goals for the new installation:

* **Use lets encrypt to have automatic and valid HTTPS certificates**
* **Have the data path for Nextcloud on a HDD and the rest of the host OS on an SSD**.

Host OS Details
---------------

For this setup, I am using Ubuntu 18.04 installed on a 30GB SSD backed partition. I then have a 1TB disk attached and mounted to /media.

Step 1 – How to install Nextcloud Snap Package
----------------------------------------------

Snap is simple and effective, this one command gives you a basic install of Nextcloud (assuming that you have your DNS records and firewall rules in place at this point):

```
sudo snap install nextcloud
```

**Do not visit the new site in your browser yet!**

Step 2 – Change the data directory to use another disk partition
----------------------------------------------------------------

You **must** make sure that your secondary drive is mounted in **/media** or **/mnt** – as described in the Nextcloud snap wiki: [https://github.com/nextcloud/nextcloud-snap/blob/master/README.md](https://github.com/nextcloud/nextcloud-snap/blob/master/README.md "https://github.com/nextcloud/nextcloud-snap/blob/master/README.md") 

When you have you drive mounted, you will need to allow the snap instance to access ‘removable media’:

```
sudo snap connect nextcloud:removable-media
```

```
sudo chown -R root:root /media/<my mount point>
```

Now we need to edit the Nextcloud config to match this data directory:

```
sudo nano /var/snap/nextcloud/current/nextcloud/config/autoconfig.php
```

Here we must change the following line to reflect our data path:

```
 // ...
 'directory' => '/media/<my mount point>',
 // ...
```

Once you have exited and saves your changes to the config file, it is time to restart the service:

```
sudo snap restart nextcloud.php-fpm
```

**Now it is safe to visit your Nextcloud URL**!

At this point, you will be visiting a HTTP page and will be asked to create your default admin Nextcloud user. Feel free to do this.

Step 3 – Enabling Lets Encrypt HTTPS for the Nextcloud Snap Package
-------------------------------------------------------------------

We now have a function Nextcloud server, but it is not secure. We need to enable HTTPS and generate a certificate with Lets Encrypt. Again, assuming that your DNS records and firewall rules are in order:

```
sudo nextcloud.enable-https lets-encrypt
```

Complete the steps by entering your recovery email address and the domain of the Nextcloud server and the tool will generate and apply your certificate. You only need to this once; Lets Encrypt handles auto-renewal.

**At this point, you can refresh your browser and you will be auto-forward to the HTTPS version of your new Nextcloud sever.**

---

Other useful Nextcloud tutorials:
---------------------------------

* Backup Nextcloud with RCLONE – [https://exitcode0.net/posts/backup-nextcloud-with-rclone/](https://exitcode0.net/posts/backup-nextcloud-with-rclone/ "https://exitcode0.net/posts/backup-nextcloud-with-rclone/")



---
Markdown version of https://exitcode0.net/posts/setup-the-nextcloud-snap-package-with-https-and-a-separate-data-path/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
