# Kasa home automation with IFTTT and python webhooks

> Automate TP-Link Kasa smart plugs with IFTTT webhooks fired from Python scripts, so any application can control smart home devices with an HTTP request.

Source: https://exitcode0.net/posts/kasa-home-automation-with-ifttt-and-python-webhooks/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-12-04
Updated: 2026-10-08
Tags: home-automation, ifttt, python, webhooks



They say that the long tail page titles are best for SEO, well this one certainly should be up there with the best of them. In this post, I am going to explain how easy it is to control Kasa a HS100 / HS110 with IFTTT and python webhooks. Using this basic code, you expand into building a very powerful home automation system which fits your needs and take your Kasa home automation to the next level.

Kasa HS100 / HS110 Introduction
-------------------------------

*HS100 (UK Version)*
Kasa is the home automation/ smart device brand offered by TP-Link. Their products office a great deal of functionality out of the box, allowing you to control a device smart plug/bulb from your phone without the need for a controller – such as the Philips hue systems. This is achieved using a cloud-hosted API model; every device communicates to/from the Kasa cloud-hosted backend. Every Kasa device is registered to your account.

---

IFTTT Webhooks Introduction
---------------------------

If This Then That (IFTTT) – is a free web-based service to create chains of simple conditional statements, called applets. An applet is triggered by changes that occur within other web services such as Gmail, Facebook, Telegram, Instagram, or Pinterest.

You must first activate the webhook service, then you can acquire your webhook URL.

### Find you webhook URL

[https://ifttt.com/maker\_webhooks/settings](https://ifttt.com/maker_webhooks/settings "https://ifttt.com/maker_webhooks/settings")

### Build a webhook trigger

Next, you need to build a trigger. In the case of a HS100/HS110, it would be sensible to build two triggers – on and off triggers. As you will see later in the code, the Event Name value is passed to IFTTT in our webhook URL; **make this unique to each webhook trigger.**

---

Kasa home automation with python
--------------------------------

Finally, it is time to write to some code – at last. This part is arguably less fiddly than the prerequisite setup for this project. As previously mentioned, this code contains my unique IFTTT trigger URL so I will be obfuscating some sensitive data to prevent everyone from turning my Kasa plug on and off.

```
#!/usr/bin/python

import requests

def funcHSTon ():
 requests.post("https://maker.ifttt.com/trigger/temp_low/with/key/<your webhook key>")

def funcHSToff ():
 requests.post("https://maker.ifttt.com/trigger/temp_high/with/key/<your webhook key>")

#funcHSToff()
funcHSTon()
```

I have split both webhook triggers into two functions – funcHSTon & funcHSToff – and I have commented out the function that I don’t want to run during testing.  
**Remember to replace ‘<your webhook key>’ with your own key!**

*Best of luck with the never ending story of home automation with python!*

---

Other useful posts:
-------------------

* **Ubuntu 19.10 – How to upgrade python 2.7 to python 3.7** – [https://exitcode0.net/ubuntu-19-10-how-to-upgrade-python-2-7-to-python-3-7/](https://exitcode0.net/posts/ubuntu-19-10-how-to-upgrade-python-2-7-to-python-3-7/)
* **Debian 9 – How to upgrade python 3.5 to python 3.7** – [https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ "https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/")



---
Markdown version of https://exitcode0.net/posts/kasa-home-automation-with-ifttt-and-python-webhooks/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
