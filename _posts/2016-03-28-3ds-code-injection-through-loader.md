---
author: yifanlu
comments: true
date: 2016-03-28 06:38:48-06:00
layout: post
slug: 3ds-code-injection-through-loader
title: 3DS Code Injection through "Loader"
wordpress_id: 1335
categories:
- 3DS
- C
- Technical
tags:
- 3ds
- code
- cxi
- debugging
- emulator
- firmware
- injection
- kernel
- loader
- modules
- patch
- system
---

I've seen many CFWs (custom firmware; actually they're just modified firmwares) for the 3DS but there seems to be a lack of organization and design in most of them. I believe that without a proper framework for patching the system, writing mods for the 3DS is extremely difficult and usually requires an in depth knowledge of the system even to make simple modifications. So here I present a plan that I hope developers will pick up on and contribute to.


## 3DS System Firmware: An Introduction


The 3DS has a microkernel architecture. This means most functionalities are implemented as modules (running as separate processes) and IPC is used by modules to talk to one another. The kernel has limited functionality (basic memory management, threading and synchronization, and access to IO memory) and these basic functions are exposed to modules (where the main work is done). All modules run in userland, but the security is not equal. There are three categories of modules: system, applets, and applications. (This is not an official categorization, as different modules have permission settings based on service-call access, access to other modules, memory space, and so on. However, you can basically group modules by these security properties into these three groups.) System modules do the backend work: installing content, TLS stack, communication with Nintendo's servers, etc. Applets are system software the user "sees": home menu, notifications, Nintendo ID login, keyboard, etc. Applications are the rest: settings app, camera app, games, etc.

The reason why it's hard to write patches or mods for the Nintendo 3DS is because of this modularization of the system. When we [take control of the kernel](https://www.3dbrew.org/wiki/3DS_System_Flaws), patching system features is not as simple as finding the right address in memory and overwriting it. Because everything is in different processes (and each have their own virtual memory space), you either have to recognize the kernel layout and parse kernel objects and map the right memory addresses to patch ([NTR does this](https://gbatemp.net/threads/release-ntr-cfw-3-2-with-experimental-real-time-save-feature.385142/)) or you have to stupidly scan all of physical memory (slow and high risk of false positives) for the right thing to patch. There's additional complications if you're using a hack that reboots the firmware or requires patching a process that's currently not running: you have to know when the process launches and modify the code when it does. It is possible to insert hooks when you first gain control in order to run code when processes are launched. However, most of the CFWs out there does something insanely non-practical: it busy-loops in a separate thread and scans all of physical memory for the right thing to patch... ad infinitum.

So we want to do the following



	
  * Patch code for any module (system, applet, or application) we want easily

	
  * Ideally we can insert the hook early in the boot process so we can patch most modules BEFORE they start (an example would be to change the region of the system: once the CFG module is loaded, every other module gets the region information from CFG; so we can either patch CFG or patch every other module that uses it)

	
  * Everything should be fast and use minimal resources (linear scan of physical memory is NOT okay)

	
  * Ideally the code should not be overly dependent on offsets--you should not have to compile the code separately for each firmware version


![3ds-boot](/images/2016/03/3ds-boot.png){: .alignleft}
Luckily there is a place in the boot-chain we can patch that does just that. The "loader" module is in charge of parsing the CXI executable format and loading the code into memory. That means that if we change "loader" to patch the code after it is read from the CXI file (and decompressed) but before it signals to the kernel to run (schedule) it, we can patch most modules in the system. The kernel bootstraps the five main modules: fs (file IO), loader (CXI loader), pm (process manager), sm (IPC), and pxi (talks to security processor) and then pm uses loader to start the rest of the modules. Other modules can start processes using pm, which in turn uses loader.

So loader is a good place to start if we want flexibility in patching the system and it fits the bill for what we require in loading patches, but there are some complications. First, the loader module does not have permission to access the SD card (where we want to use to store patches). Second, and more importantly, the code to read and parse patches from the SD card and run them when a process is being loaded seems like a good amount of code. If we wish to insert it into the loader module, we might have to move stuff around--a big pain with object code. So we need a better plan than just hijacking "loader."


> [2016/01/19 04:15:01] shinyquag: You could probably code your own "loader", I'd wager it's probably doable

Shinyquag would have won the wager because, luckily, loader is also the smallest module on the system. It has a couple of commands and no complicated logic. It's small enough (20KB) to completely reverse in a day. Once we rewrite the module, adding functionality is a trivial matter. In fact, since Nintendo uses a lot of boilerplate code so our loader implementation would actually be even more lean. Finally, since this module has not changed much since 2.x, we can likely use it on future firmwares without much changes. However, just because it is doable, does not mean it will be easy...


## Debugging without a Debugger


First I reversed the module and rewrote it in C. Then it got difficult. Since this was the first time anyone tried to build a sysmodule with the homebrew development environment, I had to make changes to [many](https://github.com/smealum/ctrulib/pull/253) [parts](https://github.com/profi200/Project_CTR/pull/31) of the [toolchain](https://github.com/devkitPro/buildscripts/pull/9). Then there's the problem of testing: we're replacing a key component of the system very early in the boot process. There's no iterative development since we cannot isolate the module without ending up rewriting the entire operating system. So, I just wrote the entire module in one sitting and hoped for the best. Unfortunately the best is a black screen and we don't have access to anything like gdb or even "printf". The only output we have is that the console boots or it does not boot. After re-reading my code over and over again and fixing bugs each time until I can't spot anything else obvious, I've discovered, thanks to #3dsdev, that prayer was not the only means of debugging this.

Enter [XDS](https://github.com/ichfly/XDS), the 3DS emulator we all forgot about. ichfly managed to get it to boot the 3DS home menu a while ago, but development has since ceased (likely because of other more promising emulators in the works). There's one thing XDS does that the other 3DS emulators does not do is that XDS fully emulates the 5 sysmodules bootstrapped by the kernel (the other guys use HLE). This is perfect because we only need to run "loader" in the boot process and let it interact with the other bootstrapped modules until enough modules are loaded by "loader" that we can consider it working. After [porting XDS to OSX](https://github.com/yifanlu/XDS), I was able to make use of the generous debug logs (comparing the IPC requests/responses from my loader with stock) to find the final bugs in my loader and get it to boot on the 3DS.

Our loader can now correctly duplicate the functionality of the original but ultimately, we wish to add more features. The most important one is SD card reading. The way that access permissions work is that each module specifies in its CXI header what filesystem devices it has access to. However, in the case of bootstrapped modules, the access fields are ignored. So in order to get "loader" the right permissions to read the SD card, it is not enough to modify the CXI headers. We need to talk with the "fs" module and request the permissions directly (this is what "pm" does with other modules after loader puts the code into memory). This involves reversing enough of the "fs" module to figure out how the permissions are stored and then making the right requests to "fs" to give "loader" those permissions. Since XDS does not emulate the SD card, I had to find a different avenue to test my code. What I decided to do was modify the URL to the Themes Store in the Home Menu to point to my server + the return value of the file IO calls. Then I would load up the Themes Store to find the return value and look through my reversed object code to see what caused the error. This worked well because I did not want to add more code (e.g: framebuffer or networking) just for debugging. (Who debugs the debugger?) With SD card access, future debugging is much easier because we can just write logs to the SD card.


## Loader Based CFWs


I believe that using a custom "loader" will make it much easier to write mods for the 3DS. We could see hacks such as a "Homebrew" button in the Home Menu or custom keyboards or custom themes outside of what Nintendo officially supports. We might also see hacks for games similar to [HANS](https://github.com/smealum/HANS) but without requiring access to a dump of the game. I hope 3DS developers will pick up on this and make cool mods and hacks for the system.

The code can be found [here](https://github.com/yifanlu/3ds_injector/tree/master). Developers should modify patcher.c and replace it with their own implementation.
