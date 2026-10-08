# Debian 9 - Running a python script at boot

> Run a Python script automatically at boot on Debian with crontab @reboot, pass parameters, redirect output and check it started with journalctl.

Source: https://exitcode0.net/posts/debian-9-running-a-python-script-at-startup/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-02-21
Updated: 2026-10-08
Tags: crontab, debian, linux, scripting, startup


> **Note:** Debian 9 is end-of-life, but the `crontab @reboot` technique shown here works on all modern Debian, Ubuntu, and Raspberry Pi OS releases.

There are a number of ways that you can run a python or bash script at system start but I am about to show you possibly the easiest way, using crontab.

First you need to make that your script is executable and can run unattended. Then you need to login as the user your wish to execute the script as and edit the crontab file:

```
crontab -e  

```

If this is your first time editing the crontab file, you might be asked which editor you wish to use – I chose nano.  
Once in the crontab file add a new line at the bottom of the file; it should look something like this

```
@reboot /the/path/to/your/script/./your_script.py [any other parameters] > /path/of/your/log.txt
```

Alternatively, if you don’t wish to record any output from your script you can send the output to /dev/null. Now it’s time to reboot your Debian instance and your script will be live.  



---
Markdown version of https://exitcode0.net/posts/debian-9-running-a-python-script-at-startup/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
