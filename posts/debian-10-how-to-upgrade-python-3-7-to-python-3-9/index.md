# Debian 10 - How to upgrade python 3.7 to python 3.9

> Upgrade Python 3.7 to 3.9 on Debian 10 by compiling from source and switching with update-alternatives, including the pip and lsb_release fixes afterwards.

Source: https://exitcode0.net/posts/debian-10-how-to-upgrade-python-3-7-to-python-3-9/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-01-05
Updated: 2026-10-08
Tags: debian, linux, python



![](https://i2.wp.com/exitcode0.net/wp-content/uploads/2021/01/debian10-pytohn3.png?resize=217%2C206&ssl=1)
I have covered this a number of times in the past and the posts have proved popular and useful to many. So, here is my guide for updating to the latest version of Python 3 (3.9) on Debian 10 Buster. 

To clarify the purpose of this guide, Debian 10 ships with Python 2 (2.7) and Python 3 (3.7) installed at my time of writing. For those wishing to upgrade from python 3.7.X to 3.8.X or 3.9.X, this is the guide for you. If you are trying to configure python 3.7 as your default interpreter when you call ‘`python`‘, try this: [CHANGING THE DEFAULT PYTHON VERSION IN DEBIAN](https://exitcode0.net/posts/changing-the-default-python-version-in-debian/ "https://exitcode0.net/posts/changing-the-default-python-version-in-debian/"). This method involves using the ‘`update-alternatives`‘ command. We will be using a similar method in this guide, however this time we only do so to give 3.9.X a higher priority to 3.7.X, rather than uninstalling older versions.

**The basic premise is, install the version of python 3 desire, 3.9, then configure Debian to use python 3.7 at a higher priority to python 3.9.**

---

This guide is written to target those using Debian 10, but the same principles apply to older versions of Debian and other operating system based on Debian, such as Kali Linux.

---

The Debian 10, Python upgrade process
-------------------------------------

**Check your version**  
Step 1 is to check your current python version:

```
python3 -V
```

or

```
python3 --version
```

---

**Download the latest or desired version of python 3**  
Next, we need to download the latest version or desired version of python 3 from the python website. In my case, I selected 3.9.1. Once downloaded we need to extract the tar file.

```
wget https://www.python.org/ftp/python/3.9.1/Python-3.9.1.tar.xz
tar xf Python-3.9.1.tar.xz
cd Python-3.9.1
```

You can find all versions of Python here: [https://www.python.org/ftp/python](https://www.python.org/ftp/python "https://www.python.org/ftp/python")

![Debian 10 upgrade to python 3.9.1](https://i0.wp.com/exitcode0.net/wp-content/uploads/2021/01/image-3.png?resize=774%2C123&ssl=1)Python 3.9.1 downloading…

---

**Make and Install**  
Now that we have the files downloaded and extracted, it is time to compile them.

```
./configure
make
make install 
```

‘make’ commands can take quite some time to run, this is normal when compiling large programs from source…

**NOTE:** if you are running a minimal install of Debian 10, you might need to install a C compiler before you can run ‘`./configure`‘ and the tool for running ‘`make`‘ commands:

```
apt-get update -y
apt-get upgrade -y
apt-get install build-essential -y
apt-get install make -y
```

---

### **Switch to the new Python version**

Finally, after compiling the new version of python from source, we can now configure Debian to make it our default version of python3.

```
update-alternatives --install /usr/bin/python python3 /usr/local/bin/python3.9 10
```

The integer at the end of this command (10) sets the priority for the python version; the greater the integer, the higher the priority. At this point, we can rerun the previously used version commands and we should see that we now have Python 3.9.1 active.

---

**Fixing and Updating Pip**
---------------------------

It was at this point that I attempted to install some required addons using pip and discovered that the upgrade to Python 3.9.1 had broken a few things. These were the commands I used to resolve issues with lsb\_release and pip:

```
ln -s /usr/share/pyshared/lsb_release.py /usr/local/lib/python3.9/site-packages/lsb_release.py

pip3 install --upgrade pip
```

---

[![Exitcode0 Tip Jar](https://i0.wp.com/exitcode0.net/wp-content/uploads/2019/06/tipjar.png?resize=195%2C238&ssl=1)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url "https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url")
**If you have found this guide useful or it has solved a burning issue for you, please consider throw a coin in the tip jar to help this site stay active:**

![](https://i1.wp.com/exitcode0.net/wp-content/uploads/2019/06/QR-code.png?resize=105%2C105&ssl=1)
[https://www.paypal.com/cgi-bin/webscr?cmd=\_s-xclick&hosted\_button\_id=BTQD4GN8TTWJN&source=url](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url "https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=BTQD4GN8TTWJN&source=url")

### Some useful guides I found along the way:

* [https://jcutrer.com/linux/upgrade-python37-ubuntu1810](https://jcutrer.com/linux/upgrade-python37-ubuntu1810 "https://jcutrer.com/linux/upgrade-python37-ubuntu1810") – How to upgrade to python 3.7 on ubuntu 18.10.
* [https://tecadmin.net/install-python-3-7-on-ubuntu-linuxmint/](https://tecadmin.net/install-python-3-7-on-ubuntu-linuxmint/ "https://tecadmin.net/install-python-3-7-on-ubuntu-linuxmint/") – How to Install Python 3.7 on Ubuntu, Debian and LinuxMint.
* [https://linuxconfig.org/how-to-change-default-python-version-on-debian-9-stretch-linux](https://linuxconfig.org/how-to-change-default-python-version-on-debian-9-stretch-linux "https://linuxconfig.org/how-to-change-default-python-version-on-debian-9-stretch-linux") – How to change default python version on Debian 9 Stretch Linux

---

### Other Useful Debian tips:

* [https://exitcode0.net/posts/debian-9-running-a-python-script-at-startup/](https://exitcode0.net/posts/debian-9-running-a-python-script-at-startup/ "https://exitcode0.net/posts/debian-9-running-a-python-script-at-startup/") – Debian 9 – Running a python script at boot.



---
Markdown version of https://exitcode0.net/posts/debian-10-how-to-upgrade-python-3-7-to-python-3-9/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
