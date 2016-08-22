---
author: yifanlu
comments: true
date: 2012-12-31
layout: post
slug: huawei-e587-t-mobile-4g-sonic-hotspot-information-and-rooting
title: 'Huawei E587 (T-Mobile 4G Sonic Hotspot): Information and rooting'
wordpress_id: 601
categories:
- Qualcomm
- Technical
tags:
- e587
- freescale
- hardware
- hotspot
- huawei
- imx
- jailbreak
- linux
- qualcomm
- root
- rooting
- software
- sonic
- sqlite
- teardown
- tmobile
- uboot
---

Earlier this year, I got my hands on the T-Mobile 4G Sonic Hotspot and as always, had to tear it apart as soon as I got it. I never wrote about it because I didn't find anything overly interesting, but now it's the end of the year, and I need to clear some inventory from my brain. If anyone remembers my post on the (older) [T-Mobile 4G Hotspot](/2012/03/07/unlocking-t-mobile-4g-hotspot-zte-mf61-a-case-study/) (sans "Sonic"), the main limitation of that device was that the processor is an obscure one that required some digging to get information on. Thankfully, the Sonic variety is much easier to break into.<!-- more -->


## Teardown


I don't usually do this, but as I couldn't find any good snapshots of the insides of this device, I took it upon myself to produce some amateur shots. One thing I want to say about the insides is that I loved how the main board is broken into three parts and they're sandwiched together to make the device small (but thick).

[![Device with faceplate removed.](/images/2012/12/e587_body-225x300.jpg)](/2012/12/31/huawei-e587-t-mobile-4g-sonic-hotspot-information-and-rooting/e587_body/)

Device with faceplate removed.
{: .wp-caption-text}



[![MCIMX283CVM4B](/images/2012/12/e587_soc-300x225.jpg)](/2012/12/31/huawei-e587-t-mobile-4g-sonic-hotspot-information-and-rooting/e587_soc/)

FreeScale MCIMX283CVM4B
{: .wp-caption-text}



[![Qualcomm MDM8220 modem](/images/2012/12/e587_modem-225x300.jpg)](/2012/12/31/huawei-e587-t-mobile-4g-sonic-hotspot-information-and-rooting/e587_modem/)

Qualcomm MDM8220 modem
{: .wp-caption-text}



[![Middle layer, containing various chips](/images/2012/12/e587_chips-225x300.jpg)](/2012/12/31/huawei-e587-t-mobile-4g-sonic-hotspot-information-and-rooting/e587_chips/)

Middle layer, containing various chips
{: .wp-caption-text}



The important information is that the device is ARM based (it even uses the same system-on-chip as older Kindles), and having a well documented SoC is always a plus. There isn't an obvious debug serial port, but I would bet that there is one knowing how the FreeScale SoCs work. However, we don't need to explore hardware hacking yet as the software is unexplored.


## Rooting


This was literally the easiest device I've ever rooted. I can honestly say that from opening the package (knowing nothing about the device) to getting a root shell took me about fifteen minutes. There was only one interface to the device and that's the management webpage. My plan was to explore every location where I could pass input to the device (settings, HTTP POST requests, MicroSD file browser, etc) and basically just try things until I get a crash or something interesting. The first thing I've tried was the settings backup/restore feature. Creating a backup of the settings allows you to download a SQLite database containing the settings. A quick SQL dump of the settings showed me some interesting options that can't be set directly from the web interface, including:


```sql
CREATE TABLE telnet
(
TelnetStatus int
);
```


Yep, setting TelnetStatus to 1 and restoring the backup database showed me that port 23 was now open from the hotspot's IP. Well, that was extremely lucky, as always the best hacks are the one which doesn't require hacking at all. Well that was only half the challenge, the next part is getting access to the root account. I'm thinking everything from brute forcing passwords to looking at privilege escalation exploits but all of that disappeared as soon as I typed "root" and enter because there was no password prompt. That's right, "root" doesn't require a password. I did a quick inventory of the filesystem and found the block devices, and using the magic of dd, nc, and the old Unix pipe, quickly dumped all the filesystems.


## Software


Here's the thing though, I spent all this time (almost 45 minutes at this point!) rooting the device and I don't even have a clear goal. I don't need to unlock the device because I was a T-Mobile customer at that point, and I didn't really want to make a pocket ARM computer/server (which would be a thing one can do with this), so I just did a quick scan of how the device works (curiosity is the best excuse) and went my way. Here's some of the things I've discovered, use this information how you will.

First of all, the device runs a stripped down build of Android running "Linux version 2.6.31 (e5@e587) (gcc version 4.4.0 (GCC) ) #1 Sun Aug 28 02:25:47 CST 2011." On startup, most of the vanilla Android processes (including adbd) are not started, but instead the Qualcomm modem driver, some pppd/networking daemons, and a custom software they call "cms" are started. "cms" makes sure stuff like their custom httpd (which is hard coded to allow the HTML portal site to perform functions on the hotspot) and power management and the OLED display are running and in good status. The Huawi device stores all data on its flash MTD device. From a quick analysis (aka, might be errors), block 0 contains the u-boot bootloader (in what I believe is a format dictated by FreeScale), block 3 contains the kernel (gzipped with a custom header, possible also dictated by FreeScale), block 4 contains the rootfs (also gzipped with a custom header) loaded with boot scripts and busybox, block 5 is Android's /system which also contains the main binaries (like cms, httpd) and the HTML pages, block 6 is Android's /data which is empty, block 8 maps to /mnt/backup which I believe is, as the name says, just backups, block 12 maps to /mnt/flash which I believe is where ephemeral data like logs are and also where the settings are stored, and block 13 maps to /mnt/cdrom which has Huewai's software and drivers for connecting to the computer with (and you see it when you plug the device into your computer).

That's a quick summary of some of the things I've found while poking around this device. Nothing interesting (unless you're a Huawei E587 fanatic I guess), but I'm sure there's someone, someday, who got here from Google.
