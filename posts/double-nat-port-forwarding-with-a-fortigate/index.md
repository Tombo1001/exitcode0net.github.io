# Double NAT port forwarding with a Fortigate

> Set up Fortigate VIP port forwarding through a double NAT, routing external traffic via a DMZ IP through nested gateways to reach services like Plex.

Source: https://exitcode0.net/posts/double-nat-port-forwarding-with-a-fortigate/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-02-23
Updated: 2026-10-08
Tags: double-nat, fortigate, networking, portforward, vip



If you are unfortunate enough to have to deal with double NAT on your gateway then you might know the troubles surrounding portforwarding or VIPs. Here is a quick how to guide for setting up a port forward on a Forgate where double NAT is inplace.

**Case Study – Plex port forward**  
  
Plex is a great tool for managing your personal media collection and it gets even better when you enable a port forward to let you access this collection from anywhere in the world. Whilst Plex ahve made a number of changes to allow you to reach your contect via a relay server, the best way to access your content from outside your LAN is by using a port forward.

Double NAT means that there is a device runing NAT service in front of your NAT enabled default gateway – this can make portforwards difficult.  

I started by setting the ‘WAN IP’ of my Fortigate to a DMZ IP on the border NAT device – this will prevent any port foltering or firewall restrictions on traffic destined for the Fortigate.

Next when creating your VIP, use the following config:

NOTE: I am currently using WAN2 as the primary WAN conenction on my Fortigate.  



---
Markdown version of https://exitcode0.net/posts/double-nat-port-forwarding-with-a-fortigate/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
