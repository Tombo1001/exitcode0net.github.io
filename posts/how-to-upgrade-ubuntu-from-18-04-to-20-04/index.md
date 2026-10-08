# How to upgrade Ubuntu from 18.04 to 20.04

> Upgrade Ubuntu 18.04 LTS to 20.04 LTS in-place using do-release-upgrade. Preserve installed packages and configuration during a major release upgrade.

Source: https://exitcode0.net/posts/how-to-upgrade-ubuntu-from-18-04-to-20-04/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-04-21
Updated: 2026-10-08
Tags: 20.04-lts, linux, ubuntu, upgrade-ubuntu


> **Note:** Ubuntu 18.04 LTS reached end-of-life in April 2023, and Ubuntu 20.04 LTS reaches EOL in April 2025. If you're starting fresh, install Ubuntu 24.04 LTS. The `do-release-upgrade` process shown here is still valid for sequential LTS upgrades.

[Ubuntu 20.04](https://releases.ubuntu.com/20.04/ "https://releases.ubuntu.com/20.04/") LTS full release is almost upon, with a scheduled release date of [April 23rd 2020](https://www.omgubuntu.co.uk/2019/10/ubuntu-20-04-release-features "https://www.omgubuntu.co.uk/2019/10/ubuntu-20-04-release-features") (2 days as of writing this). It is packed with lots of great new features and upgrades and the Linux community is very excited to get a hold of it. If you are looking to upgrade ubuntu, there is a quick guide down below.

If like me you are already running Ubuntu 18.04 LTS then you might be interested to learn how to upgrade your current version of the distro to 20.04 LTS.

Fourtnately it is relatively simple and requires a few terminal commands.

Prerequisites
-------------

It is really important to **take a backup of any important data** and only consider the upgrade in the first place if you are happy that your current software suite is supported in the new version of the OS. If you are not sure about this then I would advise building 20.04 in a VM for testing before committing to the upgrade.

Starting the upgrade
--------------------

It would be a good idea to make sure that all your current packages in Ubuntu 18.04 are up to date before jumping to 20.04:

```
sudo apt-get update && sudo apt-get upgrade && apt-get dist-upgrade
```

follow that up with a reboot:

```
sudo reboot now
```

Now we can install the Ubuntu release upgrade tool:

```
sudo apt install update-manager-core
```

**Now it it time to upgrade to the latest LTS:**

```
sudo do-release-upgrade
```

**The uprade will take quite a bit of time and require some minor intervention for confirmation during the upgrade.** Once it is complete, reboot your system and move onto the final stage.

It is important to note that if you are trying to upgrade to 20.04 before the final release date, you will get the following message and tip:

```
Checking for a new Ubuntu release
There is no development version of an LTS available.
To upgrade to the latest non-LTS develoment release 
set Prompt=normal in /etc/update-manager/release-upgrades
```

Changing ‘Prompt’ to normal from LTS will allow your system to jump to developer builds when running the ‘do-release-upgrade’. Once again, if this is a mission-critical system, consider your decision to upgrade to development versions of your OS carefully.

If you play fast and loose, you can always ugrade to a **developer version**:

```
sudo do-release-upgrade -d
```

Final step
----------

Now it is time to verify the version of Ubuntu you are running:

```
lsb_release -a
```

If you want to share your version with the world, consider using neofetch as a way to make your terminal flex that bit more exciting:

```
sudo apt install neofetch
```

```
neofetch
```

*Feel free to post your neofect flex in the comments below!*

---

More useful Ubuntu tips and tricks
----------------------------------

* Ubuntu 19.10 – How to upgrade python 2.7 to python 3.7 – [http://Ubuntu 19.10 – How to upgrade python 2.7 to python 3.7](http://Ubuntu 19.10 – How to upgrade python 2.7 to python 3.7 "http://Ubuntu 19.10 – How to upgrade python 2.7 to python 3.7")  
(this will probably work on 20.04 too!)



---
Markdown version of https://exitcode0.net/posts/how-to-upgrade-ubuntu-from-18-04-to-20-04/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
