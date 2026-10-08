# Customising the PowerShell prompt to show date and time

> Customise the PowerShell prompt to show the date, time and history ID, with coloured output and a shortened path, using $PROFILE in Windows Terminal.

Source: https://exitcode0.net/posts/customising-the-powershell-prompt-to-show-date-and-time/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-07-24
Updated: 2026-10-08
Tags: powershell, profile, windows-terminal



Depending on your reason for using the PowerShell prompt at it, it might be useful to include data such as timestamps in your recorded output, to enable better auditing. This post will walk you through the simple steps needed for customising the PowerShell prompt, including how to show the current date and time.

What exactly do I mean by the PowerShell prompt and what does the typical prompt look like by default? Well let me dial it back to default and show you:

*Windows 10 21H1 default PowerShell prompt – Windows Terminal*
The default Windows 10 (21H1) PowerShell prompt simply includes `PS` (PowerShell), followed by the current working directory; in my case: `C:\Users\admin`. In circumstances where we are looking to record our terminal sessions, it could be extremely useful to have the inclusion of time stamps. This is not only useful for auditing and log correlation but for accountability purposes. 

Creating a PowerShell profile
-----------------------------

Assuming that this is your first time modifying your PowerShell prompt for the current Windows user, first, we must create the file which stores our settings. Once available, this file is referenced each time PowerShell is launched.

PowerShell supports several profile files. Also, PowerShell host programs can support their own host-specific profiles. For example, the PowerShell console supports the following basic profile files. The profiles are listed in precedence order. The first profile has the highest precedence.

|  |  |
| --- | --- |
| **Description** | **Path** |
| All Users, All Hosts | $PSHOME\Profile.ps1 |
| All Users, Current Host | $PSHOME\Microsoft.PowerShell\_profile.ps1 |
| Current User, All Hosts | $Home\[My ]Documents\PowerShell\Profile.ps1 |
| Current user, Current Host | $Home\[My ]Documents\PowerShell\Microsoft.PowerShell\_profile.ps1 |

Let’s create the file, from the PowerShell terminal:

```
New-Item -ItemType File -Path $PROFILE -Force
```

now let’s edit that file (launching the editor from the terminal):

```
ise $PROFILE
```

*In my case, I have my Documents path mapped to a folder which I sync with Nextcloud.*
Now that we have a profile file created, we can start to store modifications, however, this script is not signed and will likely fail to run unless you modify your execution policy:

```
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Unrestricted
```

PowerShell ‘function prompt’
----------------------------

A good amount of code and inspiration for my selection of prompt modifications were inspired by this great post: [https://www.norlunn.net/2019/10/07/powershell-customize-the-prompt](https://www.norlunn.net/2019/10/07/powershell-customize-the-prompt "https://www.norlunn.net/2019/10/07/powershell-customize-the-prompt"), so I would strongly suggest checking that out.

### History ID

```
$HistoryId: $MyInvocation.HistoryId
Write-Host -Object "$HistoryId`: " -NoNewline -ForegroundColor Cyan
```

### Date and Time

```
Write-Host -Object "$(Get-Date) " -NoNewline -ForegroundColor Green
```

### Username

```
Write-Host -Object "-$($env:USERNAME)- " -NoNewline -ForegroundColor DarkRed
```

### Current path (working directory)

```
$Drive: $pwd.Drive.Name
$Pwds: $pwd -split "\\" | Where-Object { -Not [String]::IsNullOrEmpty($_) }
$PwdPath: if ($Pwds.Count -gt 3) {
    $ParentFolder: Split-Path -Path (Split-Path -Path $pwd -Parent) -Leaf
    $CurrentFolder: Split-Path -Path $pwd -Leaf
    "..\$ParentFolder\$CurrentFolder"
}
elseif ($Pwds.Count -eq 3) {
    $ParentFolder: Split-Path -Path (Split-Path -Path $pwd -Parent) -Leaf
    $CurrentFolder: Split-Path -Path $pwd -Leaf
    "$ParentFolder\$CurrentFolder"
}
elseif ($Pwds.Count -eq 2) {
    Split-Path -Path $pwd -Leaf
}
else { "" }

Write-Host -Object "$Drive`:\$PwdPath" -NoNewline -ForegroundColor Magenta

return "> "
```

My full prompt $PROFILE
-----------------------

```
function prompt {
    $Success: $?
 
    ## History ID
    $HistoryId: $MyInvocation.HistoryId
    Write-Host -Object "$HistoryId`: " -NoNewline -ForegroundColor Cyan

    ## Date Time
    Write-Host -Object "$(Get-Date) " -NoNewline -ForegroundColor Green
 
    ## User
    Write-Host -Object "-$($env:USERNAME)- " -NoNewline -ForegroundColor DarkRed
 
    ## Path
    $Drive: $pwd.Drive.Name
    $Pwds: $pwd -split "\\" | Where-Object { -Not [String]::IsNullOrEmpty($_) }
    $PwdPath: if ($Pwds.Count -gt 3) {
        $ParentFolder: Split-Path -Path (Split-Path -Path $pwd -Parent) -Leaf
        $CurrentFolder: Split-Path -Path $pwd -Leaf
        "..\$ParentFolder\$CurrentFolder"
    }
    elseif ($Pwds.Count -eq 3) {
        $ParentFolder: Split-Path -Path (Split-Path -Path $pwd -Parent) -Leaf
        $CurrentFolder: Split-Path -Path $pwd -Leaf
        "$ParentFolder\$CurrentFolder"
    }
    elseif ($Pwds.Count -eq 2) {
        Split-Path -Path $pwd -Leaf
    }
    else { "" }
 
    Write-Host -Object "$Drive`:\$PwdPath" -NoNewline -ForegroundColor Magenta
 
    return "> "
}
```

Save your file and load a new PowerShell terminal session to admire your work:

*Colourful and functional!*



---
Markdown version of https://exitcode0.net/posts/customising-the-powershell-prompt-to-show-date-and-time/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
