---
author: yifanlu
comments: true
date: 2014-01-11 05:56:44-06:00
layout: post
slug: dumping-the-vita-nand
title: Dumping the Vita NAND
wordpress_id: 720
categories:
- PS Vita
- Technical
- Vita Hardware Hacking
tags:
- binary
- emmc
- hacking
- hardware
- logic
- logic board
- nand
- packet
- usb
- vita
---

When we [last left off](/2013/12/30/updates-on-the-vita-hardware-hacking-project/), I had spent an excess of 100 hours (I'm not exaggerating since that entire time I was working, I listened to [This American Life](http://www.thisamericanlife.org/) and went through over a hundred one-hour episodes) soldering and tinkering with the Vita logic board to try to dump the eMMC. I said I was going to buy a eMMC socket from taobao (the socket would have let me clamp a eMMC chip down while pins stick out, allowing the pressure to create a connection) however, I found out that all the sellers of the eMMC socket from taobao don't ship to the USA and American retailers sell the sockets for $300 (cheapest I could find). So I took another approach.<!-- more -->


### Packet Sniffing


My first hypothesis on why it is not working is that there's some special initialization command that the eMMC requires. For example, CMD42 of the MMC protocol allows password protection on the chip. Another possibility was that the chip resets into boot mode, which the SD card reader doesn't understand. To clear any doubts, I connected CLK, CMD, and DAT0 to my [Saleae Logic](http://saleae.com) clone I got from eBay.

[![Vita eMMC points connected to logic analyzer.](/images/2014/01/IMG_0613-300x225.jpg)](/images/2014/01/IMG_0613.jpg)

Vita eMMC points connected to logic analyzer.
{: .wp-caption-text}



As you can see from the setup, I had the right controller board attached so I can get a power indicator light (not required, but useful). I also took the power button out of the case and attached it directly. The battery must be attached for the Vita to turn on. Everything is Scotch-taped to the table so it won't move around. Once all that is done, I captured the Vita's eMMC traffic on startup.

[![First command sent to eMMC on startup](/images/2014/01/Screen-Shot-2014-01-10-at-8.59.25-PM-300x149.png)](/images/2014/01/Screen-Shot-2014-01-10-at-8.59.25-PM.png)

First command sent to eMMC on startup
{: .wp-caption-text}



After reading the 200 paged [specifications](http://rere.qmqm.pl/~mirq/JESD84-A44.pdf) on eMMC, I understood the protocol and knew what I was looking at. The very first command sent to the Vita is CMD0 with argument 0x00000000 (GO_IDLE_STATE). This is significant for two reasons. First, we know that the Vita does NOT use the eMMC's boot features. The Vita does not have its first stage bootloader on the eMMC, and boots either from (most likely) an on-chip ROM or (much less likely) some other chip (that mystery chip on the other side maybe?). Second, it means that there's no trickery; the eMMC is placed directly into Idle mode, which is what SD cards go into when they are inserted into a computer. This also means that the first data read from the eMMC is in the user partition (not boot partition), so the second or third stage loader must be in the user partition of the eMMC. For the unfamiliar, the user partition is the "normal" data that you can see at any point while the boot partition is a special partition only exposed in boot mode (and AFAIK, not supported by any USB SD card reader). Because I don't see the boot partition used, I never bothered to try to dump it.


### Dumping


I tried a dozen times last week on two separate Vita logic boards trying to dump the NAND with no luck. Now that I'm on my third (and final) Vita, I decided to try something different. First, I did not remove the resistors sitting between the SoC and eMMC this time. This is because I wanted to capture the traffic (see above) and also because I am much better at soldering now and the tiny points doesn't scare me anymore. Second, because of my better understanding of the MMC protocol (from the 200 page manual I read), I no longer attempted to solder DAT1-DAT3 because that takes more time and gives more chance of error due to bad connections. I only connected CLK, CMD, and DAT0. I know that on startup, the eMMC is placed automatically into 1-bit read mode and must be switched to 4-bit (DAT0-DAT3) or 8-bit (DAT0-DAT7) read mode after initialization. My hypothesis is that there must be an SD card reader that followed the specification's recommendation and dynamically choose the bus width based on how many wires can be read correctly (I also guessed that most readers don't do this because SD cards always have four data pins). To test this, I took a working SD card, and insulated the pins for DAT1-DAT3 with tape. I had three SD card readers and the third one worked! I know that that reader can operate in 1-bit mode, so I took it apart and connected it to the Vita (CLK, CMD, DAT0, and ground).

[![As you can see, more tape was used to secure the reader.](/images/2014/01/IMG_0614-300x225.jpg)](/images/2014/01/IMG_0614.jpg)

As you can see, more tape was used to secure the reader.
{: .wp-caption-text}



I plugged it into the computer and... nothing. I also see that the LED read indicator on the reader was not on and a multimeter shows that the reader was not outputting any power either. That's weird. I then put a working SD card in and the LED light turned on. I had an idea. I took the SD card and insulated every pin except Vdd and Vss/GND (taped over every pin) and inserted the SD card into the reader. The LED light came on. I guess there's an internal switch that gets turned on when it detects a card is inserted because it tries to draw power (I'm not hooking up Vdd/Vss to the Vita because that's more wires and I needed a 1.8V source for the controller and it's just a lot of mess; I'm using the Vita's own voltage source to power the eMMC). I then turned on the Vita, and from the flashing LED read light, I knew it was successful.

[![LED is on and eMMC is being read](/images/2014/01/IMG_0615-300x225.jpg)](/images/2014/01/IMG_0615.jpg)

LED is on and eMMC is being read
{: .wp-caption-text}




### Analyzing the NAND


Here's what OSX has to say about the eMMC:


> Product ID: 0x4082
> Vendor ID: 0x1e3d (Chipsbrand Technologies (HK) Co., Limited)
> Version: 1.00
> Serial Number: 013244704081
> Speed: Up to 480 Mb/sec
> Manufacturer: ChipsBnk
> Location ID: 0x1d110000 / 6
> Current Available (mA): 500
> Current Required (mA): 100
> Capacity: 3.78 GB (3,779,067,904 bytes)
> Removable Media: Yes
> Detachable Drive: Yes
> BSD Name: disk2
> Partition Map Type: Unknown
> S.M.A.R.T. status: Not Supported


I used good-old "dd" to copy the entire /dev/rdisk2 to a file. It took around one and a half hours to read (1-bit mode is very slow) the entire eMMC. I opened it up in a hex editor and as expected the NAND is completely encrypted. To verify, I ran a histogram on the dump and got the following result: 78.683% byte 0xFF and almost exactly 00.084% for every other byte. 0xFF blocks indicate free space and such an even distribution of all the other bytes means that the file system is completely encrypted. For good measure, I also ran "strings" on it and could not find any readable text. If we assume that there's a 78.600% free space on the NAND (given 0xFF indicates free space and we have an even distribution of encrypted bytes in non-free space), that means that 808.70MB of the NAND is used. That's a pretty hefty operating system in comparison to PSP's 21MB flash0.


### What's Next


It wasn't a surprise that the eMMC is completely encrypted. That's what everyone suspected for a while. What would have been surprising is if it WASN'T encrypted, and that tiny hope was what fueled this project. We now know for a fact that modifying the NAND is not a viable way to hack the device, and it's always good to know something for sure. For me, I learned a great deal about hardware and soldering and interfaces, so on my free time, I'll be looking into other things like the video output, the mystery connector, the memory card, and the game cards. I've also sent the SoC and the two eMMC chips I removed to someone for decapping, so we'll see how that goes once the process is done. Meanwhile, I'll also work more with software and try some ideas I picked up from the [WiiU 30C3 talk](http://www.youtube.com/watch?v=hZRz0xikaAU). Thanks again to everyone who contributed and helped fund this project!


### Accounting


In the sprit of openness, here's all the money I've received and spent in the duration of this hardware hacking project:

**Collected: **$110 WePay, $327.87 PayPal, and 0.1BTC


#### Assets


Logic Analyzer: $7.85
Broken Vita logic board: $15.95
VitaTV x 2 (another for a respected hacker): $211.82
Rework station: $80
Broken 3G Vita: $31
Shipping for Chips to be decapped: $1.86

**Total:** $348.48 (I estimated/asked for $380)

I said I will donate the remaining money to EFF. I exchanged the 0.1BTC to USD and am waiting for mtgox to verify my account so I can withdraw it. $70 of donations will not be given to the EFF by the request of the donor(s). I donated $25 to the EFF on January 10, 2014, 9:52 pm and will donate the 0.1BTC when mtgox verifies my account (this was before I knew that EFF takes BTC directly).
