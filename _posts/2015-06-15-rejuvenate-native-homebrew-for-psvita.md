---
author: yifanlu
comments: true
date: 2015-06-15 04:30:39-06:00
layout: post
slug: rejuvenate-native-homebrew-for-psvita
title: 'Rejuvenate: Native homebrew for PSVita'
wordpress_id: 1185
categories:
- Announcements
- PS Vita
tags:
- beta
- homebrew
- psm
- psvita
- rejuvenate
- sdk
- toolchain
- uvloader
- vita
- vitadefiler
---

(Sadly, they did not give me a spot at the Sony E3 conference, so I have to make do with this blog post.) I am excited to announce **Rejuvenate**, a native homebrew platform for PS Vita. The tools that will be released through the next couple of weeks will allow developers (not in contract with Sony) to develop and test games, apps, and more on the PS Vita. These unofficial software can run on any PS Vita handheld device without approval by Sony. These tools _cannot_ enable pirated or backup games to run (I'm not just saying this... the exploits used does not have enough privilege to enable such tasks). **Rejuvenate** requires **[PlayStation Mobile Development Assistant](https://psm.playstation.net/static/general/all/en/psm_sdk.html#devassistant)** to be installed on your Vita! Sony will remove this from PSN soon, so if you wish to ever run homebrew apps on your PS Vita, you must download this app now!
<iframe width="560" height="315" src="https://www.youtube.com/embed/_Buiwewttcw" frameborder="0" allowfullscreen></iframe>
<!-- more -->
It's been almost three years since I found the first native exploit for the PS Vita. Many people have asked me why I did not release my tools for public consumption. Other than laziness, the two main reasons were: 1) I believed that PSM was a great platform for indie developers and feared that releasing this would ensure PSM a death like OtherOS, and 2) there was no SDK for Vita homebrew so releasing the exploit would not benefit any users at all since they have no use for it. Now that Sony is killing PSM anyways and there is [significant progress on the SDK](/2015/05/23/calling-all-coders-we-need-you-to-help-create-an-open-vita-sdk/), it seems that the time is right for a release.


### Rejuvenate


Rejuvenate is composed of three main tools which together provides a platform for developers to write Vita homebrew.



	
  * **[UVLoader 1.0](http://github.com/yifanlu/UVLoader)** allows executables to be loaded on the PS Vita. The original version was written three years ago for firmware version 1.80. Since then, Sony has taken the source code and made loading code much, much, harder. However, there is no barrier that cannot be bypassed! The latest version includes support for SCE ELF relocations, NID poison antidote, and more. It can run homebrews on Vitas up to firmware 3.51 (at time of writing)

	
  * **VitaDefiler** is a RPC (remote procedure call) system for Vita userland. Main features includes live peek/poke of userspace memory, execution of arbitrary ARM code, and a scripting interface for quickly running tasks. Originally developed for finding exploits, this tool can also be used by homebrew developers to test and debug their apps. VitaDefiler also serves as the ASLR (address-space-layout-randomization, a technique used by Sony to discourage exploitation) bypass for UVLoader.

	
  * **PSM+** is what I call the method I found to bypass the two kill-switches Sony placed into PSM to prevent this very scenario. First, app-keys which are issued by Sony to developers to sign PSM content are required for the exploit to run. These keys usually expire every three months, and Sony can refuse to issue them later. This can be bypassed. Second, every day, PSM phones home to see if it is revoked. If Sony decides to kill the Dev Assistant (and they will), it will refuse to run even if you have it installed. This can also be bypassed.


These tools, along with the open SDK (currently in development) will allow for developers to write Vita homebrew. The demonstration video above shows UVLoader running as a VitaDefiler script (which supplies information for ASLR bypass). The spinning-cube demo was coded up by me, linked together by hand (as the open SDK is currently incomplete), and launched with UVLoader. It is running natively with direct access to the GPU API calls (not within the PSM sandbox).


### Limitations


So what's the catch? The good news is you don't have to buy any obscure or expensive game (everything is free!). The bad news is that launching homebrews is not as simple as copying some files over. Hopefully, most of these limitations can be bypassed in a later release, but at this point, the following side effects will apply



	
  * USB connection is required each time you wish to launch a homebrew. The exploit requires a PC to run, so this is unfortunately a requirement. This also means that the VitaTV is not supported.

	
  * Windows PC is required. Blame Sony for never porting PSM tools to other operating systems.

	
  * Network is required once each day you decide to run homebrew. This is because PSM has to phone home every day. Although we have a means of bypassing the revoke, we currently cannot bypass the phone-home.

	
  * Firmware 3.00+ recommended. Although technically the exploit works on 1.69+, the latest version of the tools have only been tested (and will only support) 3.00+. If there is enough interest, I can port it to lower firmware versions, but it will be very low priority.




### FAQ


**I don't want to bother with [insert limitation from above], should I still download PSM Developer Assistant?**

Yes, if you _ever_ want to run homebrew at some point. PSM DevAssistant is the only application on the PS Vita that has the required permissions to run arbitrary code in memory. WebKit exploits does not allow for this. Any game exploit does not allow for this. Any system application exploit does not allow for this. PSM DevAssistant is the _only_ application allowed to execute code other than the kernel (operating system), which nobody is even close to hacking. In other words, expect at least a dozen more exploits of PSM DevAssistant (each of which may require less hassle to use) before someone finds a kernel exploit.

**Can I run backups/ISOs/copied games for Vita? For PSP? For PSOne?**

No.

**Stop acting all high any mighty with your anti-piracy stance.**

Inability to decrypt/dump/execute official software and games is not something I decided to include by choice (however, I am glad it's there). The exploits that are used physically does not give permissions for this. Sony did a really good job with security in depth, no application has more privileges than necessary. PSM DevAssistant would never be used officially to decrypt, dump, or execute signed games so it cannot do so even when exploited.

**What kind of homebrews will we see? Is it any better than PSP homebrew?**

This depends on how many developers are willing to invest time in writing homebrew for the Vita. I'm as hopeful as you are. In terms of pure statistics, the PSP-3000 has 64MB of shared memory, 333MHz CPU, and 166MHz GPU. The Vita has 512MB of main memory and 128MB of dedicated video RAM. It has four cores of CPU running at around ~1GHz and four cores of GPU running at around ~200MHz. In addition, the Vita also has the entire PSP hardware inside its silicon.

The exploit used also allow for developers to use dynamic-recompilation features for speeding up emulators.

**Can I install Android, custom themes, cheats, or plugins?**

No, this exploit does not give kernel or bootloader level of access. It cannot access the filesystem (unsandboxed), modify system files, or access other process' memory.


### Schedule





	
  * **Today**, early beta access of Rejuvenate is being distributed to SDK developers (to complete and test the SDK) as well as hackers who will begin searching for more exploits (and hopefully bypass the current limitations)

	
  * **By end of the month** (or whenever Sony removes PSM DevAssistant from PSN), a public beta goes out. However, it is only recommended that homebrew developers use it as there would be no homebrew for users to try yet. Hopefully at this point, the open SDK would be complete and ready for developers to write apps.

	
  * **When PSM is revoked**, the directions for using **PSM+** to bypass the revokes will be released.



**EDIT:** Public beta is [now out](/2015/06/20/rejuvenate-public-beta-release/).

In the meantime, we have two IRC channels for Vita discussions (sorry about the confusion, it's a long story)! This is the preferred way to speak with me and other unofficial Vita developers. For discussion of UVLoader, VitaDefiler, and other exploit related things, join [#vitadev](irc://irc.efnet.net/vitadev) on irc.efnet.net. For discussion on the open SDK, APIs, and homebrew development, join [#vitasdk](irc://chat.freenode.net/vitasdk) on chat.freenode.net.

Thanks to everyone who helped out with this, especially Davee, Proxima, and xyz. I hope that this is the start of something great. Every day, people clamor about the death of the Vita, but we will prove them wrong. We will give new life to the Vita.
