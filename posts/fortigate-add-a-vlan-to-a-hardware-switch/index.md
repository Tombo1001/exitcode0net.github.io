# Fortigate: Add a VLAN to a hardware switch

> Create VLANs on a Fortigate hardware switch with a DHCP server and inter-VLAN firewall rules, with 802.1Q trunk ports for segmentation in FortiOS 6.0+.

Source: https://exitcode0.net/posts/fortigate-add-a-vlan-to-a-hardware-switch/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-01-09
Updated: 2026-10-08
Tags: fortigate, networking, vlans



In this post, we are going to discuss how to add a VLAN to a hardware (sometimes referred to as physical) switch or interface on a Fortigate. It is worth noting that I actually do my testing on a FortiWifi, so I can assure you that this also applies there too. I will be focusing on the configuration which is relevant to FortiOS v6.0 and above, so your millage vary between versions. However, you need can usually be found over at the Fortinet Handbook: [https://docs.fortinet.com/product/fortigate/6.0](https://docs.fortinet.com/product/fortigate/6.0 "https://docs.fortinet.com/product/fortigate/6.0").

Switch mode Vs Interface Mode
-----------------------------

I’m not going to get too far into the minutia of switch vs interface mode. There are many arguments for both and I believe that the correct answer is very much dependant on your intended implementation. I have my Fortigate 60E configured in interface mode, with most of my LAN ports, 2-7, configured in a hardware switch. This hardware switch is operating in what you may class as the default VLAN. Most of the configuration which we are going to discuss is basic and will apply to a Fortigate in (software) switch mode. I will try my best to point out any obvious configuration differences where they may arise.

VLANs in NAT mode
-----------------

It is important to mention that I have my Fortigate 60E running in NAT mode. In **NAT** mode, the FortiGate supports VLAN trunk links with IEEE 802.1Q‑compliant switches or routers. The trunk link transports VLAN-tagged packets between physical subnets or networks. When you add VLAN sub-interfaces to the FortiGate physical interfaces, the VLANs have IDs that match the VLAN IDs of packets on the trunk link. The FortiGate directs packets with VLAN IDs to sub‑interfaces with matching IDs.

Whereas in **Transparent** mode, the Fortigate behaves like a layer-2 bridge but it can still provide services such as antivirus scanning, web filtering, spam filtering, and intrusion protection to traffic. There are some limitations in transparent mode because you can’t use SSL VPN, PPTP/L2TP VPN, DHCP server, or easily perform NAT on traffic. The limits in transparent mode apply to IEEE 802.1Q VLAN trunks passing through the device.

Fortigate VLAN Config Example
-----------------------------

### Physical Switch Config

I will start by splitting an interface out of the default hardware switch. I am chosing to do this because I want to preseve the rest of the LAN ports for use in my ‘regular’ LAN. Interface LAN1 will be used for VLANs and connected to a managed switch whih supports VLANs.

*Two hardware switches – one with VLANs*
The reality of having ‘two hardware switches’ is that the Fortigate created two virtual-switches, nested under one physical-switch. This is easily demonstrated in the subsequent config:

```
config system physical-switch
    edit "sw0"
        set age-val 0
    next
end
config system virtual-switch
    edit "internal"
        set physical-switch "sw0"
        config port
            edit "internal2"
            next
            edit "internal3"
            next
            edit "internal4"
            next
            edit "internal5"
            next
            edit "internal6"
            next
            edit "internal7"
            next
        end
    next
    edit "VTRUNK"
        set physical-switch "sw0"
        config port
            edit "internal1"
            next
        end
    next
end
```

### VLAN Config

Now that we have split out our hardware switches, we can start adding VLANs. Enabling a DHCP server on a VLAN is optional, but I have no alternatives in my home network and most of the devices I plan to isolate in my VLANs can be troublesome to set static addresses on.

*Basic VLAN config on a hardware switch*
It is worth noting that, I did not enable a DHCP server on my LAN1 interface, so if my device is connected and does not have a valid VLAN tag set, it will be assigned an IP address. I named internal1 one to reflect that it will be used as a trunk port for the traffic of multiple VLANs to be carried over to a managed switch. Here is the CLI config for each of my VLAN interfaces:

```
config system interface
    edit "internal"
        set vdom "root"
        set ip ---.---.---.--- ---.---.---.---
        set allowaccess ping https ssh
        set type hard-switch
        set alias "LAN"
        set stp enable
        set snmp-index 6
    next
    edit "VTRUNK"
        set vdom "root"
        set ip 172.16.30.1 255.255.255.0
        set allowaccess ping
        set type hard-switch
        set description "VLAN TRUNK"
        set alias "VLAN TRUNK"
        set device-identification enable
        set lldp-transmission enable
        set role lan
        set snmp-index 11
    next
    edit "V31-IoT"
        set vdom "root"
        set ip 172.16.31.1 255.255.255.0
        set allowaccess ping
        set description "VLAN IoT"
        set alias "VLAN IoT"
        set device-identification enable
        set role lan
        set snmp-index 10
        set interface "VTRUNK"
        set vlanid 31
    next
    edit "V32-Hue"
        set vdom "root"
        set ip 172.16.32.1 255.255.255.0
        set allowaccess ping
        set description "VLAN32 Hue"
        set alias "VLAN Hue"
        set device-identification enable
        set role lan
        set snmp-index 12
        set interface "VTRUNK"
        set vlanid 32
    next
end
```



---
Markdown version of https://exitcode0.net/posts/fortigate-add-a-vlan-to-a-hardware-switch/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
