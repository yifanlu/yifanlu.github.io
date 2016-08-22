---
archive: https://sites.google.com/a/yifanlu.com/downloads/rejuvenate-0.3.4-beta.zip
author: yifanlu
comments: false
date: 2015-06-20 19:32:34-06:00
excerpt: Native homebrew for PS Vita
layout: project
slug: rejuvenate
source: https://github.com/yifanlu/rejuvenate/
title: Rejuvenate
version: 0.3.4
wordpress_id: 1211
categories:
- PS Vita
tags:
- rejuvenate
---

See [this post](/2015/06/14/rejuvenate-native-homebrew-for-psvita/) for more information.

Please follow the directions in README.txt to use. Support will be provided in [#vitadev](irc://irc.efnet.net/vitadev) IRC channel on EFNet.

(Thanks to egarrote from elotrolado.net for the logo!)

### Changes

* **2015-06-19**: Public beta release.
* **2015-06-21**: Fixed bug where installing to a path containing a space does not work.
Fixed bug where launching homebrew with ARM entry point crashes PSM
* **2015-08-07**: Added PSM Unity support (thanks to Netrix)
* **2015-08-08**: Fixed bug with PsmSigner not working on 32-bit systems
* **2015-08-13**: Minor fixes to bugs in the setup script
Fixed connection problem with Unity for OSX/Linux (package installation still unsupported, but you can run VitaDefiler with Mono)
* **2015-08-28**: The functions to set the CPU, GPU, and bus clock frequencies can be used on PSM Unity 1.06
Fixed uvl_elf_get_module_info returning zero always
* **2015-09-15**: Add support for printf output through debugnet
VitaDefiler functionality moved to DLL--allows implementation of better frontends

### Screenshots

![Screen 0](/images/2015/06/Logo_Rejuvenate.png)

