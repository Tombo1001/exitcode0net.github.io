# How to check if RAM is running in ECC mode

> Check whether your RAM is running in ECC mode on Windows and Linux using dmidecode or WMIC, for servers and workstations.

Source: https://exitcode0.net/posts/how-to-check-if-ram-is-running-in-ecc-mode/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-05-30
Updated: 2026-10-08
Tags: dmidecode, ecc-ram, linux, windows, wmic



Error Correcting Code (ECC) RAM is a variation of coputer memory which helps to ilimintate data curruption or ‘bit rot’, but it is not always imediately apparent if your system memory is running in ECC mode; here is a quick guide on how to check if your system memory is running in ECC mode. This guide covers Windows and Linux systems, but please ensure that if you are running either system in a virtual machine configuration that this command is ran on the **host** machine.

Linux – Check if ECC Mode is enabled
------------------------------------

To check if a system’s memory is running in ECC mode on a Linux system, open up the terminal (elevation may be required in some instances) and type the following command:

```
dmidecode -t memory
```

or

```
dmidecode --type memory
```

Along with other memory statistics and information, your will receive the following item line if ECC is active and the mode in which it is running:

```
Error Correction Type: Multi-bit ECC
```

Furthermore, the **dmidecode** tool is very useful for learning more about the other areas of your system from the Linux terminal. You can find a great breakdown of some of its features here: [https://www.redhat.com/sysadmin/linux-tools-dmidecode](https://www.redhat.com/sysadmin/linux-tools-dmidecode "https://www.redhat.com/sysadmin/linux-tools-dmidecode")

---

Windows – Check if ECC Mode is enabled
--------------------------------------

Open the windows command prompt and enter the following **wmic** command:

```
wmic memphysical get memoryerrorcorrection
```

*Using CMD to check if ECC is enabled*

---

*Note: if you are struggling to run WMIC command in your command prompt and receiving an error that ‘**wmic is not recognized as an internal or external command**‘*, consider checking out this post where I demonstrate how to fix that: [https://exitcode0.net/posts/adding-wmic-command-to-the-windows-path/](https://exitcode0.net/posts/adding-wmic-command-to-the-windows-path/ "https://exitcode0.net/posts/adding-wmic-command-to-the-windows-path/")

---

As you can see from the example above, my Windows laptop returned a value of **3**, but what does this mean?

```
Value Meaning 
0 (0x0) Reserved 

1 (0x1) Other 

2 (0x2) Unknown 

3 (0x3) None 

4 (0x4) Parity 

5 (0x5) Single-bit ECC 

6 (0x6) Multi-bit ECC 

7 (0x7) CRC
```

Unsurprisingly my laptop is not running any kind of ECC memory and returns **None**.



---
Markdown version of https://exitcode0.net/posts/how-to-check-if-ram-is-running-in-ecc-mode/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
