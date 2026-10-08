# Debian 9 - How to upgrade python 3.5 to python 3.7

> Install Python 3.7 on Debian 9 from source and make it the default in place of 3.5 with update-alternatives, plus the pip fixes afterwards.

Source: https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-02-19
Updated: 2026-10-08
Tags: debian, linux, python


> **Note:** Debian 9 Stretch reached end-of-life in June 2022. The compile-from-source method shown here is still valid on any current Debian or Ubuntu system — just substitute the Python version you need.

![Python upgrade 3.5 to 3.7](https://i0.wp.com/exitcode0.net/wp-content/uploads/2019/06/Python-Upgrade.png?resize=208%2C208&ssl=1)
I recently spent 30 minutes figuring out how to upgrade to python 3.7 and subsequently pip version on a fresh install of Debian 9. I ran into a number of issues doing this so I though that I would put this quick post together to make this a little bit less complicated for anyone else trying to this.

This post relates specifically to python version 3.7.3. Although the generic commands should also apply to earlier versions, but your milage may vary.

**The basic premise is, upgrade your version of python 3 to your desired version – 3.7.3 in this instance – then configure Debian to use python 3.7 at a higher priority to python 3.5.**

---

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
Next we need to download the latest version or desired version of python 3 from the python website. In my case I selected 3.7.3. Once downloaded we need to extract the tar file.

```
wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tar.xz
tar xf Python-3.7.3.tar.xz
cd ./Python-3.7.3 
```

---

**Make and Install**  
Now that we have the files downloaded and extracted, it is time to compile them.

```
./configure  
make  
make install  

```

---

**Switch to the new version**  
After compile the new version of python from source, we can now configure Debian to make it our default version of python3.

```
update-alternatives --install /usr/bin/python python /usr/local/bin/python3.7 10
```

The integer at the end of this command (10) sets the priority for the python version; the greater the integer, the higher the priority. At this point we can rerun the previously used version commands and we should see that we now have 3.7.3 active.  

---

**Fixing and Updating Pip**  
It was at this point that I attempted to install some required addons using pip and discovered that the upgrade to python 3.7.3 had broken a few things. These were the commands I used to resolved issues with lsb\_release and pip:

```
ln -s /usr/share/pyshared/lsb_release.py /usr/local/lib/python3.7/site-packages/lsb_release.py

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
Markdown version of https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
