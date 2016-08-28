---
author: yifanlu
comments: true
date: 2016-08-27
layout: post
slug: henkaku-update
title: HENkaku Update
categories:
- Releases
- Vita
tags:
- vita
- henkaku
- jailbreak
- update
- psn
---

![Version Screen](/images/2016/08/IMG_0302.PNG){: .alignleft} It's been almost a month since the release of HENkaku. We now have over 100,000 unique installs! (That number excludes re-installs required after rebooting.) To celebrate, we are pushing the third major update and it includes features that many users have been asking for. For the impatient, you can get it right now by rebooting your Vita and installing HENkaku from **https://henkaku.xyz/**.

## Release 2

First, let's recap the features we added since the initial release.

  * **Dynarec support**: Developers can generate ARM code and execute it directly. This aids in JIT engines for emulators.
  * [Offline installer](https://github.com/henkaku/offline-installer/releases): HENkaku can now run without a network connection thanks to work by xyz. He also made a nice [writeup](https://blog.xyz.is/2016/henkaku-offline-installer.html) that you should check out if you're interested in the technical details.
  * [VitaShell 0.7](https://bitbucket.org/TheOfficialFloW/vitashell/): When we originally released HENkaku, we forked VitaShell to molecularShell because we didn't want to spend too much time writing our own file manager. Thanks to The_FloW, our changes have been merged to the official VitaShell codebase and we no longer need molecularShell. This release had added many new features and bug fixes to the shell.

## Release 3

Now, the fun stuff. Today, we are pushing the next major update to HENkaku. The following features will be available the next time you run the online [HENkaku installer](https://henkaku.xyz). [Self-hosters](https://github.com/henkaku/henkaku/releases/latest) should get the changes from Github.

  * **PSN spoofing**: You can access PSN without updating to 3.61! Please continue reading for some important notes.
  * Safe homebrew support: Developers can optionally mark their homebrews as "safe" and it will _not_ gain restricted API access. We highly recommend developers who are not using such features to update their packages as safe.
  * [VitaShell 0.8](https://github.com/TheOfficialFloW/VitaShell): Read the release notes from The_FloW for the list of changes to VitaShell.
  * Version string: A callback to the PSP days where every hack would change the system version string. We do that too now (see the screenshot) so we can provide better support to our users.
  * Update blocking: In HENkaku mode, firmware updates using the official servers are blocked. That way you won't accidentally install 3.61 and it won't download in the background regardless of your settings.

Again, you will see these updates immediately the next time you install HENkaku if you use the online installer. If you use the offline installer, you need to update the payload. To do this, you need to temporarily enable network functionality on your Vita and open the offline installer bubble (NOT the Mail application). Install offline HENkaku again and re-disable network functions. The next time you run offline HENkaku with the Mail application, you should see the new payload. Because of how the offline installer works, this will _not_ update VitaShell. You must run the online installer again to get the latest VitaShell. Optionally, you can download the [VitaShell 0.8 VPK](https://github.com/TheOfficialFloW/VitaShell/releases/latest) and install it.

### PSN Spoofing

You can access PSN after enabling HENkaku on 3.60 but please heed this warning. Using hacks on your PlayStation console is (and always has been) against the PSN terms of service and is a ban-able offense. We have had hacks of various forms on the Vita for years and nobody has ever been banned and hopefully this will stay true in the future. However, because HENkaku has opened up the console more than any previous hacks, we might be at a point when Sony decides to enforce the PSN ToS and start banning people. That is why my personal recommendation is that you _do not use PSN_ on your HENkaku enabled console even though we give you the option to (at your own risk). If you are paranoid, you may want to use only the offline installer so your Vita does not communicate with Sony's servers. Or you may want to format your console in order for the console to not be associated with your main PSN account. Again, there has not been any confirmed bans nor have I heard of an incoming ban-wave, but my gut feeling is that you should be prepared.

The PSN spoofing is only temporary! The next time Sony releases an update, I predict that spoofing will become a lot more difficult to do. So make sure you download the games you want and update your PS+ licenses while you still can.

### Safe Homebrew

HENkaku gives developers access to public APIs (the same APIs licensed developers use to make games), private APIs (hidden APIs that may be exposed to licensed developers in the future), and restricted APIs (APIs used internally by the operating system and is not meant for external developers to use). We have seen many cool homebrews that make use of restricted APIs. For example, [RegistryEditor](https://github.com/some1psv/RegistryEditor/releases) by some1 allows you to access hidden system settings not exposed in the Settings application. There are also experimental homebrew that allow you to modify system files (at your own risk) in order to change layouts and to find exploits. Unfortunately, it also allows for malicious developers to write homebrew that wipes your memory card or (although we have not seen such an application yet) even brick your console. We have always warned the community to be vigilant, but from a design perspective, it does not make sense to give every homebrew full access.

Therefore we added the option for developers to specify their homebrew as "safe" and not get access to restricted APIs and not disable the filesystem sandbox. All you have to do is download the [latest toolchain](https://bintray.com/package/files/vitasdk/vitasdk/toolchain?order=desc&sort=fileLastModified&basePath=&tab=files) and change the call to `vita-make-fself` in your Makefile to `vita-make-fself -s`. Safe homebrews can still access all public APIs and private APIs (so you still have dynarec, changing clock speed, etc) as well as specific directories on the memory card, but there is no access to restricted APIs (registry, system partitions, etc).

Most homebrews would already be considered "safe" (you would know if you used a restricted API). However, the big catch is that `ux0:` (memory card) access is now restricted to `ux0:data` (for arbitrary data), `app0:` (a mount of your application directory at `ux0:app/TITLEID`), and `savedata0:` (a mount of your application save). There is no direct access to `ux0:app/TITLEID` since safe homebrews are sandboxed. If you wish to use and store custom data on the memory card, please use `ux0:data` as it can be accessed by all applications and is not deleted when your bubble is deleted (useful for emulators).

So what about unsafe homebrew? HENkaku still supports running them, but VitaShell will now throw a nice and scary warning message whenever the user attempts to install an unsafe homebrew. The hope is that if someone decides to package up a bricking malware as a "game", the user can be alerted because games wouldn't need extended permissions. However, in order for this warning system to work, developers of safe homebrew must update their current packages to be safe. We do not want to numb users to the warning as all "legacy" applications are currently considered "unsafe."

To recap, if you do nothing, your `.vpk` is by default considered to be unsafe and can still have access to restricted APIs and all filesystems. If you do not wish to have the unsafe message pop up every time a user installs your vpk, then you should download the [latest toolchain](https://bintray.com/package/files/vitasdk/vitasdk/toolchain?order=desc&sort=fileLastModified&basePath=&tab=files) and change the call to `vita-make-fself` in your Makefile to `vita-make-fself -s`. All current homebrew are still supported and still work and there are no changes to the behavior of anything already installed.

## On Piracy

Now for the elephant in the room. For those who aren't familiar, I recommend reading [my reply on how I approach piracy](https://www.reddit.com/r/vitahacks/comments/4w0iej/henkaku_stance_on_piracy/d63jyt4). The short of it is that, as I've stated countless times, I do not care if _you_ pirate games or not. I personally will not write piracy-enabling or piracy-aiding tools, but if _you_ do it, then that's your business and not mine. We did not add DRM/anti-piracy code nor did we add anti-DRM/piracy code. The whole point of HENkaku is owning your own device. Sony does not get to tell you what you can or cannot do with the device you bought. Same with molecule. That is what I believe. I'm writing this because I have been receiving a lot of harassment lately for things I have not said and for ideals I do not have. Please do not waste both of our time trying to convince me that piracy is/is not bad.

## On KOTH Challenge

We have seen many great progress on the [KOTH challenge](/2016/08/05/henkaku-koth-challenge/) to reverse HENkaku. The first stage has been reversed, and as promised, xyz did an [amazing writeup](https://blog.xyz.is/2016/webkit-360.html) that filled in the rest of the details. We have seen participants piecing together stage two and I think we can expect some of them to talk publicly about it soon. Once that happens, more information will be revealed by us. I am happy to hear that the participants are really enjoying the challenge and am even more delighted to hear that non-participants are really not enjoying the challenge 😉.
