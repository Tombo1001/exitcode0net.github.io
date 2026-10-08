# Fortigate: AP-Bridge with a hardware switch

> Configure Fortigate access point bridge SSIDs mapped to isolated VLANs, tagging wireless clients into security zones with CAPWAP management.

Source: https://exitcode0.net/posts/fortigate-ap-bridge-with-a-hardware-switch/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-01-17
Updated: 2026-10-08
Tags: ap-bridge, fortigate, vlans



Following on from a previous post on [how to setup a VLAN on a Fortigate hardware switch](https://exitcode0.net/posts/fortigate-add-a-vlan-to-a-hardware-switch/ "https://exitcode0.net/posts/fortigate-add-a-vlan-to-a-hardware-switch/"), this post is going to explain how we can link an AP-bridge SSID to a hardware switch and VLAN.

For the most part, the only reference material you will need to complete this configuration can be found here: [https://docs.fortinet.com/document/fortiap/6.4.0/fortiwifi-and-fortiap-cookbook/252439/configuring-the-fortigate-interface-to-manage-fortiap-units](https://docs.fortinet.com/document/fortiap/6.4.0/fortiwifi-and-fortiap-cookbook/252439/configuring-the-fortigate-interface-to-manage-fortiap-units "https://docs.fortinet.com/document/fortiap/6.4.0/fortiwifi-and-fortiap-cookbook/252439/configuring-the-fortigate-interface-to-manage-fortiap-units"). However, if you have been working with Fortigates and by extension FortiOS for quite some time, you may be wonder where the CAPWAP option vanished to? Fortinet adopted this into the security fabric naming convention. CAPWAP has even been replaced in the CLI by ‘fabric’.

AP-Bridge with a VLAN
---------------------

The confif that I a looking to build will give me an SSID for a given VLAN; a client which is connected to this SSID wil be given a DHCP address and will be subject to firewall policy for that VLAN. The client will not be required to set their own VLAN tag – traffic will be tagged by the SSID interface.

*V33 is an AP-brige to VLAN 33 on the hardware switch.*
We are also going to have an SSID on the ‘default VLAN’ aka a typical wireless LAN – sharing the same subnet and multicast zone as the ports in our hardware switch.

Ultimately we will have an SSID which is isolated from all other ‘LAN’ traffic, perfect for wireless CCTVs cameras or IoT device which we don’t trust.

The Configuration
-----------------

### Hardware Switch and VLAN

Much of this configuration was covered in a previous post, see here: [https://exitcode0.net/fortigate-add-a-vlan-to-a-hardware-switch](https://exitcode0.net/posts/fortigate-add-a-vlan-to-a-hardware-switch/ "https://exitcode0.net/posts/fortigate-add-a-vlan-to-a-hardware-switch/").

At this point we will have a hardware switch and a VLAN assigned to it – VLAN 33.

One important change is to enabled security fabric management (previously CAPWAP) on our hardware switch:

*Security Fabric Connection is enabled – previous known as CAPWAP.*
If you choose to operate in the CLI, here is how you would implement this:

```
config system interface
edit LAN      #(hardware switch name)
set allow-access fabric
set ap-discover enable
next
end
```

### SSID Configuration

Setting traffic mode to AP-bridge and Optional VLAN ID to 33, we create an SSID with a suitabe BSSID and security settings as follows:

*an SSID for VLAN 33*
In AP-bridge mode, all traffic on SSID V33 will be tunneled to the Fortigate’s security fabric controller and to the VLAN interface there onwards.

**And that is it! It really is that simple.** It is worth mentioning, that this is also possible with a software switch, but not without significant throughput penalties – all switching passes through the the Fortigate’s CPU in a software switch.



---
Markdown version of https://exitcode0.net/posts/fortigate-ap-bridge-with-a-hardware-switch/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
