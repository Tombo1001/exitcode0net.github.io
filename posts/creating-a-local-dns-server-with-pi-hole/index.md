# Creating a local DNS server with Pi Hole

> Set up Pi-Hole as a local DNS resolver on Debian for network-wide ad blocking. Configure DHCP integration and custom DNS overrides for a home lab environment.

Source: https://exitcode0.net/posts/creating-a-local-dns-server-with-pi-hole/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-02-16
Updated: 2026-10-08
Tags: adblocker, configuration, dns, networking, pi-hole



This is a local DNS server for local DNS requests.

This post is going to explain the why and how I created a local DNS server in my home network environment. I used the PiHole project to make a network wide advert blocking a reality.

[https://pi-hole.net/](https://pi-hole.net/ "https://pi-hole.net/")

I have previously created a DNS server using Bind, running on CentOS 7. Thankfully digital ocean came to my rescue with some of the config in this home lab project, so be sure to check out their guide if you are looking for slightly higher level DNS setup:  
[https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-centos-7](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-centos-7 "https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-centos-7")

**PiHole Setup**  
**Step 1 – Server Installation**  
Every great **Linux** project starts with a tedious operating system install. I started this project by setting up a Debian 9 VM on one of my local hypervisors. However as the name suggest, PiHole is very comfortable running on a Raspberry Pi single single board computer.  
My VM specs: 1 vCPU, 512MB RAM, 15GB Storage

  
**Step 1 – Server Installation**  
Every great **Linux** project starts with a tedious operating system install. I started this project by setting up a Debian 9 VM on one of my local hypervisors. However as the name suggest, PiHole is very comfortable running on a Raspberry Pi single single board computer.  
My VM specs: 1 vCPU, 512MB RAM, 15GB Storage

**Step 2 – Update all the things**  
Be sure to update and upgrade the package repositories in your OS once the install is complete

> sudo apt-get update
> 
> sudo apt-get upgrade
> 
> 

You might also want to take this opportunity to install the nano text editor as well as a couple of other handy utilities

**Step 3 – Set a static IP**  
Now that our CentOS box is configured how you like it, its is time to make sure that it has a static IP. If you are going to be distributing this DNS server to your network clients, then it going to be important that this server does not get different DHCP lease addresses

> sudo nano /etc/network/interfaces
> 
> 

Change the file to reflect the following config, changing the network setting to match your environment:  

> auto lo 
> 
> iface lo inet loopback 
> 
>  iface eth0 inet static 
> 
> address 192.168.0.100
> 
> netmask 255.255.255.0
> 
> network 192.168.0.0
> 
> broadcast 192.168.0.255
> 
> gateway 192.168.0.1
> 
> 

**Step 4 – Install PiHole**  
This final step pretty much rounds up the PiHole install process. The following command runs a guided installer which lets you chose your settings but offer a very functional system when the defaults are selected

> `curl -sSL https://install.pi-hole.net |bash`
> 
> 



---
Markdown version of https://exitcode0.net/posts/creating-a-local-dns-server-with-pi-hole/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
