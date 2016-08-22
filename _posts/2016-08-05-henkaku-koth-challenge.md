---
author: yifanlu
comments: true
date: 2016-08-05
layout: post
slug: henkaku-koth-challenge
title: HENkaku KOTH Challenge
wordpress_id: 1493
categories:
- Announcements
- PS Vita
tags:
- ctf
- hacking
- henkaku
- koth
- molecule
- release
- source
- vita
---

We released [HENkaku](/2016/07/28/henkaku-vita-homebrew-for-everyone/) a week ago and were blown away by the reception. There has been over 25k unique installs and every day new homebrew are being announced. This is all thanks to those who contributed to the SDK project back when [Rejuvenate](/2015/06/14/rejuvenate-native-homebrew-for-psvita/) was announced. Without a working toolchain for developers and a couple of working homebrews at the time of HENkaku's launch, I doubt the reception would have been as popular.

Since the release, there have been a couple of questions we've been getting over and over again: When will this work on older firmware versions? How does HENkaku work? Where is the source code? I am going to address these questions in a bit. First, I want to thank Sony. It is common for hackers to laugh and poke fun at companies on the receiving end of hacks. But I think that's unfair--security issues are a learning experience for all sides and we should all be thankful for it. For myself, I started my work on the Vita since its North America release in 2012. Although Davee beat me in hacking the PSP compatibility mode and getting ROP on WebKit, I was the first to run native code and [dump the memory](https://www.youtube.com/watch?v=w1GICNXTOhM) through PSM. Since then, Davee, Proxima, I, and later xyz (collectively "molecule") have been working on the Vita on and off through the years. It is a tremendous learning experience both working with these smart individuals and getting my hands dirty with real world hacks. I think I owe a large portion of what I know about security due to my work on the Vita. It has, hands down, the most well designed security infrastructure of any consumer electronics device. In 2012, the iPhone, Android, and 3DS were no match. Even today, I think the Vita rivals the security of devices in the market.

There's no single reason that led me to this conclusion, but there's a couple of factors. First, the Vita has really good security-in-depth: multiple layers of abstraction, exploit mitigation, proper input sanitation, etc. Second, the software and firmware are mostly proprietary. Now that's interesting because usually this is usually a point against security: trust the audited code of the open source community because rolling your own features will expose you to more bugs. However, in their case, this worked in Sony's favor. They managed to not make any major security mistakes (I hypothesize that they hired an external security firm to audit their code) and this made it harder for us to put a foot in the door (because we have nothing to go on). While known WebKit exploits provide a common way into a new device, the Vita is unlike the PS4 where we can exploit known FreeBSD9 bugs on an older firmware to get higher privileges. The calculated risk they took in using proprietary code paid off since nobody has been able to decrypt their firmware files yet--and until someone does, it is unlikely that anyone would write any advanced exploit code. However, the risk is that if their code is indeed buggy, then once the floodgates open (someone finds a single exploit), there is no closing it (all the bugs will be found). Finally, the Vita is not exposed to hardware attacks simply because it would be too expensive to perform. Unlike the 3DS, the Vita's RAM is on the same chip as the CPU so we cannot dump the contents through external hardware without access to a sophisticated lab and experienced technicians. That means as long as someone doesn't dump the memory, because of the exploit mitigation features, it would be extremely difficult to find vulnerabilities and exploit them. However, it is very difficult to dump the memory because we do not have the funds to do it with hardware and must resort to exploiting the software. But then we have a chicken-and-egg problem.

All this is to say that we of team molecule wish to share our learning experience with the rest of you. We feel that the Vita has been neglected by hackers because of it's unpopularity. However, they are missing out on a great challenge. The barrier of entry has been lowered since you can buy a PS TV for less than $50 USD. Don't take my word for it, take a stab at it yourself and see if the device is really secure or if I'm just too inexperienced.



## KOTH Challenge



[CTF](https://en.wikipedia.org/wiki/Capture_the_flag#Computer_security) challenges are common in the hacking community. The goal is to hack a system in a controlled environment to get a "flag" and is a fun and educational experience. I highly recommend it to anyone interested in security. We are hosting a variation of this challenge. The first king-of-the-hill challenge will take place on Vita Island.

The idea is as follows: we (molecule) are currently the kings of the hill. You (challenger) can claim the throne by reversing our hack (HENkaku) and explaining it. Once we have been knocked off, we will post all our source code, build scripts, and a special bonus... We won't say what it is yet, but it can be claimed by anyone who beats the challenge (not just the first) and is only valuable to people who have an interest in the Vita and Vita hacking. Since all the "prizes" are available to everyone and not just the first, we strongly encourage collaboration.

To make the challenge as interesting as possible, we used minimal obfuscation in our code. The goal isn't to see who can write the best deobfuscation tool but to invite all the skilled security researchers of the world to look at what we believe is one of the most secure device on the market today. Therefore most of the difficulties in the challenge will be posed by the system and not us.



## Releases



The source for HENkaku will be released in parts. Today, we released the [files for offline hosting](https://github.com/henkaku/henkaku/releases). This allows the challengers to start in reversing our code and also allows for anyone to mirror HENkaku. It also allows those with slow or intermittent internet access to use HENkaku.

Next, when someone completely reverses the second stage ROP and explains properly how it works, we will release the source code up to that point as it might aid in the next part. I don't think it would take more than a couple of weeks for someone to get to this point. Some questions to be thinking about are: how do we manage to run unsigned code? do we get kernel access? if so, how? if not, what other ways are there?

Finally, when someone figures out the entire HENkaku installation process, we will release all our source and tools. I hope this would be done in no longer than a couple of months (if interest takes off) however it may take a year (if there is minimal interest). I'm not going to hold the HENkaku sources for hostage, so if there is no interest for a long time, I'll reevaluate the options.

Until then, molecule will be taking a break from hacking for an indeterminate amount of time. We will still maintain HENkaku and post fixes from time to time. However, we will not be actively working so we won't be able to port HENkaku to lower firmware versions. For me this is because the amount of free time I have is slowly diminishing and I have other things to do. I hope I have inspired others to take up on hacking the Vita so molecule won't be the only people to hack it. My hope is that in a year, HENkaku would no longer be needed and molecule can quietly retire.
