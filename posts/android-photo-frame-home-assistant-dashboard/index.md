# Turn a £50 Android Photo Frame Into a Home Assistant Dashboard: Root From Windows, Then Fully Kiosk

> Root a Humblestead PF1007L photo frame from Windows with Claude Code, swap its WebView and run Home Assistant in Fully Kiosk with brightness control.

Source: https://exitcode0.net/posts/android-photo-frame-home-assistant-dashboard/
Author: Tom Cocking (https://tomcocking.com)
Published: 2026-10-13
Updated: 2026-10-13
Tags: home-assistant, android, photo-frame, fully-kiosk, claude-code, root, adb, dashboard, allwinner


Wes Bos put out a video about rooting a $50 Amazon photo frame and running a family dashboard on it. The frames are Android tablets with a vendor app bolted on, and his [photo-frame-dashboard](https://github.com/wesbos/photo-frame-dashboard) repo is written so you can point a coding agent at it and let it do the fiddly bits. I wanted the same thing for Home Assistant, so I ordered the UK listing of the frame he linked and spent a day on it with Claude Code.

Two things turned out differently from the video. The frame that arrived has a different board from the one he rooted, and the route that worked for it had only ever been run from a Mac. This post is what it took from a Windows laptop, then from a Linux box, to get from stock to a locked-down Home Assistant kiosk that HA can dim.

I've done the wall-tablet thing before with a [Fire HD 8](https://exitcode0.net/posts/home-assistant-firehd8-dashboard/). The frame is cheaper, has a proper stand and a light sensor, and the result is in the cover image.

## What you actually get for £50

The listing is a Humblestead 10.1" Wi-Fi frame ([Amazon UK, B0GGB7LWX8](https://link.amazon/B0eMmC9U4)). The box says PF1007 and it ships with the AiMOR app. Inside:

| Item | Mine |
| --- | --- |
| Board | Allwinner A33, Android 6.0.1, 1 GB RAM |
| Firmware | `YHK.F-null-A33-1001-D1024-PF1007L-V10-1680.0.20260529` |
| Screen | 1280 x 800 IPS, touch |
| Ports | USB-C only, microSD, reset pinhole |
| Stock ADB | Off, over USB and over the network |
| SystemUI | None. No status bar, no navigation bar, no Back button |

That `PF1007L` in the firmware string is the thing to check first. Wes's video frame was a Rockchip RK3126 "Synergy 10X1" with ADB on out of the box and SuperSU already installed. The same Amazon listing now ships the A33 board more often than not, and his repo has a separate [start page](https://github.com/wesbos/photo-frame-dashboard/blob/main/docs/pf1007l-start-here.md) for it. Settings, About, read the string, then pick the guide.

> **Note:** Don't put it on Wi-Fi before you've read about the vendor app. AiMOR keeps an unencrypted TCP connection to the vendor's server on port 7806, and a command type on that connection is piped straight into `su`. The vendor, or anyone who can tamper with that traffic, has a root shell on the frame. Keep it offline or block outbound 7806 at the router until the app is disabled.

## The cable

The cable in the box carries power only. USB-C to USB-C gives you nothing at all, no power and no data, because the board lacks the resistors a USB-C host needs to see a device. Use a USB-A to USB-C cable you know carries data, in a USB-A port on the laptop or a powered hub. With the right cable Windows shows a `1F3A:1000` mass storage device. Without it, nothing.

## Part 1: root from a Windows laptop, no SD card

With stock ADB off and no network ADB, every remaining route writes a patched boot image. The repo's documented routes both need an SD card and a Linux or macOS host. There's an unmerged [PR #65](https://github.com/wesbos/photo-frame-dashboard/pull/65) that avoids the card: during a factory reset, stock recovery briefly exposes root ADB over USB, and you push the patch in that window. It had been run once, from a Mac. I ran it from Windows 11 with Claude Code driving adb.

### The recovery window

1. Hold the reset pin in while plugging the frame in. Release after two or three seconds.
2. AiMOR's Recovery Mode screen appears. Choose Factory reset.
3. For about 15 seconds the frame shows up as USB `18D1:D001` and `adb get-state` says `recovery`. It's root, but there's no shell, only `adb push`, `adb pull` and `adb ls`.
4. Recovery reboots itself. Android is back about 20 seconds later.

Every attempt is a factory reset, and AiMOR makes you click through its setup screens before it'll offer another one. Windows bound a working driver to `18D1:D001` on its own, so no Zadig and no Google USB driver.

It took five resets to get the patch on. Two of those were wasted because I pressed reset before Claude had armed the script, so the window opened with nothing listening. The fix was procedural: tell Claude a reset is coming, wait for it to say it's waiting on recovery, then count ten seconds before holding the pin. After that every window was caught.

### Two Windows problems

The PR's scripts are bash, so they run under Git Bash. Git Bash rewrites anything that looks like a POSIX path before it reaches `adb.exe`:

```text
adb: error: remote object 'C:/Program Files/Git/dev/block/by-name/boot' does not exist
```

Every `/dev/...` and `/sys/...` argument broke. One whole reset went on discovering that. The fix is one line before any adb command:

```sh
export MSYS_NO_PATHCONV=1
```

Also check `python3` isn't the Microsoft Store stub, and pull the PR's scripts out of git objects with `git show pr65:<path> > <path>` so `core.autocrlf` doesn't give them CRLF endings.

The second problem would have bricked the frame. The PR's write script does a lot inside the window: scans every partition's sysfs entry (15 adb calls), re-reads the whole 32 MiB boot partition to confirm it's still original, then pushes 15.9 MB. On the author's Mac, adb ran at 8.5 MB/s and his pushes finished three seconds in. Windows adb managed about 3 MB/s and 0.3 seconds per call. Claude ran the script's `check` mode, which does everything up to the write, and measured where the pushes would start:

| Step on Windows | Seconds into the window |
| --- | --- |
| Recovery ADB up | 0 |
| Boot node found | 5.1 |
| Boot re-read finished | 15.0 |
| Window closed | about 16 |

The push would begin as recovery rebooted. Half a ramdisk with the original header is a frame that doesn't boot, and the only way back from that is the SD card route. So the published script was not run on this host.

### A trimmed write script, rehearsed first

Claude wrote a shorter script with the same payloads and as few in-window adb calls as it could get away with. It fixes the boot node to `mmcblk0p6` and checks it with one pull of sysfs `start` and `size`, skips the in-window re-read (boot had been confirmed original in two earlier windows, and a factory reset doesn't touch it), then pushes the ramdisk slot, then the header, then reads back if there's time.

The useful trick was a `rehearse` mode that runs the identical steps and timing but pushes to a file in recovery's RAM-backed `/dev` instead of the eMMC. Reset 4 was the rehearsal: node checked at 0.3 s, both pushes done by 2.6 s, read-back at 10.9 s showing the boot still original. Reset 5 was the real write: pushes done by 3.3 s, read-back showed the patched hash at 10.1 s, and the frame came back as `1F3A:1002` with `adb shell id` returning `uid=0(root)`.

Two details from the script worth keeping even if you never run it:

- Push to `/dev/block/mmcblk0p6`, never to `/dev/block/by-name/boot`. The by-name entries are symlinks, and Android 6's adbd unlinks the symlink before writing. The push reports success and writes nothing.
- The script checks every payload hash against the plan before it will arm, and aborts on any mismatch. Don't edit a guard to make it pass.

The script is in my notes rather than this post because its hashes and sectors are specific to the May firmware. On any other firmware the plan has to be rebuilt, and if the boot image hash isn't in the repo's firmware table, stop.

### After root, before anything else

Back up every partition except user data, verifying each one on the frame:

```bash
for n in mmcblk0boot0 mmcblk0boot1 mmcblk0p2 mmcblk0p3 mmcblk0p5 mmcblk0p6 mmcblk0p7 mmcblk0p8 \
         mmcblk0p9 mmcblk0p10 mmcblk0p11 mmcblk0p12 mmcblk0p13 mmcblk0p14 mmcblk0p15 mmcblk0p16; do
  sz=$(adb -d shell "cat /sys/class/block/$n/size" | tr -d '\r[:space:]')
  adb -d exec-out "dd if=/dev/block/$n bs=1048576 2>/dev/null" > "$out/$n.img"
  [ "$(wc -c < "$out/$n.img")" -eq $((sz * 512)) ] && \
  [ "$(md5sum < "$out/$n.img" | cut -d' ' -f1)" = "$(adb -d shell "md5sum /dev/block/$n" | tr -d '\r' | cut -d' ' -f1)" ]
done
```

Seven minutes over USB. Then disable the vendor software, install a launcher so there's still a home screen, and fix the time zone (it ships set to New York):

```sh
adb -d install rootless.apk     # Rootless Launcher from F-Droid
for p in com.efercro.os.aimor com.yhk.qeota com.DeviceTest; do adb -d shell "pm disable $p; am force-stop $p"; done
adb -d shell 'setprop persist.sys.timezone Europe/London; settings put global auto_time 1'
```

> **Warning:** `pm disable` returns before Package Manager has written the change to disk, and `sync` doesn't force it. Poll `/data/system/users/0/package-restrictions.xml` until every disabled package shows `enabled="2"` before you pull the power. We lost a disable to a power cycle once; the second time it took 12 seconds to land.

### ADB over Wi-Fi

```sh
adb -d shell setprop persist.adb.tcp.port 5555
```

Then power-cycle. Do not run `stop adbd; start adbd` through an adb shell. Stopping adbd kills the shell that was about to start it, and the frame drops off USB entirely. We did exactly that. A power cycle recovered it, because init starts adbd at boot and the property had already been saved.

The Wi-Fi chip has no stored MAC address, so the MAC and the DHCP lease change every boot. Set a static IP on the frame itself (long-press the network, Modify network, IP settings: Static). Getting back out of Settings afterwards needs adb, because there's no Back button:

```sh
adb shell am start -a android.intent.action.MAIN -c android.intent.category.HOME
```

## Part 2: from a root shell to a Home Assistant kiosk

At this point the laptop handed over. The frame was on a static IP with root ADB open on port 5555, Firefox 143 installed and pointed at HA. From here on I worked from my Linux server, which could reach the frame directly:

```sh
adb connect 192.168.1.50:5555
adb -s 192.168.1.50:5555 shell id     # uid=0(root)
```

One thing that cost ten minutes: Android 6's adbd will only serve one host over TCP. While the laptop's adb server still held the connection, the server saw the frame as `offline` and a raw handshake to port 5555 got no reply. `adb kill-server` on the laptop fixed it.

Firefox rendered HA fine, and that's worth knowing as a fallback. But it has a URL bar, no kiosk mode, no auto-start, and nothing HA can talk to. I wanted Fully Kiosk Browser, because its [HA integration](https://www.home-assistant.io/integrations/fully_kiosk/) gives you a brightness slider, a screen on/off switch and a screensaver as entities. Fully renders through the system WebView, and the system WebView on this frame is Chromium 44 from 2015. HA's frontend won't load in it.

### Swap the system WebView

Wes's repo documents replacing the stock WebView with a Chromium 106 build from LineageOS's prebuilts. It's old, but it renders today's HA frontend, and it's the newest engine that runs on Android 6. The procedure is a `/system` write, so: back up, stage, verify, swap, verify, reboot.

```sh
F="-s 192.168.1.50:5555"
# 1. Back up the stock tree to the host, read it twice, compare hashes
adb $F exec-out 'toybox tar -C /system/app -cf - webview' > stock-webview.tar

# 2. Stage the new APK and its native libs under /data (not /system yet)
unzip -j webview.apk 'lib/armeabi-v7a/*.so' -d stage/lib/arm
adb $F push webview.apk /data/local/tmp/webview106/webview.apk
adb $F push stage/lib/arm /data/local/tmp/webview106/lib/

# 3. Create the new tree on /system, read every file back and compare
adb $F shell 'mount -o remount,rw /system
  mkdir -p /system/new-webview/lib/arm
  cp /data/local/tmp/webview106/webview.apk /system/new-webview/
  cp /data/local/tmp/webview106/lib/arm/*.so /system/new-webview/lib/arm/
  chown -R root:root /system/new-webview
  find /system/new-webview -type d -exec chmod 0755 {} \;
  find /system/new-webview -type f -exec chmod 0644 {} \;'

# 4. Swap, keeping the stock tree (and its odex) intact outside /system/app
adb $F shell 'mv /system/app/webview /system/stock-webview
  mv /system/new-webview /system/app/webview
  sync; mount -o remount,ro /system'
adb $F shell 'svc power reboot'
```

The frame has no `sha256sum`, so verify by `adb exec-out cat` of each file piped into `sha256sum` on the host. After the reboot:

```text
$ adb shell dumpsys package com.android.webview | grep versionName
    versionName=106.0.5249.126
```

The native libraries have to be extracted and placed under `lib/arm`, not left inside the APK. Android 6 doesn't unpack them for system apps, so an APK-only swap reports 106 and then fails to load a page.

> **Note:** `adb reboot` hangs on these units more often than it works. `adb shell 'svc power reboot'` is reliable.

### Fully Kiosk

Install Fully (1.61.4 declares a minimum of Android 6.0, so the current build is fine) and launch it on the HA URL:

```sh
adb $F install /data/local/tmp/fully.apk
adb $F shell am start -n de.ozerov.fully/.MainActivity -a android.intent.action.VIEW -d http://192.168.1.10:8123/lovelace/serverroom
```

On a frame with no accounts, Fully immediately offers to become the device owner. Android's managed-provisioning app crashed halfway through that on my unit, so I set it directly with root:

```sh
adb $F shell dpm set-device-owner de.ozerov.fully/.MyDeviceAdmin
```

Fully also needs three permissions that Android 6 would normally prompt for through the UI. Grant them over adb instead:

```sh
adb $F shell 'appops set de.ozerov.fully SYSTEM_ALERT_WINDOW allow
  appops set de.ozerov.fully GET_USAGE_STATS allow
  pm grant de.ozerov.fully android.permission.READ_EXTERNAL_STORAGE
  pm grant de.ozerov.fully android.permission.WRITE_EXTERNAL_STORAGE
  pm grant de.ozerov.fully android.permission.WRITE_SECURE_SETTINGS'
```

To configure it without tapping through menus on a screen with no Back button, push a one-time settings file and restart the app. Fully reads `fully-once-settings.json` from shared storage at start, as long as it has the storage permission above:

```json
{
  "startURL": "http://192.168.1.10:8123/lovelace/serverroom",
  "remoteAdmin": true,
  "remoteAdminOnLocalNetwork": true,
  "remoteAdminPassword": "choose-something-long",
  "kioskMode": false
}
```

```sh
adb $F push fully-once-settings.json /sdcard/fully-once-settings.json
adb $F shell 'am force-stop de.ozerov.fully; am start -n de.ozerov.fully/.MainActivity'
```

Once Remote Admin is up on port 2323, everything else goes through its REST API from the host. Booleans must use the boolean endpoint; the string endpoint silently refuses them:

```sh
A="http://192.168.1.50:2323/?password=choose-something-long&type=json"
for kv in launchOnBoot=true kioskMode=true confirmExit=false reloadOnInternet=true autoImportSettings=false; do
  curl -s "$A&cmd=setBooleanSetting&key=${kv%%=*}&value=${kv#*=}"
done
curl -s "$A&cmd=setStringSetting&key=kioskPin&value=123456"
curl -s "$A&cmd=listSettings" | python3 -m json.tool | less    # 405 keys, find anything else here
```

Turning on kiosk mode makes Fully the preferred Home app, so it's what comes back after a power cut. Turning auto-import off afterwards means a stray JSON file on the SD card can't reconfigure it later.

Fully showed a device provisioning wizard once after becoming owner. Skip it. It didn't come back.

![Home Assistant login page rendered inside Fully Kiosk, offering a choice of user and a local password login](https://exitcode0.net/images/android-photo-frame-home-assistant-dashboard-login-user-choice.jpg)

### The HA side

Add the Fully Kiosk Browser integration with the frame's IP and the Remote Admin password. Mine came up with every entity named `unknown_*`, because Fully reports its device name as "unknown" on this firmware no matter what you set, so I renamed the device and the 33 entity ids in the registry. Then the test that was the whole point:

```sh
# HA service call: number.set_value on number.photo_frame_screen_brightness = 80
$ adb shell 'dumpsys display | grep mActualBacklight'
  mActualBacklight=80
```

The backlight follows the entity. From here, dimming the frame when the room lights go off is an ordinary automation on that number, and `switch.photo_frame_screen` turns the panel off at night. The garage graph in the screenshot is the [mini-graph-card](https://exitcode0.net/posts/mini-graph-card-temperature-humidity-home-assistant/) I wrote up earlier; it renders fine in Chromium 106. The free memory sensor is handy too: a 1 GB frame running HA's frontend sits at about 670 MB free with nothing else open.

![Home Assistant overview with printer camera, garage temperature graph and 3D printer card, full screen in Fully Kiosk on the frame](https://exitcode0.net/images/android-photo-frame-home-assistant-dashboard-fully-kiosk-overview.jpg)

One HA config change was needed on my install. I had `allow_bypass_login: true` on the trusted-networks auth provider, which logs every LAN device in as a guest user with no way to pick anyone else. Removing that one line brings the login page back with a user choice, while the provider itself keeps working for scripts. I also made a non-admin `frame` user for the panel. The frame is logged in as me for now, which is lazy, but the sidebar is collapsed and the top bar is useful for flicking between dashboards.

### Lock ADB to one key

Root ADB with no authentication on the LAN was fine for an afternoon, not for a wall. The patched boot image sets `ro.adb.secure=0`, and with no SystemUI there's no "Allow USB debugging" prompt, so the fix is a known key plus flipping the flag in build.prop:

```sh
adb $F push ~/.android/adbkey.pub /data/local/tmp/adb_keys
adb $F shell 'mkdir -p /data/misc/adb
  cp /data/local/tmp/adb_keys /data/misc/adb/adb_keys
  chown system:shell /data/misc/adb/adb_keys; chmod 0640 /data/misc/adb/adb_keys'
# edit build.prop on the host (ro.adb.secure=0 -> 1), push it back, byte-exact
adb $F shell 'mount -o remount,rw /system; cp /data/local/tmp/build.prop /system/build.prop
  chown root:root /system/build.prop; chmod 0644 /system/build.prop; sync; mount -o remount,ro /system'
adb $F shell 'svc power reboot'
```

Edit the file on the host. The frame's toybox `sed` mangled the line on the first try and left `ro.adb.secure=1 ro.adb.secure=0`, which I only caught by diffing the file against the backup. After the reboot, a client with a fresh key gets `unauthorized` and the known key gets root. Keep a copy of `~/.android/adbkey` somewhere safe: if adb ever regenerates it, you're locked out and the only way back in is the SD card.

## Quick reference

| Want | Do |
| --- | --- |
| Which board is this? | Settings, About. `PF1007L` and `A33` is this post. `rk312x` is the video's guide |
| USB shows nothing | USB-A to C data cable, direct port or powered hub. The boxed cable is power only |
| Recovery window | Reset pin held while plugging in, Factory reset, about 15 s of root ADB as `18D1:D001` |
| Git Bash breaks adb paths | `export MSYS_NO_PATHCONV=1` |
| Frame won't take a second adb host | `adb kill-server` on the first one |
| Reboot | `adb shell 'svc power reboot'`, not `adb reboot` |
| Fully won't take a boolean | Use `setBooleanSetting`, not `setStringSetting` |
| Entities named unknown | Rename the device and entity ids in HA; Fully's name setting is ignored |
| Back out of Settings | `am start` the Home activity over adb. There is no Back button |

## Would I do it again

Yes, but I'd budget the day, not the hour the video suggests. The rooting half only went smoothly because the agent measured the window before writing into it, and because a factory reset is cheap to repeat. If the frame that turns up is the Rockchip one, you'll skip most of Part 1. If it's the A33 and you own an SD card and a Linux box, the repo's Route U is the better-trodden path than the recovery window.

## Or just hand this page to your agent

Pointing Claude Code at this page and Wes's repo is how most of it was done:

```text
Read https://exitcode0.net/posts/android-photo-frame-home-assistant-dashboard/ and the PF1007L start page in https://github.com/wesbos/photo-frame-dashboard, then help me turn my photo frame into a Home Assistant kiosk. Before anything else, ask me for the firmware string from the frame's About screen and stop if it isn't in the repo's firmware table. Never write to the boot partition or to /system without showing me exactly which bytes change and how to undo it, and get my explicit yes for each one. Ask me for my Home Assistant URL, the dashboard path to show, and the frame's static IP rather than guessing them. Don't put the frame on Wi-Fi until the AiMOR app is disabled.
```

The page has the cable rule, the recovery-window timing, the WebView swap, the Fully settings and the ADB lock-down. Only you know which firmware you've got, your HA address, which dashboard belongs on the wall, and whether you're prepared to lose a frame if a write goes wrong.

Enjoy. ✌️


---
Markdown version of https://exitcode0.net/posts/android-photo-frame-home-assistant-dashboard/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
