---
author: yifanlu
comments: true
date: 2011-03-03 01:28:05-06:00
layout: post
slug: quickguide-bypassing-lenovo-s10-bios-whitelist
title: 'Quickguide: Bypassing Lenovo S10 BIOS Whitelist'
wordpress_id: 254
categories:
- Assembly
- Guides
- Technical
tags:
- bios
- hardware
- hex
- lenovo
- phoenix
- quickguide
- s10
- whitelist
- wwan
---

Lenovo loves to assert their dominance to you by whitelisting what WWAN (3G modem) card you can install in your laptop. There has been a way to bypass or remove the whitelist on most models, except the S10. Now I found a great guide here: [http://www.sbbala.com/DellWWAN/Whitelist.htm](http://www.sbbala.com/DellWWAN/Whitelist.htm) that shows you how the remove the whitelist, but as many found out, it doesn't always work. The problem is that... well, I don't know what the problem is, but I'm guessing there's additional checks. I've been trying to find the format of the S10 whitelist, but I'm having no luck, so we'll do it the easy way. Brute force. Put your WWAN card into every whitelist entry. It'll have to work then, right?<!-- more -->

Now this is a "quickguide" which means I won't spoon feed you. This is mostly because I don't have the time to write a full guide, but maybe if I ever find the format of the whitelist or find a way to disable it completely, I'll write an actual guide.

Basically, follow [sbbala's guide](http://www.sbbala.com/DellWWAN/Whitelist.htm) up until "Save and now you can close the hex-editor." Instead of pulling out after replacing one entry, we're going to replace a couple of others in MISER00.ROM. Take the PID/VID (little-endian reversed) and replace the follow entries with it:


```
DB 0B 00 19 (this one was in the guide)
D1 12 01 10 (this one will appear twice, replace both)
D1 12 03 10
C6 05 01 92
D2 19 F1 FF (this one will appear twice, replace both)
```


Now, I'm sure there are more devices in the whitelist, but for safety reasons, the ones I choose are 1) WWAN cards (I don't want to accidentally remove the camera from the whitelist), and 2) in the Linux VID/PID list. If this doesn't work, then try looking and replacing some more values in the whitelist. Although I haven't completely reversed the whitelist format yet, I THINK it's something like this. 1 Byte: FA followed by 4 bytes VID (little-endian) followed by 4 bytes PID (little-endian) followed by X bytes of don't-know-what. The offset is different for every BIOS version, but it's always in MISER00.ROM and is before DB 0B 00 19 and a bit after a bunch of 00s.
