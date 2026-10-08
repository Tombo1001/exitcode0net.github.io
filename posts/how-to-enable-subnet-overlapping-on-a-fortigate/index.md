# How to enable subnet overlapping on a Fortigate

> Enable subnet overlapping on a Fortigate so the same IP range can sit on multiple interfaces. Set allow-subnet-overlap in the FortiOS CLI for HA management IPs.

Source: https://exitcode0.net/posts/how-to-enable-subnet-overlapping-on-a-fortigate/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-01-07
Updated: 2026-10-08
Tags: fortigate, networking, subnet-overlapping



A short and sweet problem/resolution. If you are looking to enable subnet overlapping on a Fortigate so that you can give multiple interfaces an IP in the same subnet, this is the post for you.

NOTE: This feature can only be enabled in the Fortigate’s CLI. To enable the overlapping feature, enter the following commands:

```
config system settings
  set allow-subnet-overlap [enable/disable]
end
```

What is subnet overlapping?
---------------------------

Subnet overlapping is disabled by default in fortiOS and for good reason; if you misuse subnet overlapping it can cause massive routing issues for your clients and their traffic. Subnet overlapping lets you apply IPs from the same subnet (e.g 192.160.1.X/24) to multiple interfaces that are not in the same virtual/physical switch.

When trying to set an overlapping IP to an interface without enabling overlapping, the FortiGate will give the following error messages, CLI or GUI respectively:

```
'Subnets overlap between 'port2' and the primary IP of 'port2'
object set operator error, -54 discard the setting'
```

or

```
'IP address is in same subnet as the others.'
```

When might I use subnet overlapping?
------------------------------------

One of my most common reasons for using subnet overlapping on a Fortigate is to give a HA interface a management IP on the same subnet as the shared (floating) management interface for your Fortigate. This makes it easy to access the web interface or SSH to the CLI of a HA slave if you need to do some troubleshooting.

Supported FortiOS versions
--------------------------

All of the following branches support subnet overlapping, but all have it disabled by default:

FortiGate v4.0 MR2  

FortiGate v4.0 MR3  

FortiGate v5.0  

FortiGate v5.2  

FortiGate v5.4  

FortiGate v5.6  

FortiGate v6.0  

FortiGate v6.2

---

Online Resources
----------------

Fortinet KB for Subnet Overlapping – [https://kb.fortinet.com/kb/documentLink.do?externalID=FD30014](https://kb.fortinet.com/kb/documentLink.do?externalID=FD30014 "https://kb.fortinet.com/kb/documentLink.do?externalID=FD30014")

---

Other Fortigate Tips and How-To posts
-------------------------------------

Double NAT port forwarding with a Fortigate – [https://exitcode0.net/posts/double-nat-port-forwarding-with-a-fortigate/](https://exitcode0.net/posts/double-nat-port-forwarding-with-a-fortigate/ "https://exitcode0.net/posts/double-nat-port-forwarding-with-a-fortigate/")



---
Markdown version of https://exitcode0.net/posts/how-to-enable-subnet-overlapping-on-a-fortigate/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
