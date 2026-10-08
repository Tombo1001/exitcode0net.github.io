# Adding WMIC command to the Windows path

> Add C:\Windows\System32\wbem to the Windows PATH to restore wmic and fix 'wmic is not recognized' errors on Windows 10 after updates or PATH changes.

Source: https://exitcode0.net/posts/adding-wmic-command-to-the-windows-path/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-05-30
Updated: 2026-10-08
Tags: path-variables, windows, wmic


> **Note:** WMIC is deprecated as of Windows 11 22H2. For equivalent functionality use `Get-WmiObject` or `Get-CimInstance` in PowerShell instead.

**wmic is not recognized as an internal or external command** – I was quite shocked to find that a command I use on a very regular basis was not working on a fresh installation of Windows 10 1909 on my trust old ThinkPad.

I don’t want to spend hours trying to find out why this was not correct in my system path, but instead, I fixed it and spent the time sharing how to fix the problem.

Adding WBEM to the Windows path
-------------------------------

Adding the **WMIC** command to the Windows path is a very simple process (administrator privileges required) and is completed as follows:

Find the **Advanced System Settings** item in your start menu search:

*View advanced system settings*
Open the **Advanced** tab and select the **Environment Variables** button:

*On the advanced tab, open Environment Variables*
Under **System variables**, highlight the **Path** variable and select the edit button:

*Select the Path system variable and Edit…*
Now we need to added the following line to our Path variable:

```
C:\Windows\System32\wbem\
```

We can do that as follows, ensuring to include the trailing backslash as this is a folder:

Once you have clicked **OK**, closing all the windows we have opened so far, you need to **REBOOT** your computer to apply the change.

Once you have completed the reboot, open up the command prompt, and test the **WMIC** command:

*Learn more about this WMIC command from this link.*



---
Markdown version of https://exitcode0.net/posts/adding-wmic-command-to-the-windows-path/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
