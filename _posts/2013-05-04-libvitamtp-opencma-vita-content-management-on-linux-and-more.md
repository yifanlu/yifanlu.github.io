---
author: yifanlu
comments: true
date: 2013-05-04 23:47:57-06:00
layout: post
slug: libvitamtp-opencma-vita-content-management-on-linux-and-more
title: 'libVitaMTP & OpenCMA: Vita content management on Linux (and more)'
wordpress_id: 619
categories:
- C
- PS Vita
tags:
- cma
- libusb
- linux
- mtp
- opencma
- usb
- vita
- vitamtp
- xml
---

More than a year ago, I've analyzed how the [Vita communicates with the computer](/2012/02/18/playstation-vitas-usb-mtp-connection-analyzed/). I mentioned at the end that I started a project that will be an open source implementation of the protocol that the Vita uses. This protocol is just MTP (media transfer protocol) with some additional commands that I had to figure out. MTP is used by most Windows supported media players and cameras, so I was able to use a lot of existing code from [libmtp](http://libmtp.sourceforge.net/) and [gphoto2](http://www.gphoto.org/). After lots of on and off work, I am happy to announce the first (beta) version of libVitaMTP and OpenCMA.<!-- more -->

**What is libVitaMTP and OpenCMA?**

In anticipation of allowing developers to include Vita support in their media management applications, and for allowing developers to create custom applications that use the Vita (for example, it would be possible to write an application that uses the Vita as a general purpose storage device), I've placed all the backend code into libVitaMTP. This includes connecting of the Vita through libusb-1.0, processing various XML data sent from the device with libxml2, and supports an interface that allows the developer to send the various custom MTP commands to the device.

OpenCMA is the first user-level software that uses libVitaMTP. It serves two purposes. First, it's a stripped down, open source implementation of CMA, that allows the user to send and receive games, backups, and media to and from the Vita. It is stripped down because Sony's official CMA supports reading the metadata from media files, and I chose not to include this feature because it would complicate things.

More information can be found on the [Github page](https://github.com/yifanlu/VitaMTP).

**Testing**

If you are a developer and wish to include libVitaMTP in your projects, please give feedback on any bugs or difficulties/bad documentation/inconsistant documentation.

If you are a user and wish to test OpenCMA, I've provided some Debian packages (created and tested on Ubuntu 12.04) for you to try. After installing, just invoke "opencma -h" in the terminal to see how to use it.

A sample run of OpenCMA could be


```bash
$ opencma -u /path/to/dir/containing/updatelist -p /path/to/photos -v /path/to/videos -m /path/to/music -a /path/to/apps -l 4
```


the -l 4 means to show more logging information. This is very important for filing bug reports. If you encounter a crash or bug, please make sure to include the output of the OpenCMA session with -l 4. Note that more logging means slower transfers.

Please be careful not to pass in large directories because OpenCMA tries to create an index of all files (recursively) for each path.

Please submit all bug reports and feature requests to the [Github issues page](https://github.com/yifanlu/VitaMTP/issues). It would make things easier if you include the -l 4 output.

**Downloads**

As always, the most up to date packages will be uploaded to [the project page](/p/vitamtp/).
