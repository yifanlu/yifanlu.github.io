---
author: yifanlu
comments: true
date: 2010-08-19 11:55:34-06:00
layout: post
slug: load-creative-zen-v-plus%e2%80%99s-firmware-on-your-zen-v
title: Load Creative Zen V Plus’s firmware on your Zen V
wordpress_id: 178
categories:
- Guides
- Technical
tags:
- creative
- firmware
- hacks
- patch
- zen v
- zen v plus
---

So thanks to a Napster promotion, I’ve got a free 1GB Creative Zen V. If you know anything about me, you’d know that the first thing I did was pop open IDA Pro, and see what I can make this device do that it’s not made for doing. After some quick Googleing, I’ve noticed there’s no modifications or anything for this POS music player. However, I did notice that Creative sells a higher priced player that plays videos too. Anyways, enough talk, here’s how to turn your Zen V to a Zen V Plus (NOTE: You still won’t get radio because it’s not in the hardware)

**Directions:**



	
  1. Download the ZEN V Plus firmware 1.32.01 [here](http://support.creative.com/downloads/download.aspx?nDownloadId=10334). It’s last update was in 07, so I don’t think there’s going to be a newer version, but if there somehow is, follow the “DIY” instructions in the next section to do it manually.

	
  2. Make a copy of the **ZENVPlus_PCFW_L22_1_32_01.exe **file you just downloaded. Name it **ZENV_Patch.exe**.

	
  3. Patch **ZENV_Patch.exe **with this [IPS file](http://www.multiupload.com/JYCQOVQS15) using any [IPS patching utility](http://www.zophar.net/utilities/patchutil.html).

	
  4. Run **ZENV_Patch.exe **and let it reboot your Zen V.

	
  5. Now, you should be getting an error on the device. _THIS IS NORMAL_. The firmware update should fail and put you in recovery mode.

	
  6. In recovery mode on the Zen, choose “Reload Firmware”

	
  7. Now, on your PC, force quit **ZENV_Patch.exe **and open up **ZENVPlus_PCFW_L22_1_32_01.exe**

	
  8. Wait until the update is done, and your Zen V is now a Zen V Plus!


**DIY**

Now, how does this work? Well, basically the first “firmware update” with ZENV_Patch.exe makes the device think it’s a Zen V Plus, and the second update with the official file actually copies the firmware on. ZENV_Patch.exe is just the Zen V Plus updater hex-edited to run on the Zen V. You can make your own ZENV_Patch.exe by taking the official update, opening a HEX editor, and replacing every instance of “C.r.e.a.t.i.v.e. .Z.e.n. .P.l.u.s” to “C.r.e.a.t.i.v.e. .Z.e.n” (Please note that the periods represent the ASCII character 00 (null)). After doing so, the updater will accept the Zen V.

Now, maybe one day, I’ll port RockBox or something to it…
