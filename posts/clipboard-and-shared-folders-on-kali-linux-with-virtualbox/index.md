# Clipboard and Shared Folders on Kali Linux with VirtualBox

> Enable clipboard sync and shared folders for Kali Linux in VirtualBox by installing virtualbox-guest-x11.

Source: https://exitcode0.net/posts/clipboard-and-shared-folders-on-kali-linux-with-virtualbox/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-11-21
Updated: 2026-10-08
Tags: kali, linux, tidbits, virtualbox



I spent more time that care to admit trying to setup a shared folder between my windows host and Kali VirtualBox VM. So hopefully the Google algos pick this one up and save you the time trying to find the right packages to fix this a clipboard sync.

```
apt-get update
apt-get install -y virtualbox-guest-x11
```

Go for a quick reboot once the above commands are complete and you should have clipboard sync (text) and shared folders should mount successfully.

[![](https://i.ytimg.com/vi/GD6qtc2_AQA/maxresdefault.jpg)](https://www.youtube.com/watch?v=GD6qtc2_AQA "https://www.youtube.com/watch?v=GD6qtc2_AQA")
More Linux Tidbits:
-------------------

* Changing the default python version in Debian [https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ "https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/")
* Debian 9 – Running a python script at boot [https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ "https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/")
* Changing the default python version in Debian [https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ "https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/")



---
Markdown version of https://exitcode0.net/posts/clipboard-and-shared-folders-on-kali-linux-with-virtualbox/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
