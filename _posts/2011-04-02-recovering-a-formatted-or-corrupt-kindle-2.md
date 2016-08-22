---
author: yifanlu
comments: true
date: 2011-04-02
layout: post
slug: recovering-a-formatted-or-corrupt-kindle-2
title: Recovering a formatted or corrupt Kindle 2
wordpress_id: 266
categories:
- Guides
- Kindle
- Technical
tags:
- arm
- assembly
- corrupt
- formatted
- hex
- ida pro
- kernel
- kindle
- linux
- recovery
- serial
- ttl
- unix
- update
---

One day, while playing around with a Kindle 2, I accidentally deleted the /lib folder. Oops. Now, no command beyond "ls" and "rm" work. If this was a computer, I could have simply inserted a installation DVD and copied the files over, but this was an eBook reader, and I was in for a world of pain. This was a month ago, and I've finally recovered the Kindle. I'm posting what I did online to save anyone else who's in the same boat a ton of time. This tutorial is only designed for the Kindle 2, but it MAY work for the DX. It will NOT work for the Kindle 3, but directions should be similar.<!-- more -->



**First**

If you've think you "bricked" your Kindle, don't panic yet. There could be a easy solution. Chances are, if you can see the startup progress bar loading, the solution should be easier (although I can't tell you exactly what your problem is). I would follow [Amazon's troubleshooting directions](http://www.amazon.com/gp/help/customer/display.html/ref=hp_200127470_k3land_k2trouble?nodeId=200375470#screen) first. Only proceed if you are absolutely sure nothing else can be done.

**Overview**

Here's what you'll need



	
  1. TTL to RS232 or USB connector. I used [this one](http://www.geeetech.com/index.php?main_page=product_info&products_id=158). For that, use the jumper on pin 1 and 2 on the side (with the three pins, pin 1 is towards the USB port). Connect Kindle Tx -> Tx, Rx -> Rx, GND -> GND, VDD -> VDD

	
  2. Windows with [HyperTerminal](http://www.hilgraeve.com/hyperterminal/) (I tried Kermit on Linux, but it couldn't send files. HyperTerminal is the only program I've tested that works sending files to the Kindle)

	
  3. Linux or Unix-based computer with "dd" and "ssh"

	
  4. My [custom recovery kernel](/files/kindle-kernel.uImage) which allows jailbreak signed recovery packages and exporting MMC0 without a password. If you want to know how I've made it in technical details, see the appendix.


Here's what we'll be doing:

	
  1. Attaching the recovery port

	
  2. Flashing the custom patched recovery kernel

	
  3. Obtaining a backup of Kindle system files

	
  4. Restoring your Kindle


**Attaching the recovery port**

First open up the Kindle 2 to reveal the PCB board. You should remove both the metal casing and the white plastic with all the screws. On the top, to the left of the headphone jack, you should see four pads labeled J4. Either solder (recommended) or tape (make sure it isn't lose!) four wires to these pads. The order of these ports (left to right, where left is towards the volume switch) are: VDD, Rx, Tx, GND. Connect these lines to your TTL adapter and connect the adapter to your computer.

**Flashing the custom patched recovery kernel**

Open up HyperTerminal, and connect to your adapter. Make sure to click "Configure" and fill in the settings. The settings are: BPS: 115200, Data bits: 8, Parity: none, Stop bits: 1, Flow control: none. Then, restart your Kindle either by removing & reconnecting the battery, holding the sleep switch for 30 seconds, or tapping the reset switch on the PCB. Press Enter when text pops up in HyperTerminal. You only have one second, so be quick. In uBook, type in "run prg_kernel_serial" (make sure to type fast or uBoot will timeout). Then right click on the text, and choose "Send File". Under protocol, choose Ymodem-G and for the file, select my custom kernel. Wait a few minutes for it to upload and install, then type in "bootm 0xa0060000" to boot the kernel. The Kindle has two kernels that it alternates on each boot, so if you want to use my recovery kernel, you need to either flash the other kernel also or type in "bootm 0xa0060000" into uboot on startup. Hold down Enter either on your computer or on the Kindle to enter the recovery menu. The recovery menu times out in 10 seconds, so you need to be quick. First type "I" to recreate all the partitions, then type "3" to export the MMC. Again, these can be typed from either your keyboard in HyperTerminal, or the Kindle keypad. If you do not have access to HyperTerminal because you are in Linux restoring, you can get back here by holding Enter on the Kindle keypad and pressing 3 on the recovery menu.

**Obtaining a backup of Kindle system files**

Let's put your broken Kindle aside. You need a working copy of Kindle's system files. I cannot provide this for legal reasons, but if you obtain another Kindle 2 (preferably the same model and version as your broken one, but others MAY work [not Kindle 3 though... yet]), jailbreak it and install the [usbNetwork hack](http://www.mobileread.com/forums/showthread.php?t=88004) for SSH access. Make sure that Kindle has at least 500MB of free space on the FAT partition to store the backup image. Once you SSH'd into the working Kindle (there are tons of tutorials around on this), issue the following command:


```bash
dd if=/dev/mmcblk0p1 of=/mnt/us/rootfs.img bs=1024
```


Note that this will only make a copy of the OS files. All personal information, passwords, books, etc are not copied. You can tell your friend that. This may take five to fifteen minutes to run, but when the command returns with the blocks written, you can disable usbNetwork and enable USB storage again. Copy the rootfs.img file over to your recovery computer and prepare to restore.

**Restoring your Kindle**

Back to your broken Kindle. You need to reformat the root and copy over the backup files. I moved the Kindle over to a Linux computer because it is easier. You can also use OSX or maybe even cygwin, but I haven't tested. In shell, type in the following commands:


```bash
sudo su # Become root, so you don't need to sudo everything
fdisk -l # Look for your Kindle's identifier, it should be something like /dev/sdc, it should be a 2GB drive with 4 partitions. I will use /dev/sdX to represent this drive
mkfs.ext3 /dev/sdX2 # Make a ext3 partition for /var/local
dd if=/path/to/rootfs.img of=/dev/sdX1 bs=1MiB # This will take a long time to finish
```


Note that an alternative method is to gzip rootfs.img and place it into a recovery package created using kindle_update_tool.py, but I'll leave that as an exercise for the reader.

**Appendix**

So, what is in the magical Kindle recovery kernel? It's actually just a regular Kindle kernel recompiled with a modified initramfs with a patched recovery script. Using the regular kernel, you'll run into two difficulties when trying to recover. First, if you press 3 to export MMC0, you'll get a password prompt. Good luck brute forcing it. Second, if you build a custom recovery package using kindle_update_tool.py m --k2 --sign --fb, it will not work because of signature checks. What I did was patch the two checks.

First, I extracted the recovery-utils file by gunzipping uImage (with the extra stuff stripped out), and gunzipped initramfs.cpio from that. Then I extracted initramfs.cpio and found recovery-utils under /bin.

Next, the easy part is patching the updater package signature checks. What I did is extract the updater RSA public key file from the Kindle, found under /etc/uks and used OpenSSL to extract the public key from it (n and e). Then I opened recovery-utils with a hex editor, searched for the public key, and replaced it with the jailbreak key (found in kindle_update_tool.py).

Finally, the challenging part was to patch the password check from export MMC0. First I opened recovery-utils with IDA Pro. Then I located the check_pass function. I worked backwards from that and saw it was called from loc_94A8. Here's a snippet of the check along with my interpretation of what it does:


```
BL	check_pass # Call the check_pass function
CMP	R0, #0 # check_pass sets register R0 with the return value, we will check if R0 equals 0x0
BEQ	loc_9604 # If the previous instruction is true, then password check has failed
LDR	R0, =aDevMmcblk0 ; "/dev/mmcblk0" # We did not jump out, so CMP R0, #0 is false
BL	storage_export # Call this function
```


It's always easy to patch the least amount of instructions with the least amount of changes, so this is what I did. (Note that IDA Pro doesn't allow editing instructions directly, so I have to find the machine code in hex to know what to replace. Luckily, I have tons to instructions to look at and see what their corresponding machine codes are).


```
NOP # Instead of calling check_pass, I did MOV R0, R0 which is the same as NOP
CMN R0, R0 # Negative compare R0 with itself. Basically, always return false.
... rest is the same
```


Now, I saved the file and luckily, it was the same size. So I didn't have to recreate the initramfs.cpio, I just replaced the file inside with my hex editor (note that cpio files do not have checksum checks unlike tar files). I copied this to the kernel source folder and compiled the kernel. Lucky for you, I have already done all of this so you don't have to.
