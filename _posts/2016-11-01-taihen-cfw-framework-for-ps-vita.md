---
author: yifanlu
comments: true
date: 2016-11-01
layout: post
slug: taihen-cfw-framework-for-ps-vita
title: "taiHEN: CFW Framework for PS Vita"
categories:
- Vita
tags:
- vita
- henkaku
- jailbreak
- taihen
- updates
---

Ever since I first bought the Vita, I have dreamed of running a custom firmware on it. I don't mean just getting kernel code running. I want an infrastructure for adding hooks and patches to the system. I want a system for patching that was properly designed (or actually has a design), clean, efficient, and easy to use. That way, firmware patches aren't a list of hard coded offset and patches. I've seen [hacks](https://en.wikipedia.org/wiki/Nintendo_3DS) that busy loops the entire RAM looking for a version string pattern so it can replace it with a custom text. I've seen [hacks](https://en.wikipedia.org/wiki/PlayStation_Portable) that redirect the "open" syscall so every file open path is string compared with a list of files to redirect. The examples go on and on. Needless to say, good software design is not a strong point for console hacking. For HENkaku, we did not commit any major software development sins, but the code was not perfect. It had hard coded offsets everywhere, abuse of C types, and lots of one-off solutions to problems but it got the job done. Part of the reason we didn't want to release the source right away was that we didn't want people to build on that messy code-base (the other reason was the [KOTH challenge]({% post_url 2016-10-20-henkaku-koth-solved %})). I remember the dark days of 3DS hacking where every homebrew that needed kernel access would just bundle in the exploit code. This is why I decided to create taiHEN.

## taiHEN

taiHEN is a framework for writing application and system level patches. Simply put, it lets you run game and kernel plugins anywhere. taiHEN is _not_ a new exploit. The HENkaku update (which we lovingly call taiHENkaku) uses the same chain of exploits (and therefore still requires firmware 3.60) but the actual firmware patches have been moved to the taiHEN system. taiHEN is designed to be firmware and exploit agnostic--that means it should run on any firmware if you bring your own exploit. Right now the only exploit is HENkaku and it requires WebKit to work. However, if someone finds a boot exploit or an exploit for 3.61/3.63, all they have to do is load `taihen.skprx` and (ideally) every plugin should just work. This also means that when someone ports over the HENkaku exploit to lower system versions, they do not have to re-build every patch from scratch.

In addition to adding hooks to the kernel, taiHEN also allows hooking system applications and games. Add elements to LiveArea? Enable more options in Settings? Cheats in games? The possibilities are endless. More information is at the official site: [tai.henkaku.xyz](https://tai.henkaku.xyz/) and from [Davee's blog](https://www.lolhax.org/2016/11/02/the-vision-behind-taihen/).

## taiHENkaku

As promised, this is the _big_ HENkaku update. In addition to the major plumbing overhaul, we added some new features to HENkaku too:

  * Loading compressed FSELFs are supported now
  * VitaShell is updated to [1.42](https://github.com/TheOfficialFloW/VitaShell/releases/tag/1.42) with a brand new HENkaku configuration menu that allows user configuration of PSN version spoofing. (Note at the time of writing, VitaShell has not been updated yet. I will push an update as soon as it is out.)
  * **Unsafe homebrew is disabled by default** This change means that some of your homebrew will not launch immediately. Before you panic, go into molecularShell, press Start, enter the HENkaku configuration menu and choose to enable unsafe homebrew. You also need to do this to use system and kernel plugins. More information on this change can be found [here](https://gist.github.com/yifanlu/3b36b987c9099b620c5b65ce8e93801d). (Note, this feature is disabled in the beta currently because the VitaShell configuration options is not out yet. It will be enabled as soon as that's done.)

Because this is a major update with a significant increase in complexity, we are releasing it as an open beta. The changes mostly benefit developers wanting to write plugins with taiHEN so that is the target audience. To install it, reboot your Vita and visit **http://beta.henkaku.xyz/**. You can always go back to the last stable release from the regular site **https://henkaku.xyz/**.

**Note on the beta**: It is currently in an unstable state. Some features such as PSN spoofing do not currently work. I hope to resolve the issues in the upcoming days. Meanwhile, I hope that developers can start writing plugins immediately while I iron out the issues. Again, the beta is only recommended for developers making plugins and is of no benefit currently for regular users.

## Plugin SDK

Davee did a wonderful job implementing SDK support for user and kernel plugins. The changes are not in the mainline yet, so please help us test it. You need the new toolchain updates to build taiHEN and your own plugins.

## Development Wiki

This brings me to the last point. For the kernel, there needs to be a lot of reverse engineering to figure out all the functionalities exported by the kernel. We at molecule have done a lot of work in the past few years but we have not even covered 10% of what the kernel exports. This was the prize given to those who completed the KOTH challenge and now it is released for the public. It contains just about everything that molecule has discovered and reversed about the Vita since 2012 and includes a lot of low level information about the system. It is a good place to start for anyone who wishes to get into Vita hacking: [wiki.henkaku.xyz](https://wiki.henkaku.xyz/).

## What's next?

To summarize, today we are releasing four things

  * [taiHEN](https://tai.henkaku.xyz/), a CFW framework for the Vita enabling kernel and user plugins for everyone
  * [taiHENkaku](http://beta.henkaku.xyz/), the latest update to HENkaku that uses taiHEN
  * [Plugin supporting SDK](http://github.com/vitasdk/), for creating kernel and user plugins
  * [Vita Development Wiki](https://wiki.henkaku.xyz/), the largest resource for Vita hacking and revere engineering

All this is due to the gracious work done by my friends in molecule: Davee, Proxima, and xyz. I am extremely lucky to have worked with such talented individuals and they have my sincere thanks. All our releases have been made with a level of polish and professionalism unparalleled by anyone else in the console hacking scene because of them. This also marks the end of our active development on the Vita. We'll release bug fixes from time to time and we'll continue to look into hacking the F00D processor (lv0), but we will not have the time to create user facing content anymore. I want to thank the community for the encouragement and support and I want to thank Sony for building the Vita and making it secure. Finally, I want to thank everyone who participated in the KOTH challenge and proved to me that there is indeed still interest in hacking the Vita. I know that we leave the scene in good hands!
