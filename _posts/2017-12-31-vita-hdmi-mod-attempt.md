---
author: yifanlu
comments: true
date: 2017-12-31
layout: post
slug: vita-hdmi-mod-attempt
title: "Vita HDMI Mod (Attempt)"
categories:
- Vita
tags:
- vita
- hardware
- hacking
- pcb
- oscilloscope
- i2c
- hdmi
- mipi
---

For the last couple of months, I've been developing an HDMI mod for the Vita on my free time. I thought it would be a fun project to practice my hardware design skills even though the end product would not be too useful (the VitaTV already exists). Unfortunately, this project did not end in success but I want to write about it anyways so you can see what I've been doing with some of the leftover money from my [adapter project]({% post_url 2017-08-22-psvsd-custom-vita-microsd-card-adapter %}).

## Overview

The Vita's SoC (named Kermit) has two MIPI DSI output ports. On OLED units, the first port is connected to a custom 40-pin high speed board-to-board connector that mates with an [AMS495QA01](https://wiki.henkaku.xyz/vita/File:AMS495QA01_datasheet.pdf) OLED panel. On LCD units, the same port goes to a ZIF connector. The second port is unused on handheld Vitas and is connected to an [ADV7533](http://www.analog.com/en/products/audio-video/hdmidvi-switches/adv7533.html) on the PSTV. On development kits, both ports are used (one to OLED and another to ADV7533) and I suspect that's why the SoC has two ports in the first place. I would like to comment here that the Kermit SoC does not have native support for HDMI/TMDS signaling and therefore any rumors of handheld Vita consoles having HDMI output capabilities are false. No, that "mystery port" does not have video output capabilities (it is a USB host port with a custom physical connector).

Can we hook up the unused MIPI DSI port? Unfortunately no because those pins are not routed so it is impossible to get to them, so instead the idea is to "hijack" the DSI output to the OLED panel and let the same signals drive a custom board that can convert it to HDMI. This requires us to solder some wires to the video signals and thanks to the OLED datasheet along with some connectivity tests, it was easy to locate test points for the desired signals.

[![Testpoints]({{ site.url }}/images/2017/12/ngptv-vita-pinout.png)]({{ site.url }}/images/2017/12/ngptv-vita-pinout.png)

## ngptv

My original idea is to use the same components as the PSTV, namely the ADV7533 MIPI DSI to HDMI conversion chip. It is the only ASIC on the market that does this so there was little choice. Using some other implementation as reference, I drew up a schematic for the board including the recommended circuits to adhere to HDMI standards.

[![ngptv]({{ site.url }}/images/2017/12/ngptv-schematic.png)]({{ site.url }}/images/2017/12/ngptv-schematic.png)

A couple of big problems quickly came up that made this design infeasible

* I wanted to expose a mini-HDMI port on the bottom of the OLED Vita right next to the multi-connector. There is unused space inside the Vita near that region but it is only about 15mm x 15mm. That means all the components I choose will have to be extremely space efficient and therefore expensive.
* The ADV7533 only comes in a 49-BGA package which means layout requires at least a 6 layer board with low pitch and drill sizes. This means that prototyping the boards will be very expensive. A normal 2 layer PCB with standard drills can be fabricated for about $10 for each prototype run. A 6 layer board with small drill sizes goes for about $300 for each prototype run.
* I do not have the equipment to solder and test small pitch BGA parts which I would have to use to meet the space constraints.
* You cannot buy the ADV7533 from standard US suppliers because the part is under NDA and requires you to have a HDMI license which costs thousands of dollars per year.

Since I do not plan to produce these boards at a profit, I cannot justify investing the time and money for this design. However, another approach presented itself to me.

## ngptv lite

ST makes an [adapter board](http://www.st.com/en/development-tools/b-lcdad-hdmi1.html) for their MCU evaluation boards (which only has MIPI DSI support) to hook up to external displays. We can easily purchase these for $30 a pop (no license or NDA required) and then build a custom "host" board for it. That's exactly what I did. I built a small 15mm x 15mm breakout board that can be placed into the Vita and soldered 36AWG wires from the testpoints to the breakout board. Then I built a "host" board that connects to my breakout board and the ST adapter. The host board also has pins to connect to my RaspberryPi so I can power it as well as program the ADV7533. It quickly became a colorful mess of wires.

[![Setup]({{ site.url }}/images/2017/12/ngptv-vita-setup.JPG)]({{ site.url }}/images/2017/12/ngptv-vita-setup.JPG)

Wiring the breakout board.
{: .wp-caption-text}

[![RPI setup]({{ site.url }}/images/2017/12/ngptv-rpi-setup.JPG)]({{ site.url }}/images/2017/12/ngptv-rpi-setup.JPG)

RaspberryPi and what the adapter looks like.
{: .wp-caption-text}

[![Setup]({{ site.url }}/images/2017/12/ngptv-hooked-up.JPG)]({{ site.url }}/images/2017/12/ngptv-hooked-up.JPG)

Everything connected together.
{: .wp-caption-text}

## Driver

Since the ADV7533 is under NDA, Analog Devices does not give out the programming guide to the public. This makes no sense because there are quite a few open source implementations out there:

* [Linux driver](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=2437e7cd88e8781cef5fd2c254c85aa62b305d04)
* [Marvell Android driver](https://github.com/manakeri/android_kernel_samsung_xcover/blob/master/common/drivers/media/video/adv7533.c)
* [Qualcomm Android driver](https://android.googlesource.com/kernel/msm/+/android-lego-6.0.1_r0.2/drivers/video/msm/msm_dba/adv7533.c)
* [ST drivers for evaluation board](https://os.mbed.com/teams/ST/code/BSP_DISCO_F769NI/file/145e714557cf/Drivers/BSP/Components/adv7533/adv7533.c/)
* [xerpi's vita-baremetal-sample which uses above](https://github.com/xerpi/vita-baremetal-sample/blob/master/src/hdmi.c)

By looking at the different implementations, I was able to piece together the proper configuration flow (as well as find benign bugs and wrong comments in different implementation leading me to believe not all the drivers above were original work). I wrote the I2C configuration sequence as a Python script to run on the RPI, which was able to communicate with the ADV7533 successfully.

However, no video showed up on screen. It's time to bring out the oscilloscope.

## Debugging

[![Oscilloscope]({{ site.url }}/images/2017/12/ngptv-oscilloscope.JPG)]({{ site.url }}/images/2017/12/ngptv-oscilloscope.JPG)

[![Powered on]({{ site.url }}/images/2017/12/ngptv-powered-on.JPG)]({{ site.url }}/images/2017/12/ngptv-powered-on.JPG)

After sniffing the clock lanes with my oscilloscope, I've noticed something strange: the clock signal is off every 30us.

[![Clock signal]({{ site.url }}/images/2017/12/ngptv-clock-signal.JPG)]({{ site.url }}/images/2017/12/ngptv-clock-signal.JPG)

The [MIPI D-PHY specifications](http://diyprojector.info/forum/index.php?app=core&module=attach&section=attach&attach_id=27737) defines two modes: HS (high-speed) and LP (low-power). In HS mode (also called video mode for MIPI DSI), the clock lanes act as a high speed differential clock while the data lanes transfer the data. This is typically used to send each frame. In LP mode (also called command mode for MIPI DSI), the video source and sink can communicate during v-blank periods and send auxiliary information. The clock lanes are not used when the data lanes are in LP mode and therefore to save battery, the clock lanes can also enter LP mode and is seen as off. Unfortunately, the [ADV7533 datasheet](http://www.analog.com/media/en/technical-documentation/data-sheets/ADV7533.pdf) states the following on the first page:

> The DSI Rx implements DSI video mode operation only.

This implies that there is no logic to handle the clock lane LP transition. To test this hypothesis, I used xerpi's [vita-baremetal](https://github.com/xerpi/vita-baremetal-sample/) to set up the MIPI DSI clock the same way the PSTV does and sure enough I see in my oscilloscope that the clock no longer turns off and I can see test patterns on the screen.

[![Test working]({{ site.url }}/images/2017/12/ngptv-test-working.JPG)]({{ site.url }}/images/2017/12/ngptv-test-working.JPG)

Cursory tests shows that the Vita OLED does not like the clock running continuously so it does not seem possible to have the OLED and HDMI working at the same time. I also don't want to limit the adapter to only working with hacked Vitas, so I thought to find another way. I tried [asking around](https://electronics.stackexchange.com/questions/342657/how-to-generate-a-continuous-clock-from-one-that-periodically-turns-off) to see if there is some magical IC that can derive a fixed clock that is phase synced to the pixel clock but stays on. Initially I thought I found a solution with jitter attenuators/cleaners but then I was told by an engineer that a jitter cleaner would average out the clock rather than ignore the "off" periods. It would be way too expensive to build a custom solution using FPGA or op-amps and PLLs that can handle > 250MHz differential signals.

## Redesign

Nevertheless, I decided to redesign my host board just to hone my design skills (which is the whole point of the project anyways). I wanted my board to be the same size as the ST board and have the connectors align so they can sit on top of each other. I also added a MIPI DSI redriver with a configurable equalizers. This ensures the video signal going to the ST adapter is clean. Finally, I added a microcontroller so I can program in the I2C configuration sequence for the redriver and ADV7533 without needing a RPI. The end result was a pretty packed board.

[![New board]({{ site.url }}/images/2017/12/ngptv-v11-board.JPG)]({{ site.url }}/images/2017/12/ngptv-v11-board.JPG)

The microcontroller is not soldered on as it is easier to debug by connecting to my RPI.
{: .wp-caption-text}

[![New board setup]({{ site.url }}/images/2017/12/ngptv-v11-setup-1.JPG)]({{ site.url }}/images/2017/12/ngptv-v11-setup-1.JPG)

What it looks like stacked.
{: .wp-caption-text}

[![New board setup 2]({{ site.url }}/images/2017/12/ngptv-v11-setup-2.JPG)]({{ site.url }}/images/2017/12/ngptv-v11-setup-2.JPG)

Top down view.
{: .wp-caption-text}

## Future

I don't plan to pursue this project any further because I got the experience I wanted out of it. However, for people who are interested in continuing where I left off, the designs are [open source](https://github.com/yifanlu/ngptv). I think there are a couple of ways going forward.

* If you only care about hacked Vitas, you can try to get the existing design to work with a custom driver that sets [the auto clock configuration](https://wiki.henkaku.xyz/vita/DSI_Registers) to output to the screen or to the external adapter. You can also try to find the test-points on a Vita slim. Finally, if you want sound, you need to find the an I2S output somewhere.
* If you want to try another part, you can look at one of various MIPI DSI to eDP chips ([for example this](http://www.ti.com/product/SN65DSI86)) and chain it with a DP to HDMI chip or with a DP cable. Make sure the chip you're using supports LP mode!
* If you want to design your own part using a FPGA, that might be the best route but you need to make sure your FPGA supports MIPI D-PHY, which most likely it won't and you'll have to make a [level translation circuit](https://www.xilinx.com/support/documentation/application_notes/xapp894-d-phy-solutions.pdf). I think this is what the existing Vita video out mod does.
