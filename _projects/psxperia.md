---
archive: https://github.com/yifanlu/PSXperia/downloads
author: yifanlu
comments: true
date: 2011-08-10
excerpt: Converts any PSX game to work on the Xperia Play
layout: project
slug: psxperia
title: PSXperia
version: '0.1'
wordpress_id: 410
categories:
- Assembly
- Java
- Xperia Play
---

This tool will take a PSX image that you legally own and convert it to be playable on the Xperia Play with the emulator extracted from the packaged game "Crash Bandicoot."

To build, you need to copy the following to the "lib" directory
* apktool.jar from [http://code.google.com/p/android-apktool/](http://code.google.com/p/android-apktool/)
* commons-io-2.0.1.jar from [http://commons.apache.org/io/download_io.cgi](http://commons.apache.org/io/download_io.cgi)
* sdklib.jar from Android SDK (under tools/lib)
* swing-layout-1.0.4.jar from Netbeans (under platform/modules/ext)

You also need a copy of "aapt" from Android SDK (under platform-tools)
* OSX version named aapt-osx
* Windows version named aapt-windows.exe
* Linux version named aapt-linux
Put these in the "resources" directory

Finally, you need my PSXperia wrapper library (compiled) in the "resources" directory

To run the GUI, use "java -jar PSXperiaTool.jar"
To run the command line tool, use "java -cp PSXperiaTool.jar com.yifanlu.PSXperiaTool.Interface.CommandLine" to see usage directions, which is also listed below for your convenience.

Usage:
Extract and patch data files
psxperia e[x]tract [-v|--verbose] input.apk input-data.zpak output
[-v|--verbose] Verbose output
input.apk Either com.sony.playstation.ncua94900_1.apk or com.sony.playstation.ncea00344_1.apk
input-data.zpak Either NCUA94900_1_1.zpak or NCEA00344_1_1.zpak (must match region of APK)
output Directory to extract the files

Convert ISO to Xperia Play APK and ZPAK
psxperia [c]onvert [OPTIONS] titleId image.iso output
titleId An unique ID, usually from the game in the format NCXAXXXXX_1
image.iso Input PSX image. You must rip it on your own!
output Directory to output files
Options (unset options will be set to defaults):
-v|--verbose Verbose output, including image creation progress
-D directory Custom location for extracted data files, default is "./data"
--load-xml Load options from Java properties XML
--game-name Name of the game
--description Description of the game
--publisher Publisher of the game
--developer Developer of the game
--icon-file Path to image for icon
--store-type Where to find this title (any string will do)
--analog-mode true|false, Turn on/off analog controls (game must support it).

Convert image.ps to ISO
psxperia [d]ecompress [-v|--verbose] input.ps output.iso
[-v|--verbose] Verbose output
input.ps image.ps from ZPAK
output.iso ISO file to generate

### Changes

* **2011-08-10**: 

### Screenshots

![Screen 0](/images/2012/01/psxperia_screen.png)

