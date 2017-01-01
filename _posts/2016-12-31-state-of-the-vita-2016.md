---
author: yifanlu
comments: true
date: 2016-12-31
layout: post
slug: state-of-the-vita-2016
title: "State of the Vita 2016"
categories:
- Vita
tags:
- vita
- henkaku
- taihen
- hacking
---

Although it hasn't been a good year for all of us, 2016 was a great year for the Vita. In [August]({% post_url 2016-07-28-henkaku-vita-homebrew-for-everyone %}), molecule released the first user-friendly Vita hack which builds on four years of research and a year of building a SDK platform from scratch. Since then, we saw dozens of homebrews, new hackers showing up in the scene, and the creation of a community that I am proud to be a part of. In [November]({% post_url 2016-11-01-taihen-cfw-framework-for-ps-vita %}), I released taiHEN, a CFW framework that makes it easy to extend the system and to port future hacks. As such, it was a busy year for molecule. We are a team of five individuals and we served as pen testers, exploit writers, web developers, UI designers, web masters, IT, moderators, PR, recruiters, software architects, firmware developers, support, and lawyers for the Vita hacking community. These are roles we took out of necessity because Vita hacking is such a niche interest. However, these are not roles we can hold forever. Back in November, I said that I (and I am assuming the rest of molecule but I do not speak for them) would retire from the scene after taiHENkaku was stable enough and that time has finally come. Aside from a parting gift from Davee that should be released in a couple of days we will be retiring from all non-research tasks. Since we entered the scene with no drama, no bullshit, and no corruption, we will leave in the same manner. Firstly, all our work are either already [open sourced](https://github.com/henkaku) or are in the process of being tidied up and released. Second, we have [extensively documented](https://wiki.henkaku.xyz/) all our findings on the Vita with the exception of our TrustZone (lv1) hacks which we left out at the request of other hackers who wish to try the challenge without aid. Lastly, we revamped the process for [setting up development](https://vitasdk.org) and making homebrew is easier than ever. Fixing the toolchain required a lot of boring and tedious work and I want to thank everyone who helped with the process. I am proud that our toolchain is the only unofficial toolchain that was [designed](https://wiki.henkaku.xyz/vita/File:Vita_SDK_specifications.pdf) rather than hacked together.

# The community

We leave the rest to you, the hacking community. We hope [HENkaku](https://github.com/henkaku/henkaku) will be ported to other firmwares. We hope that [taiHEN](https://github.com/yifanlu/taiHEN) will be used to make spectacular extensions. We hope that someone will make a debugger for the SDK. We hope someone will find a way to dump the latest firmware and enable PSN spoofing. We **[opened a new forums](https://forums.vitasdk.org)** for Vita developers and hackers alike to share ideas and creations. I know realistically, because of how small the user base is, we will not have the level of activity that exists on the 3DS or iOS jailbreaking community. But nevertheless, I am thankful for everybody who has participated in Vita hacking.

# What is left

There are four distinct security levels on the Vita. Userland, kernel (lv2), TrustZone (lv1), and F00D (lv0). We have hacked the first three levels, but owning F00D is particularly challenging. It uses a proprietary [instruction set](https://github.com/yifanlu/toshiba-mep-idp) and an architecture that is severely underdocumented. It has minimal attack surface, and we can't see any code that runs on it because of multiple levels of encryption. Even if we get a crash through fuzzing, it is unclear how we can exploit any vulnerability. Hardware attacks are not useful here because we don't have any control of the code running on it (typically hardware attacks involve escalating the privilege of running code). Attacking F00D will be my only focus in Vita hacking at this point and I welcome anyone who wants to help me in this journey.

## Hardware mods

If you are a skilled hardware hacker and can wire together an external eMMC flasher for the Vita or PS TV, please contact me. I am willing to pay for such a device and it would help speed up fuzzing efforts. The pinouts can be found [here]({% post_url 2014-01-04-ps-vita-nand-pinout-updated %}). The problem is there there are no test points for the eMMC (unlike most other devices) so the only way to get access is by cutting the trace or by soldering to the tiny (~0.5mm) noise reducing resistors next to the CPU. I believe that replacing these resistors with solder bridges would be safe. Then external wires can be soldered onto the bridge and connected to some port that we can drill into the case. However, the scale of all this is beyond my skills and equipment. If you know anyone who can help with this, please forward this request to them. It would be an immense service to molecule and the Vita hacking scene.

# Final Words

In this day and age when hacking has been politicized, fetishized, and commoditized, we should remember [where hacking came from](http://phrack.org/issues/7/3.html). Hacking is about freedom of knowledge not an ego contest about who knows what. Hacking is about control over the devices we own by us not control by other hackers. Hacking is about fun and exploration and challenges not about showing off and making profits. As our skills becomes ever more relevant for the connected world and generates power and revenue for many organizations, it is easy to forget that. But luckily for us, we are Vita hackers. Nobody has ever profited off the Vita.
