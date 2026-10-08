# How To: Raspberry Pi boot from USB

> Boot a Raspberry Pi 4 from an external SSD over USB by updating the EEPROM bootloader. Faster and more reliable than an SD card.

Source: https://exitcode0.net/posts/how-to-raspberry-pi-boot-from-usb/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-05-23
Updated: 2026-10-08
Tags: linux, pi4, raspberry-pi



The Pi enthusiasts have been waiting for official USB boot support on the Raspberry Pi for what feels like a lifetime, but finally it is on the horizon. In this post I will explain how to make your Raspberry Pi boot from USB.

**WARNING**: Although this is official, it is still in beta testing, so rock-solid stability is far from certain. Learn more about this beta release [here](https://www.raspberrypi.org/forums/viewtopic.php?t=274595&p=1663644#p1663644 "https://www.raspberrypi.org/forums/viewtopic.php?t=274595&p=1663644#p1663644").

Why should I make my Raspberry Pi boot from USB?
------------------------------------------------

Firstly let me address why you would want to do this and what makes many people relieved that the feature is on the horizon.

**A more robust, long-lasting boot device.** Micro SD cards, particularly [high-quality cards](https://amzn.to/2TvRhs4 "https://amzn.to/2TvRhs4"), have come a long way and the days of a dead micro SD card seem to be dwindling. However, the small flash cards have a much shorter read/write life expectancy, and are not really designed to be used as a bootable OS media. Being able to use a SATA (over USB) device which was designed for the typical IO expected from an active operating system will ensure a long-lasting Pi setup.

[](https://uk.camelcamelcamel.com/product/B073JWXGNT "https://uk.camelcamelcamel.com/product/B073JWXGNT")32GB Sandisk Card Price over time – **Camel Camel Camel**
**SATA based storage is cheaper per GB.** Prices of micro SD cards have dropped significantly over the years, but SATA based storage is till better value for money at larger capacities, as well as the reliability benefits. 

How do I make my Raspberry Pi boot from USB?
--------------------------------------------

#### Difficulty:

**Medium**

There are a number of steps involved in this process and it will require you to bounce some files between devices that can be done on the Pi or by connecting the two bootable devices to another computer – this method won’t be covered in this guide.

#### Requirements:

* **Raspberry Pi**
	+ [https://amzn.to/3eiF2XT](https://amzn.to/3eiF2XT "https://amzn.to/3eiF2XT") Pi 4 Starter kit
* **The latest version of Raspbian Buster**
	+ [https://www.raspberrypi.org/downloads/raspbian/](https://www.raspberrypi.org/downloads/raspbian/ "https://www.raspberrypi.org/downloads/raspbian/")
* **USB hard drive/flash drive**
	+ [https://amzn.to/2Twk4wm](https://amzn.to/2Twk4wm "https://amzn.to/2Twk4wm") 256GB Sandisk USB drive.
* **Micro SD Card** (still required at this point in BETA)
	+ [https://amzn.to/3ggknoX](https://amzn.to/3ggknoX "https://amzn.to/3ggknoX") 32GB SanDisk Micro SD card

Burn the Rasbian ISO to your SD card using something like Etcher: [https://www.balena.io/etcher/](https://www.balena.io/etcher/ "https://www.balena.io/etcher/") and plug it into the Pi.

#### Process (as of 23-05-2020):

The breakdown of the process:  
1. Change the content of the EEPROM on the Raspberry Pi4 and set the boot device to USB.  
2. Copy files from the boot directory of the SD card to the boot directory of the USB boot device.

We must start by updating the Raspbian installation:

```
sudo apt update -y && sudo apt upgrade -y && sudo rpi-update -y
```

Now that we are up to date, we must reboot before installing rpi-eeprom:

```
sudo reboot now
```

```
sudo apt install rpi-eeprom
```

Now we have to change the path of the Pi’s EEPROM firmware- this is done by moving from the critical channel to beta channel: 

```
sudo nano /etc/default/rpi-eeprom-update
```

Replace ‘**critical**‘ with ‘**beta**‘ followed by **‘ctrl+x**, **y**‘ to save and exit.

Now we can program the EEPROM as follows:

```
sudo rpi-eeprom-update -d -f /lib/firmware/raspberrypi/bootloader/beta/pieeprom-2020-05-15.bin
```

Followed by a reboot:

```
sudo reboot now
```

Once the reboot is complete, we can check the bootloader version as follows:

```
vcgencmd bootloader_version
```

If we run the follow…

```
vcgencmd bootloader_config
```

… we should see ‘BOOTORDER=0xF41. **4**‘  
‘.4’ means that USB boot is active; ‘.1’ means that SD boot is active.

**At this point the Pi is ready, but the USB media is not.** All that is left is to prepare the USB bootable media with the necessary files form our SD card installation.

---

Start by plugging in the USB getting rasbian flashed to your USB device in the same manner as the SD card – personally I use Etchor: [https://www.balena.io/etcher/](https://www.balena.io/etcher/ "https://www.balena.io/etcher/")

Now we need to copy all ‘.elf’ and ‘.dat’ files from the boot directory of our SD card to the boot directory on the USB. Let’s start by mounting the USB drive.

```
sudo mkdir /mnt/usbdisk
sudo mount /dev/sd1 /mnt/usbdisk
```

Now that the USB drive is mounted, we can copy the required files:

```

sudo cp /boot/*.elf /mnt/usbdisk
sudo cp /boot/*.dat /mnt/usbdisk
```

**We are done!**

Now we can now power down (sudo shutdown now), remove the SD card, then power back up and check that we have successfully booted from out USB!

---

More Raspberry Pi Antics:
-------------------------

* [SOLVED] – RUNNING DOCKER ON A RASPBERRY PI 4 – [https://exitcode0.net/posts/solved-running-docker-on-a-raspberry-pi-4/](https://exitcode0.net/posts/solved-running-docker-on-a-raspberry-pi-4/ "https://exitcode0.net/posts/solved-running-docker-on-a-raspberry-pi-4/")
* USING A DHT11 SENSOR WITH A RASPBERRY PI – [https://exitcode0.net/using-a-dht11-sensor-with-a-raspberry-pi/](https://exitcode0.net/posts/using-a-dht11-sensor-with-a-raspberry-pi/)
* CREATING A LOCAL DNS SERVER WITH PI HOLE – [https://exitcode0.net/creating-a-local-dns-server-with-pi-hole/](https://exitcode0.net/posts/creating-a-local-dns-server-with-pi-hole/)



---
Markdown version of https://exitcode0.net/posts/how-to-raspberry-pi-boot-from-usb/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
