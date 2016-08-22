---
author: yifanlu
comments: true
date: 2015-06-15
layout: post
slug: how-to-register-and-download-for-psm
title: How To Register and Download for PSM (Shutdown Bypass)
wordpress_id: 1195
categories:
- Announcements
---

**Update:** It seems that Sony has closed this loophole. However, if you own a PS3, there is [another way](http://wololo.net/talk/viewtopic.php?p=388138#p388138).

Did you [miss](/2015/05/07/you-should-register-for-psm/) the call to register for PSM? People found out today that if you did not register for PSM (not the publisher's license that requires manual approval, but the general developer registration), they cannot download the Developer Assistant app on their Vita in order to run [Rejuvenate](/2015/06/14/rejuvenate-native-homebrew-for-psvita/). Not to worry, I have discovered a workaround. But act quickly because Sony will patch this loophole soon and possibly also remove the Developer Assistant from PSN, so you should download it _now_.
<!-- more -->

Please note that the steps below requires running Javascript in the URL bar. Most browsers support this features, but I know that Chrome does not allow you to paste any text beginning with "javascript:" into the URL bar. So on Chrome (and possibly other browsers), you must paste just the text after "javascript:" into the URL bar and the manually type in "javascript:" at the beginning.




	
  1. First, begin the registration process by following [this link](https://psm.playstation.net/portal/en/#register/signin).

	
  2. Enter the following into the URL bar on registration page `javascript:$("#lbl-dvrg-signin-signin").removeClass('disabled');`

	
  3. You should now be able to click the "Sign In" button. Proceed to sign into PSN.

	
  4. Enter the following into the URL bar on the license agreement page
`javascript:$(".btn,.btn-primary,.btn-large,.spn2").attr('id','lbl-dvrg-agreement-next').removeClass('disabled').removeAttr('disabled');`

	
  5. You should now be able to click the "Next" button. Proceed with the rest of the registration. Complete the registration by clicking the link sent to your email and logging in again to PSN.

	
  6. **PlayStation®Mobile Developer Assistant** should now show up in your Downloads List on your Vita. You can download it from PSN on your Vita.



This is likely your _last chance_ to get PSM. Please do it soon before it's gone forever! Remember, even though the current exploit to be released has many limitations (including no VitaTV support), any future homebrew exploit (which may have less limitations) will likely use PSM DevAssistant, so if you ever want homebrew on your Vita device, you must download the DevAssistant application.
