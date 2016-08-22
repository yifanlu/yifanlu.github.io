---
archive: https://sites.google.com/a/yifanlu.com/downloads/vitamtp1_2.0.0_i386.deb
author: yifanlu
comments: true
date: 2013-05-04
excerpt: Library to interact with Vita's USB MTP protocol
layout: project
site: https://github.com/yifanlu/VitaMTP
slug: vitamtp
source: https://sites.google.com/a/yifanlu.com/downloads/vitamtp-2.0.0.tar.gz
title: VitaMTP
version: 2.0.0
wordpress_id: 618
tags:
- cma
- library
- libusb
- libvitamtp
- linux
- mtp
- opencma
- sony
- usb
- vita
- vitamtp
---

libVitaMTP is a library based off of libMTP that does low level USB communications with the Vita. It can read and recieve MTP commands that the Vita sends, which are a proprietary set of commands that is based on the MTP open standard.

OpenCMA is a frontend that allows the user to transfer games, saves, and media to and from the PlayStation Vita. It makes use of libVitaMTP to communicate with the device and other libraries to intrepet the data sent and recieved. OpenCMA is a command line tool that aims to be an open source replacement to Sony's offical Content Management Assistant.

More information can be found on the [Github](https://github.com/yifanlu/VitaMTP) page.

### Changes

* **2013-05-03**: Initial binary release.
* **2013-05-05**: Removed the need for psp2-updatelist.xml
Added more help information about usage and behavior
Fixed HUGE memory leak in sending Vita/backup objects
Fixed HUGE memory leak in receiving PSP saves
Fixed a minor memory leak in renaming function
Fixed crash in Linux systems when sending apps
* **2013-05-16**: Added wireless device support
Fixed video titles not showing
Fixed invalid disk space reporting on Linux

