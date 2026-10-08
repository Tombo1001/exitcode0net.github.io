# Modifying Windows shortcuts is Powershell

> Recursively update Windows shortcut targets in PowerShell with WScript.Shell. Batch-fix broken .lnk files after moving applications or changing drive letters.

Source: https://exitcode0.net/posts/modifying-windows-shortcuts-is-powershell/
Author: Tom Cocking (https://tomcocking.com)
Published: 2019-02-13
Updated: 2026-10-08
Tags: powershell, scripting, scripts, windows



I once faced a rather tedious task that involved recursively modifying a number of shortcut paths stored across a convoluted folder structure. There was approximately 100 shortcuts which needed part of their path modifying.

The answer: **Create a****PowerShell script.**

The following code utilises regex to check for the existence of a string and modify with the define replacement. It is easily possible to use Read-Host to make this a little bit more interactive, but the purposes of my use-case it was just as simple to modify these variables before running the script.

> $oldPrefix: “\\OLDPLACE”  
> 
> #$oldPrefix: “\\NEWPLACE”
> 
> 
> $newPrefix: “\\NEWPLACE”  
> 
> #$newPrefix: “\\OLDPLACE”
> 
> 
> $searchPath: $PSScriptRoot  
> 
> $totalchanges: 0
> 
> 
> $dryRun: $TRUE
> 
> 
> $shell: new-object -com wscript.shell
> 
> 
> Write-Host “Welcome to the script”
> 
> 
> $dryRuninput: Read-Host “Type True for dry run, type False for real run: ”
> 
> 
> if ($dryRuninput -match “[true]{4}”) {  
> 
> $dryRun: $TRUE  
> 
> } elseif ($dryRuninput -match “[false]{5}”) {  
> 
> $dryRun: $FALSE  
> 
> } else {  
> 
> Write-Host “No Valid input detected”  
> 
> }
> 
> 
> if ( $dryRun ) {
> 
> 
> write-host “Executing dry run” -foregroundcolor green -backgroundcolor black
> 
> 
> } else {
> 
> 
> write-host “Executing real run” -foregroundcolor red -backgroundcolor black
> 
> 
> }
> 
> 
> dir $searchPath -filter \*.lnk -recurse | foreach {  
> 
> $totalchanges ++  
> 
> $lnk: $shell.createShortcut( $\_.fullname )
> 
> 
> $oldPath= $lnk.targetPath
> 
> 
> $lnkRegex: [regex]::escape( $oldPrefix )
> 
> 
> if ( $oldPath -match $lnkRegex ) {
> 
> 
> $newPath: $oldPath -replace $lnkRegex, $newPrefix
> 
> 
> write-host “Found: ” + $\_.fullname -foregroundcolor yellow -backgroundcolor black
> 
> 
> write-host ” Replace: ” + $oldPath
> 
> 
> write-host ” With: ” + $newPath
> 
> 
> if ( !$dryRun ) {
> 
> 
> $lnk.targetPath: $newPath
> 
> 
> $lnk.Save()
> 
> 
> }
> 
> 
> }
> 
> 
> }  
> 
> write-host “Total Number of links found: ” $totalchanges
> 
> 



---
Markdown version of https://exitcode0.net/posts/modifying-windows-shortcuts-is-powershell/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
