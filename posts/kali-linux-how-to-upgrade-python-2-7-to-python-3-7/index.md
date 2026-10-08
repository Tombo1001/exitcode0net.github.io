# Kali Linux - How to upgrade python 2.7 to python 3.7

> Make Python 3.7 the default on Kali Linux in place of Python 2.7 by setting interpreter priority with update-alternatives.

Source: https://exitcode0.net/posts/kali-linux-how-to-upgrade-python-2-7-to-python-3-7/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-11-26
Updated: 2026-10-08
Tags: python, kali-linux, linux



> **Note:** Python 2.7 reached end-of-life in January 2020 and Python 3.7 followed in June 2023. On current Kali Linux, Python 3 is the default — `python3` is available out of the box and `python` typically symlinks to it already. This article is kept for historical reference and for anyone on older installations.

I have covered changing the default version of python in Debian, however for those looking to Google for a quick fix on Kali, I hope that this reaches you well.

This was tested on a completely fresh install of Kali Linux with no other alterations made prior.

**The basic premise is to configure Kali to use python 3.7 at a higher priority to python 2.7 or any other version installed on the system.**

---

**Check your python version**
-----------------------------

Step 1 is to check your current python version:

```
python -V
```

or

```
python --version
```

**Kali default output:**  
Python 2.7.17

Set your Python Default
-----------------------

Now it is time configure the priority for the versions of python that we have installed, 2.7 and 3.5/7. You can list all of the available alternatives installed by running: 

```
ls /usr/bin/python*
```

**To set your version priorities, with 3.7 being the high priority:**

```
update-alternatives --install /usr/bin/python python /usr/bin/python2.7 1
```

```
update-alternatives --install /usr/bin/python python /usr/bin/python3.7 2
```

**We have just set 3.7 (2) to have a priority great than 2.7 (1).** Now when we list the python priorities we see see 3.7 is higher that 2.7:

```
update-alternatives --config python
```

![](https://i0.wp.com/exitcode0.net/wp-content/uploads/2019/11/python-versions3.png?resize=689%2C184&ssl=1)
This is also a great way to easily switch those priorities around once they have been set.

---

### Check you default version, again…

```
python -V
```

Now this command should return the default which you configured above.

![](https://i0.wp.com/exitcode0.net/wp-content/uploads/2019/11/python-versions4.png?resize=191%2C43&ssl=1)

---

### Other Useful Python tips:

* [https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ "https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/") – Debian 9 – How to upgrade python 3.5 to python 3.7
* [https://exitcode0.net/posts/changing-the-default-python-version-in-debian/](https://exitcode0.net/posts/changing-the-default-python-version-in-debian/ "https://exitcode0.net/posts/changing-the-default-python-version-in-debian/") – Changing the default python version in Debian

---

[![Exitcode0 Tip Jar](https://i0.wp.com/exitcode0.net/wp-content/uploads/2019/06/tipjar.png?resize=195%2C238&ssl=1)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url "https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url")
**If you have found this guide useful or it has solved a burning issue for you, please consider throw a coin in the tip jar to help this site stay active:**

![](https://i1.wp.com/exitcode0.net/wp-content/uploads/2019/06/QR-code.png?resize=105%2C105&ssl=1)
[https://www.paypal.com/cgi-bin/webscr?cmd=\_s-xclick&hosted\_button\_id=BTQD4GN8TTWJN&source=url](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url "https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url")



---
Markdown version of https://exitcode0.net/posts/kali-linux-how-to-upgrade-python-2-7-to-python-3-7/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
