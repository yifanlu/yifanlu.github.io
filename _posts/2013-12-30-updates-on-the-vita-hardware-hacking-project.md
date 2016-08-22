---
author: yifanlu
comments: true
date: 2013-12-30 06:02:39-06:00
layout: post
slug: updates-on-the-vita-hardware-hacking-project
title: Updates on the Vita Hardware Hacking project
wordpress_id: 687
categories:
- Guides
- Vita Hardware Hacking
tags:
- emmc
- hacking
- hardware
- microsd
- sd
- soldering
- test points
- vdd
- vita
---

After a week of trying to dump the eMMC (spoilers: didn't happen yet), I've decided to post this update about things I've tried to do (and how I tried to do it) and where the money is going to.<!-- more -->


### Supplies


I had two Vita logic boards. The first one, which I removed the SoC and eMMC to find the trace points (shown previously), came from eBay. The second board came from a Vita with a broken screen generously donated by [@Amxomi](https://twitter.com/Amxomi). I also bought a professional rework station, the [X-Tronic 4040](http://www.amazon.com/dp/B003TC8EQS/) which was paid partially by your donations (I returned the heat gun) and partially thanks to [wololo](http://wololo.net/). For wiring, the thinnest wire I could find is enamel-coated magnet wire. For soldering the wires, I got 60/40 Rosin solder and a Rosin flux pen.


### Attempt One


The first thing I did was remove the EMI shield base blocking the test point resistors. With the reflow station's hot air gun, it was much easier than the heat gun I used last time. Next I warmed up my soldering skills by hooking wires up to a microSD to SD card adapter. My plan was to attach the wires to the test point and plug the SD card into a SD card reader. To expose the copper in the enamel-coated wire, I melted a blob of solder and kept it on the tip of the iron at 400C. Then I dipped the tip of the wire into the liquid solder, which both coats the tip of the wire with solder and also removes the coating. It's a neat trick that I used all the time throughout the rest of the ordeal.

Then I brushed the pins of the microSD adapter with flux and quickly melted a small blob of solder on each pin. Then with a pair of tweezers, I held each wire next to the pin, and as soon as it is heated, the small bit of solder on the wire joins with the blob on the pin and they connect.

It gets much harder connecting the other end. There is very little exposed solder on the tiny resistors, and it is very hard to add more because you might accidentally short circuit two adjacent pads. I made sure there is a bit of solder on the end of the wire using the trick. Then I held the end of the wire steady with the tweezer while tapping it with the iron. It takes many tries for it to stick on, and many times when trying to attach the neighboring pads, the heat from the iron loosens the other wires. In addition, accidentally bumping into the wire causes enough stress to rip the solder off the resistor (because there is so little solder), so I just taped everything to a piece of cardboard. I also can't test if my joints are correct and not shorted with any other joints because of how small and close everything is.

After a couple of hours, the wires are soldered to the points but there are a couple of problems. First, as mentioned, I couldn't test the correctness of my connections. Second, I don't know if in the process of soldering to the tiny resistors, I damaged any resistors and if so, would it still work. Third, I never found a test point for Vdd because for some reason, Vdd shorts to Vss/Ground on my first board. As expected, after plugging the microSD adapter into a reader into the computer, nothing shows up. Because there could be so many problems, I removed all the wires and started over.


### Attempt Two


First, I located a test point for Vddf (Vdd is power to the eMMC controller while Vddf is power to the actual NAND chip). My hypothesis is that the same power source that powers Vddf also powers Vdd (although the Samsung documents recommends against this). This point is on top of the tiny resistor to the left of the audio jack.

Next, I decided to remove all of the 150ohm resistors on the test points in order to get more solder surface area. Looking at the [eMMC testpoints picture](/images/2013/12/vita_nand_pinout.jpg) from last time, it's important to note that the pad on the left of each resistor is the one coming from the eMMC while the pad on the right of each resistor is the one going to the SoC. The resistors themselves may be for current limiting or noise removal. Removing them is as simple as pointing at it with the hot air gun set to 380C for half a minute and then using tweezers to to remove them.

I also found it easier to solder wires directly to the SD card reader instead of to an microSD to SD card adapter. I first verified that the card reader still works and that my wires are not too long by soldering the other end of the wire directly to an old 128MB SD card. After verifying, I removed the wires from the SD card and attached them to the now exposed test points.

Unfortunately, it still didn't work. The computer sees the SD card reader but believes no card is inserted. Again, there could be any number of problems including (still) bad soldering, Vdd not receiving power, or even read protection in the eMMC.


### Attempt Three


Next I made another attempt to find Vdd. The problem is that my multimeter shows a short from Vdd to Vss. This means that Vdd is somehow shorted to ground either because I broke something with all the heat and bad soldering or because that is by design (which I don't think is true because all documents I read say that you need to power Vdd for the eMMC controller to work). I thought maybe I can experiment by sending test voltages through various locations on the first logic board (the one with the chips removed) and see if I get a voltage drop in the Vdd pad. I used an old broken MP3 player as my voltage source (since it was just laying around and I didn't want to buy a power supply, rip open any working cable/device, or solder to a battery). I attached the positive end to a pointed screwdriver and the negative end to the Vita's ground. Then I attached one probe of my voltmeter to the same ground. Then with the screwdriver in one hand and the voltmeter probe in the other hand, I tried to send voltage through every location on the board. Unfortunately, the only response was sparks on capacitors here and there but no response in Vdd.

Back to the second Vita, I tried to attach the battery and charger and turned it on. At first, I got excited and saw a voltage drop on the eMMC's decoupling capacitor (meaning there's power going to the eMMC). However, after going back and reattaching the wires, I could no longer reproduce the result. In addition, the power light no longer responds to the power switch. I believe that I shorted something and the first time I powered it on, it destroyed some component; so the next time I attempted to power it on, it fails before even attempting to power the eMMC.

Regardless, I tried to reattach all the wires with better soldering on the assumption that my only problem is still the bad soldering (likely not true). Being the fourth or fifth time doing this, I am getting better at soldering these extremely tiny points. My trick was to first align the wire to the board and then using the tweezer, make a 90 degree bend on the end of the wire. This makes the end of the wire the same length as the original resistors. Then I quickly dip the end in solder, flux the board, and attach the wire to two pads instead of one. This makes a stronger connection. Even though I did a much better job and soldering the test points, I still could not get anything to show up on my computer.


### Attempt Four


Now that I have experience in soldering tiny points, I made an attempt at soldering directly to the eMMC removed from the first Vita. However, after a quick test (nothing shows up on the computer), I didn't look any farther because I believe that the eMMC must be part of a circuit of capacitors and resistors in order for it to work (and not break the chip). All documents I've read supports this.

I also made yet another attempt at resoldering to the board again and still no luck. At this point, I believe that either I still am not powering Vdd correctly, or I broke the eMMC at some point. I also suspect that perhaps my SD card reader does not support the Samsung eMMC or that it is not being initialized correctly.


### What's Next


I still haven't given up. I will continue to try resoldering the points. I still want to find a way to surely power Vdd; I bought another Vita from eBay because I believe the second Vita is now broken. I also ordered a [eMMC socket](http://item.taobao.com/item.htm?id=14178433577) with the last of the usable donations, but it will take at least a month to arrive from China. There's also the possibility that the eMMC does something unsupported by my SD card to USB adapter and I want to do some raw signal interaction with an Arduino. If you want live updates of progress as I'm working, join [#vitadev](irc://irc.efnet.org/vitadev) on EFnet.

<style type="text/css">#gallery-6{margin:auto;}#gallery-6 .gallery-item{float:left;margin-top:10px;text-align:center;width:33%;}#gallery-6 img{border:2px solid #cfcfcf;}#gallery-6 .gallery-caption{margin-left:0;}</style>
<div id="gallery-6" class="gallery galleryid-687 gallery-columns-3 gallery-size-thumbnail"><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i1.wp.com/yifan.lu/images/2013/12/IMG_0582.jpg"><img width="150" height="150" src="//i1.wp.com/yifan.lu/images/2013/12/IMG_0582.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Professional soldering and reflow station fresh out of the box." aria-describedby="gallery-6-688" srcset="http://i1.wp.com/yifan.lu/images/2013/12/IMG_0582.jpg?resize=150%2C150 150w, http://i1.wp.com/yifan.lu/images/2013/12/IMG_0582.jpg?resize=46%2C46 46w, http://i1.wp.com/yifan.lu/images/2013/12/IMG_0582.jpg?zoom=2&amp;resize=150%2C150 300w, http://i1.wp.com/yifan.lu/images/2013/12/IMG_0582.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-688">
Professional soldering and reflow station fresh out of the box.
</dd></dl><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0583.jpg"><img width="150" height="150" src="//i2.wp.com/yifan.lu/images/2013/12/IMG_0583.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="microSD to SD adapter torn apart and wires soldered in" aria-describedby="gallery-6-689" srcset="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0583.jpg?resize=150%2C150 150w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0583.jpg?resize=46%2C46 46w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0583.jpg?zoom=2&amp;resize=150%2C150 300w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0583.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-689">
microSD to SD adapter torn apart and wires soldered in
</dd></dl><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0585.jpg"><img width="150" height="150" src="//i2.wp.com/yifan.lu/images/2013/12/IMG_0585.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Shield removed on new board. Much easier this time with a reflow station." aria-describedby="gallery-6-690" srcset="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0585.jpg?resize=150%2C150 150w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0585.jpg?resize=46%2C46 46w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0585.jpg?zoom=2&amp;resize=150%2C150 300w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0585.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-690">
Shield removed on new board. Much easier this time with a reflow station.
</dd></dl><br style="clear: both"><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0586.jpg"><img width="150" height="150" src="//i0.wp.com/yifan.lu/images/2013/12/IMG_0586.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Another angle." aria-describedby="gallery-6-691" srcset="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0586.jpg?resize=150%2C150 150w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0586.jpg?resize=46%2C46 46w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0586.jpg?zoom=2&amp;resize=150%2C150 300w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0586.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-691">
Another angle.
</dd></dl><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0588.jpg"><img width="150" height="150" src="//i2.wp.com/yifan.lu/images/2013/12/IMG_0588.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Old logic board. The trace points with the resistors removed." aria-describedby="gallery-6-692" srcset="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0588.jpg?resize=150%2C150 150w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0588.jpg?resize=46%2C46 46w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0588.jpg?zoom=2&amp;resize=150%2C150 300w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0588.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-692">
Old logic board. The trace points with the resistors removed.
</dd></dl><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i1.wp.com/yifan.lu/images/2013/12/IMG_0589.jpg"><img width="150" height="150" src="//i1.wp.com/yifan.lu/images/2013/12/IMG_0589.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Old board. Coupling capacitors next to the eMMC removed." aria-describedby="gallery-6-693" srcset="http://i1.wp.com/yifan.lu/images/2013/12/IMG_0589.jpg?resize=150%2C150 150w, http://i1.wp.com/yifan.lu/images/2013/12/IMG_0589.jpg?resize=46%2C46 46w, http://i1.wp.com/yifan.lu/images/2013/12/IMG_0589.jpg?zoom=2&amp;resize=150%2C150 300w, http://i1.wp.com/yifan.lu/images/2013/12/IMG_0589.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-693">
Old board. Coupling capacitors next to the eMMC removed.
</dd></dl><br style="clear: both"><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0591.jpg"><img width="150" height="150" src="//i2.wp.com/yifan.lu/images/2013/12/IMG_0591.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Attempt 1: Wires soldered directly to resistors and to an adapter." aria-describedby="gallery-6-694" srcset="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0591.jpg?resize=150%2C150 150w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0591.jpg?resize=46%2C46 46w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0591.jpg?zoom=2&amp;resize=150%2C150 300w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0591.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-694">
Attempt 1: Wires soldered directly to resistors and to an adapter.
</dd></dl><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0592.jpg"><img width="150" height="150" src="//i0.wp.com/yifan.lu/images/2013/12/IMG_0592.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Look at the horrible soldering job" aria-describedby="gallery-6-695" srcset="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0592.jpg?resize=150%2C150 150w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0592.jpg?resize=46%2C46 46w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0592.jpg?zoom=2&amp;resize=150%2C150 300w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0592.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-695">
Look at the horrible soldering job
</dd></dl><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0593.jpg"><img width="150" height="150" src="//i0.wp.com/yifan.lu/images/2013/12/IMG_0593.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Attempt 2: Resistors removed. Vddf found and soldered to Vdd from SD reader." aria-describedby="gallery-6-696" srcset="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0593.jpg?resize=150%2C150 150w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0593.jpg?resize=46%2C46 46w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0593.jpg?zoom=2&amp;resize=150%2C150 300w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0593.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-696">
Attempt 2: Resistors removed. Vddf found and soldered to Vdd from SD reader.
</dd></dl><br style="clear: both"><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0594.jpg"><img width="150" height="150" src="//i0.wp.com/yifan.lu/images/2013/12/IMG_0594.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Wired directly to a USB SD card reader" aria-describedby="gallery-6-697" srcset="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0594.jpg?resize=150%2C150 150w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0594.jpg?resize=46%2C46 46w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0594.jpg?zoom=2&amp;resize=150%2C150 300w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0594.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-697">
Wired directly to a USB SD card reader
</dd></dl><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0595.jpg"><img width="150" height="150" src="//i0.wp.com/yifan.lu/images/2013/12/IMG_0595.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Contraption built to send test voltages to board." aria-describedby="gallery-6-698" srcset="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0595.jpg?resize=150%2C150 150w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0595.jpg?resize=46%2C46 46w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0595.jpg?zoom=2&amp;resize=150%2C150 300w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0595.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-698">
Contraption built to send test voltages to board.
</dd></dl><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i1.wp.com/yifan.lu/images/2013/12/IMG_0596.jpg"><img width="150" height="150" src="//i1.wp.com/yifan.lu/images/2013/12/IMG_0596.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Attempt 3: Refluxed pads and tried again" aria-describedby="gallery-6-699" srcset="http://i1.wp.com/yifan.lu/images/2013/12/IMG_0596.jpg?resize=150%2C150 150w, http://i1.wp.com/yifan.lu/images/2013/12/IMG_0596.jpg?resize=46%2C46 46w, http://i1.wp.com/yifan.lu/images/2013/12/IMG_0596.jpg?zoom=2&amp;resize=150%2C150 300w, http://i1.wp.com/yifan.lu/images/2013/12/IMG_0596.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-699">
Attempt 3: Refluxed pads and tried again
</dd></dl><br style="clear: both"><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0597.jpg"><img width="150" height="150" src="//i2.wp.com/yifan.lu/images/2013/12/IMG_0597.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Better soldering job this time" aria-describedby="gallery-6-700" srcset="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0597.jpg?resize=150%2C150 150w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0597.jpg?resize=46%2C46 46w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0597.jpg?zoom=2&amp;resize=150%2C150 300w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0597.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-700">
Better soldering job this time
</dd></dl><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0598.jpg"><img width="150" height="150" src="//i2.wp.com/yifan.lu/images/2013/12/IMG_0598.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Attempt 4: Tried hooking wires up directly to eMMC and SD adapter" aria-describedby="gallery-6-701" srcset="http://i2.wp.com/yifan.lu/images/2013/12/IMG_0598.jpg?resize=150%2C150 150w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0598.jpg?resize=46%2C46 46w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0598.jpg?zoom=2&amp;resize=150%2C150 300w, http://i2.wp.com/yifan.lu/images/2013/12/IMG_0598.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-701">
Attempt 4: Tried hooking wires up directly to eMMC and SD adapter
</dd></dl><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0599.jpg"><img width="150" height="150" src="//i0.wp.com/yifan.lu/images/2013/12/IMG_0599.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Attempt 4: Also tried resoldering pads one more time" aria-describedby="gallery-6-702" srcset="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0599.jpg?resize=150%2C150 150w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0599.jpg?resize=46%2C46 46w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0599.jpg?zoom=2&amp;resize=150%2C150 300w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0599.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-702">
Attempt 4: Also tried resoldering pads one more time
</dd></dl><br style="clear: both"><dl class="gallery-item">
<dt class="gallery-icon landscape">
<a href="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0600.jpg"><img width="150" height="150" src="//i0.wp.com/yifan.lu/images/2013/12/IMG_0600.jpg?resize=150%2C150" class="attachment-thumbnail size-thumbnail" alt="Zoomed out" aria-describedby="gallery-6-703" srcset="http://i0.wp.com/yifan.lu/images/2013/12/IMG_0600.jpg?resize=150%2C150 150w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0600.jpg?resize=46%2C46 46w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0600.jpg?zoom=2&amp;resize=150%2C150 300w, http://i0.wp.com/yifan.lu/images/2013/12/IMG_0600.jpg?zoom=3&amp;resize=150%2C150 450w" sizes="(max-width: 150px) 100vw, 150px"></a>
</dt>
<dd class="wp-caption-text gallery-caption" id="gallery-6-703">
Zoomed out
</dd></dl>
<br style="clear: both">
</div>
