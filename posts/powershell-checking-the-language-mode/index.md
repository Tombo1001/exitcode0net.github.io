# Powershell - Checking the Language Mode

> Check the PowerShell language mode on Windows to spot ConstrainedLanguage restrictions before running administrative or automation scripts.

Source: https://exitcode0.net/posts/powershell-checking-the-language-mode/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-02-05
Updated: 2026-10-08
Tags: code, powershell, scripting, scripts



For security purposes, it is possible to control the language mode in a given Powershell session. These language modes can constrict which modules can be loaded during the life of a powershell session.

Learn mode about Powershell langeuage modes: **[About Language Modes – Microsoft](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_language_modes?view=powershell-6 "https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_language_modes?view=powershell-6")**

Detect the Current Language Mode
--------------------------------

> **$sLangMode: $ExecutionContext.SessionState.LanguageMode**  
> 
> If ($sLangMode -ne “FullLanguage”){  
> 
> Write-Host ” !! Unable to run scrit – Powershell Using Wrong Language Mode !! ”  
> 
> }  
> 
> Else{  
> 
> **#RUN THE MAIN FUNCTION**  
> 
> }
> 
> 

Try putting this simple statement at the start of your powershell scripts to avoid any unhandled exceptions caused by constrictive language modes.



---
Markdown version of https://exitcode0.net/posts/powershell-checking-the-language-mode/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
