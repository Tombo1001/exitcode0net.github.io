# Kali Linux 2020.1 - No more root by default

> Kali Linux 2020.1 drops root as the default user. What the change means for penetration testing workflows and how sudo is configured in the new setup.

Source: https://exitcode0.net/posts/kali-linux-2020-1-no-more-root-by-default/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-01-28
Updated: 2026-10-08
Tags: kali-linux-2020.1, linux



[*](https://www.kali.org/ "https://www.kali.org/")*
Rejoice! Kali Linux 2020.1 is here! The first release of 2020 has arrived and this post will contain some updates and my initial thoughts on the fresh release.

Kali Linux 2020.1 features and changes
--------------------------------------

* **[Non-Root by default](https://www.kali.org/news/kali-default-non-root-user/ "https://www.kali.org/news/kali-default-non-root-user/")**
* Kali single installer image
* [Kali NetHunter Rootless](https://www.kali.org/docs/nethunter/nethunter-rootless/ "https://www.kali.org/docs/nethunter/nethunter-rootless/")
* Improvements to theme & kali-undercover
* New tools

Full Kali release notes as per their release blog post: [https://www.kali.org/releases/kali-linux-2020-1-release/](https://www.kali.org/releases/kali-linux-2020-1-release/ "https://www.kali.org/releases/kali-linux-2020-1-release/") 

Got root?
---------

Kali has installed with root as the default user since the dawn of its existence, Backtrack, WHAX and Whoppix, but finally, the devs have decided that it time to bring Kali in line with Debian on this one and have a default user which is no longer root.

Many might see this as an unnecessary move as the OS is not intended or condoned for use as a daily driver (even 2020.1) – although some people ignore this anyway so not your browser as root is always a good thing.

>  While we don’t encourage people to run Kali as their day to day operating system, over the last few years more and more users have started to do so *(even if they are not using it to do penetration testing full time)*, including some members of the Kali development team. When people do so, they obviously don’t run as default root user. With this usage over time, there is the obvious conclusion that default root user is no longer necessary and Kali will be better off moving to a more traditional security model. – 
> 
>  [https://www.kali.org/news/kali-default-non-root-user/](https://www.kali.org/news/kali-default-non-root-user/ "https://www.kali.org/news/kali-default-non-root-user/") 

The maintainers have discovered that a large number of tools no longer require root (with the exception of NMAP as an example) to run, so it made sense to align with a more traditional security model.

First Impressions
-----------------

I grabbed the 64bit ISO and spun up a VM – the host is running KVM, but this makes no difference.

*Shock horror no more root by default – text-based install method.*
Pro-tip for first time Kali users… don’t give a name by which you could be identified.

I like that you can now select the [metapackages](https://www.kali.org/docs/general-use/metapackages/ "https://www.kali.org/docs/general-use/metapackages/") which you install and the Desktop environment (unless you use the automated installer method). So you can make your installation as fat or lightweight as you need it.

Once you’ve completed your install, everything else feels more or less the same. The maintainers are committed to binning as many python2 dependant tools as they can as technically they are [no longer supported](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/). We have some new tools, which I excited to test at a given opportunity:

* cloud-enum
* emailharvester
* phpggc
* sherlock
* splinter.

Upgrade from Kali 1904.1 or earlier
-----------------------------------

If you already have an installation and you don’t want to nuke and pave, then it is also possible to upgrade.

```
cat <<EOF | sudo tee /etc/apt/sources.list
deb http://http.kali.org/kali kali-rolling main non-free contrib
EOF
```

```
apt update -y && apt full-upgrade -y
```

Once that is complete, go for a quick reboot then check that you are now running the latest and greatest:

```
lsb_release -a
```

---

Other useful Kali posts:
------------------------

* Kali Linux – How to upgrade python 2.7 to python 3.7 [https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ "https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/")
* How to make a VirtualBox VM the same resolution as host – [https://exitcode0.net/how-to-make-a-virtualbox-vm-the-same-resolution-as-host/](https://exitcode0.net/posts/how-to-make-a-virtualbox-vm-the-same-resolution-as-host/)
* Clipboard and Shared Folders on Kali Linux with VirtualBox – [https://exitcode0.net/clipboard-and-shared-folders-on-kali-linux-with-virtualbox/](https://exitcode0.net/posts/clipboard-and-shared-folders-on-kali-linux-with-virtualbox/)
* AWUS036AC (rt8812au) driver setup in Kali Linux – [https://exitcode0.net/posts/awus036ac-rt8812au-driver-setup-in-kali-linux/](https://exitcode0.net/posts/awus036ac-rt8812au-driver-setup-in-kali-linux/ "https://exitcode0.net/posts/awus036ac-rt8812au-driver-setup-in-kali-linux/") (pending validation with 2020.1)



---
Markdown version of https://exitcode0.net/posts/kali-linux-2020-1-no-more-root-by-default/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
