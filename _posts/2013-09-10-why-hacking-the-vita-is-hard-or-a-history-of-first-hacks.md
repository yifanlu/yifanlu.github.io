---
author: yifanlu
comments: true
date: 2013-09-10 05:28:53-06:00
layout: post
slug: why-hacking-the-vita-is-hard-or-a-history-of-first-hacks
title: 'Why hacking the Vita is hard (or: a history of first hacks)'
wordpress_id: 629
categories:
- Information
- PS Vita
tags:
- devices
- exploit
- hack
- hackers
- hardware
- security
- vita
---

It's been about a year since I revealed the [first userland Vita exploit](/2012/12/12/playstation-vita-the-progress-and-the-plan/) and I still occasionally get messages asking "what happened" (not much) or "when can I play my downloaded games" (hopefully never) or "I want homebrew" (me too). While I don't have anything new exploitwise (same problems as before: no open SDK, lack of interest in the development community, lack of time on my part), I do want to take the time and go over why it's taking so long.<!-- more -->


### Where are the hackers?


A common (and valid) complaint I hear is that there is a lack of hackers (a word I hate) working on the Vita. The fail0verflow team has [a great post about console hacking](http://fail0verflow.com/blog/2013/espresso.html) that applies just as well to the Vita. In short, there isn't as much value to hacking a console now than before. Not too long ago, the PSP and DS were the only portable device people owned that plays games and, for many people, the only portable device they owned period. I had a DS Lite that I carried everywhere long before I had a smartphone. But then I got a smartphone (and so did everyone else). iPhones and Androids (and don't forget Windows Phone) are the perfect platform for what we used to call homebrew. Indie developers who wanted to write a portable game no longer has to use a hacked PSP and an open SDK. Writing apps is much easier and much more profitable. Meanwhile users can play all the emulators they want on their Android phone or their jailbroken iPhone. The demand for hacked consoles shrunk dramatically with those two audiences gone. Plus with smartphones gaining a larger audience while the Vita barely sells (which by the way is a tragedy since it's a pretty awesome console), a hacker can get a lot more attention (for for those who seek "donations", a lot more money) spending time rooting phones that are coming out every month.


### But [insert device here] was hacked very quickly, we just need more people working, right?


To some extent, that is true, but even with a large group of talented reverse engineers, I would not bet that the Vita would be hacked any time soon. To be clear here, when I say "hacked," I refer to completely owning the device to the point that decryption keys are found and unsigned code can be run in kernel mode (or beyond). The problem is that even talented reverse engineers (who can read assembly code and find exploits) are out of luck when they don't have the code to work with. I mentioned this circular problem before, but to restate it: you need to have access to the code before you can exploit it, and to get access to the code, you need to exploit it. But, if that's the case, you ask, how would any device ever be hacked? That is why I believe that the first (real) hack of any device is the most important. Let's look at some examples of "first hacks" and see why it doesn't work with the Vita.


#### Insecure First Version


This is the most common situation. Let's look at the PSP. The 1.00 firmware ran unsigned code out of the box. Someone found a way to access the filesystem, and saw that the kernel modules were unencrypted. They analyzed the kernel modules and found an exploit and owned the system. All it takes is to have an unreleased kernel exploit from one firmware version; then update to the next one; exploit it and dump the new kernel to find more exploits. Rinse and repeat.

Same with the iPhone. The first version(s) allowed you to read from the filesystem through iBoot. It was a matter of dumping the filesystem, analyzing the (unencrypted) binaries, and creating exploits. Plus, the kernel is from the same codebase as OSX, so analyzing it was not as difficult as looking at a new codebase.

The Vita however, has a fairly secure original firmware. No filesystem access (even to the memory card), proper encryption of things that do come out of the device, and very little areas of interaction in general (you have CMA and that's pretty much it).


#### Similarities to other Devices


Most Android phones fall into this category. One Android root will most likely work across multiple manufactures. Plus, Android is open source, so it's a matter of searching for an exploit. Once the device is rooted, someone has to find a way to dump the bootloader (which for many phones is just a matter of reading from a /dev/ endpoint), and analyze the bootloader for a way to root it.

The Kindle Touch (which I was the first to jailbreak), ran essentially the same software as the Kindle 3 and had a debugging console port.

The Vita has similarities to the PSP, but most of the system is different. With multitasking support, the Vita memory model is completely different from PSP and has proper abstraction of virtual memory. The Vita has NetBSD code, but the kernel is completely proprietary. No PSP exploit will work on the Vita.


#### Hardware Methods


This is usually the "last resort" because it takes the most skill and money to perform. This usually involves physically dumping the RAM with hardware to analyze the code. The most recently hacked console, 3DS had this done. I believe the first Wii hack was developed with a hardware RAM dumper. Many consoles had some kind of hardware analyzing done before the first hack is developed.

It would be very hard to do a hardware hack on the Vita. The system memory is on the same chip as the CPU, so you cannot try to piggyback the RAM. Plus anyone doing a hardware hack would have to have expert electrical engineering skills and access to expensive tools.



The story always starts with getting access to the code, then finding an exploit, and then using that exploit to get more code to find more exploits in the future. Most of the jailbreaks, roots, and hacks you see are developed with information gathered from a previous hack. I believe that Sony knows this and really made sure that their device does not suffer any of the flaws I listed. Lots of people make fun of Sony for not handing security well, but after spending countless hours on the Vita, I could honestly say that the Vita is one of the most secure devices I've ever seen. So far, they seem to have done everything well; using all the security features in modern computers and not trusting any code. But, as we learned countless times, nothing is completely secure.

EDIT: I'm seeing a lot of comments speculating that Vita slim or Vita TV may help hacking it. In my opinion, this is grasping at straws. There are no evidence that a minor revision of the console will magically create software or hardware holes.
