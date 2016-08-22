---
author: yifanlu
comments: true
date: 2012-02-19 02:54:07-06:00
layout: post
slug: playstation-vitas-usb-mtp-connection-analyzed
title: Playstation Vita's USB MTP Connection Analyzed
wordpress_id: 535
categories:
- PS Vita
- Technical
tags:
- cma
- libusb
- linux
- mtp
- packet
- playstation
- psp
- psvita
- usb
- vita
---

This is the first of (hopefully) many posts on the PS Vita. Before I attempt anything drastic with the device, such as getting unsigned code to run, I hope I can try something easy (well, easier) to get used to the device. Ultimately, I want to make a content manager for the PS Vita for Linux. Unlike the PSP, the Vita does not export the memory card as a USB storage device, but instead relies on their custom application to copy content to and from the device. This post will give just a peek into how the communication between the Vita and the PC works.<!-- more -->

There are two ways of approaching this. One is to sniff the USB packets to figure out what data gets sent to and from the device, and second is to disassemble the content manager application to find out how it communicates with the device. I tried both methods.

**Reverse engineering the Content Manager**

The biggest problem here is that the PC version of Sony's content manager has its symbols removed. This makes everything a hundred times harder as you would have a harder time guessing what each function does. Luckily, the OSX version of the content manager does have the symbols intact. The problem here is that IDA does not work perfectly with Objective-C (it works, but you get a C++ish interpretation of Objective-C). I have a good idea of how the application is laid out, but there isn't much point giving all the details (not useful). I will give some important points:



	
  * The Vita uses the [Media Transfer Protocol](http://en.wikipedia.org/wiki/Media_Transfer_Protocol), however I am not sure if it adheres completely to standards or if it uses a custom implementation

	
  * USB endpoint 0x2 is for input, while 0x81 is for data output, and 0x83 is for MTP event output

	
  * There is support for passing PSN account information to and from the Vita (password would be in plain text!), but it is unimplemented

	
  * CMA uses a SQLite database to index media information and licenses




Also, I've compiled a list of MTP operation codes for the Vita that are referenced in CMA (and therefore implemented in the Vita). Note that some of the codes are not in the [standards](http://www.usb.org/developers/devclass_docs/MTP_1.0.zip) while others are. For events, the second number is for reference with regards to the jump-table inside CMA only.


```
Events:
0xC104 0: RequestSendNumOfObject
0xC105 1: RequestSendObjectMetadata
0xC107 3: RequestSendObject
0xC108 4: RequestCancelTask
0xC10B 7: RequestSendHttpObjectFromURL
0xC10F 11: RequestSendObjectStatus
0xC110 12: RequestSendObjectThumb
0xC111 13: RequestDeleteObject
0xC112 14: RequestGetSettingInfo
0xC113 15: RequestSendHttpObjectPropFromURL
0xC115 17: RequestSendPartOfObject
0xC117 19: RequestOperateObject
0xC118 20: RequestGetPartOfObject
0xC119 21: RequestSendStorageSize
0xC120 28: RequestCheckExistance
0xC122 30: RequestGetTreatObject
0xC123 31: RequestSendCopyConfirmationInfo
0xC124 32: RequestSendObjectMetadataItems
0xC125 33: RequestSendNPAccountInfo
0xC801 1789: Unimplemented (seen when getting object from Vita)

Commands:
0x1001: GetDeviceInfo
0x1002: OpenSession
0x1007: GetObjectHandles
0x1008: GetObjectInfo
0x1009: GetObject
0x100C: SendObjectInfo
0x100D: SendObject
0x101B: GetPartialObject
0x9511: GetVitaInfo
0x9513: SendNumOfObject
0x9514: GetBrowseInfo
0x9515: SendObjectMetadata
0x9516: SendObjectThumb
0x9518: ReportResult
0x951C: SendInitiatorInfo
0x951F: GetUrl
0x9520: SendHttpObjectFromURL
0x9523: SendNPAccountInfo
0x9524: GetSettingInfo
0x9528: SendObjectStatus
0x9529: SendHttpObjectPropFromUR
0x952A: SendHostStatus
0x952B: SendPartOfObject (?)
0x952C: SendPartOfObject (?)
0x952E: OperateObject
0x952F: GetPartOfObject
0x9533: SendStorageSize
0x9534: GetTreatObject
0x9535: SendCopyConfirmationInfo (?)
0x9536: SendObjectMetadataItems
0x9537: SendCopyConfirmationInfo (?)
0x9538: KeepAlive
0x9802: ?
0x9803: GetObjectPropValue
0x9805: GetObjectPropList

Response:
0x2001: OK
0x2002: GeneralError
0x2006: ParameterNotSupported
0x2007: IncompleteTransfer
0x200C: StoreFull
0x200D: ObjectWriteProtected
0x2013: StoreNotAvailable
0x201D: InvalidParameter
0xA002: ?
0xA003: ?
0xA004: ?
0xA00A: ?
0xA00D: ?
0xA008: ?
0xA010: ?
0xA012: ?
0xA017: ?
0xA018: ?
0xA01B: ?
0xA01C: ?
0xA01F: ?
0xA020: ?
0xA027: ?

Data Types:
0xDC01: StorageID
0xDC02: ObjectFormat
0xDC03: ProtectionStatus
0xDC04: ObjectSize
0xDC05: AssociationType
0xDC07: ObjectFileName
0xDC08: DateCreated
0xDC09: DateModified
0xDC0A: Keywords
0xDC0B: ParentObject
0xDC0C: AllowedFolderContents
0xDC0D: Hidden
0xDC0E: SystemObject
0xDC41: PersistentUniqueObjectIdentifier
0xDC42: SyncID
0xDC43: PropertyBag
0xDC44: Name
0xDC45: CreatedBy
0xDC46: Artist

Object Formats:
0x3000: Undefined
0x3001: Association
0x3008: WAV
0x3009: MP3
0x3801: EXIF/JPEG
0x3804: BMP
0x3806: UndefinedReserved
0x380A: PICT
0x380B: PNG
0xB007: PSPGame
0xB00A: PSPSave
0xB014: VitaGame
0xB400: ?
0xB411: MNV
0xB984: MNV2
0xB982: MP4/MGV/M4V/MNV3
```

**USB packets**

I've also captured the USB packets for initializing the device (from device plug-in to Vita displaying the content menu) and gave my best interpretation of it. First line is PC to Vita packet or Vita to PC packet

, followed by packets captured by VMWare running Windows 7, followed by the same action on OSX (dumped from memory using GDB on CMA, not from capturing USB packets), followed by my interpretation of what the packet does (question mark means not sure). EDIT: Some of my comments in the log I know are wrong now.


**Next time**

I'm hoping to decode these packets and implement them using libusb. I hope Sony is using the MTP standard so I can also make use of libmtp. I also need to be more familiar with how the USB protocol works so I can understand the packet layout better.

**EDIT:** I've begun work on a [new project](https://github.com/yifanlu/VitaMTP) to create an open source content manager for the Vita. As of this post, it can init the device and tell it to show the main menu.
