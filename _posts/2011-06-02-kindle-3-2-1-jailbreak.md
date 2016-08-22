---
author: yifanlu
comments: true
date: 2011-06-02
layout: post
slug: kindle-3-2-1-jailbreak
title: Kindle 3.2.1 Jailbreak
wordpress_id: 278
categories:
- Kindle
- Releases
tags:
- 3.2.1
- alpha
- exploit
- glitch
- jailbreak
- kindle
- linux
- pid
---

**UPDATE: **[Serge A. Levin](http://www.mobileread.com/forums/showpost.php?p=1725629&postcount=151) has kindly modified my "temporary" jailbreak into a more permanent solution. The information below is now considered old and should be disregarded. [Link to jailbreak for all devices on all versions](/p/kindle-jailbreak).



So I never intended to release a jailbreak for Kindle 3.2.1 because 1) people who got a discount for their Kindles should stick by their commitment and keep the ads and 2) this was an update made purely to disable jailbreaks, so there are no new features. However, from what I heard, more and more people are receiving 3.2.1 as stock firmware (not just ad-supported Kindles) and that people who exchanged their broken Kindles also have 3.2.1. I don't want to reveal the exploit I found yet (I'm saving it for the next big update), but thankfully, after half an hour of digging, I've found another glitch that I can use. The bad news is that this isn't an "easy one click" jailbreak, it will actually take some effort as some precise timing needs to be correct in order to work.<!-- more -->

**Technical Details**

What is this new glitch you ask? It's pretty simple and pretty stupid and I feel almost embarrassed to use it (that's why I'm not even using the word exploit). First of all, the last bug I found was fixed by a regex name check that prevent spaces in names. Now, whenever the Kindle gets an update, before doing anything, it looks for the signature of every file in the update (minus the signature files themselves). They do this by using the "find" command to get a file list and piping the output to "read" where "read" feeds each data (separated by a whitespace) into the signature check function where the function proceeds to use OpenSSL to check the signature. Simple enough. Well, what I want to do is make the signature check ignore a file, and to do it, I make a blank file called "\" (literally a backslash). Now it's hard to explain what happens, so I'll show you.

Here's the output of the find command usually:


```bash
$ find /tmp/update
/tmp/update/file.ext
/tmp/update/file2.ext
...
```


Now, when I insert my slash-file:


```
$ find /tmp/update
/tmp/update/file.ext
/tmp/update//tmp/update/file2.ext
...
```

What happened? The backslash is used in Linux as an escape character. Basically it says to treat the next character as not-special. Remember that "read" splits the data to be read using whitespace (in this case a new line character), so by escaping the whitespace, I can get the system to ignore /tmp/update/file2.ext, and instead get it to read /tmp/update/tmp/update/file2.ext. In that file, I will include an already signed file from an old Amazon update, and when the updater runs, it ignores the extra files and reads the unsigned file. But we're not done yet. Amazon doesn't extract the update to a set folder, it extracts it to /tmp/.update-tmp.$$ where $$ means the process id of the script running. This can be any number from 1 to 32768. So what's the elegant solution to this problem? I don't know yet. Until someone can come up with a better idea, I'm going to include PIDs 5000-7000. From my tests, if you run it immediately after a reboot, it will be 64xx, so it's a test of how bad you want to jailbreak ;)

**Installation**

Since this jailbreak is time and luck based, I've included very detailed directions on the exact timing for doing things in the readme. I suggest reading over the directions before starting, because timing is everything. It works only in a certain window of time after startup, so if it doesn't work you need to restart and try again. If it doesn't work after three or more tries, it's mostly my fault as I only tested it with a Kindle 2 so the timing might be different on the Kindle 3. If you have serial port access on your Kindle 3, send me the otaup log and I'll change the pid set.

**Download**

Since this is a temporary fix, I'm not going to add this to my projects list.

<del>Download source and binaries here</del>

**EDIT**: I've heard from some users that you have more chance of succeeding if you don't have any books to load. So, before doing anything, rename the documents folder to documents.bak and the system folder to system.bak, install the jailbreak, and rename everything back. This should allow more chance of succeeding.

**EDIT 2:** Some people report turning the wireless off before starting also increases success rates.
