---
archive: https://sites.google.com/a/yifanlu.com/downloads/kindle_jailbreak_1.1.zip
author: yifanlu
comments: true
date: 2011-12-09
excerpt: Run unsigned code on the Amazon Kindle Touch
layout: project
slug: kindle-touch-jailbreak
source: https://sites.google.com/a/yifanlu.com/downloads/kindle_jailbreak_1.1_src.zip
title: Kindle 4/Touch Jailbreak
version: '1.1'
wordpress_id: 344
categories:
- Kindle
- Scripts
---

**Information**

What does the jailbreak do? All it does is open the door to unsigned modifications by installing a developer key into the device. It does not modify any existing files and it only writes one new file.
It does NOT do anything useful or noticeable other than this. You must find and install modifications that extend the device (the jailbreak only allows that to be possible.)

This jailbreak works on the Kindle 4 and Kindle Touch. If you have a Kindle 3, Kindle 2, or Kindle DX, check out my jailbreak for these [older devices](/p/kindle-jailbreak/).

Thanks to **[ixtab](http://www.mobileread.com/forums/showthread.php?p=1902438)** for finding out this method of jailbreaking.

After installing the jailbreak, there is NO side effects at all (battery life, stability, etc). However, because you are no longer limited to Amazon's sandbox, you could potently damage your device by installing modifications that are improperly coded or by incorrectly using the modifications. Just a warning.

**Installation**

This jailbreak is designed for usage on both the Kindle 4 and Kindle 5 (Touch) and packs in three different methods of jailbreaking into one package. Please follow the methods in order if one doesn't work.

_Method 1_:



	
  1. Plug in the Kindle and copy "data.tar.gz" to the Kindle's USB drive's root

	
  2. Safely remove the USB cable and restart the Kindle (Menu -> Settings -> Menu -> Restart)

	
  3. After the Kindle restarts, you should see a new book titled "You are Jailbroken", if you see this, the jailbreak has been successful. If you DON'T see this, continue.


_Method 2_:



	
  4. Restart the Kindle again (Menu -> Settings -> Menu -> Restart)

	
  5. After the Kindle restarts, you should see a new book titled "You are Jailbroken", if you see this, the jailbreak has been successful. If you DON'T see this, continue.


_Method 3_:



	
  6. Plug in the Kindle and copy "data.tar.gz" to the Kindle's USB drive's root

	
  7. Create a blank text file named "ENABLE_DIAGS" and save it on the Kindle's USB drive's root

	
  8. Remove the USB cable and restart the Kindle (Menu -> Settings -> Menu -> Restart)

	
  9. Once the device restarts into diagnostics mode, select "D) Exit, Reboot or Disable Diags" (using the touchscreen or 5-way keypad)

	
  10. Select "R) Reboot System" and "Q) To continue"

	
  11. You should restart back into diagnostics mode, select "D) Exit, Reboot or Disable Diags"

	
  12. Select "R) Reboot System" and "Q) To continue"

	
  13. You should restart back into diagnostics mode, select "D) Exit, Reboot or Disable Diags"

	
  14. Select "D) Disable Diagnostics" and "Q) To continue"


If you wish to run a shell script after the jailbreak process, create a file named "runme.sh" on the root of the Kindle's USB partition. Use this like a regular shell script. Make sure to remount root as read-write if you plan to modify the file system. It is safe to run the jailbreak multiple times.

**Important Notices**



	
  * **Packages on the Kindle Touch cannot work on the Kindle 4 as is and vice versa!**

	
  * **Again, the jailbreak itself does NOTHING except open the door for other packages.**

	
  * **Do not expect the jailbreak to remove ads, I don't know why so many people ask me that.**

	
  * **If you have a Kindle Touch, you should try some of my Kindle mods: SSH (see usbnetwork in downloads below), [custom screensavers](/p/custom-screensaver/), and [GUI launcher (including screen rotation)](/p/kindlelauncher/).**


**Installing Packages**

You should NOT copy any packages until AFTER the jailbreak is successful. To install a package that you obtained as a .bin file, copy it to the Kindle's USB drive's root. Then go to Menu -> Settings -> Menu -> Update Your Kindle to install.

**Uninstallation**

If you wish to uninstall the jailbreak, it is recommended that you first uninstall all packages first because you cannot run any other uninstallers after removing the jailbreak.



	
  1. Plug in the Kindle and copy the uninstaller .bin for your device to the Kindle's USB drive's root (update_jailbreak_X.Y_k4_uninstall.bin = Kindle 4, update_jailbreak_X.Y_k5_uninstall.bin = Kindle Touch)

	
  2. Safely remove the USB cable

	
  3. On the device, go to Menu -> Settings -> Menu -> Update Your Kindle


**Development**

Development for the Kindle is usually done in one of two ways.

_Java Kindlets_

Kindlet is the "official" way of writing Kindle applications. These are known as "Kindle Active Content" and are written in Java either using the official SDK or unofficially imported JARs.

[More information on writing unofficial Kindlet](http://www.mobileread.com/forums/showthread.php?t=102386)

After creating your Kindlet, you must sign it with the jailbreak Kindlet key to run it on any Kindle that installed this jailbreak.

With the official SDK, to use the jailbreak Kindlet key:



	
  1. Open up Eclipse

	
  2. Open up "Workspace Preferences" in Eclipse

	
  3. Select the "Kindle Active Content" item

	
  4. Set the "Keystore Path:" to the "developer.keystore" file found in the "keys" directory of this package

	
  5. Set the "Keypass:" to "password" (without the quotes)


To manually sign your Kindlet JAR, use the following commands:
`
jarsigner -keystore /path/to/developer.keystore -storepass password JAR_FILE Kindlet
jarsigner -keystore /path/to/developer.keystore -storepass password JAR_FILE KindletInteractionSupport
jarsigner -keystore /path/to/developer.keystore -storepass password JAR_FILE KindletNetworkSupport
`
where /path/to/developer.keystore is the actual path to the "developer.keystore" file found in the "keys" directory of this package and JAR_FILE is the name of your Kindlet JAR.

**Other Apps**

Any other ARM Linux application (Linux ELFs, Shell Scripts, etc) can be installed to the device using a signed update package. This is more advanced, and the developer should take care of startup scripts, framebuffers, GUI, etc. All Kindles run the Linux 2.6 kernel and contains all standard GNU libraries.
To cross compile ARM Linux code, you must use a toolchain. Below are two examples of ARM toolchains that you could use:
[ http://www.scratchbox.org/](http://www.scratchbox.org/) (There is evidence that Amazon uses this)
[ http://www.codesourcery.com/gnu_toolchains/sgpp/](http://www.codesourcery.com/gnu_toolchains/sgpp/) (I personally use this)

After creating your native application, you can install it on any jailbroken device by creating an update package. It is recommended that you use a packager such as my [Kindle Tool](/p/kindletool/) (see the project link for more information) to generate these packages. To make an installer package, create a shell script named anything (.sh) in a directory containing all the files in your update. This script will run as root on the Kindle when your update package is installed, so use it to add, remove, or modify files. The working directory for the script is the same directory that the script is in, so everything in the input directory passed to Kindle Tool will be in the update.
If you wish to manually sign update packages (no information is provided, check the Kindle Tool source if you're curious), the RSA private key for signing jailbreak update packages is provided in the "keys" directory of this archive under "updater_key.pem".

Also, [here](/download/36) is the original Kindle Touch MP3 jailbreak for archival purposes.

### Changes

* **2011-12-08**: First release.
* **2012-01-27**: Update for Kindle Touch 5.0.3 support and Kindle 4 support.

