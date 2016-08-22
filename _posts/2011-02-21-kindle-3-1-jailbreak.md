---
author: yifanlu
comments: true
date: 2011-02-21
layout: post
slug: kindle-3-1-jailbreak
title: Kindle 3.1 Jailbreak
wordpress_id: 246
categories:
- Kindle
- Releases
- Technical
tags:
- amazon
- custom
- exploit
- jailbreak
- keys
- kindle
- script
- security
- shell
- Updates
---

I was bored one weekend and decided to jailbreak the new Kindle firmware. It was time consuming to find bugs, but not difficult. Unlike the iPhone, the Kindle doesn't really have security. They have a verified FS and signed updates and that's it, but I will still call my jailbreak an "exploit" just to piss you off. Previous Kindle 3 jailbreaks worked (AFAIK, I haven't really looked into it) by tricking the Kindle into running a custom script by redirecting a signed script using a syslink. This worked because the updater scans only "files" that do not end with ".sig" (signature files to validate the file). They fixed this now by scanning all non-directorys that do no end with ".sig". This is the first bug I've exploited. Part one is getting the files into the update, which I did by conventionally renaming them to ".sig" even though they're not signature files. Part two is harder, getting the unsigned script to run.<!-- more -->

How the Kindle updater works is that first it gets a list of all files (including files in subfolders, excluding signature files) in the update and checks it's signature with Amazon's public keys. If you modify any of the scripts from a previous update, the signature is broken and the Kindle won't run it. If you add your own scripts, you can't sign it because you don't have Amazon's keys, and finding them would take more then the lifespan of the universe. (SHA256 HMAC). They also use OpenSSL to check the signatures, so trying to buffer overflow or something is out of question (or is it? I haven't looked into it). Afterwards, when all files are matched with their signatures and checked, the updater reads a ".dat" file which contains a list of all scripts, their MD5 hash and size (to verify, I don't see the point since they were just signature checked. Maybe a sanity check?). It finds the ".dat" file using `find update*.dat | xargs` which means all the .dat file has to be is start with update and end with .dat. They don't care what is in between. Next, they read the file using "cat" and with each entry, verify the hash and loads the script. Well, conventionally, "cat" can read multiple files if more then one filename is given in the input. This means if the update*.dat file contains spaces, then "cat" will read every "filename" separated by a space. I took a signed .dat from one of Amazon's update. Renamed it "update loader.sig .dat" and placed my actual .dat (containing an entry to the script jailbreak.sig, a shell script renamed) in loader.sig. jailbreak.sig untars payload.sig, a renamed tgz file which contains the new keys we want to use to allow custom updates. Amazon's updater only signature checks "update loader.sig .dat" which is valid. Then cat tries to read the files "update", "loader.sig", and ".dat", one of which exists and the others silently fail. Loader.sig points to the script jailbreak.sig which the updater happily loads thinking it's already signature checked. Jailbreak.sig, calls tar to extract payload.sig and copies the new keys to /etc/uks and installs a init.d script to allow reverting to Amazon's keys for installing future updates. Now we own the system again!

**tl;dr:**

A download to the jailbreak can be found [here](/p/kindle-jailbreak). Directions are provided in the readme file. Use it at your own risk (I'm not responsible if you somehow brick it) and note that it most likely will void your warranty. Make sure to uninstall all custom updates before you uninstall the jailbreak, as after uninstalling the jailbreak, you cannot run custom packages until you jailbreak again. Directions for switching between custom updates and Amazon updates can be found in the readme file.
