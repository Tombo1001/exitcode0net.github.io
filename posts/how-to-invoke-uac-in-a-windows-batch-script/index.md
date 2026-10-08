# How to invoke UAC in a Windows Batch script

> Trigger a UAC elevation prompt from a Windows batch script with a VBScript helper, then re-run the original script with administrator rights.

Source: https://exitcode0.net/posts/how-to-invoke-uac-in-a-windows-batch-script/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-12-29
Updated: 2026-10-08
Tags: batch, code, scripting, windows



Here is my quick and easy way to raise the privilage level of a Windows Batch script; allowing you to run your code at an administrator level.

This is not a new question and has been asked many times on StackOverflow [forums](https://stackoverflow.com/questions/1894967/how-to-request-administrator-access-inside-a-batch-file/10052222#10052222 "https://stackoverflow.com/questions/1894967/how-to-request-administrator-access-inside-a-batch-file/10052222#10052222"). The best answer I was able to find was the following from **[dbenham](https://stackoverflow.com/users/1012053/dbenham "https://stackoverflow.com/users/1012053/dbenham")**:

[https://stackoverflow.com/questions/1894967/how-to-request-administrator-access-inside-a-batch-file/10052222#10052222](https://stackoverflow.com/questions/1894967/how-to-request-administrator-access-inside-a-batch-file/10052222#10052222 "https://stackoverflow.com/questions/1894967/how-to-request-administrator-access-inside-a-batch-file/10052222#10052222")

That being said there are many ways to skin this cat, so I came up with my own method, all be it derivative. The main difference in the following code is that **you will always get a UAC prompt** in my variant of the code, even if you are currently running it as an account that is a member of the local administrators group:

*UAC prompt invoked!*
Full Code:
----------

```
:::::::::::::::::::::::::::::::::::::::::
:: Automatically check & get admin rights
:::::::::::::::::::::::::::::::::::::::::
@echo off
CLS
ECHO.
ECHO =============================
ECHO Running Admin shell
ECHO =============================

:checkPrivileges
NET FILE 1>NUL 2>NUL
if '%errorlevel%' == '0' ( goto gotPrivileges ) else ( goto getPrivileges )

:getPrivileges
if '%1'=='ELEV' (shift & goto gotPrivileges)
ECHO.
ECHO **************************************
ECHO Invoking UAC for Privilege Escalation
ECHO **************************************

setlocal DisableDelayedExpansion
set "batchPath=%~0"
setlocal EnableDelayedExpansion
ECHO Set UAC: CreateObject^("Shell.Application"^) > "%temp%\OEgetPrivileges.vbs"
ECHO UAC.ShellExecute "!batchPath!", "ELEV", "", "runas", 1 >> "%temp%\OEgetPrivileges.vbs"
"%temp%\OEgetPrivileges.vbs"
exit /B

:gotPrivileges
::::::::::::::::::::::::::::
::START
::::::::::::::::::::::::::::
setlocal & pushd .

ECHO Hello World, I am your admin:
whoami
pause
```

Find the code on GitHub: [https://github.com/Tombo1001/UACprod](https://github.com/Tombo1001/UACprod "https://github.com/Tombo1001/UACprod")



---
Markdown version of https://exitcode0.net/posts/how-to-invoke-uac-in-a-windows-batch-script/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
