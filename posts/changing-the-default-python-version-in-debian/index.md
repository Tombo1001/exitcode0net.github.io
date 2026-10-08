# Changing the default python version in Debian

> Change the default Python version on Debian from 2.7 to 3.x with update-alternatives and switch between installed versions.

Source: https://exitcode0.net/posts/changing-the-default-python-version-in-debian/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-06-17
Updated: 2026-10-08
Tags: python, debian, linux, update-alternatives



In a previous [post](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ "https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/"), I explained how to upgrade from Python3.5 to Python3.7, however is is still the case in most fresh Debian installs that the default python version is 2.7. This post is going to show you the simple steps you need to take when changing the default python version in Debian. 

This guide is based on a Debian 9 installation, but work for multiple releases. 

---

### Check you default version

First we need to check our current default version of python. This is the version of python that you Debian OS will try to execute python scripts with unless otherwise specified.

```
python -V
```

or

```
python --version
```

My fresh install of Debian 9 told me that my default was: **2.7.17.** Old and unwanted.

---

### Install Python 3

This is **optional** and only applies for those who don’t already have python 3 installed:

```
apt install python3
```

---

### Set your Python Default

Now it is time configure the priority for the versions of python that we have installed, 2.7 and 3.5/7. You can list all of the available alternatives installed by running: 

```
ls /usr/bin/python*
```

To set your version priorities, with 3.5 being the high priority:

```
update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
```

```
update-alternatives --install /usr/bin/python python /usr/bin/python3.5 2
```

**We have just set 3.5 to have a priority great than 2.7.** Now when we list the python priorities we see see 3.5 is higher that 2.7:

```
update-alternatives --config python
```

This is also a great way to easily switch those priorities around once they have been set.

---

### Check you default version, again…

```
python -V
```

Now this command should return the default which you configured above.

---

[![](https://i0.wp.com/exitcode0.net/wp-content/uploads/2019/06/tipjar.png?resize=156%2C190)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url "https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url")
**If you have found this guide useful or it has solved a burning issue for you, please consider throw a coin in the tip jar to help this site stay active:**

![](https://i1.wp.com/exitcode0.net/wp-content/uploads/2019/06/QR-code.png?resize=90%2C90)
[https://www.paypal.com/cgi-bin/webscr?cmd=\_s-xclick&hosted\_button\_id=BTQD4GN8TTWJN&source=url](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url "https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url")

---



---
Markdown version of https://exitcode0.net/posts/changing-the-default-python-version-in-debian/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
