---
author: yifanlu
comments: true
date: 2017-08-22
layout: post
slug: psvsd-custom-vita-microsd-card-adapter
title: "psvsd: Custom Vita microSD card adapter"
categories:
- Vita
tags:
- vita
- henkaku
- ensō
- hardware
- hacking
- pcb
---

One thing I love about Vita hacking is the depth of it. After investing so much time reverse engineering the software and hardware, you think you would run out of things to hack. Each loose end leads to another month long project. This all started in the development of [HENkaku Ensō](https://enso.henkaku.xyz/). We wanted an easy way to print debug statements early in boot. UART was a good candidate because the device initialization is very simple and the protocol is standard. The Vita SoC (likely called _Kermit_ internally as we'll see later on) has seven UART ports. However, it is unlikely they are all hooked up on a retail console. After digging through the kernel code, I found that `bbmc.skprx`, the 3G modem driver contain references to UART. After a trusty [FCC search](https://apps.fcc.gov/oetcf/eas/reports/ViewExhibitReport.cfm?mode=Exhibits&RequestTimeout=500&calledFromFrame=N&application_id=8WJilWr8O8ec1EzV2bqcsQ%3D%3D&fcc_id=QDJZOE), it turns out that the Vita's 3G modem uses a mini-PCIe connector but with a custom pin layout and a custom form factor. The datasheet gives some useful description for each pin, and `UART_KERMIT` seemed like the most likely candidate (there's also `UART_SYSCON` which is connected to the SCEI chip on the bottom of the board, which serves as a system controller and a `UART_EXT` which is not hooked up on the Vita side). So finding a debug output port was a success, but with the datasheet in front of me, the USB port caught my attention. Wouldn't it be neat to put in a custom USB device?

## On USB in the Vita

A quick aside on the various USB ports found in the different models of the Vita.

  * The top port on OLED models (commonly referred to as the "mystery port" and incorrectly referred to as a "hidden video out port") is a USB host. It is unknown if the port is enabled by default or how to enable it.
  * The bottom port on OLED models (sometimes called the "multiconnector") supports UDC (USB client) but can also enable USB host support. It is unknown how this switch is controlled, but I'm guessing the syscon is involved and it's likely USB OTG.
  * The microUSB port on LCD models have the ID pin connected which implies support for USB OTG or something like it. However, it is unknown how to activate this feature.
  * There is a USB type A port on PS TV.
  * There is also a USB to Ethernet chip in the PS TV for the Ethernet port that is connected to Kermit via USB.
  * The audio codec chip is connected to Kermit via USB for all models.
  * and of course, the 3G modem on OLED models is connected by USB. On Wifi only models, VDD to the unfilled mini-PCIe pad is missing a bridge. The USB D+/D- signals are also missing a ferrite bead under the adjacent shield. It is unknown if bridging these three locations will enable the USB port on wifi models or if extra work is needed.

## Designing a microSD adapter

In order to become more familiar with hardware design as well as understand how USB works on the Vita, I thought it would be fun to create a custom Vita USB device that fits on the modem port. The main reason I chose this port aside from the other USB ports is that it is the easiest to build. It is just a matter of designing and fabricating a PCB, which is simple to do. In comparison, connecting to any of the other USB ports would require creating custom adapters, molding plastic, and dealing with mechanical issues. Creating an adapter for the external ports is also not exactly a usable solution as the Vita is supposed to be portable, and having to dangle a USB port is not something most people are willing to do. In addition, my custom Vita modem card can expose the UART port to work as a console output device (which started this whole project). For this first project, I wanted to build a microSD adapter. Vita memory cards are notoriously expensive, with 32GB cards retailing for $79.99 USD. In comparison, a microSD card with similar performance and capacity goes for $12 USD. Therefore, it would be immensely useful to use microSD cards as a USB storage replacement for the proprietary Vita memory cards.

### Choosing parts

SD to USB ICs are pretty cheap and common--you find them in any USB SD adapter. A quick research shows that most cheap adapters use an Alcor or Genesys chip. There is also the MAX14500 series chip from Maxim that is no longer in production and the Microchip USB2244 chip. The documentation for the cheap Asia manufactured chips were lacking so I went with the USB2244 even though it is more expensive (I don't plan to mass produce it anyways). Microchip provides good documentation in comparison, complete with layout guidelines and a reference design. Unfortunately, I can't find an Eagle library for the USB2244 so I had to design it myself (using [Sparkfun's tutorial](https://learn.sparkfun.com/tutorials/designing-pcbs-smd-footprints)).

Next, I needed an Eagle part for the Vita modem form factor. Luckily, I found a good part for [mini-PCIe](http://www.diymodules.org/eagle-show-library?type=usr&id=3314&part=mini-pci-e.lbr) and was able to modify it to the custom size that Vita uses thanks to the drawing in the datasheet.

![Vita 3G modem drawing]({{ site.url }}/images/2017/07/vita-modem-drawing.png)

### Schematic

Next is connecting the parts together. Having no experience whatsoever, I turned again to [Sparkfun's tutorials](https://learn.sparkfun.com/tutorials/using-eagle-schematic). Copying the [reference design](http://ww1.microchip.com/downloads/en/DeviceDoc/evb2240schematic%20Lycus_C.pdf.pdf), I came up with a board with the microSD adapter and pin headers for the UART.

![Schematics]({{ site.url }}/images/2017/07/psvsd-schematics.png)

### Layout

I learned board layout again from [Sparkfun](https://learn.sparkfun.com/tutorials/designing-pcbs-advanced-smd) making sure to follow the design guidelines from [Microchip](http://www.microchip.com/wwwAppNotes/AppNotes.aspx?appnote=en562735). I also cheated by looking at the layout for the [reference board](http://ww1.microchip.com/downloads/en/DeviceDoc/DS50002298B-EVB-USB2240-IND_C-UsersManual-20140902.pdf) and ensuring that relative distance between objects match from my design. The main challenge is in routing because of the constrained size, but through some creativity, I managed to hook everything up.

![Layout front]({{ site.url }}/images/2017/07/psvsd-front.png)
![Layout back]({{ site.url }}/images/2017/07/psvsd-back.png)

## Manufacturing

Next step is to produce some prototypes. Thankfully this is extremely easy in this day and age. [Pcbshopper](http://pcbshopper.com) allows you to choose your design requirements and it will search across many PCB manufacturers for the best price. The price (plus shipping) is similar across many Chinese manufacturers--about $15 for 10 boards with standard options. The catch is slow lead time and even slower shipping. Throughout the project, I've tried [EasyEda](https://easyeda.com/), [SeeedStudio](https://www.seeedstudio.com), [DirtyPCBs](http://dirtypcbs.com), and [PCBway](https://www.pcbway.com/setinvite.aspx?inviteid=43203). Below is a mini-review of my experiences with each fab.

I used DirtyPCBs for the breakout adapters. The shipping time is the fastest per dollar (using the cheapest shipping rate, I got the package in two and a half weeks). The board quality was good but a couple of the adapters had the PCIe connector cut improperly and therefore won't fit the Vita without some sanding. There was no problem with the wiring or drills even though I used the smallest allowed sizes.

I purchased the first three prototypes from SeeedStudios because their website was the easiest to use and the cleanest of everyone on PCBshopper. The cheapest shipping was slow (took almost a month to arrive) and more than half the adapters I received had the PCIe connectors not cut properly. I found no electrical problems.

EasyEDA had the best quality of all the fabs I've used. All the cuts were good and the drill holes were very precise and exactly centered. They do not offer cheap shipping and build time was a couple days longer than their estimate of 2-4 days. I also ordered a stencil from them and that came out great as well.

PCBway would be my recommended fab. Although the quality was not as excellent as EasyEDA, it was still better than the other fabs (no issues with the connector). They also do not offer cheap shipping but their build time is a couple day faster than EasyEDA. More importantly, PCBway offers a competitive rate (5x cheaper than SeeedStudios) for PCB assembly and eventually became the fab that produced the final production run for this project.

## Prototyping

What's the most cost effective way to debug the design? Considering how cheap it is to build these boards, it is no surprise that the best way to debug is to build _another_ board. I created a second mini-PCIe based design--this time with a mini-PCIe socket on the card to act as a breakout board. Because the design for the breakout board is simple, the only requirement to verify the board is to do connectivity test on each pin after it arrives. Then I can probe the pads on the breakout port to debug the signals on the main design.

![Breakout 1]({{ site.url }}/images/2017/07/breakout-1.jpg)
![Breakout 2]({{ site.url }}/images/2017/07/breakout-2.jpg)

Using the breakout board, I can inspect the signals from the 3G modem in anticipation of some sort of custom handshake protocol. Fortunately, there wasn't such a sequence and the USB port works as-is. When the first boards came back (a month of waiting), I was able to test it by connecting the USB pads on the breakout board to a USB cable and connect the psvsd card to the computer.

![Breakout 2]({{ site.url }}/images/2017/07/psvsd-test.jpg)

Immediately, I found some errors and fixed it in the design. Having a test plan ready by the time the boards arrived really sped up the process.

## Funding

The nice thing about software hacking as a hobby is that it costs nothing but time. But for this hardware hack, I have spent a little over $100 on this project in parts, supplies, and boards. That's less than buying two video games, so I have no qualms about the cost, but considering the interest the community showed, I think it would be more than fair to spread the cost across everyone who is interested. My idea is this: I will make a limited production of 100 boards (no more because I will be shipping the packages myself and it's fairly laborious). These boards will be sold at cost and an extra $1 will be added to cover my expenses. I have heard many horror stories of crowd funding gone wrong, so I took many steps to ensure that this will be a success.

First, I made a spreadsheet covering all the costs: supplies, boards, shipping materials, platform fees, etc. Then I added a $100 buffer for any extraneous expenses (another prototype run, for example). Next, I made sure to be very clear upfront about what contributors are paying for: the supplies for me to develop this project. Because undoubtedly, manufacturing 100 boards at such a low cost will not have a perfect yield, I know a small number of these boards will have defects. I don't have the time or money to deal with customer service for these issues, so part of the low price of the boards is that each contributor takes some amount of risk that their board is defective. Finally, I set a fixed goal so I do not receive the money until after 60 days. I am spending my own money in the meantime. My hope is that after 60 days, I'll either complete the project and use the unlocked money to reimburse myself and fund the limited production. Or, I'll run into some major unresolvable issue, in which I will refund everyone and just lose the ~$200 I spent so far. However, after a month of steady progress I felt confident enough to take 400 more orders for a total of 500. Then after getting lots of good samples from the fab I also felt it was fine to test and ensure that every adapter works before shipping it out.

The feedback was tremendous, and the funding goal was met in a day [after it was posted](https://www.indiegogo.com/projects/ps-vita-3g-to-microsd-card-adapter). That gives me enough confidence and motivation to continue the project and ensure it is a success.

## Software

Fortunately, the driver is pretty easy to create. The Vita already has drivers for USB storage (it's used on PS TV safe mode for reinstalling firmware), but is normally disabled. A simple patch running on HENkaku Ensō enables it at boot and using The_FloW's patches for mounting USB storage as a memory card, it all pretty much just works.

## Testing

Next is an important part that I feel many ambitious project leaders skip--which is testing. I want some real-world usage data and more importantly, I want to know what the battery impact of my design is. This was the first hurdle I ran into. Initial results showed that the battery life lasted an hour less with psvsd installed when idle. Worse, the battery was consumed even when powered off (not lasting overnight). This is unacceptable for daily use. I took one of my breakout boards and re-purposed it to act as a current measurement harness by cutting the trace to the power input and attaching each end to an ammeter.

![Power Measurement]({{ site.url }}/images/2017/07/psvsd-power-measure.jpg)

Then, I was able to measure the exact current consumption during various usage cases (read, write, idle, etc). Below is a video of some of these tests.

<iframe width="560" height="315" src="https://www.youtube.com/embed/zJOYfw9lmQM" frameborder="0" allowfullscreen></iframe>

After testing the power usage of a couple of different USB devices and asking around on hardware forums, I found out the problem was two-fold. First, when the Vita is powered off, it does not power off the USB voltage line, but it does pull both USB data lines low. Unfortunately, this leaves the USB device in "reset" mode instead of "low power suspend" mode. Likely this wasn't an issue for the 3G modem because it was a custom design meant only to pair with the Vita and has a separate power management IC that is smarter than just looking at the USB data lines. The second problem is that the USB2244 is a power hog of a chip. It draws an average and minimum of 100mA when not in "low power suspend" (which the Vita does not support) even if there is no activity on the SD card.

As a result, I had no choice but to go for an "cheap Asia manufactured chips" even though there was less documentation and support. Luckily I found some datasheets and reference schematics for GL823 [online](https://www.eevblog.com/forum/projects/gl823-sd-card-reader-no-response/) and was able to buy a couple of them to play with. I discovered that cheaper doesn't always means lesser quality. Not only did the GL823 consume less power (only 30mA average and 1.5mA in "reset" mode) but it also outperformed the USB2244 in read and write speeds as well! Even better, the GL823 does not require an external crystal so I can remove some of the area footprint as well. I really should have chosen this chip to start with.

I also purchased a dedicated USB power tester at this point so I was able to get quick data measurements.

![Power Measurement 2]({{ site.url }}/images/2017/07/psvsd-power-measure-2.jpg)

![Power Measurement 3]({{ site.url }}/images/2017/07/psvsd-power-measure-3.png)

Because the extra hardware must be powered somehow, some dip in battery life is expected, but in the final design this dip is not noticeable at all.

## What's next?

In the end, I made five prototypes and a breakout adapter. Here's a family photo along with the final product to the right.

![Family]({{ site.url }}/images/2017/07/psvsd-family.jpg)

![Family 2]({{ site.url }}/images/2017/07/psvsd-family-2.jpg)

Thanks to everyone who contributed to this project! There are more detailed posts (with more pictures) for each step in the process on the [Indiegogo page](https://www.indiegogo.com/projects/ps-vita-3g-to-microsd-card-adapter/x/16550489#/updates) for those who are interested. You can find the design on [psvsd.henkaku.xyz](https://psvsd.henkaku.xyz/). Since the design is open source and free for commercial use, I think someone will manufacture, sell, and support it. Here's another free idea: buy a large number of < 3.60 firmware 3G motherboards (they are around $15-25 a piece on Aliexpress) and screws (M1.6x4mm flat-head no countersink) and bundle them together with a psvsd adapter and a microSD card to form a Vita hacking starter kit.

I don't plan to mass produce this myself but I do have at most 50 extra units due to canceled orders and extra parts. As a result, I've decided to auction them off to those who most want one up until September 2017. You can find more information about that [here](https://goo.gl/forms/YeOCwo4ARa89B6S92).
