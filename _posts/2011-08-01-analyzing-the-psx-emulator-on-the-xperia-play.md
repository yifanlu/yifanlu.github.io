---
author: yifanlu
comments: true
date: 2011-08-01 21:20:22-06:00
layout: post
slug: analyzing-the-psx-emulator-on-the-xperia-play
title: Analyzing the PSX emulator on the Xperia Play
wordpress_id: 289
categories:
- Assembly
- Technical
- Xperia Play
tags:
- android
- crash bandicoot
- decompilinh
- decryption
- disassembling
- encryption
- ida pro
- information
- ps1
- reverse engineering
- root
- sony
- xperia play
- zeus
---

I've been playing around with the new Xperia Play (well, with the speed of these Android phone releases, it's already old). I've decided it would be a challenge to try to figure out how the PSOne emulator works and eventually be able to inject any ISO and play it with Sony superior PS1 emulator. Just to be clear, nothing is done yet, and this is just a technical post to aid whoever else is trying to do the same thing. Also, because information should be free.<!-- more -->

**Decompiling and disassembling**

Before we can do any analyzing, we need to break everything open. I found a couple of useful tools to aid with reverse engineering Android applications. First up is [apktool](http://code.google.com/p/android-apktool/), which is like an all-in-one Android app decompression and decompilation tool. It uses various other tools to do stuff like decompress resources, convert the meta files to be readable, and use [baksmali](http://code.google.com/p/smali/) to disassemble Dalvik bytecode to assembly code. Another useful tool is [dex2jar](http://code.google.com/p/dex2jar/), which converts Dalvik bytecode to regular Java bytecode and generates a jar that can be decompiled to Java code using a decompiler like, my favorite, [JD-GUI](http://java.decompiler.free.fr/). Last, but not least, we have the big guns: [IDA Pro](http://www.hex-rays.com/idapro/), which I've used religiously for many projects. If you don't know, it can disassemble almost any binary, including native ARM libraries.

**Stepping through**

The first thing to do once we reversed all the code is to read it. A good way to start is to follow an application from start to finish through the code. Looking in the Android manifest file, we find the main activity that is started is com.sony.android.psone.BootActivity. We open that up, look at onCreate() and read what it does, follow whatever methods it calls and read through all those too. It may get a bit complicated, so I suggest thinking like a stack. From what I can understand, the first thing the app does is check if the data is downloaded. "Crash Bandicoot" is a 500MB game, so it would use up all the system space, so what Sony did is pack the binaries into the APK installed on the system, and the game data (textures, images, etc) is a ZPAK (renamed PKZIP) file that is downloaded from their servers if deleted. Once the data is verified to exist or downloaded from Sony's servers, the baton is passed to a native JNI library to do the actual work.

**Native code**

Sony sees the Xperia Play not as just an Android phone, but a game platform. They call it "zplatform", or as I guess: Zeus Platform (Zeus was the codename for the Xperia Play). The platform APIs is found on libzplatform.so, which is linked by all platform compliant (read: only on Play) games. It contains functions for extracting/creating ZPAK files as well as a lot of encryption/decryption commands and other stuff like networking. Another library is libjava-activity.so, which contains the actual emulator. Well, sort of. libjava-activity.so contains almost 2MB worth of crypto-security functions. It's sole purpose is to decrypt and load into memory, three files (two of which are stored encrypted inside libjava-activity.so). They are: image_ps_toc (I can only guess it relates to the ROM file, image.ps), ps1_rom.bin (the PS1 BIOS, found in the data ZPAK), and libdefault.so (the main executable, aka: the emulator).

**ZPAK files**

The ZPAK file is basically a ZIP file that stores the game data. I only looked through "Bruce Lee" and "Crash Bandicoot", but from what I can see there, all ZPAK files contain a metadata XML and one or more encrypted data files. For example, Crash Bandicoot's ZPAK data contains image.ps, which I can guess from the size, is the ROM file for the game. I do not know if it's an ISO or if it's compressed, but that's not important right now. There's also ps1_rom.bin, which I can say for certain after reading the code to decrypt it, is the PS1 BIOS file, compressed using zlib. There's also pages from the manual named for their page number and have no extensions. I can assume that they're encrypted too because they contain no image header and the first two bytes are not the same throughout. The main thing I need to figure out is if the encryption key is common or not.

**The white box**

The main executable, libdefault.so, is completely encrypted and obfuscated by libjava-activity.so, which implements a white box security. If you read anything about white box cryptography (Google), you'll see that it's sole purpose in existence is to prevent itself from being reverse engineered. It hides the decryption key in a giant table or an even bigger finite-sized key. Nevertheless, it would take <del>someone</del>, a group of people smarter than me (not that that's hard to find) to crack this file.

**What's next**

Unfortunately, that's all I know for now. Why? Because the CDMA version of the Xperia Play has not been rooted yet, and any farther analysis would require client access. I'm in the process to locating a R800i model of the Play to test with, but for now, I hope that someone who knows what they're doing reads this and continue where I left off.

There are two giant problems that's preventing us from injecting any PS1 image into the emulator. First of all, everything is encrypted. My hope is that it's a single key used in zplatform (seeing that there's functions such as zpCryptDecrypt and zpCryptEncrypt in the platform APIs) is used by Sony to encrypt image.ps and the manuals. Second of all, we need libdefault.so, the emulator. This may be easier then imagined. White box cryptography is used to hide the decryption key, not the decrypted content. My hope is that libdefault.so is loaded into RAM after libjava-activity.so decrypts it. There is a high chance of that because it would be hard to hide an executable and run it at the same time. If that is the case, disassembling the emulator will produce more results. If you have a rooted Xperia Play, set up USB debugging, and open up Crash Bandicoot. Connect the Play, and call "adb shell dd if=/dev/mem > memdump.bin" and then "adb shell dd if=/dev/kmem >> memdump.bin" (I don't know which one would work, so try both). That will (hopefully) produce a memory dump that will contain the emulator executable. Once we have this, even if we cannot decrypt image.ps, it may be possible to write an alternative wrapper application that will load ISOs or something.
