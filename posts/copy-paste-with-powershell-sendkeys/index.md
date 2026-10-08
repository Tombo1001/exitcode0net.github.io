# 'Copy, Paste' With Powershell Sendkeys

> Paste complex strings to UAC prompts with PowerShell SendKeys scripting. Bypass clipboard restrictions in elevated dialogs with automated keystroke injection.

Source: https://exitcode0.net/posts/copy-paste-with-powershell-sendkeys/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-02-02
Updated: 2026-10-08
Tags: code, powershell, scripting, scripts



This is really quick nugget of Powershell for anyone who is struggling to copy and paste into a particular window or dialog box.

Perhaps it is a case of a website which prevents text form being sent to a field from the clipboard, or in my case, a windows UAC prompt. If you are following password best practices, your passwords should be long, complex and contain zero dictionary words. Furthermore, you should have a different password for every site and service. **So**, when a user interface doesn’t allow you to paste a password, life gets that little bit more difficult.

**Powershell to the rescue!**

> function FuncMain {
> 
> 
> add-type -AssemblyName System.Windows.Forms
> 
> 
> Write-Host “”  
> 
> Write-Host ” — Welcome to the UAC Clipboard Utility — ”  
> 
> Write-Host “”
> 
> 
> $sUsername: Read-Host “Enter Username”  
> 
> $sPassword: Read-Host “Enter Password”
> 
> 
> Write-Host “User name and password ready…”  
> 
> Write-Host “”  
> 
> Write-Host “!!! PLACE THE CURSOR IN THE UAC USERNAME FIELD NOW !!!”  
> 
> Write-Host “”  
> 
> Start-Sleep -Seconds 1
> 
> 
> **$i: 5**  
> 
> Write-Host “Begin Countdown:”  
> 
> while( $i -gt 0){  
> 
> write-host $i  
> 
> $i —  
> 
> Start-Sleep -Seconds 1  
> 
> }
> 
> 
> Try {  
> 
> #Sends the entered username followed by tab and then the password.  
> 
> [System.Windows.Forms.SendKeys]::SendWait($sUsername + “{TAB}”)  
> 
> Start-Sleep -Seconds 1  
> 
> [System.Windows.Forms.SendKeys]::SendWait($sPassword)  
> 
> }  
> 
> catch {  
> 
> Write-Host “There was an error sending username / password!”  
> 
> }
> 
> 
> Write-Host “”  
> 
> Write-Host “…Paste of Credentials Complete”
> 
> 
> Read-Host “Press Enter to Clear Console and close the script”  
> 
> Clear-Host  
> 
> }
> 
> 
> $sLangMode: $ExecutionContext.SessionState.LanguageMode  
> 
> If ($sLangMode -ne “FullLanguage”){  
> 
> Write-Host “”  
> 
> Write-Host ” !! Unable to run scrit – Powershell Using Wrong Language Mode !! ”  
> 
> Write-Host “”  
> 
> }  
> 
> Else{  
> 
> #RUN THE MAIN FUNCTION  
> 
> FuncMain  
> 
> }
> 
> 

Modifications:
--------------

If this doesn’t quite fit your use-case, you can easily modify this code by commenting out the read-host line which asks for a username, as well as commenting out the line where the username keys are sent – This converts the script into a password only tool.

Changing the value of **$i** changes the countdown timer.

**Thanks again to Powershell scripts for making our lives a little bit easier.**



---
Markdown version of https://exitcode0.net/posts/copy-paste-with-powershell-sendkeys/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
