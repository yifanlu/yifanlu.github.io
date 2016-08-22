---
author: yifanlu
comments: true
date: 2016-03-28 07:21:54-06:00
layout: post
slug: cosmo3ds-the-cfw-nobody-wanted
title: 'Cosmo3DS: The CFW nobody wanted'
wordpress_id: 1414
categories:
- 3DS
- C
- Releases
tags:
- 3ds
- cfw
- cosmo3ds
- eshop
- firmware
- injection
- loader
- n3ds
- system
---

In the [last article](/2016/03/28/3ds-code-injection-through-loader/), I talked about my plan for creating 3DS mods. Now, I will put that plan to the test with a CFW (modified firmware) that nobody wants except me.

The idea for this CFW is that I want



	
  * Keep my 3DS on the hackable 9.2 firmware but still use the latest system software (emuNAND)

	
  * Play games region free right from the home menu

	
  * Change the system region without possibly bricking the device

	
  * [Use the eShop with region changed systems](http://gbatemp.net/threads/creating-a-north-american-non-xl-new-3ds.381775/page-37#post-5459760)


and _only_ those things. Specifically, I don't want to patch signatures that allow for installing pirated content (personal choice, I don't care what you do). I don't want threads running in the background to support fancy features such as replacing version strings or taking screenshots or in-game menus. So I created my own CFW in two components: a stripped down version of [ReiNAND](https://github.com/Reisyukaku/ReiNand) where everything except emuNAND is removed and an implementation of my [custom loader](https://github.com/yifanlu/3ds_injector/tree/master) which does all the patches. No threads. No glut.

I understand that I'm the only one who needs such a CFW and most people can make do with one of the ones out there. This is mostly written just for my own use and for me to test out my idea of patching the system. However, I'm happy that my N3DS is now a _cosmopolitan_.

If you want to use it, you need the [CFW](https://github.com/yifanlu/Cosmo3DS/releases/tag/v0.1) as well as the [custom loader](https://github.com/yifanlu/3ds_injector/releases/tag/v0.1). Then, the important part is [injecting the custom loader into the right FIRM file](https://github.com/yifanlu/Cosmo3DS/blob/cosmo3ds/README.md). (If you search for "reinand 3.1 firmware.bin pastebin" you should be able to find it)
