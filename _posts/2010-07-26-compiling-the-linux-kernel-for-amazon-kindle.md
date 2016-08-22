---
author: yifanlu
comments: true
date: 2010-07-26
layout: post
slug: compiling-the-linux-kernel-for-amazon-kindle
title: Compiling the Linux kernel for Amazon Kindle
wordpress_id: 173
categories:
- Guides
- Kindle
- Technical
tags:
- amazon
- arm
- compile
- hacks
- kernel
- kindle
- linux
- mod
---

So, I recently bought a Kindle 2. As usual, the minute it arrived, I ripped it apart, poked every chip, and then started to reverse engineer the damn thing. Wait. I didn't have to! I found this out days late, after messing with IDA Pro. Amazon has generously released most of the back end code for the Kindle as open source. (The front end, aka the stuff you see, is written in Java and we might get to that another day). So I decided to compile my own Kindle kernel. Why? Why not. Here's how:

**Part 1: Prerequisites**



	
  * Get a root shell of your Kindle. If you don't know, Google "usbNetworking"

	
  * A Linux computer for compiling code

	
  * Amazon's sources for your version of the Kindle: [http://www.amazon.com/gp/help/customer/display.html?nodeId=200203720](http://www.amazon.com/gp/help/customer/display.html?nodeId=200203720)

	
  * An ARM cross-compiler. You can compile Amazon's code, or if you're lazy, use CodeSourcery's precompiled toolchain: [http://www.codesourcery.com/sgpp/lite/arm](http://www.codesourcery.com/sgpp/lite/arm)

	
  * The following packages, get them from your distro's repo: _libncurses-dev_ (for menuconfig), _uboot-mkimage_ (for making the kernel image), and _module-init-tools _(depmod)


**Part 2: Compiling the kernel**



	
  1. Extract the source to anywhere. If you can't decide, use "_~/src/kernel/_" and "_cd_" to the source files.

	
  2. Now, you need to configure for the Kindle, type "_make mario_mx_defconfig_"

	
  3. Edit the "_.config_" file and look for the line that starts with "_CONFIG_INITRAMFS_SOURCE_". We don't need that, delete that line or comment (#) it out.

	
  4. Here's the part were you make all your modifications to the kernel. You might want to do "_make menuconfig_" and add extra drivers/modules. I'll wait while you do that.

	
  5. Back? Let's do the actual compiling. Type the following: "make ARCH=arm CROSS_COMPILE=~/CodeSourcery/Sourcery_G++_Lite/bin/arm-none-linux-gnueabi- uImage". This will make the kernel image. I assume you installed CodeSourcery's cross compiler to your home folder (default). If your cross compiler is elsewhere, change the command to match it.

	
  6. Compile the modules into a compressed TAR archive (for easy moving to the kindle): "_make ARCH=arm CROSS_COMPILE=~/CodeSourcery/Sourcery_G++_Lite/bin/arm-none-linux-gnueabi- targz-pkg_" (again, if your cross compiler is installed to a different location, change it).

	
  7. For some reason, depmod refuses to run with the compile script, so we're going to do it manually. Do the following "_depmod -ae -F System.map -b tar-install -r 2.6.22.19-lab126 -n > modules.dep_" Change 2.6.22.19-lab126 to your compiled kernel version.

	
  8. Open modules.dep up with a text editor and do a search & replace. Replace all instances of "kernel/" with "/lib/modules/2.6.22.19-lab126/kernel/" (again, use your version string). I'm not sure this is needed, but better safe then brick.

	
  9. Now copy _arch/arm/boot/uImage_, _linux-2.6.22.19-lab126.tar.gz _(or whatever your version is), and _modules.dep_ to an easy to access location.


**Part 3: Installing on Kindle**



	
  1. Connect the Kindle to your computer, and open up the storage device. Copy the three files you moved from the previous part to your Kindle via USB.

	
  2. This part is mostly commands, so get a root shell to your Kindle, and do the following commands line by line. Again, anywhere the version string "_2.6.22.19-lab126_" is used, change it to your kernel's version. Explanation follows.




```bash
mv /mnt/us/linux-2.6.22.19-lab126.tar.gz /mnt/us/modules.dep /mnt/us/uImage /tmp
mv /lib/modules /lib/modules.old
cd /tmp & tar xvzf /tmp/linux-2.6.22.19-lab126.tar.gz
mv lib/modules /lib/
chmod 644 modules.dep
mv modules.dep /lib/modules/2.6.22.19-lab126/
/test/flashtools/update-kernel-both uImage
sync
shutdown -r now
```


Wow, that's a lot of commands. What did that do? Well, line by line:



	
  1. Move the files we compiled to the temp folder. That way, we don't have to clean up.

	
  2. Back up the old kernel modules

	
  3. Go to the temp folder and untar the modules

	
  4. Install the modules

	
  5. Correct the permissions for the modules.dep file (in case something happened after copying from your computer)

	
  6. Move the module dependencies list to it's correct folder.

	
  7. Flash the kernel (I don't know why it has to be flashed twice to two different partitions, but if you don't, it won't load, maybe sig checks?)

	
  8. Make sure everything is finished writing

	
  9. Reboot


