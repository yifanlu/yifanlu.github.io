---
author: yifanlu
comments: true
date: 2015-06-25 01:45:04-06:00
layout: post
slug: on-the-future-of-rejuvenate
title: On the future of Rejuvenate
wordpress_id: 1257
categories:
- Information
- PS Vita
tags:
- homebrew
- psm
- psvita
- rejuvenate
- sdk
- unity
- vita
---

Since, the [announcement](/2015/06/14/rejuvenate-native-homebrew-for-psvita/) ten days ago, Rejuvenate received tons of positive reception and thousands of downloads. Progress on [both](https://github.com/173210/psp2sdk) [SDK](https://github.com/vitasdk) projects is moving at fast speeds. There are already Vita [homebrew](http://vitadevwiki.com/index.php?title=Homebrew) projects in the works. No doubt, there are more to come. However, Sony's response has not been positive. Yesterday, Sony released firmware 3.52 which revokes access to PSM DevAssistant and PSM Unity DevAssistant along with a [friendly request](https://en-support.psm.playstation.net/app/answers/detail/a_id/347) for PSM developers to delete the DevAssistant from their devices. This means that **if you ever want to run homebrew on your Vita, regardless of your opinions on the current limitations and regardless of your ability to use PSM, do NOT update to 3.52**.

[![CHIP-8 emulator by @xerpi, picture by @Chihab_rm](/images/2015/06/CIOYTHtUkAAJ8ba-300x169.jpg){: .aligncenter}](https://github.com/xerpi/VITA-8)
CHIP-8 emulator by @xerpi, picture by @Chihab_rm
{: .wp-caption-text}

<!-- more -->



### PSM+



When I first announced the Rejuvenate hack, I promised a solution for people who do not have a publisher license (and therefore cannot generate keys needed to run the hack) which I dubbed **PSM+**. PSM+ requires the use of the filesystem writing trick discovered by mr.gas and Major_Tom and also requires materials to be hosted on a server. Therefore, I gave **mr.gas**, **Major_Tom**, and **wololo** the prerogative to release PSM+ at a time they deem fit. Please follow [wololo.net](https://wololo.net/) for the release schedule. They will post instructions and provide support for PSM+ when the time comes.

![Credits to egarrote from elotrolado.net for the logo!](/images/2015/06/Logo_Rejuvenate.png){: .aligncenter}
Credits to egarrote from elotrolado.net for the logo!
{: .wp-caption-text}



### Rejuvenate



If you remember the [original announcement](/2015/06/14/rejuvenate-native-homebrew-for-psvita/), the hack came with quite a few limitations that made it less useful for the general audience. I have been working hard to try to address some of these issues, and I think I found a temporary solution to a few of them. I have confirmed earlier today that **PlayStation®Mobile Development Assistant for Unity** contains the same bug as the regular PSM DevAssistant application. That means it [can be exploited](/2015/06/21/hacking-the-ps-vita/) to run UVLoader too. The Unity app is also a better solution for running homebrew.



	
  * It runs the debugger over Wifi, so after the initial setup over USB, it is possible (for example) to launch homebrew with a custom app on your smartphone, which is a step forward from being tethered to a computer.

	
  * For the same reason as above, the VitaTV can be supported. The only requirement is installing the files either with the same filesystem writing trick as PSM+ (which can be a hassle) or swapping the memory card into a PS Vita to install over USB (which can also be a hassle). Installation only needs to be done once though.

	
  * The application is DRM free which means that you can install it with the Package Installer or through a PS3 _without needing any license_. This means that even if you did not register for PSM and even if Sony removes the app from PSN, you can still install PSM for Unity.



Again, all of this will only be for those who are on **firmware 3.51 and below**! Do NOT update past 3.51 if you ever want a chance of running homebrew on your PS Vita or VitaTV.

It will be a bit of work to port VitaDefiler over to the Unity DevAssistant app. I have already started a [branch of VitaDefiler](https://github.com/yifanlu/VitaDefiler/tree/unity-support) that can connect to Unity DevAssistant and its debugger. However, more work is needed to work out the kinks in the debugger connection as well as finding the offsets to get UVLoader working. I will likely be busy with other things so I will not have time to work on PSM stuff for the next couple of months. However, I hope that someone else can pick up the source code and finish what I have started. I'm looking forward to seeing all the wonderful homebrew you guys make!
