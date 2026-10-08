# Prevent Screensavers and Lock-screens with Powershell

> Stop the Windows screensaver and lock screen kicking in with a PowerShell SendKeys loop. Keeps a PC awake with a timed keypress and no Group Policy changes.

Source: https://exitcode0.net/posts/prevent-screensavers-and-lock-screens-with-powershell/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-03-26
Updated: 2026-10-08
Tags: code, powershell, windows



There might be a number of reasons to want to prevent your windows screensavers and lock-screens from engaging and in some cases (no doubt yours if you have hit this article from a search engine) local policy on the machine preventing you from changing these settings.

Powershell lets us work around this problem and prevent the machine from locking or activating a screensaver. 

Prevent Screensavers and Lock-screens
-------------------------------------

The bulk of the code below has been lifted from this great write up: [https://dmitrysotnikov.wordpress.com/2009/06/29/prevent-desktop-lock-or-screensaver-with-powershell/](https://dmitrysotnikov.wordpress.com/2009/06/29/prevent-desktop-lock-or-screensaver-with-powershell/ "https://dmitrysotnikov.wordpress.com/2009/06/29/prevent-desktop-lock-or-screensaver-with-powershell/") I just took things one step further by adding an infinite loop. My version of this code will run indefinitely.

*Affectionately named: **PScaffeine***
The following code sends a relatively harmless (for most cases – change where appropriate) key to the system emulating a keypress on a human interface device:

```
#powershell caffeine

param($minutes: 29)
$myshell: New-Object -com "WScript.Shell"
Write-Host "Pouring out fresh coffee..."
while($true){
    for ($i: 0; $i -lt $minutes; $i++) {
        Start-Sleep -s 60
        $myshell.sendkeys('{NUMLOCK}')
        $myshell.sendkeys('{NUMLOCK}')
    }
    Write-Host "*Takes another sip*"
}
Write-Host "Caffeine has worn off..."
```

Change the **$minutes** parameter to 1 minute less than your systems lock/screensaver start initiation time.

Combine this with a scheduled task if you want this to run on startup or system login.

---

More useful code snippets
-------------------------

* ‘Copy, Paste’ With Powershell Sendkeys – [https://exitcode0.net/posts/copy-paste-with-powershell-sendkeys/](https://exitcode0.net/posts/copy-paste-with-powershell-sendkeys/ "https://exitcode0.net/posts/copy-paste-with-powershell-sendkeys/")
* Powershell – Checking the Language Mode – [https://exitcode0.net/posts/powershell-checking-the-language-mode/](https://exitcode0.net/posts/powershell-checking-the-language-mode/ "https://exitcode0.net/posts/powershell-checking-the-language-mode/")
* Modifying Windows shortcuts is Powershell – [https://exitcode0.net/modifying-windows-shortcuts-is-powershell/](https://exitcode0.net/posts/modifying-windows-shortcuts-is-powershell/)
* All other codes samples – https://exitcode0.net/code-samples/



---
Markdown version of https://exitcode0.net/posts/prevent-screensavers-and-lock-screens-with-powershell/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
