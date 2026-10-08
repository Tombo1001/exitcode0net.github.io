# [FIXED] Bose Bluetooth headphones with Windows 10

> Fix poor Bose Bluetooth audio and the microphone feedback loop on Windows 10 by switching from the generic driver to Microsoft A2DP Source.

Source: https://exitcode0.net/posts/fixed-bose-bluetooth-headphones-with-windows-10/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-08-30
Updated: 2026-10-08
Tags: bose, drivers, windows-10



![SoundLink® around-ear wireless headphones II - winows 10 drivers](https://images-na.ssl-images-amazon.com/images/I/71jDdUuRi8L._AC_SL1500_.jpg)
I recently moved to a new Windows 10 laptop and experience a number of problems with my [Bose SoundLink II](https://www.bose.co.uk/en_gb/products/headphones/over_ear_headphones/soundlink-around-ear-wireless-headphones-ii.html#v=soundlink_ae_headphones_ii_black "https://www.bose.co.uk/en_gb/products/headphones/over_ear_headphones/soundlink-around-ear-wireless-headphones-ii.html#v=soundlink_ae_headphones_ii_black"), Bluetooth headphones. Fortunately, after some playing around with drivers I was able to solve my problem and this post explains how…

The Problem
-----------

**Poor sound quality** – with the default drivers, the sound quality was poor and there was a considerable amount of audible noise being delivered to the headphones via the Bluetooth interface. The audio was very lo-fi and the frequency range was dramatically reduced. This resulted in a very thin, tinny sound.

**Mic feedback loop** – the Bose Soundlink headphones have a built-in mic, meaning that you can use it as a mic enabled headset for calls. With the default drivers which Windows was allocating to the headphones, I could hear a constant mic feedback loop whilst there was active audio output from the system. It was possible to hear yourself tapping on your keyboard through your headphones.

The Solution
------------

1. Connect your headphones to the laptop via Bluetooth.
2. Open device manager (devmgmt.msc) as an administrator.
3. Open the properties of the Bluetooth Audio device, go to the Driver tab:

*My headphones are called ‘KeepYourVinyl’*
4. Update Driver and Browse my computer for driver software. Pick from a list of available drivers on my computer.  
5. Find and select ‘Microsoft Bluetooth A2dp Source’ and proceed with the installation.

*Driver selection*
6. Your audio device will likely stop working at this point and you will receive a dialog prompting you to reboot to complete the driver setup.

*Full driver details*
**After a reboot, my headphones connected o the laptop and audio was as you would expect! Audio quality is now on par with that when connecting to an iPhone or Android device.**



---
Markdown version of https://exitcode0.net/posts/fixed-bose-bluetooth-headphones-with-windows-10/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
