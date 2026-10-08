# Copy Paste in vSphere before installing VMware tools

> Send text into a vSphere VM before VMware Tools is installed using AutoIT keystrokes, getting round the clipboard limit during fresh OS installs.

Source: https://exitcode0.net/posts/copy-paste-in-vsphere-before-installing-vmware-tools/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-05-18
Updated: 2026-10-08
Tags: autoit, code, vsphere



I’ve spent enough time building VMs in vSphere to know that the first few minutes between starting the VM and getting VMware Tools installed is agonising. Similar to my ‘[Copy Paste With Powershell Sendkeys](https://exitcode0.net/posts/copy-paste-with-powershell-sendkeys/ "https://exitcode0.net/posts/copy-paste-with-powershell-sendkeys/")‘ script, I have another tool written in AutoIT which sends defined keyboard strokes to the system. So this is not _exactly_ ‘copy paste in vSphere’, but it achieves the same goal.

If you actually want to enable clipboard sync between host and VM, you can follow this guide here: [Enable Copy and Paste Operations Between the Guest Operating System and Remote Console](https://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vsphere.security.doc_50%2FGUID-367D02C1-B71F-4AC3-AA05-85033136A667.html "https://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vsphere.security.doc_50%2FGUID-367D02C1-B71F-4AC3-AA05-85033136A667.html"). This is going to need VMware Tools installed on the host though.

## Copy Paste in vSphere with AutoIT

Yes this tool is written in AutoIT, apparently my favorite script kiddie language from the 201X years, so Windows only on this one – sorry. Quite a simple bit of code, we prompt for the text we want to put on the ‘clipboard’ announce the waiting period before sending the keystrokes (allowing the user time to place the cursor in the VM) and then fire off the keystrokes.

```
#include <MsgBoxConstants.au3>

Local $sAnswer: InputBox("Send Keys", "Enter the text you want to send:", "", "", _
             - 1, -1, 0, 0)
Local $iTimeout: 7
Local $iSecUntil: 10
MsgBox($MB_SYSTEMMODAL, "Send Keys", "Your keys will be sent in " & $iSecUntil & " seconds or select the OK button to start a 3 second countdown.", $iTimeout)
Sleep(3000)
Send($sAnswer)
Exit
```

### Code Enhancements?

This tool has got me out of a number of binds where the clipboard has failed me – usually some form of remote session. I don’t trust myself to type things accurately in high-pressure situations, especially when character obfuscation is in play.

At the moment this is a one-shot executable; you run it once and then the process terminates. One handy feature/enhancement would be to have it run perpetually and listen for a hotkey (definable) on the host triggering the main function of the code to be called.

That being said, there are **lots** of other things I am working on right now, and digging back into an AutoIT project from 2016 is not very high on the priorities list. Perhaps one for a rainy day…

---

## More AutoIT ramblings:

- Automatically pause Spotify with the 3CX client – [https://exitcode0.net/posts/automatically-pause-spotify-with-the-3cx-client/](https://exitcode0.net/posts/automatically-pause-spotify-with-the-3cx-client/ "https://exitcode0.net/posts/automatically-pause-spotify-with-the-3cx-client/")


---
Markdown version of https://exitcode0.net/posts/copy-paste-in-vsphere-before-installing-vmware-tools/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
