---
author: yifanlu
comments: true
date: 2012-12-12 22:49:43-06:00
layout: post
slug: playstation-vita-the-progress-and-the-plan
title: 'PlayStation Vita: the progress and the plan'
wordpress_id: 592
categories:
- Information
- PS Vita
tags:
- cfw
- emulator
- hardware
- homebrew
- kernel
- psvita
- sdk
- userland
- uvloader
- vita
---

Sorry that it's been a while since I've said anything about the Vita. I was caught by surprise the last time of all the media attention from just a simple call for help. While I still don't want to say too much right now, I do want to answer some common questions I've been getting and also go over what needs to be done.<!-- more -->

If this is news to you, please read this [interview](http://www.playstationlifestyle.net/2012/09/06/hacking-the-vita-yifan-lu-talks-homebrew-security-and-why-games-still-cant-be-pirated/) I've done a while ago about it.

**Did you hack the Vita? **That's a very vague question. What I _have_ done, is run native code on the Vita with the same permissions as the game being exploited. This means I can load homebrews written and optimized for the Vita's CPU and take full advantage of the CPU speed and RAM (unlike the PSP emulator or PSM, both impose artificial limits on resources and system functions). What has NOT been done (yet) is unlocking the system completely for tasks like USB interfacing, custom themes/system mods/plugins, and (fortunately) pirating games.

**What's [UVLoader](https://github.com/yifanlu/UVLoader) and how far along is it? **The last I've spoken, I was beginning work on UVL and asked for any help I could get. Even though, I did not really get help, I did find people who were interested in what I was doing and we exchanged information. I also want to brag that I finished the main functionalities of UVL in a couple of weeks, and it has been "done" for about three months now. (Quotes around "done" because I decided to not worry about some [features](https://github.com/yifanlu/UVLoader/wiki) yet). That means, I can basically load most (most being the few that I manually built without an open sdk) compiled homebrews. You can run your standard hello worlds and spinning cubes and such, but in theory, it should load any homebrew built.

**When's the release? What's taking so long? **So as I've said, the loader was done three months ago. I have a couple of reasons for not releasing yet. The main reason is that currently, there is no open SDK for compiling and linking Vita homebrew like [pspsdk](https://github.com/pspdev/pspsdk) did for the PSP. That means, even with the loader, it would be useless for users because there are no homebrew games, emulators, etc to run, and it would be useless for developers because they can't build homebrews either. So what's the progress on the open sdk? Zero, as I'm typing this right now. I have an idea of what it should look like and I spoke to a couple of people who are interested in helping, but so far, no code is written. Why is that? Because for me, I am very busy with lots of other unrelated things, and unfortunately, only me and a handful of other people know enough about the device and the executable format and etc to make the open sdk and none of us have the time currently.

The second reason is that having a Vita exploit at this stage (when it is really hard to find exploits) is very rare if not a once in a lifetime thing. Me and others I've talked to agree that right now it's more important to use this exploit to gather more information about the system in order to find more exploits and such than it is to run homebrews right now. We have PSM for homebrew games and PSP emulator for homebrew emulators, so there really isn't a huge demand for native PSVita homebrews yet. As I'll expand on below, we've only scratched the surface of Vita hacking and there's so much more to see.

**Are you looking for testers/can I test UVLoader? **There's no need to "test" UVLoader right now because, as I've stated before, there isn't any compiled homebrew and nothing to compile them anyways. Yes, UVL works with some of the custom still I've built manually, but it is unwise to write complex stuff without a working SDK.

**Can _I _help?** Depends who you are. If you're an _established _reverse engineer, you know how to contact me. If you just want to "beta test," read above. If you know any other way of helping me, don't ask, just do it™, since UVL is open source. Even though I don't accept monetary donations before I release anything, if you have access to broken Vitas, memory cards, games, etc, or any unused hardware reversing tools like logic analyzers; anything you wouldn't mind parting with, one of the things me and others involved don't have access to is funds for materials to test some of the more... risky ideas and if you could help with that respect, just use the contact link at the top of the page to get in touch with me.

**What needs to be done to "hack" the Vita? **Again, that term is very vague, but I know what you mean. This is the perfect time to describe (as far as I know) the Vita's security structure and what needs to be done at each level.


### PSP emulator


I'll start with the PSP emulator just because that is what's "hacked" right now. How much control do you have of the Vita when you use [vHBL](http://wololo.net/vhbl/)? Almost none. On the PSP itself, games are "sandboxed" (meaning some other process tells it what functions of the PSP can be used by the current game, main thing being that one game can't load another game). Because the Vita emulates the PSP, it also emulates this structure.


### PSP kernel


One level up, we have "kernel exploits" on the PSP, which means that we are no longer limited to what functions of the PSP we can use. Any PSP function that is emulated by the Vita can be used, that's why you see ISO loading as the main thing. However, all of this, the PSP emulator, sits in the Vita game sandbox. This sandbox is just like the PSP one, in that another Vita process tells the game (in this case, the PSP emulator running some PSP game) what Vita functions can be used in a similar fashion. For example, if a game doesn't explicitly declare that it's going to use the camera or bluetooth (and Sony approves), any code that tries to use these functions will crash.


### Vita userland


This is where UVLoader works; we exploited some game to run code inside it's sandbox, meaning that if that game doesn't have camera functions, no UVLoader Vita homebrew can use the camera either. This also means, of course, we can't load pirated Vita games and so on. A fun fact here is that, _in theory_, if someone finds an exploit in Kermit, the system inside the PSP emulator that talks to the Vita through a virtual serial port, they can run UVLoader in the process hosting the emulator (one level higher than a PSP kernel exploit), meaning they may be able to modify the emulator to have more RAM or faster CPU or etc. Another advantage of running UVLoader here is that because the PSP emulator has access to more Vita hardware than most games (bluetooth, camera, etc), homebrews could have more access too.

However, it's easier said than done. It's hard to appreciate  how hard it is to get a Vita userland exploit. Let's work backwards: we want to somehow run native ARM code, how? Well, the classic route is some stack smash. But wait, modern ARM processors have XN (eXecute Never), which is a feature that only allow code in memory to run at specific locations (these locations are determined by the kernel and are read only). Ok, we have some other choices here: heap overflows, ROP (google if you don't know), and so on (assuming you even know you got a working exploit, which in itself is hard to know without additional information; most "crashes" are useless), but all of these choices require that you know enough about the system to create a payload fitted for the system. That means, you need either a memory sniffer or somehow dump the memory. Well, let's rule out hardware memory sniffing since the Vita has the RAM on the same system-on-a-chip as the CPU. How do we dump the memory then? Usually, you need to run some code to dump the memory or do some kind of oracle attack on crashes or error messages or something. Option one only works if we hacked the system before, and the second one, AFAIK, won't work because the Vita doesn't give any information when it crashes. So how did I get the first userland exploit? I'll leave that as an exercise to the reader...


### Vita kernel (lv2?)


Vita userland is the most we have access right now and PSP kernel mode is the most that is public. What comes after? Remember all information at this point could be wrong and is based off of the little evidence I have currently. We are in the Vita sandbox right now, which means we can run homebrew, but we can't use functions that the game doesn't use (camera, bluetooth, USB, etc). We also can't modify the system (run Linux, change the theme, add plugins, etc). For those to work, we need to go one level up: the Vita kernel, which might be called lv2. Even with complete userland access, we can't even poke at the kernel. The kernel acts like a black box, providing functions to the system through syscalls. You pass input into these syscalls and it returns some output, without revealing how the output is created. The kernel's memory is separate from userland obviously, and even guessing what syscalls do (there's no names in the memory, only numbers) is a challenge. In order to hack the kernel, we have a problem that is very much like the one I've stated above about getting Vita userland, except with even more limitations. Again, there's the circular problem of needing a kernel RAM dump to inspect for exploits and requiring a kernel exploit to dump the RAM. Now, there's even less "places" to inspect (visually and programmatically). In order of likelihood, one of the following needs to happen before there's even a CHANCE of a kernel exploit: 1) Sony does something stupid like the PS3 keys leak, 2) we get REALLY lucky and basically stumble upon an exploit by just testing one of the several hundreds of syscalls with one of an infinite amount of different inputs, 3) some information leaks out from Sony HQ.

It's still unknown how much control we would have if kernel mode is compromised, but me and some others think that we MAY at least be able to do something like a homebrew enabler (HEN) that patches signature checks temporarily until reboot, allowing for homebrews with no sandbox limitations (access to camera, BT, etc) and POSSIBILITY system plugins and themes. It is very unlikely at any keys will be found at this point or being able to create or run a CFW.


### Hypervisor? (lv1?)


At this point, it is purely a thought experiment, as we literally have no information beyond what we THINK the kernel does. It is highly possible that there is a hypervisor that makes sure everything running is signed and the kernel isn't acting up and such. Getting to this would be EVEN HARDER than getting kernel, which I already think is impossible. Even at kernel, it seems to be over my skill limit, but this would definitely be above me, and someone with real skills would have to attack this. I'm thinking at least, decaps will have to be attempted here. If somehow this gets hacked, we may be able to run CFWs, but like the PS3 before the lv0, newer firmwares would not be able to be CFW'd until...


### Bootloader? (lv0?)


Again, only conjecture at this point, but this is the holy grail, the final boss. Once this is compromised, the Vita would be "hacked" in every sense of the word. We may never get here (and by never, I mean maybe 5-10 years, but I would most likely not be working on the Vita at this point). Here's is where I think the keys are stored. With this compromised, CFW of any past, present, or future firmwares could be created, and anything would be possible.


## Summary


I guess to summarize, the reason there's no release in the foreseeable future isn't just because I don't have time to make an sdk so there won't be homebrews to use even if UVL is released. Even if the SDK does get done, at this point, it would be more attractive to use the control we currently have, double down, and try to get more control. If the exploit is revealed prematurely, getting the game pulled, and the firmware patched, sure we may get a fast N64 emulator in a couple of months when somebody has the chance to write it (and at that point, most people might be enticed to upgrade anyways for new firmware features and PSN access), but we will have to start at square one (read above about finding userland exploits) before having another chance at exploring the full potential of the system. Deep down, I am a researcher, and would have more interest in reversing the system than I would at making a release for users just so I could be the "first". Like all gambles, I may end up with nothing, but that's a risk I'm willing to take.
