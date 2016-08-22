---
author: yifanlu
comments: true
date: 2015-06-05 18:10:14-06:00
layout: post
slug: secure-your-emmc-devices
title: Secure your eMMC devices!
wordpress_id: 1179
categories:
- Information
tags:
- arm
- bootloader
- emmc
- kernel
- root
- security
---

Most of our embedded devices use eMMC, but security into eMMC (as far as I know) has not been extensively studied or taken account of in threat models. In the small sample of devices I've looked at, the ability to send raw commands to the eMMC only requires kernel access. If you look at the Android platform, kernel hacks are not uncommon and remote kernel hacks are also not a rarity. There are certain commands that a hacker can send which can permanently disable (brick) a device.<!-- more -->

**Permanent write protect** can be enabled on the main partition or any of the boot partitions. A malicious entity with kernel access to the device can wipe the bootloader and enable permanent write protect and prevent any (even hardware based) recovery.

**Card lock** will lock the eMMC with a password. No data would be accessible without the password. An adversary could hold your data for hostage and demand payment for a hardware unlocking solution.

This is not meant to be an exhaustive list. There are other attack vectors to watch out for too. For example, if the device uses the RPMB to prevent downgrades, an attacker may corrupt it and prevent boot. The main takeaway here is that having raw eMMC access is dangerous.

So far, the only eMMC based attack in the wild was from a piracy dongle on the 3DS called [Gateway](https://gbatemp.net/threads/warning-gateway-team-bricks-card-on-purpose.360568/). This only affected a small number of users who choose to hack their 3DS with Gateway. However, in the future, we may see an expansion of these kinds of attacks on mobile devices.



### Solutions



The eMMC specifications provides a means of disabling any potentially destructive features either at power-on or permanently. If disabled at "power-on" then it is the trusted bootloader's prerogative to disable destructive features so when the untrusted kernel is compromised, the attacker won't be able to use any destructive features. Care must be taken in implementing this solution as reset attacks have to be prevented. Also, this will not prevent any hardware based bricking, although it is unlikely that such a thing can happen remotely.

Permanent disable of features like card lock and write protect will prevent the commands for enabling them from ever working. Device manufacturers who choose to do this must be sure that they would never need to use such a feature in the future. This is the safest solution.

Another potential solution is to treat the eMMC as a secure device under an ARM TrustZone implementation, but this would be specific to ARM based systems.

As a user who wants to secure their eMMC based devices, you can obtain root access and run the permanent disable commands, although such a task is obviously dangerous and should only be attempted by someone with knowledge of eMMC. I hope to write an Android app at some point that can perform this operation, although it is very low priority for me at this point.

Overall, I think more research should be done on the topic of eMMC security. I only have a small sample of devices that I've tested and found the disable commands for the destructive actions are not used (therefore an attacker with kernel access can run them). Someone with the resources should perform a study to see how many percentage of devices in the wild has the potential to be compromised with an eMMC attack.
