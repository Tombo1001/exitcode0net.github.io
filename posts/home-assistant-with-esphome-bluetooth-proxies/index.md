# Home Assistant With ESPhome Bluetooth Proxies

> Extend Home Assistant's Bluetooth range with ESPHome proxies on an ESP32. Connect BLE sensors like SwitchBot thermometers over WiFi without a USB adapter.

Source: https://exitcode0.net/posts/home-assistant-with-esphome-bluetooth-proxies/
Author: Tom Cocking (https://tomcocking.com)
Published: 2025-01-08
Updated: 2026-10-08
Tags: home-automation, home-assistant, esphome, switchbot


In this blog post, I’ll walk you through how I set up a SwitchBot BLE (Bluetooth Low Energy) Indoor/Outdoor Thermo-Hygrometer in Home Assistant (HA) using an ESP32 running ESPHome as a Bluetooth proxy. If you’re working with Home Assistant in a containerized environment (as I am), you might face challenges with Bluetooth connectivity, in particular, passing through a bluetooth adapter. This approach solves that elegantly and opens the door to other Bluetooth-based projects by providing extended bluetooth connectivity over a WiFi backbone. 

The switchbot sensor is great for a number of reasons, check it out on the [Smart Home Index](https://smarthomeindex.com/?deviceId=cac98b86-c500-4aea-bbe6-81ffad793cb6).

---

## Why I Needed a Bluetooth Proxy

My Home Assistant instance is running in a Docker container on a server that doesn’t have a Bluetooth device installed or configured. While USB Bluetooth dongles and passthrough are options, they often come with limited range and reliability issues - there are number of well tested devices and chipsets listed here: [known working high-performance adapters](https://www.home-assistant.io/integrations/bluetooth/#known-working-high-performance-adapters). Using an ESP32 with ESPHome as a Bluetooth proxy has been a game-changer. Not only does it extend the Bluetooth range across my house and garden, but it also integrates seamlessly with Home Assistant.

---

## Setting Up ESPHome on the ESP32

#### Flashing ESPHome to the ESP32
To get started, you’ll need an ESP32. I’ve used a simple ESP32 development board and a 3D-printed case to house it neatly. There are two main ways that you can flash your ESP32 with ESPhome firmware, through a self hosted instance of the ESPhome server, or using the [ready-made projects page](https://esphome.io/projects/index.html).

If you are going to follow the path of flashing from your own local ESPhome instance, here is a suitable config to do that (adjust your esp32 board value to suit):

```yaml
esphome:
  name: esp32-bluetooth-proxy
  friendly_name: Bluetooth Proxy
  min_version: 2024.6.0
  name_add_mac_suffix: true
  platformio_options:
    board_build.f_flash: 40000000L
    board_build.flash_mode: dio
    board_build.flash_size: 4MB

esp32:
  board: esp32-c3-devkitm-1
  framework:
    type: esp-idf
    sdkconfig_options:
      CONFIG_BT_BLE_42_FEATURES_SUPPORTED: y
      CONFIG_BT_BLE_50_FEATURES_SUPPORTED: n

wifi: # values are stored in the esphome secrets vault or you can manually entire the values in double quotes.
  ssid: "Your_WiFi_SSID"
  password: "Your_WiFi_Password"

api:
logger:

ota:
  - platform: esphome
    id: ota_esphome

esp32_ble_tracker:
  scan_parameters:
    # We currently use the defaults to ensure Bluetooth
    # can co-exist with WiFi In the future we may be able to
    # enable the built-in coexistence logic in ESP-IDF
    active: true

bluetooth_proxy:
  active: true

button:
  - platform: safe_mode
    id: button_safe_mode
    name: Safe Mode Boot

  - platform: factory_reset
    id: factory_reset_btn
    name: Factory reset
```

This `bluetooth_proxy:` line enables the ESP32 to act as a bridge between Bluetooth devices and Home Assistant. The `api:` line enabled seamless communication with Home Assistant - you can choose to encrypt this API.

There are more bluetooth proxy yaml examples to be found here: [https://github.com/esphome/bluetooth-proxies](https://github.com/esphome/bluetooth-proxies)

#### 3D Printed Case

My particular [*ESP32-WROOM-32*](https://amzn.to/4aaZFDx) has the GPIO header pins installed as I hope to add additional sensors down the road. Because of this, the 3D printed case is less slender, but overall the footprint of the cased device is very small and easily tucked away throughout the house.

![3D print from tispokes on Thingiverse](https://exitcode0.net/images/esp32-3d-printed-case.png)

credit: [tispokes](https://www.thingiverse.com/thing:5361695/files)

It's even possible to power these devices using PoE-USB breakout adapters, so you could place these devices anywhere that you can run network cable! These are available as USB-C and Micro USB-B.

---

## Connecting the Bluetooth Proxy to Home Assistant

With the ESP32 configured and powered on, it will automatically register in Home Assistant if you’ve set up ESPHome integration. Here’s how to ensure everything is working:

1. Go to **Settings > Devices & Services** in Home Assistant.
2. Look for the ESP32 device under the ESPHome integration.
3. Confirm that the Bluetooth proxy is active.

![ESPhome Bluetooth proxy detected in Home Assistant](https://exitcode0.net/images/esphome-bt-proxy.png)

One limitation that I have noticed with the wireless ESP32 is that it tends to show up in Home Assistant as Disconnected. Unfortunately this is just a limitation of the shared wireless/bluetooth chipset - not an issue for ESP32 devices with ethernet.

---

## Adding the SwitchBot BLE Thermo-Hygrometer

With the Bluetooth proxy/proxies active, adding the SwitchBot Thermo-Hygrometer is now incredibly straightforward:

1. Remove the rear cover, pull out the transit battery disconnect slip - this powers on the device.
2. In Home Assistant, go to **Settings > Devices & Services**.
3. The Thermo-Hygrometer should appear automatically. Once added, you’ll begin to see temperature and humidity readings.

![Switchbot Temperature sensor in Home Assistant settings](https://exitcode0.net/images/ha-switchbot-sensor.png)

#### Configuring the Home Assistant Dashboard

To display the temperature and humidity on your dashboard, add a new card. Here I am using the awesome [mini-graph-card](https://github.com/kalkih/mini-graph-card) HACS plugin, this is what the dhasboard yaml looks like:

```yaml
type: custom:mini-graph-card
entities:
  - sensor.gargen_thermo_hygrometer
show:
  labels: true
color_thresholds:
  - value: -10
    color: "#3255a8"
  - value: 0
    color: "#3293a8"
  - value: 10
    color: "#82d300"
  - value: 15
    color: "#d39b00"
  - value: 19
    color: "#d35400"
  - value: 23
    color: "#c0392b"
```

This will show real-time readings directly on your Home Assistant dashboard.

![Home Assistant Mini Graph Card showing switchbot temperature data](https://exitcode0.net/images/mini-card-temp.png)

Yes... it's cold out there! This demonstrates just how great the Switchbot sensor is for this use case; its IP rating makes it a resilient outdoor temperature sensor and arguably more reliable than a local weather forecast service.

---

## The Benefits of a Bluetooth Proxy

Using an ESP32 as a Bluetooth proxy has significantly extended the range of my Home Assistant setup. With strategic placement of ESP32 devices, I now have full Bluetooth coverage across my home and garden, far beyond what a single USB Bluetooth adapter could achieve.

This project has been a great starting point for other ESPHome-based Bluetooth integrations. For example, I’ve also started experimenting with:

- Presence detection using ESP32 and BLE tags.
- Additional ESP32 devices with built-in temperature and humidity sensors.

---

## Conclusion

By using an ESP32 running ESPHome as a Bluetooth proxy, I’ve unlocked the full potential of Bluetooth integrations in Home Assistant, even in a containerized setup. The SwitchBot BLE Thermo-Hygrometer is just one example of the possibilities this setup offers. Whether you’re looking to monitor environmental conditions, enable presence detection, or control Bluetooth-enabled devices, this approach provides flexibility and scalability.



---
Markdown version of https://exitcode0.net/posts/home-assistant-with-esphome-bluetooth-proxies/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
