---
author: yifanlu
comments: true
date: 2015-04-22 23:53:41-06:00
layout: post
slug: opening-up-cardboard-crafting-an-american-new-3ds-non-xl
title: 'Opening Up CARDBOARD: Crafting an American New 3DS (non-XL)'
wordpress_id: 1060
categories:
- 3DS
- Assembly
- C
- Guides
- Technical
tags:
- 3ds
- act
- arm11
- cardboard
- eshop
- gateway
- nand
- nim
- nintendo
- nnid
- rdt
- rest
- soap
- uds
---

Last time, I analyzed now [update checks](/2015/03/23/nintendo-3ds-system-updater/) worked on the 3DS. That was a straightforward process. CARDBOARD (known colloquially as "System Transfer") is a bundle of complexity with no less than three separate servers communicating with each other as well as the device. A custom proprietary protocol is used for 3DS to 3DS communication. Finally, we have multiple unique identifiers the console uses to identify itself with Nintendo (serial, certificates, console id, account id, etc). I can't imagine this will be comprehensive, but I hope that whoever is reading can gain new insight on the complexity of the 3DS ecosystem.<!-- more -->


## Nintendo Servers


Before we begin, it would be beneficial to have a quick breakdown of the different Nintendo servers and what their uses are with respect to account management.


### SOAP Servers


First, we have the SOAP servers which the 3DS communicates with using XML SOAP. The client that speaks to the SOAP servers is the "[nim](http://3dbrew.org/wiki/NIM_Services)" module.

**NetUpdateSOAP (NUS)**

This server is used to handle update requests and is detailed in my last post.

**ECommerceSOAP (ECS)**

This is the gateway to all 3DS communication. Whenever the 3DS needs to speak with any of the SOAP servers, it first makes a request to "GetAccountStatus", which returns the console's (legacy) account id, region lock information, and other identifying information. It is also lets the 3DS "sync up" with Nintendo. For example, if the account id changed (off-line transfer, system format, etc) then this request returns error code 903 which forces NIM to obtain the proper account id and token through IVS. The account Country (region lock) ensures that requests are being made to servers in the right country. The AccountStatus returned is usually 'R' (no NNID linked) or 'T' (NNID linked). However, it can also be 'U' (device unregistered) or 'P' (account transfer is in process). If any of those responses are read, NIM will try to re-register the device (aka, create a new legacy account or transfer an old one over). Next, a list of the server URLs for all the other SOAP servers is returned. Finally, a list of ticket-ids associated with the account is returned. This is a list of all eShop titles and DLCs that the account is licensed for (which can be very large I assume). If the device sees any ticket ids it does not have currently installed, it will download the tickets. All of this is done during an Initialization request to speak with ECS servers from whatever system title made the request. That means every time any system application wishes to speak with Nintendo SOAP servers (for whatever reason), the device syncs up with Nintendo.

**IdentityAuthenticationSOAP (IAS)**

This handles what I call "legacy accounts." Before NNIDs were introduced, the account id is unique per device. When you connect a new 3DS to the internet for the first time, a "Register" request is sent to IAS along with the device serial, unique certificate, and a signature over the entire request using the certificate. Nintendo assigns your console an account id and that is used in the eShop. If you choose to delete your unlinked account from the eShop settings menu, the "Unregister" request is made which delinks your legacy account from your unique console certificate. Then the next time GetAccountStatus is called, it will return 'U' for AccountStatus which will force the device to register for a new account.

**CatalogingSOAP (CAS)**

I think this is used to get information about system-wide parameters like "how often can an account be moved". It is not used much and I did not look into it because it does not handle any account unique requests.


### NNID Server


Next we have the "account.nintendo.net" server which appears to be REST based. The "[act](http://3dbrew.org/wiki/Title_list#00040130_-_System_Modules)" module is responsible for speaking to it. This was introduced with the update that brought NNID. When you create a NNID, it is linked to your device serial and certificate (until you transfer it). Logging in to the NNID server consists of sending a username and hashed password along with the device serial, certificate, and region information. If everything matches what Nintendo has on file, an OAuth token is returned and used for further communications. It is important to note that NNIDs must be associated with a legacy account in addition to a device id. Upon first linking, titles associated with the legacy account is transfered over to the NNID. This means if you bought a console with a game pre-installed and you transfer your NNID over, that game will now be linked with your NNID. Linking an account also means that it is not possible to use that legacy account without the same NNID. When you link an account and format the system, you are forced to create a new legacy account. (I'm unsure if this means that if you buy content with this new legacy account and re-link your NNID if that will transfer the contents over. I believe that would be the case.)


### eShop Servers


The eShop servers are also RESTful and are named "samurai", "ninja", "ccif", and "eou". Samurai is used for eShop requests (game details, screenshots, etc) and ninja is used for account information. The other two I have not observed yet. If the account is not linked (legacy account), getting an authentication token for ninja involves passing in just the serial number, region, and account token (from IVS). If the account is linked to an NNID, then a "service token" obtained from the NNID server (requested with the OAuth token) is also passed to ninja. This is how ninja is linked to IVS and NNID servers.


## System Transfer


**Initialization**

When you first launch CARDBOARD (either system), as with any SOAP communication, the initialization requests GetAccountStatus which ensures the console is registered and up to date (account-wise, firmware-wise is done through NUS). If the Country returned here does not match the CARDBOARD title's region, an error is shown. Next a request for GetCountryAttributes is made to CAS which returns MoveAccountMinDays, MoveAccountMaxTimes, and other information. Following that, an IVS request for GetAccountAttributesByProfile is made to check if the account information adheres the limits set by the CAS response. Finally, a couple of ECS requests are made to query the status of the account (balance, tickets, etc) which are likely cached for later use.

**Receiving Console**

At this point, you are given the option to send or receive from this console. If you are the receiving console, the console begins to broadcast UDS (Nintendo's custom encrypted local wireless protocol) packets with the profile name, if the device is a development unit, and a hash of the console's serial.

**Sending Console**

On the sending console, if it is NNID linked, the user will be prompted to input their NNID credentials. Once successful, the sender listens for UDS broadcasts and displays the ones it sees on screen. Once a receiver is selected, the two exchange basic information about the protocol version (there are current four versions) and various details on the devices. Because UDS does not guarantee packet integrity or arrival, a CRC is added to each packet header. To possibly prevent replay attacks, a random number is placed in each packet and a MD5 hash of the entire packet is included at the end.

Once both consoles are satisfied with the other's reliability and status, the sending console disconnects from local wireless and connects back to the internet. Using either the NNID service token or just the IVS account token (for unlinked devices), a session with ninja (eShop) is established. This checks that the account is valid and the session is kept open for a later step. A request is made to IVS for MoveAccount with CheckOnly set to true. It passes the account ids, device ids, and tokens for both devices to check if a transfer is allowed. If, for example, the two devices are in different regions, a transfer is not allowed. If no error is returned, the user is now prompted to confirm the transfer (decide how to move the SD card data, move DSiWare titles, and other housekeeping tasks).

**Transferring Data**

Now the two consoles connect back using UDS and data transfer takes place over RDT, Nintendo's reliable data transfer protocol implemented over UDS. The contents of [nand:/data/ID0](http://3dbrew.org/wiki/Flash_Filesystem#CTR_partition) is copied from the old console over to the new one. I haven't looked into the specifics of how it works, but I speculate that the receiving console's SEED (in memory) is changed temporary to the sending console's SEED. This allows the receiving console to create extdata and sysdata under the sending console's ID0. Then it creates the data archives through the usual [FSUser](http://3dbrew.org/wiki/Filesystem_services) APIs and fills them with the data received from the old console. After the transfer is complete but before it is validated with Nintendo, the old SEED is restored so the new data cannot be used until the receiving console is synced up with Nintendo.

In the last step (transferring eShop shows up on screen), on the sending console, another request to IVS MoveAccount is made, this time CheckOnly is not set. This effectively transfers the legacy account over. Now, the NNID is transfered over with a request to the NNID server in order to link the new console serial/certificate with the NNID. At this point the old console's CTRNAND is formatted (even before the "format" prompt shows up).

**Validation**

I have never reached this step since I was not able to perform a proper system transfer (not having access to two consoles from the same region) however I can speculate what happens next. The receiving console creates an empty shared extdata (0xe0000000) and reboots. Home menu sees this extdata and loads up CARDBOARD. It connects to ECS and sees the account status is 'P' and gets the new account details. As part of the GetAccountStatus response, a list of all tickets associated with the account is returned (and all tickets are downloaded). Note that since GetAccountStatus is made any time ECS communication is established, this can also happen the next time you enter eShop (if you interrupt the system transfer after the last step above). That is also how offline (Nintendo Support) transfers work. Then it writes the sending console's SEED to the NAND and restores its own configuration save (wifi settings, profile name, etc) and reboots.


## Faking this Process


I was able to fake a system transfer from my USA 3DS XL to my JAP N3DS by augmenting and substituting each of these steps. While the complete process is more complicated than I have time to describe (plus I haven't kept notes of everything I did), I can give the basic outline of what I did here.

First, I performed a region swap using the [SecureInfo replacement trick](https://gist.github.com/yellows8/f15be7a51c38cea14f2c) (using my old 3DS's SecureInfo means I did not need to patch signature checks, but it also means I must manually patch my old serial number in memory every time I wish to enter eShop). Next I transfered my NNID by running CARDBOARD with the region checks patched out (one patch to replace GetAccountStatus response Country to be US and another one to ignore the error code from MoveAccount). Although the IVS MoveAccount fails, the NNID transfer seemed to have worked. At this point, my American NNID is associated with my Japanese console and Japanese legacy account. Since my legacy account did not transfer over, the validation step will always fail. So I dumped my NAND and manually imported the movable.sed over. Since the system data was already transfered over, I had to "activate" it by removing the 0xe0000000 extdata and copying over my old configuration save. Then I re-flashed my modified NAND and linked my NNID (after patching my old serial in memory). Finally, I was able to enter eShop and trigger the ticket download. Now my old SD card works and all my saves and games transfered over.

[![World's first US ambassador N3DS non-XL!](/images/2015/04/FullSizeRender-280x300.jpg)](/images/2015/04/FullSizeRender.jpg)

World's first US ambassador N3DS non-XL!
{: .wp-caption-text}




## Some Tools


In this long journey to produce an American 3DS (non-XL) with all my information transferred over, I made a couple of homebrew tools and patches that others may find helpful.



	
  * [3DSInstallTickets](https://github.com/yifanlu/3DSSystemTools/tree/master/3DSInstallTickets) lets you import tickets and CIAs into the system. Not really useful now since [FBI](http://gbatemp.net/threads/release-fbi-open-source-cia-installer.386433/) has a much nicer interface and more features.

	
  * [3DSTransferDevice](https://github.com/yifanlu/3DSSystemTools/tree/master/3DSTransferDevice) lets you export movable.sed and import SecureInfo and movable.sed. It uses official APIs which does verification checks on the data you're importing so it is "safer" than manually writing the files to the NAND. Of course, you can still brick your device with this so be careful!

	
  * [CardboardPatches](https://github.com/yifanlu/3DSSystemTools/tree/master/CardboardPatches) are the patches I wrote to log and analyze CARDBOARD (especially the local communication stuff that disconnects you from NTR debugger). Not useful to anyone except hackers wanting to continue this work.

	
  * [Spider3DSTools (NIM Patcher)](https://github.com/yifanlu/Spider3DSTools/tree/nim-patching) patches NIM to use your own servers on the O3DS. This allows you to bypass updates on 9.2 and use CARDBOARD. The code is really hacky and was put together in an hour or so.

	
  * [service-patch (Spider fork)](https://github.com/yifanlu/service-patch) attempts to port archshift's wonderful ARM11 service patching utility to work with Spider on O3DS and was an attempt to make the code above more beautiful. This currently does not work and I don't think I'll ever finish it now that I have no use for my O3DS anymore, but it should be "mostly done" (like [spiderninja](https://github.com/yifanlu/ninjhax/tree/spiderninja)).

	
  * [Patches for eShop](http://gbatemp.net/threads/creating-a-north-american-non-xl-new-3ds.381775/page-37#post-5459760) to work are needed every time you wish to connect to eShop on your region-swapped NNID linked console. This is because officially your legacy account is still in another region (and eShop application errors on this) even though your NNID is in the right region and only the NNID account is used for eShop. This also patches the update checks so you can enter eShop on 9.2.

	
  * [NNID transfer guide](https://gist.github.com/yifanlu/5d16e134332622d26944) for advanced readers. Exact procedure/patches I've made to get my NNID transferred over. For archival purposes.


