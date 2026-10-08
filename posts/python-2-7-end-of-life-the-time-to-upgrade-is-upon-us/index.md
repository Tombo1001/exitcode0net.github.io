# Python 2.7 end-of-life - The time to upgrade is upon us

> Python 2.7 reached end of life on 1 January 2020 with no further security patches. How to plan the move to Python 3 for production scripts and dependencies.

Source: https://exitcode0.net/posts/python-2-7-end-of-life-the-time-to-upgrade-is-upon-us/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-01-05
Updated: 2026-10-08
Tags: linux, python, python2.7


> **Note:** Python 2.7 EOL has now passed. This article is kept for historical reference — if you're still running Python 2.7 in production, the migration guidance here remains relevant, but the urgency framing is dated.

1st January 2020 marked the official end of python 2.7 development, including feature support and security fixes.

![Python 2.7 end-of-life](https://i1.wp.com/exitcode0.net/wp-content/uploads/2019/11/Python-PNG-Clipart.png?resize=231%2C144)
Python 2.7 was over 9 years old in development years, making it the longest supported version to date. The code freeze is no in place, with the final release – 2.7.18 – scheduled for an April 2020 release. So yes there will be one more version to come down the tubes but it’s probably best that the new python project you were thinking of starting is written in 3.7 or above.

Migrating away from Python 2.7
------------------------------

One useful resource for tracking the impending demise of a Python version: [https://python-release-cycle.glitch.me/](https://python-release-cycle.glitch.me/ "https://python-release-cycle.glitch.me/")

You might be inclined to believe that any version of 3.X is good enough and better than running or developing in 2.7, however, there are a number of versions of 3 which have already reached the end of support – 3.2, 3.3, 3.4 – and others with only a few months on the clock – 3.5 and 3.6.

For those working on a Python project written in 2.7, here is the official porting guide: [https://docs.python.org/3/howto/pyporting.html](https://docs.python.org/3/howto/pyporting.html "https://docs.python.org/3/howto/pyporting.html")

For those who just wish to update the operating system’s primary python version used for running scripts and commands, I have composed a number of simple guides to make 3.7 your default version of python, including subsequent fixes for pip:

* Changing the default python version in Debian – [https://exitcode0.net/posts/changing-the-default-python-version-in-debian/](https://exitcode0.net/posts/changing-the-default-python-version-in-debian/ "https://exitcode0.net/posts/changing-the-default-python-version-in-debian/")
* Ubuntu 19.10 – How to upgrade python 2.7 to python 3.7 – [https://exitcode0.net/ubuntu-19-10-how-to-upgrade-python-2-7-to-python-3-7/](https://exitcode0.net/posts/ubuntu-19-10-how-to-upgrade-python-2-7-to-python-3-7/)
* Kali Linux – How to upgrade python 2.7 to python 3.7 – [https://exitcode0.net/posts/kali-linux-how-to-upgrade-python-2-7-to-python-3-7/](https://exitcode0.net/posts/kali-linux-how-to-upgrade-python-2-7-to-python-3-7/ "https://exitcode0.net/posts/kali-linux-how-to-upgrade-python-2-7-to-python-3-7/")

and if you are looking to move from 3.5 to 3.7:

* Debian 9 – How to upgrade python 3.5 to python 3.7 – [https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/](https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/ "https://exitcode0.net/posts/debian-9-how-to-upgrade-python-3-5-to-python-3-7/")

The final goodbye for Python 2.7?
---------------------------------

In conclusion, whilst the new year marks the official sunset for Python 2.7, the road to migration to newer versions has been famously messy and in some sense a failure. Don’t be surprised to encounter 2.7 projects for years to come; consider 2.7 an annoying requirement for so time to come.



---
Markdown version of https://exitcode0.net/posts/python-2-7-end-of-life-the-time-to-upgrade-is-upon-us/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
