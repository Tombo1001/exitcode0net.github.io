# Turn off TPLink Smart plug LED

> Turn off the status LED on TP-Link Kasa HS100 and HS110 smart plugs with a Python script, for rooms that need to stay dark.

Source: https://exitcode0.net/posts/turn-off-tplink-smart-plug-led/
Author: Tom Cocking (https://tomcocking.com)
Published: 2021-08-22
Updated: 2026-10-08
Tags: hs100, hs110, kasa, tplink



*The HS100/HS110 LED can be bright and obnoxious.*
If you are looking to turn off the LED status lights on the TPLink Kasa smart plugs, then look no further – the solution resides in this post. It does involve some Python interaction, but I promise that it is a gentle passing with the programming language.

This solution works for:

* HS100
* HS103
* HS105
* HS110

The only real drawback is that when the LED is disabled, you can not see when the device might have become disconnected from the wireless network – this is normally indicated by a RED or orange LED status.

---

The problem:
------------

>  [How do you turn off the LED Indicator in a TPLink HS105 Smart plug?](https://www.reddit.com/r/HomeNetworking/comments/8c1fx2/how_do_you_turn_off_the_led_indicator_in_a_tplink/?ref_source=embed&ref=share "https://www.reddit.com/r/HomeNetworking/comments/8c1fx2/how_do_you_turn_off_the_led_indicator_in_a_tplink/?ref_source=embed&ref=share") from [HomeNetworking](https://www.reddit.com/r/HomeNetworking/ "https://www.reddit.com/r/HomeNetworking/") 

 

### The reddit solution:

`put some tape over it`

The Reddit users go on to explain that the tape solution was working rather well. I am sure that using tape is quite effective, but if you don’t want the light emitted by the LED, then you might as well not have the energy consumption either. 

The real solution:
------------------

   [**Turn off Kasa smartplug LED lights (HS100/HS110)**](https://github.com/Tombo1001/Kasa-Dark-Mode "https://github.com/Tombo1001/Kasa-Dark-Mode")    
 [https://github.com/Tombo1001/Kasa-Dark-Mode](https://github.com/Tombo1001/Kasa-Dark-Mode "https://github.com/Tombo1001/Kasa-Dark-Mode")  
 [0](https://github.com/Tombo1001/Kasa-Dark-Mode/network "https://github.com/Tombo1001/Kasa-Dark-Mode/network") forks.  
 [1](https://github.com/Tombo1001/Kasa-Dark-Mode/stargazers "https://github.com/Tombo1001/Kasa-Dark-Mode/stargazers") stars.  
 [0](https://github.com/Tombo1001/Kasa-Dark-Mode/issues "https://github.com/Tombo1001/Kasa-Dark-Mode/issues") open issues.  
  Recent commits: * [bug fixdevice discovery notice bug fix](https://github.com/Tombo1001/Kasa-Dark-Mode/commit/dd23852bb825f93050b0cdc32f7a072c0dea2ed5 "https://github.com/Tombo1001/Kasa-Dark-Mode/commit/dd23852bb825f93050b0cdc32f7a072c0dea2ed5"), Tom Cocking
* [Update README.mdimproved readme](https://github.com/Tombo1001/Kasa-Dark-Mode/commit/a8fe7e1a1126a7b0bbddddc515360bd4bbfd058f "https://github.com/Tombo1001/Kasa-Dark-Mode/commit/a8fe7e1a1126a7b0bbddddc515360bd4bbfd058f"), Tom Cocking
* [Initial commitv1.0 code, requirements and a basic readme](https://github.com/Tombo1001/Kasa-Dark-Mode/commit/14074b56266f7fa2cae92156c6bb36c25f99ca1b "https://github.com/Tombo1001/Kasa-Dark-Mode/commit/14074b56266f7fa2cae92156c6bb36c25f99ca1b"), Tom Cocking
* [Initial commit](https://github.com/Tombo1001/Kasa-Dark-Mode/commit/a159ab8278710126494ff433be65add67c0bee17 "https://github.com/Tombo1001/Kasa-Dark-Mode/commit/a159ab8278710126494ff433be65add67c0bee17"), Tom Cocking

  

The project’s README file has a full set of instructions on setup and how to use the python script. The code has two usage modes:

1. One for all – apply LED status on or off for all discovered smart plugs:

```
> python kasa-dark-mode.py -d
Plug Alias: Fan
Current LED state: True
New LED state: False
---
Plug Alias: Network
Current LED state: True
New LED state: False
```

2. Interactive mode – walk through each discovered smart plug and make choice on the LED status:

```
> python kasa-dark-mode.py -i
You have selected interactive mode
Searching for smartplugs...
--plug found--
Plug Alias: Fan
Current LED state: False
Do you wish to turn ON 'Fan' LED [Y/n]: n
---
--plug found--
Plug Alias: Network
Current LED state: True
Do you wish to turn OFF 'Network' LED [Y/n]:
New LED state: False
---
```

**No tape. No LED.**



---
Markdown version of https://exitcode0.net/posts/turn-off-tplink-smart-plug-led/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
