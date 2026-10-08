# Using a DHT11 sensor with a raspberry pi

> Connect a DHT11 temperature and humidity sensor to a Raspberry Pi over GPIO and read it in Python, no soldering needed with a pre-made 3-pin sensor.

Source: https://exitcode0.net/posts/using-a-dht11-sensor-with-a-raspberry-pi/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-02-24
Updated: 2026-10-08
Tags: dht11, electronics, gpio, raspberry-pi, temperature-sensor



This post is going to share how to use DHT11 sensors with a the raspberry pi , using the GPIO pins. Furthermore the particular sensor that I am going to be discussing requires zero soldering and is purely plug and play.

**The Sensor:**  
The sensor that we will be using is the DHT11 – the exact variant is the 3 pin version. You can find the exact version that I used here

![](https://images-na.ssl-images-amazon.com/images/I/61rvpJDSylL._SL1010_.jpg)
[https://amzn.to/2XkclSq](https://amzn.to/2XkclSq "https://amzn.to/2XkclSq")

This pack of 5 sensors also comes with the cables you will need to hook up the DHT11 to the GPIO pinout of the raspberry pi.

**The PIN Layout**  
The sensor pin are a positive power pin, a neutral pin and a data pin. Depending on the version of your raspberry pi, the GPIO layout can very. The following diagram shows the GPIO layout for the raspberry pi 3 – the board I used:

The 40-pin header layout is on [pinout.xyz](https://pinout.xyz/), which is the reference I keep open when wiring anything to the Pi.
The pins to connect the DHT11 to in this example would be 2, 6, and 7. That being said, it would be possible to use any of the other available data pins (green), if this was the only device attached via the GPIO pins.



---
Markdown version of https://exitcode0.net/posts/using-a-dht11-sensor-with-a-raspberry-pi/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
