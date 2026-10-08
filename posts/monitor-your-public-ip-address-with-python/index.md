# Monitor your public IP address with Python

> Monitor your public IP address in Python with OpenDNS lookups. Detect VPN drops and confirm IP masking with a polling loop and alerts.

Source: https://exitcode0.net/posts/monitor-your-public-ip-address-with-python/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-12-13
Updated: 2026-10-08
Tags: python, networking, ip-address, monitoring



This is my lean and effective way to monitor your public IP address with Python, specifically Python3. The script – code and GitHub link below – runs on a continuous loop, which I covered in a previous post: [here](https://exitcode0.net/posts/how-to-python-infinite-loops-with-while-true/ "https://exitcode0.net/posts/how-to-python-infinite-loops-with-while-true/").

**NOTE:**  this code is reliant on the `dig` system command – so this essentially only work on a unix based system. I was running this on a Debian install.

```
import os
import subprocess
import re
import time

print("start monitor vpn monitor check")

expected_IP: "0.0.0.0" # ENTER YOUR EXPECTED PUBLIC IPv4 ADDRESS HERE

current_IP: subprocess.check_output("dig +short myip.opendns.com @resolver1.opendns.com", shell=True)

try:
    while True:
        if expected_IP in str(current_IP):
            print("IPs Match - Things are normal")
        else:
            print("Current IP: " + str(current_IP) + "\nIP NOT AS EXPECTED!")
            #Code to complete actions called here
        time.sleep(60) # Change this timer to reduce/increase time between checks (seconds)
except KeyboardInterrupt:
    print("\nHard Exit Initiated. Goodbye!")
```

[https://github.com/Tombo1001/PyIPmonitor](https://github.com/Tombo1001/PyIPmonitor "https://github.com/Tombo1001/PyIPmonitor")

Alternative uses
----------------

There are a number of use cases for this code, mine was checking if my computer is still connected to a VPN. Monitor my public IP address meant that I could tell if the VPN had disconnected.

*Checking my IP matches my VPN endpoint IP*
The only thing that was changed here from the current repository code is the print statements – the output just matches the context of the code’s use.

---

More python snippets
--------------------

Here are some links to some of my other python code samples:

* My python code samples – https://exitcode0.net/code-samples/#python
* Python3 SSH with paramiko – [https://exitcode0.net/posts/python-3-ssh-with-paramiko/](https://exitcode0.net/posts/python-3-ssh-with-paramiko/ "https://exitcode0.net/posts/python-3-ssh-with-paramiko/")



---
Markdown version of https://exitcode0.net/posts/monitor-your-public-ip-address-with-python/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
