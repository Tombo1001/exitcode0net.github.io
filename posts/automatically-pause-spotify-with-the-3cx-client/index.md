# Automatically pause Spotify with the 3CX client

> Automatically pause Spotify when a 3CX VoIP call comes in, using a small AutoIT script that sits in the tray.

Source: https://exitcode0.net/posts/automatically-pause-spotify-with-the-3cx-client/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-05-16
Updated: 2026-10-08
Tags: 3cx, autoit, spotify


> **Note:** This article covers an older 3CX CTI integration method from the Windows 10 era. The 3CX client has changed significantly since this was written — the core AutoIT approach may still work but exact steps may differ.

This is a really old script that I just found in my archive, but it still works all the same. The goal was to write a small script that would automatically pause Spotify with the [3CX client](https://www.3cx.com/voip/softphone/ "https://www.3cx.com/voip/softphone/") when there was an inbound call.

The script is written in AutoIT, meaning that it is a Windows-only solution. The Windows 3CX CTI client allows you to run a particular executable when an inbound call is received, i.e. when the phone rings. I wanted to pause the manic euro beats so that I could answer my phone and here the caller and uphold professionalism.

The AutoIT Code
---------------

```
; *****************************************************************************
; Auto Pause Spotify
; Pauses spotify when called
; Author: Tom Cocking - www.tomcocking.com
; *****************************************************************************
#include <MsgBoxConstants.au3>
SpotifyPause()

Func SpotifyPause()
; Retrieve the handle of the Spotify window using the classname of SpotifyMainWindow.
    Local $hWnd: WinGetHandle("[CLASS:SpotifyMainWindow]")
	WinActivate($hWnd)
	Local $sText: WinGetTitle("[ACTIVE]")

	If $sText: "Spotify" Then
	   Exit
	Else
	   Send("{SPACE}")
	   Exit
	EndIf

    If @error Then
        ConsoleWrite("An error occurred when trying to retrieve the window handle of Spotify")
        Exit
    EndIf
EndFunc
```

As you can see, the code is very straight forward and to the point; we are asking 3CX to run the compiled exe file every time there is an inbound call, fast and reliable code is essential. It is also really important that you conclude you executable with an exit statement to prevent stale processes lingering on your system. 

One subtle thing that this script does is check if the Spotify instance has a song/artist data in the window title (which would signal that music was playing) before trying to activate the window and pause playback. One thing to mention – it would be a lot easier if there was a global hotkey to play/pause music, particularly one that Spotify adhered to. This would make this script 3 lines long (not counting comments).

**Editors Note:** I fully intend on uploading this to a public Github repository so that you can clone the repo, rather than just copying/pasting.

### Configure the 3CX softphone client

Once you have compiled your script into an executable file, you can find the setting in the 3CX client to run an exe upon an incoming call. So when you have a call from the boss, you can rest easy knowing that your secret adoration for Taylor Swift remains undisclosed.

*please recognise my sarcasm*

---

You can find more useful AutoIT snippets over on my Code Samples Page.



---
Markdown version of https://exitcode0.net/posts/automatically-pause-spotify-with-the-3cx-client/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
