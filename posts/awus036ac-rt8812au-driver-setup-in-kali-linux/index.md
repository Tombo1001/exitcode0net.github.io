# AWUS036AC (rtl8812au) driver setup in Kali Linux

> Install Alfa AWUS036AC (rtl8812au) drivers on Kali Linux for monitor mode and packet injection with aircrack-ng, using a manual dkms build.

Source: https://exitcode0.net/posts/awus036ac-rt8812au-driver-setup-in-kali-linux/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-12-03
Updated: 2026-10-08
Tags: alfa, awus036ac, drivers, kali


> **Note:** This guide was written for Kali Linux 2019.4. The core dkms build method still works on current Kali, but package names and kernel headers may differ slightly on newer releases.

> **Authorised use only:** monitor mode and packet injection are standard tools for testing wireless networks you own or have written permission to assess. Using them against anyone else's network is illegal in the UK under the Computer Misuse Act 1990. This post covers getting the adapter's driver working and nothing else.

Wireless security testing with aircrack-ng needs a wireless adapter that supports monitor mode and packet injection. I decided on the Alfa AWUS036AC, but some work was required to get the drivers installed.

This guide is based on Kali Linux 2019.4 – but the drivers are certified for earlier versions of Kali and the kernel that 2019.4 uses. See updates below for getting this working on Kali 2020.4.

Hardware – AWUS036AC
--------------------

My choice of adapter for wireless testing was the Alfa Networks AWUS036AC – [https://amzn.to/34LlqXY](https://amzn.to/34LlqXY "https://amzn.to/34LlqXY")

Full manufacturer specifications – [https://www.alfa.com.tw/products\_detail/3.htm](https://www.alfa.com.tw/products_detail/3.htm "https://www.alfa.com.tw/products_detail/3.htm")

 **Increased Wireless Signal Penetration**   
 *With unmatched Wi-Fi signal strength and coverage. AWUS036AC not only has maximum WiFi range, it helps to penetrate walls, and eliminate Wi-Fi dead spots in your living space easily.*   
If you are conductin a wireless assessment, this is great news because you can conduct your testing from the comfort of your desk chair.

|  |  |
| --- | --- |
| Chipset
  | 
 Realtek RTL8812AU
  |
| 
 WiFi Standards
  | 
 IEEE 802.11ac/a/b/g/n
  |
| 
 WiFi Frequency
  | 
 Dual Band 2.4GHz or 5GHz
  |
| 
 Antenna Connector
  | 
 RP-SMA female x 2
  |
| 
 Antenna Type
  | 
 2.4G/5GHz Dual-Band 5dBi dipole antenna
  |
| 
 Wireless Performance
  | 
 802.11a: up to 54Mbps
 802.11b: up to 11Mbps
 802.11g: up to 54Mbps
 802.11n: up to 300Mbps  
 802.11ac: up to 867Mbps
  |
| 
 Wireless Security
  | 
 64/128 bit WEP,WPA/WPA2,WPA-PSK/WPA2-PSK,WPS
  |
| 
 Interface
  | 
 USB 3.0
  |
| 
 OS Requirement
  | 
 Windows XP, Vista, 7, 8/8.1 and Windows 10 32/64bit, 
 macOS 10.5 to 10.14 or later
 Linux |

AWUS036AC (rtl8812au) Driver Installation
-----------------------------------------

Assuming that you are loged into a terminal session on your kali linux machine as root, the following commands are required to download and install the drivers from source:

Clone the aircrack-ng git repository:

```
git clone https://github.com/aircrack-ng/rtl8812au
```

Enter the newly downloaded directory:

```
cd rtl8812au
```

Build and install the source files:

```
make
```

```
make install
```

Reboot the Kali instance to complete:

```
reboot now
```

There are a number of other git repositories where you can obtain drivers for this usb wireless adaptor. My findings were that the aircrack-ng repo was the only place which supplied drivers supported in the current kernel version for Kali 2019.4 – kernel 5.3.9

Verify your device
------------------

To ensure that your device is available and ready to be used in Kali, you can run the following command to confirm that the OS can recognise the adapter:

```
iwconfig
```

Update: 11/01/2021
------------------

As you can see from the Github issue reports, [https://github.com/aircrack-ng/rtl8812au/issues](https://github.com/aircrack-ng/rtl8812au/issues "https://github.com/aircrack-ng/rtl8812au/issues"), there are a lot of active issues with these drivers.

As of 11/01/2021, I was able to get this chipset working on Kali with the following driver install method:

```
apt-get update
apt install realtek-rtl88xxau-dkms
```

And just to verify, here is my curent Kali version:

```
root@kali:~# lsb_release -a
No LSB modules are available.                                                                                                                         
Distributor ID: Kali                                                                                                                                  
Description:    Kali GNU/Linux Rolling
Release:        2020.4
Codename:       kali-rolling

```

…and airodump-ng was my check that the adapter worked in monitor mode.

It is worth noting that `iwconfig` shows the interface `wlan0` in monitor mode; a `mon0` interface is not created like most online tutorials demonstrate: [https://www.computerweekly.com/tip/Step-by-step-aircrack-tutorial-for-Wi-Fi-penetration-testing](https://www.computerweekly.com/tip/Step-by-step-aircrack-tutorial-for-Wi-Fi-penetration-testing "https://www.computerweekly.com/tip/Step-by-step-aircrack-tutorial-for-Wi-Fi-penetration-testing").

---

### Other useful articles:

* [https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ "https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/")
* [https://exitcode0.net/clipboard-and-shared-folders-on-kali-linux-with-virtualbox/](https://exitcode0.net/posts/clipboard-and-shared-folders-on-kali-linux-with-virtualbox/)



---
Markdown version of https://exitcode0.net/posts/awus036ac-rt8812au-driver-setup-in-kali-linux/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
