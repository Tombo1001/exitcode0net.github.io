# Pimoroni Enviro Urban Battery Drain: A Low-Power MicroPython Firmware Fix

> The Pimoroni Enviro Urban flattens batteries when its firmware hangs with the power latch on. Replacement MicroPython firmware and a battery dashboard fix it.

Source: https://exitcode0.net/posts/pimoroni-enviro-urban-low-power-firmware-battery-life/
Author: Tom Cocking (https://tomcocking.com)
Published: 2026-11-22
Updated: 2026-11-22
Tags: pimoroni, enviro-urban, pico-w, micropython, mqtt, low-power, battery, air-quality, home-assistant, docker



The Pimoroni Enviro Urban is a lovely little air quality board with a reputation for flattening its batteries. It's designed to sleep at a few microamps and wake on a timer, which should mean months on a set of AAs. Pimoroni's issue tracker has years of reports of boards that stop waking and drain their batteries instead. The cause is in the firmware, not the board, and Pimoroni's repo hasn't had a commit to main since April 2024.

So I replaced the firmware. The new one runs on stock MicroPython, reads the sensors, sends one MQTT message and cuts its own power. I also built a dashboard for it, because voltage is the only fuel gauge on this board and I wanted a number of days, not a number of volts. Both are on GitHub: [Tombo1001/enviro-urban-lowpower](https://github.com/Tombo1001/enviro-urban-lowpower).

**The short version**, if you only want the shape of it:

- The stock firmware can hang with the power latch held on. A hung board draws tens of mA until the batteries are flat.
- The replacement adds a hardware watchdog and lets the RTC's own countdown timer do the waking, so a hang ends in power off and the next wake still happens.
- On a 2000 mAh LiPo mine measured about 33 mAh a day at 15 minute wakes, which works out at roughly 57 days a charge. That's from under a day of data, so treat it as an estimate.
- The dashboard is one Docker container. It logs the MQTT messages, estimates battery life, changes the wake interval remotely and sets up new boards.

I built both with Claude Code doing most of the typing, from the [dev VM setup I've written about before](https://exitcode0.net/posts/dumb-terminal-dev-vm-tailscale-claude-code/). The numbers below are from my one board.

## Why the Enviro Urban drains its batteries

The hardware is a good low-power design. The RTC and a tiny regulator sit on an always-on rail. A P-FET latch disconnects the whole Pico W from the battery. The latch closes when the RTC's interrupt line fires, when you press the button, or while GP2 is held high. Asleep, nothing but the RTC is powered.

The trouble starts when the RP2040 hangs while GP2 is high. The latch stays closed, the board stays fully powered, and nothing recovers it. The evidence is in Pimoroni's own issue tracker:

- In [enviro#87](https://github.com/pimoroni/enviro/issues/87), sjefferson99 put a meter on a hung board. GP2 sat at 3.3 V with the rails up. Shorting `3V3_EN` to ground shut the board down, and "the RTC woke it up shortly after". The latch worked and the RTC worked. The CPU was stuck.
- [enviro#210](https://github.com/pimoroni/enviro/issues/210) blames the board for thinking it's on USB. The log line behind that, `on usb power (so can't shutdown)`, is printed after every shutdown with no check at all. On battery it still gets written, because the supply takes a few milliseconds to collapse after the latch opens.
- Battery monitoring was switched off in [v0.0.9](https://github.com/pimoroni/enviro/releases/tag/v0.0.9). The battery pin is shared with the WiFi chip, and reading it while WiFi ran upset the WiFi.

There's no watchdog in the stock firmware, so a hang is permanent until the batteries give out.

## What the replacement does differently

| Stock firmware | This firmware |
|---|---|
| A hang leaves the latch held, and nothing recovers it. | An 8 second hardware watchdog resets the chip. A reset releases GP2, so a hang ends in power off. |
| The next wake depends on software re-arming an RTC alarm against a wall clock. | The RTC countdown timer reloads itself in hardware. The board wakes again even if the last run crashed. No clock or NTP needed. |
| Battery reading disabled. | The battery is read before the WiFi chip is powered in that wake, and again with the fan running. |
| A crash leaves no trace. | A marker in the RTC's spare RAM byte records how far each wake got. The next message carries `last_fault: wifi`, or whichever stage it died in. |
| Particulate readings jump about ([enviro#172](https://github.com/pimoroni/enviro/issues/172)). | The fan runs for 30 seconds first. Frames are checksum-checked and three are averaged. |
| Needs Pimoroni's custom MicroPython build. | Runs on stock MicroPython for the Pico W. |

It's a different firmware, not a patch, and it drops things. There's no web provisioning on the board, no local CSV logging, no InfluxDB or Adafruit IO upload, and no support for the Indoor, Weather or Grow boards. It sends MQTT and nothing else.

## What you need

- A Pimoroni [Enviro Urban](https://shop.pimoroni.com/products/enviro-urban).
- A battery. AAs work. I used a 2000 mAh single-cell LiPo.
- An MQTT broker the board can reach, such as Mosquitto.
- Python on your computer for `mpremote`, and a micro USB cable.
- Optional: a machine that runs Docker, for the dashboard.
- Optional: an Adafruit Micro Lipo charger. More on that below.

## Install the firmware

Clone the repo and work from its `firmware` folder:

```bash
git clone https://github.com/Tombo1001/enviro-urban-lowpower.git
cd enviro-urban-lowpower/firmware
pip install mpremote
```

1. **Wipe the board.** Unplug the battery. Hold BOOTSEL on the Pico W and plug in USB. A drive called `RPI-RP2` appears. Copy [flash_nuke.uf2](https://datasheets.raspberrypi.com/soft/flash_nuke.uf2) onto it. A different UF2 can keep the old filesystem, and you don't want Pimoroni's files left behind.
2. **Flash MicroPython.** BOOTSEL again, then copy the latest stable `RPI_PICO_W` build from [micropython.org](https://micropython.org/download/RPI_PICO_W/) onto the drive.
3. **Write your config.** Copy `config_example.py` to `config.py` and fill in the WiFi and broker details:

   ```python
   NAME = "enviro-001"              # used in MQTT topics and as the hostname

   WIFI_SSID = "your-ssid"
   WIFI_PASSWORD = "your-password"
   WIFI_COUNTRY = "GB"

   MQTT_BROKER = "192.168.1.10"     # use an IP, not a hostname: avoids a DNS lookup
   MQTT_PORT = 1883
   MQTT_USER = None
   MQTT_PASSWORD = None

   INTERVAL_MIN = 15                # RTC countdown timer, 1..255 minutes
   PM_EVERY_N = 4                   # run the particulate sensor on every Nth wake
   ```

4. **Copy everything except `main.py`:**

   ```bash
   mpremote mkdir :lib
   mpremote mkdir :lib/umqtt
   mpremote cp lib/urban_hw.py lib/pcf85063a_lite.py lib/bme280_lite.py :lib/
   mpremote cp lib/umqtt/simple.py :lib/umqtt/
   mpremote cp config.py diag.py boot.py :
   ```

   Stock MicroPython for the Pico W doesn't include an MQTT client, so the repo carries `umqtt.simple` from micropython-lib.

5. **Run the hardware check:**

   ```bash
   mpremote run diag.py
   ```

   It prints PASS, FAIL or WARN for the battery reading, both I2C devices, the RTC, the wake signal path, the BME280, the microphone, the particulate sensor, WiFi and a test MQTT publish. The particulate check takes a while because the fan has to warm up.

6. **Install the firmware itself** once the check passes:

   ```bash
   mpremote cp main.py :
   ```

Unplug USB, connect the battery, and the board wakes, publishes and powers off.

> **Note:** once `main.py` is on the board, the watchdog will reset it in the middle of an `mpremote` session. To get back in, hold **STATUS** for 3 seconds while pressing **RESET**. The white LED stays on and the board waits with the watchdog off.

## What it sends

One retained message per wake on `enviro/<name>/state`. This is a real one from mine:

```json
{"fw": "urban-lp 0.1", "wake": "timer", "reset_cause": "power_on",
 "battery_v": 3.843, "temperature": 19.61, "humidity": 54.1, "pressure": 992.66,
 "noise": 0.029, "rssi": -57, "usb": 0, "awake_ms": 6195,
 "interval_min": 5, "pm_every_n": 4}
```

`reset_cause: power_on` on battery is the line to look for. It means the board really lost power between wakes. `awake_ms` is about 5 seconds normally and about 40 when the fan runs. Particulate wakes add a second message on `enviro/<name>/pm` with PM1, PM2.5 and PM10.

With `HA_DISCOVERY = True` the board also announces its sensors to Home Assistant over MQTT, so they appear without any YAML. If Home Assistant is where your readings end up, the [presence sensor post](https://exitcode0.net/posts/home-assistant-presence-sensor-desk-screens/) shows the sort of automation a room sensor can drive, and [Mini Graph Card](https://exitcode0.net/posts/mini-graph-card-temperature-humidity-home-assistant/) will chart the temperature and humidity on one card.

You can change settings without touching the board. Publish a retained JSON object to `enviro/<name>/set` and it's applied at the next wake:

```bash
mosquitto_pub -h 192.168.1.10 -r -t 'enviro/enviro-001/set' -m '{"INTERVAL_MIN": 15, "PM_EVERY_N": 4}'
```

## The dashboard

A stream of voltages doesn't tell you when to go and fetch the board. I wanted days left, a record of every fault, and a way to change the wake interval without plugging the board in.

![The dashboard for one board: readings pinned to a drawing of the board's front and back, a battery card showing about 48 days left, and wake interval controls](https://exitcode0.net/images/pimoroni-enviro-urban-low-power-firmware-battery-life-dashboard.jpg)

It's one container:

```bash
cd enviro-urban-lowpower
cp .env.example .env        # put your broker's address and login in here
docker compose up -d
```

Open `http://<that machine>:8870`. A board gets its own tab the first time one of its messages reaches the broker, so there's nothing to add by hand.

> **Note:** there's no login. Anyone who can open the page can change a board's wake interval. Keep it on a network you trust.

The diagrams above are drawn. By default the container downloads the makers' own product photos on first start and uses those instead. I kept them out of the repo because they aren't mine to publish. Set `PHOTOS=off` to stay with the drawings.

### How the battery estimate works

A LiPo's voltage is flat through the middle of its charge, so fitting a straight line to voltage and extending it to empty gives nonsense. The dashboard does this instead:

1. Each battery reading goes through a voltage-to-charge table to give the charge left.
2. Each wake gets a modelled cost: seconds awake at 70 mA, plus 30 seconds of fan at 150 mA on particulate wakes.
3. Charge left is fitted against the running total of modelled cost. The slope says how many times the real drain exceeds the model.
4. Days left is the usable charge, down to 3.5 V, divided by the calibrated daily cost at the current wake interval.

Fitting against charge spent, and not against time, means the estimate survives a change of interval halfway through. It needs 20 readings over 2 hours on battery before it trusts a fit. Until then it shows a range.

![History charts from the dashboard: battery voltage flat near 3.85 V overnight then stepping up to 4.1 V during a charge, with particulates, temperature and humidity alongside](https://exitcode0.net/images/pimoroni-enviro-urban-low-power-firmware-battery-life-history.jpg)

### Changing the wake interval

The **On battery** chips set how often the board wakes and how often the fan runs. Picking one only updates the estimate, so you can see what a setting would cost before committing to it. **Send to board** publishes the retained config message. The board is asleep, so the page shows "Waiting for the board" until a wake reports the new interval.

**On charge** switches the board to 1 or 5 minute wakes while it's charging and puts the old interval back afterwards. It runs in the container, so the page doesn't need to be open.

### Setting up a board from the browser

The **Provision** tab builds `config.py` from a form and copies the firmware over USB using Web Serial, with the hardware check in the middle. The config file is built in your browser, so the WiFi password never goes to the dashboard server.

![The Provision tab: a form for the board's name, WiFi and MQTT details, followed by the steps for flashing MicroPython](https://exitcode0.net/images/pimoroni-enviro-urban-low-power-firmware-battery-life-provision.jpg)

Two catches. A browser can't write to the Pico's boot drive, so flashing MicroPython is still drag and drop. And browsers switch Web Serial off on plain HTTP, so the page has to be on HTTPS or on `localhost`. If your services already sit behind HTTPS on a tailnet, as in my [Home Assistant HTTPS with Tailscale post](https://exitcode0.net/posts/homeassistant-tls-with-tailscale-traefik/), you're set. Otherwise the tab offers the firmware as a zip with `mpremote` instructions.

I haven't run the browser install against a real board yet. Mine was installed with `mpremote`, using the steps above.

## Charging the LiPo with an Adafruit Micro Lipo

Enviro boards can't charge a battery. Pimoroni's product page says "there's no battery charging included on Enviro boards, so you'll need a separate LiPo battery charger". Plugging USB into the Pico W powers the board and leaves the cell exactly where it was. Mine sat on USB for eight minutes and the cell read 3.854 V before and 3.857 V after.

I used an [Adafruit Micro Lipo](https://www.adafruit.com/product/4410), wired to the cell in parallel with the board. The cell stays plugged into the Enviro's battery socket and a second pair of leads runs to the charger.

> **Note:** check polarity before you plug anything in. JST-PH cells aren't all wired the same way round, and a reversed one will destroy the charger or the board.

Wiring it this way has a useful side effect. The board stays on battery throughout, so it keeps reading the cell's voltage as it climbs, and the dashboard can see the charge happen. The charger diagram on the page lights up, and the battery card switches from days left to time to full.

The Micro Lipo charges at 100 mA as shipped and 500 mA with its jumper soldered closed. Tell the dashboard which, because it can't measure charge current. It counts charge in from the moment the voltage steps up, at the rate you set. When I plugged mine in, the cell jumped from 3.84 V to 4.00 V in one reading. That's a big step, and it's what a 500 mA charge looks like.

## Enviro Urban battery life: the numbers so far

One board, one 2000 mAh LiPo, about 18 hours and 222 wakes.

| | |
|---|---|
| Drain at 5 minute wakes, fan every 4th | about 107 mAh a day |
| Drain at 15 minute wakes, fan every 4th | about 33 mAh a day |
| Full charge at 15 minute wakes | about 57 days |
| Measured drain against the firmware's power budget | 0.6 times |
| Wakes that never arrived | 1 |
| WiFi timeouts, recovered at the next wake | 1 |

The drain figures come from a fit over 15 hours on battery. They depend on a generic LiPo voltage table, so I wouldn't quote them to better than a quarter either way. A recorded full discharge would tighten that up, and I haven't done one.

I haven't put a meter on the sleep current. The few microamps asleep is what the schematic says it should be.

## Things that tripped me up

### One bad voltage reading ended a charge

Halfway through the first charge, one reading came back at 4.073 V among a run of 4.13 V. The dashboard took that as the charger being unplugged, called the charge finished and put the board back on slow wakes. A charge now ends only on two low readings in a row.

### Capacity doesn't change the days

The battery card has a setting for the cell's capacity. Once the drain is fitted, changing it moves the mAh figures and leaves the days alone. I checked: 1000 mAh and 2000 mAh gave the same answer to a decimal place. The fit works in percent of charge per day, so capacity cancels out. It still matters before a fit exists, and for the charge time.

### The board rejects config it can't parse, and tells you

My first two remote config messages were rejected. The payload had been escaped twice on the way to the broker, so the board received a string, not an object. The next state message carried `config_error: ValueError('syntax error in JSON',)`, which is how I found out. Send plain JSON.

### "PM off" isn't a wake setting

`PM_EVERY_N` runs from 1 to 15. To switch the particulate sensor off altogether, set `PM_MIN_V = 99` in `config.py`, which makes the board skip it at any battery voltage.

## Troubleshooting

- **No messages at all.** Run `mpremote run diag.py` in maintenance mode. It tests WiFi and a publish separately, so you can see which one fails.
- **`last_fault` in a message.** The previous wake died at that stage and the watchdog recovered it. One now and then on a weak WiFi signal is the system working. A run of them at the same stage is worth a look.
- **`reset_cause` isn't `power_on` on battery.** The board restarted without losing power, so the latch didn't open. Check nothing else is feeding the board, then read `log.txt` off it with `mpremote cp :log.txt .`.
- **The red LED stays on.** The RTC lost power and reset. The firmware turns its clock output off again at the next wake and reports `rtc_power_lost`.
- **The dashboard shows no boards.** Check the Settings tab. It says whether the container is connected to the broker and how many messages it has logged.

## Worth a weekend

The board is on its second day on a LiPo, reporting every fifteen minutes, and the page says it has about seven weeks before I need to find the charger. My older [DHT11 on a Raspberry Pi](https://exitcode0.net/posts/using-a-dht11-sensor-with-a-raspberry-pi/) needed a wall socket to do less.

## Or just hand this page to your agent

The install is mostly copying files in the right order, which an agent is good at.

```text
Read https://exitcode0.net/posts/pimoroni-enviro-urban-low-power-firmware-battery-life/ and set up the low-power firmware on my Pimoroni Enviro Urban, using the repo at https://github.com/Tombo1001/enviro-urban-lowpower. Before you write anything to the board, ask me for my WiFi name and password, my MQTT broker's address and login, and what to call the board, and show me the finished config.py. Don't run flash_nuke or flash MicroPython yourself: tell me when to hold BOOTSEL and which file to copy, because wiping the board deletes any old readings on it. Copy the support files, run diag.py and show me its full output before installing main.py. If I want the dashboard too, ask which machine and port to run it on before starting the container, and remind me it has no login.
```

The page gives the agent the file order, the commands, the config keys and what a healthy message looks like. Only you know your WiFi, your broker and whether there's anything on the board worth keeping.

Leave it on a 1 minute wake for the first evening and watch the messages arrive. Then set it to 15 and forget about it.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/pimoroni-enviro-urban-low-power-firmware-battery-life/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
