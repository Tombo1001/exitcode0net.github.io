# Using an Amazon FireHD8 as a Home Assistant Dashboard Control Panel

> Using an Amazon Fire HD 8 as a wall-mounted Home Assistant control panel: setup, docking, the pros and cons, and the alternatives I considered.

Source: https://exitcode0.net/posts/home-assistant-firehd8-dashboard/
Author: Tom Cocking (https://tomcocking.com)
Published: 2024-06-11
Updated: 2026-10-08
Tags: home-automation, home-assistant, hardware, review


I have covered Home Assistant (HA) content a number of times in previous posts, and am I a strong advocate for this project. The home automation scene is a wash with consumer offerings, but no single silicone valley big hitter has managed to provide a fully interoperable solution; most certainly not an affordable one. Ultimately, everyone has there own idea of how home automation should work. Home Assistant doesn't solve the problem of standards dilution, but it offers an extremely robust way to tie the wide variety of standards and products together.

Some related posts which you might want to save for later:

- [Home Assistant HTTPS Certificates with Tailscale, Traefik and CoreDNS](https://exitcode0.net/posts/homeassistant-tls-with-tailscale-traefik/)
- [Home Assistant Enable MagicDNS and HTTPS Certificates in Tailscale](https://exitcode0.net/posts/homeassistant-tls-with-tailscale/)
- [How to Use a Docker Compose File for Wyoming Whisper](https://exitcode0.net/posts/wyoming-whisper-docker-compose/)
- [Setting up Wyoming Piper with Docker Compose](https://exitcode0.net/posts/wyoming-piper-docker-compose/)

## Home Assistant Introduction

Home Assistant (HA) allows you to connect your various home automation systems as 'Integrations', then you can start to build routines, scene, and actions dependant on a vast range of conditions or controls. As the project has matured, the options and abilities of Home Assistant have reached a point where they are only limited by your imagination. There is also a fantastic community would are constantly pushing the boundaries and sharing their experiences and knowledge over at the HA forums: [https://community.home-assistant.io/](https://community.home-assistant.io/).

### Dashboards

Once your integrations on setup, a common way to interface with them is through the HA lovelace dashboards that you can build; particularly if you have sensors which are collecting data about your home environment.

#### Users

It is often the case that there is one home automation enthusiast in the household and everyone else just wants to turn the lights without the needing to understand what a zigbee network is... Home Assistant allows you to configure multiple users within the configuration. Users can then be given a role, user/admin, and be allocated access to only the lovelace dashboards that they need. For example: your partner only needs to be given access to light switch toggles and not access to the smart plugs which power your home server.

![Our HA dashboard on a FireHD8+](https://exitcode0.net/images/ha-dashboard-firehd8.png)

We can also create a user with need-to-know dashboard access for our tablet. In 99% of scenarios, this tablet will be left in an easy to access place, where even guests can interact with it. Giving the 'tablet_user' account read only access to limited lovelace dashboards means that they cannot edit HA settings, or see excessive data.

Here is how we limit dashboard access for this tablet account:

![Limiting Home Assistant dashboard visibility](https://exitcode0.net/images/ha-dashboard-user-access-control.png)

Edit the dashboard, edit the view, locate the visibility tab and modify the toggle options, before hitting the save button. Remember to refresh all your other dashboard displaying devices to apply the changes.

#### Rooms

Similarly, if we have multiple tablets, possible one per room, we can create users on a per room basis and use access controls to set a default dashboard for that room. Room specific dashboards might include shopping list cards in the kitchen or light and CCTV controls in the bedroom. The possibilities are almost endless and the best path to get the most out of the system would be to build automations and scenes for each room; then add controls or triggers for these on that room's dashboard.

## Amazon FireHD8 Control Panel

This where we reach the main focus of this article, testing an Amazon FireHD8 Plus (with ads) as a dashboard control panel.

I chose the 'with ads' version of the FireHD8, which is ~£10 cheaper. Here are some resources for removing the bloat and getting GApps and the HA app installed in Kiosk mode:

- Fire Toolbox V34.0: https://xdaforums.com/t/windows-linux-tool-fire-toolbox-v34-0.3889604/
- Fire Toolbox tutorial: https://liliputing.com/hack-your-amazon-fire-tablet-with-fire-toolbox-v10/

The fireHD8/8+ is restricted meaning that Amazon has locked down the firmware. When a firmware is restricted it means we cannot fully block OTAs, use custom launchers without Fire Launcher, lockscreen ads can no longer be permanently removed, and some system apps can't be disabled. Even though a tablet is marked as Restricted in the table above, sometimes it may only apply to certain versions of FireOS. If your tablet is on a firmware before the listed firmware then your tablet is NOT restricted.

It is still possible to disable some of the fluff, and side-load GAPPs (Play Store).

---

**Reach out if you need help with this:**

> :bird:/:heavy_multiplication_x:: [~~Tweet~~ X me](https://x.com/TomC20924637)

> :camera:: [Instagram direct message](https://www.instagram.com/exitcode_0)

> :email:: [Drop me an email](https://tomcocking.com/#footer)

---

### Pros

#### Cost

The FireHD8 is relatively cheap and can often be had a very reasonable price on the various Prime sales which occur through the year. And if you are willing to by used, the Amazon warehouse deals make these tablets an even more affordable option. The build quality is understandably poor, but there Fire tablets are a known quantity when compared to the 'no name' android tablet alternatives.

- [**Lowest Ever: £61.99 (Nov 18, 2022)** - Unlikely to be this price again due to inflation](https://3cmls.co/UK/B09BG59L4Y)

- [**Current: £119.99 (May 16, 2024)**](https://www.amazon.co.uk/fire-hd-8-plus/dp/B09BG59L4Y)

#### Docking Station

> **NOTE: this only applies to the FireHD8 Plus.**

The goal of the tablet is to look like an intentional home automation control panel, so the official docking station with _wireless_ (a concealed DC power adapter is still needed for the dock) charging is a relative stylish addition to your space. If we intend to place multiple tablets in multiple rooms, a clean consistent integration is important (to some people).

![Amazon Fire HD8+ Wireless charging Dock](https://exitcode0.net/images/fire-tablet-wireless-dock.jpg)

### Cons

#### Screen Resolution

The FireHD8 (and FireHD8 Plus) has an 8 inch screen with a resolution of 1280 x 800 (189 ppi). This works out to be a on the border between just enough and too little screen real estate for the official Home Assistant app, when running in full screen mode. You can optimise your dashboards with smaller card styles to fit more data and controls into a single viewport, but if you are looking to add an entire home's worth of controls, you will probably fail to do so.

I'd recommend looking into this HACS UI plugin for more dashboard design flexibility, in particular

There are other Fire tablets out there with higher resolution displays, but unfortunately, they don't support the wireless charging protocol and this means that there is no dock with a concealed power cable.

![The Amazon Fire Tablet Line up](https://exitcode0.net/images/fire-tablet-lineup.png)

#### Amazon Bloat and Ads

This is a very small problem if you never intend to lock the device or use it for anything other than a Home Assistant Dashboard display, but it is worth mentioning that should you ever wish to repurpose the device, you will need to pay a higher price to remove embedded adds on the tablet UI. This type of behaviour from Amazon clearly demonstrates how little they respect the user experience if they are willing to sell the same hardware with a different software image that is littered with adverts.

#### Wall Mounting

Like many android tablets, they were never really designed with a wall mounted use case in mind and you might struggle to find a tidy wall mounting solution; particular one where the power cables are concealed.

It would be fantastic to emulate control devices such as those offered in Control4 setups, at a fraction of the cost and using the Home Assistant as the basis of our automation. At the time of writing this, the best option for wall mounting is 3D printed enclosures.

There are some novel 3D printed solutions for wall mounting:

![Printable FireHD10 wall mount](https://exitcode0.net/images/fire-tablet-wall-mount.png)

- https://www.printables.com/model/383856-wall-mount-for-fire-tablet-hd-10-11th-gen-2021

## Alternatives

There are more cheap android tablets than you can shake a stick at, but your mileage may vary when it comes down to quality and suitability for this use case. Here are some alternatives that I have tested...

### Samsung Tab S6 Lite

Possibly not the cheapest option, but I have one and it would be remiss of me to not test it. The Tab S6 lite is a few year old now, but still available, offering a few benefits over the FireHD8. The build quality is superior and the screen resolution is higher, meaning that you can fit more dashboard items on screen without scrolling.

There is no wireless charging option for the Tab S6 lite, so unfortunately, you're going to need a USB-C cable sticking out the side. I opted for the cheap, yet stylish Ikea bamboo tablet stand, but unfortunately, I can't allow this visible wire to exist beyond my office desk. If you want to perpetuate the sunk cost fallacy, there are always adapters to hide the USB cable:

![180 Degree USB-C Adapter](https://exitcode0.net/images/180-degree-USB-C.png)

- 180 Degree USB-C Adapter - https://www.amazon.co.uk/gp/aw/d/B0BXCTRN7F

- Slim USB C to A Power Cable - https://www.etsy.com/listing/771904711/smart-home-mount-slim-usb-power-cable

### Raspberry Pi Official Touch Screen

If you are into home automation, there's is a strong probability that you have an unloved Raspberry Pi sitting in a draw somewhere. I'm here to tell you that it could be put to work as a home assistant dashboard host. The official touch screen and case allow us to build a dashboard device that we can customise to include sensors, using spare GPIO pins.

However, the huge downside is once again, a low screen resolution; extenuated by smaller overall dimensions. At least with the Pi, you are running in a full screen chromium instance, so you can define the zoom level, but you soon reach a point where it becomes unusable.

![Official Pi touchscreen case](https://exitcode0.net/images/pi-touchscreen.jpg)

I have taken the choice to resign this touch screen to my 3D print area, where it runs the OctoPrint service and displays a dashboard with minimal, 3D printing related information. To get the HA dashboard to open on Pi boot up, we can use the following LXDE autostart entry:

```bash
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```

At the end of the file, add the following, adjusting for your home assistant IP/hostname and dashboard URL:

```
...
#End of file
@chromium-browser --kiosk https://homeassistant:8123/lovelace/default_view?
```

You will need to sign on the first visit, but enabling `remember` on sign-in will cache the authentication details in the chromium browser. You can set the zoom level of the webpage using `Ctrl -` or `Ctrl+`, this will also be stored in chromium appdata.

An added benefit of the Pi is that you can SSH or VNC into it, take make changes like this quite easily... it's still just a Linux computer.

## Conclusion

I'm still on the lookout for a sensible alternative to my FireHD8, with fewer drawbacks and compromises, but still at an affordable price point. If you can find the tablet hardware that fits your needs, they make fantastic Home Assistant controllers and go a long way to helping your setup pass the family/friends test - where anyone can walk into your home (with an invite) and interact with your smart home.

As the HA project progresses, I expect that someone will see the need for a dedicated dashboard/control device which offers wall mounting or neat side table aesthetics, without compromises on software, screen quality and at an strong value price point. Until then, the Amazon FireHD8 tablet is a strong candidate for use as a Home Assistant controller or dashboard viewing device, with very few compromises or caveats for most consumers. If you have a large number of dashboard cards, perhaps take a look at something with a higher DPI.


---
Markdown version of https://exitcode0.net/posts/home-assistant-firehd8-dashboard/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
