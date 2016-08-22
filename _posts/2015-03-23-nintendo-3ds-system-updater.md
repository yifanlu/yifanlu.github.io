---
author: yifanlu
comments: true
date: 2015-03-23
layout: post
slug: nintendo-3ds-system-updater
title: Nintendo 3DS System Updater
wordpress_id: 1051
categories:
- 3DS
- Technical
tags:
- 3ds
- cdn
- downloader
- nim
- nintendo
- nus
- soap
- ssl
- titles
- updater
- xml
---

Since there isn't much public documentation on how 3DS updater and the NIM module works, I thought I should write something up.<!-- more -->


## SSL


The 3DS talks with the Nintendo update servers (as well as eShop) through SSL with a client certificate that is common to all 3DS. The client certificate, its private key, and the Nintendo root CA are found in the title [0004001B00010002](http://3dbrew.org/wiki/ClCertA). The two files found inside the title's RomFS are additionally encrypted. The [SSL system module](http://3dbrew.org/wiki/SSL_Services) [decrypts](http://3dbrew.org/wiki/PS:EncryptDecryptAes) the files and stores it into the process heap. The certificate, key, and root CA are all stored in DER format, so you may want to convert it to a PKCS12 format before using it to communicate with NUS on your own.


## NIM


The [NIM](http://3dbrew.org/wiki/NIM_Services) module is how the 3DS communicates with Nintendo's servers through SOAP (and over SSL). The following is a typical update process.

The following request is made to https://nus.c.shop.nintendowifi.net/nus/services/NetUpdateSOAP (potentially identifying information is stripped out)

```xml
POST /nus/services/NetUpdateSOAP HTTP/1.1
User-Agent: CTR NUP 040600 Mar 14 2012 13:32:39
Connection: Keep-Alive
Accept-Charset: UTF-8
Content-type: text/xml; charset=utf-8
SOAPAction: urn:nus.wsapi.broadon.com/GetSystemTitleHash
com.broadon.RequesterName: unitTest
com.broadon.RequesterHash: zzz
com.broadon.RequesterTimestamp: 1427146068799
Transfer-Encoding: chunked

<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:xsd="http://www.w3.org/2001/XMLSchema"
xmlns:nus="urn:nus.wsapi.broadon.com">
<SOAP-ENV:Body>
<nus:GetSystemTitleHash xsi:type="nus:GetSystemTitleHashRequestType">
<nus:Version>1.0</nus:Version>
<nus:MessageId>EC-xxx-142714927</nus:MessageId>
<nus:DeviceId>xxx</nus:DeviceId>
<nus:RegionId>JPN</nus:RegionId>
<nus:CountryCode>JP</nus:CountryCode>
</nus:GetSystemTitleHash>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

Since the 3DS firmware is a collection of "titles" that can be updated independently, the updater has to make sure each title is up to date. To save time, it first gets a hash and checks if anything needs to be updated. The server responds:

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<soapenv:Body>
<GetSystemTitleHashResponse xmlns="urn:nus.wsapi.broadon.com">
<Version>1.0</Version>
<DeviceId>xxx</DeviceId>
<MessageId>EC-xxx-154274329</MessageId>
<TimeStamp>1427146232957</TimeStamp>
<ErrorCode>0</ErrorCode>
<TitleHash>7E745F7B67D553BEA847859404790C93</TitleHash>
</GetSystemTitleHashResponse>
</soapenv:Body>
</soapenv:Envelope>
```

If the title hash matches the current system's hash, then the updater exits. Otherwise, it continues and makes a request to https://ecs.c.shop.nintendowifi.net/ecs/services/ECommerceSOAP to get the latest update server URLs

```xml
POST /ecs/services/ECommerceSOAP HTTP/1.1
User-Agent: CTR NUP 040600 Mar 14 2012 13:32:39
Connection: Keep-Alive
Accept-Charset: UTF-8
Content-type: text/xml; charset=utf-8
SOAPAction: urn:ecs.wsapi.broadon.com/GetAccountStatus
com.broadon.RequesterName: unitTest
com.broadon.RequesterHash: zzz
com.broadon.RequesterTimestamp: 1427146232982
Transfer-Encoding: chunked
Host: ecs.c.shop.nintendowifi.net

<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:xsd="http://www.w3.org/2001/XMLSchema"
xmlns:ecs="urn:ecs.wsapi.broadon.com">
<SOAP-ENV:Body>
<ecs:GetAccountStatus xsi:type="ecs:GetAccountStatusRequestType">
<ecs:Version>2.0</ecs:Version>
<ecs:MessageId>EC-xxx-143998661</ecs:MessageId>
<ecs:DeviceId>xxx</ecs:DeviceId>
<ecs:DeviceToken>yyy</ecs:DeviceToken>
<ecs:AccountId>yyy</ecs:AccountId>
<ecs:ApplicationId>0004013000002c02</ecs:ApplicationId>
<ecs:TIN>1234</ecs:TIN>
<ecs:Region>JPN</ecs:Region>
<ecs:Country>JP</ecs:Country>
<ecs:Language>ja</ecs:Language>
<ecs:SerialNo>zzz</ecs:SerialNo>
<ecs:ECVersion>EC 4.6.0 Mar 14 2012 13:32:39</ecs:ECVersion><ecs:Locale>ja_JP</ecs:Locale><ecs:ServiceLevel>SYSTEM</ecs:ServiceLevel>
</ecs:GetAccountStatus>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

and the server responds with the URLs to use

```xml
<?xml version="1.0" encoding="UTF-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<soapenv:Body>
<GetAccountStatusResponse xmlns="urn:ecs.wsapi.broadon.com">
<Version>2.0</Version>
<DeviceId>xxx</DeviceId>
<MessageId>EC-xxx-121712521</MessageId>
<TimeStamp>1427134562983</TimeStamp>
<ErrorCode>0</ErrorCode>
<ServiceStandbyMode>false</ServiceStandbyMode>
<AccountStatus>R</AccountStatus>
<ServiceURLs>
<Name>ContentPrefixURL</Name>
<URI>http://ccs.cdn.c.shop.nintendowifi.net/ccs/download</URI>
</ServiceURLs>
<ServiceURLs>
<Name>UncachedContentPrefixURL</Name>
<URI>https://ccs.c.shop.nintendowifi.net/ccs/download</URI>
</ServiceURLs>
<ServiceURLs>
<Name>SystemContentPrefixURL</Name>
<URI>http://nus.cdn.c.shop.nintendowifi.net/ccs/download</URI>
</ServiceURLs>
<ServiceURLs>
<Name>SystemUncachedContentPrefixURL</Name>
<URI>https://ccs.c.shop.nintendowifi.net/ccs/download</URI>
</ServiceURLs>
<ServiceURLs>
<Name>EcsURL</Name>
<URI>https://ecs.c.shop.nintendowifi.net/ecs/services/ECommerceSOAP</URI>
</ServiceURLs>
<ServiceURLs>
<Name>IasURL</Name>
<URI>https://ias.c.shop.nintendowifi.net/ias/services/IdentityAuthenticationSOAP</URI>
</ServiceURLs>
<ServiceURLs>
<Name>CasURL</Name>
<URI>https://cas.c.shop.nintendowifi.net/cas/services/CatalogingSOAP</URI>
</ServiceURLs>
<ServiceURLs>
<Name>NusURL</Name>
<URI>https://nus.c.shop.nintendowifi.net/nus/services/NetUpdateSOAP</URI>
</ServiceURLs>
</GetAccountStatusResponse>
</soapenv:Body>
</soapenv:Envelope>
```

Now, NIM sends the full list of title versions on the system as the next request to the SOAP server defined in NusURL.

```xml
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:nus="urn:nus.wsapi.broadon.com">
<SOAP-ENV:Body>
<nus:GetSystemUpdate xsi:type="nus:GetSystemUpdateRequestType">
<nus:Version>1.0</nus:Version>
<nus:MessageId>EC-xxx-147358457</nus:MessageId>
<nus:DeviceId>xxx</nus:DeviceId>
<nus:RegionId>JPN</nus:RegionId>
<nus:CountryCode>JP</nus:CountryCode>
<nus:Language>ja</nus:Language>
<nus:SerialNo>zzz</nus:SerialNo>
<nus:TitleVersion>
<nus:TitleId>1126106602178562</nus:TitleId>
<nus:Version>10</nus:Version>
</nus:TitleVersion>
...
<nus:TitleVersion>
<nus:TitleId>1126106065308162</nus:TitleId>
<nus:Version>7168</nus:Version>
</nus:TitleVersion>
</nus:GetSystemUpdate>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

The server responds with the versions and metadata of all the titles corresponding to the device type and region.

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<soapenv:Body>
<GetSystemUpdateResponse xmlns="urn:nus.wsapi.broadon.com">
<Version>1.0</Version>
<DeviceId>xxx</DeviceId>
<MessageId>1</MessageId>
<TimeStamp>1414627502761</TimeStamp>
<ErrorCode>0</ErrorCode>
<ContentPrefixURL>http://nus.cdn.c.shop.nintendowifi.net/ccs/download</ContentPrefixURL>
<UncachedContentPrefixURL>https://ccs.c.shop.nintendowifi.net/ccs/download</UncachedContentPrefixURL>
<TitleVersion>
<TitleId>0004001000021000</TitleId>
<Version>8203</Version>
<FsSize>4931584</FsSize>
<TicketSize>848</TicketSize>
<TMDSize>4708</TMDSize>
</TitleVersion>
...
<TitleVersion>
<TitleId>0004013820000202</TitleId>
<Version>4816</Version>
<FsSize>1032192</FsSize>
<TicketSize>848</TicketSize>
<TMDSize>4660</TMDSize>
</TitleVersion>
<UploadAuditData>1</UploadAuditData>
<TitleHash>7E745F7B67D553BEA847859404790C93</TitleHash>
</GetSystemUpdateResponse>
</soapenv:Body>
</soapenv:Envelope>
```

If any titles are new to the system, NIM will request to get the [common ticket](http://3dbrew.org/wiki/CommonETicket) for those titles.

```xml
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:SOAP-ENC="http://schemas.xmlsoap.org/soap/encoding/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:nus="urn:nus.wsapi.broadon.com">
<SOAP-ENV:Body>
<nus:GetSystemCommonETicket xsi:type="nus:GetSystemCommonETicketRequestType">
<nus:Version>1.0</nus:Version>
<nus:MessageId>EC-xxx-170576756</nus:MessageId>
<nus:DeviceId>xxx</nus:DeviceId>
<nus:RegionId>JPN</nus:RegionId>
<nus:CountryCode>JP</nus:CountryCode>
<nus:Language>ja</nus:Language>
<nus:SerialNo>zzz</nus:SerialNo>
<nus:TitleId>0004001000021000</nus:TitleId>
...
<nus:TitleId>000400DB20016302</nus:TitleId>
</nus:GetSystemCommonETicket>
</SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

The server returns base64 encoded tickets for each title. It also returns a certificate chain for the tickets. As an aside, common tickets are used to sign firmware components. Regular games use tickets tied to a specific account (not "common").

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
<soapenv:Body>
<GetSystemCommonETicketResponse xmlns="urn:nus.wsapi.broadon.com">
<Version>1.0</Version>
<DeviceId>xxx</DeviceId>
<MessageId>EC-xxx-244400570</MessageId>
<TimeStamp>1427142110949</TimeStamp>
<ErrorCode>0</ErrorCode>
<CommonETicket>...</CommonETicket>
...
<CommonETicket>...</CommonETicket>
<Certs>...</Certs>
<Certs>...</Certs>
</GetSystemCommonETicketResponse>
</soapenv:Body>
</soapenv:Envelope>
```

Now, the system is ready to download the updated titles. For each title in the GetSystemUpdateResponse, if the version is higher than the current installed version, NIM first gets the title metadata from the ContentPrefixURL. For example, downloading version 8203 of title 0004001000021000 will be from: http://ccs.cdn.c.shop.nintendowifi.net/ccs/download/0004001000021000/tmd.8203?deviceId=xxx&accountId=zzz

It then parses the title metadata, which contains a list of content archives to download. For the example above, it will download http://nus.cdn.c.shop.nintendowifi.net/ccs/download/0004001000021000/00000043 and http://nus.cdn.c.shop.nintendowifi.net/ccs/download/0004001000021000/00000045

Once all the titles are downloaded, it makes another GetSystemTitleHash request (presumably to check if there's an update released while the device was being updated).


## More Information


For more information on downloading titles from Nintendo CDN, check out Rely's [CDN downloader script](https://github.com/Relys/3DS_Multi_Decryptor/blob/master/to3DS/CDNto3DS/CDNto3DS.py). For more information on talking with Nintendo's SOAP servers, check out yellows8's [ninupdates](https://github.com/yellows8/ninupdates) (you need the client certificate and key from the SSL module as described above) or his [update reports](http://yls8.mtheall.com). These tools were indispensable for figuring out the updater.


## Appendix: Updating N3DS 8.1.0-0J to 9.2.0-20J


The motivation behind figuring out how the update process worked was so I could manually update my Japanese N3DS from the stock 8.1 (which does not support [ninjhax](http://smealum.net/ninjhax/)) to 9.2 (the last version that supports ninjhax). I'll quickly describe how it is done, but since the process is a bit involved, I would not recommend anyone not experienced to try it (you can easily brick/update to the latest version). I only attempted this on a Japan N3DS on 8.1.0-0J to 9.2.0-20J, but it should work with other configurations provided you get the right files and offsets.


### Prerequisites





	
  * Cubic Ninja

	
  * [NTR CFW 2.0](http://gbatemp.net/threads/release-ntr-cfw-2-0-for-new-3ds.384640/) and [NTR Debugger](http://gbatemp.net/threads/release-ntr-debugger-2-with-source-the-first-public-real-time-debugger-for-n3ds.384858/)

	
  * A web server with support for some kind of scripting language (PHP for example)

	
  * Clear any pending update by entering [recovery mode](http://3dbrew.org/wiki/Recovery_Mode) and exiting (I don't think this is needed but better safe than sorry)




### Steps





	
  1. Host the SOAP response for the version you want to update to on your web server. You can find all the raw SOAP responses from yellows8's [update report](http://yls8.mtheall.com/ninupdates/reports.php) site. For example, [here](http://yls8.mtheall.com/ninupdates/titlelist.php?date=10-10-14_12-05-04&sys=ktr&reg=J&soapreply=1) is the one for 9.1.0-20J. According to yellows8, there was a bug and his bot did not capture 9.2.0-20J. However, since there was only two titles changed in that update, I just manually crafted a [9.2.0-20J](https://gist.github.com/yifanlu/bdcd22073cb5096942bb) response.

	
  2. Host the SOAP response for the update title hash. [Here](https://gist.github.com/yifanlu/3a6fab2ea14a2b2f081f) is the template. You need to change the value of the TitleHash to match the TitleHash at the end of your update response from step 1.

	
  3. Create a script that responds with one of the two SOAP responses above depending if the request header is for "urn:nus.wsapi.broadon.com/GetSystemUpdate" or "SOAPAction: urn:nus.wsapi.broadon.com/GetSystemTitleHash". I made a two lined PHP script called "update.php" that does this.

	
  4. Host the SOAP response for getting the server URLs. The template for this is [here](https://gist.github.com/yifanlu/e0787ca48473cdf992f8). You only need to change the value of NusURL to point to your NUS responder script created in step 3. (In my case, it would be http://myhost.com/update.php)

	
  5. Boot your 3DS into NTR CFW 2.0 and connect the debugger

	
  6. Use listprocess() to get the PID for "nim". On 8.1.0-0J, it should be 0x25.

	
  7. Patch NIM to use your server for NetUpdateSOAP (this offset is for 8.1.0-0J): write(0x15E424, tuple(map(ord, "http://myhost.com/update.php\0")), pid=0x25)

	
  8. Patch NIM to use your server for ECommerceSOAP. Since you're only responding to GetAccountStatus, it is okay to hard code this: write(0x15E0EC, tuple(map(ord, "http://myhost.com/GetAccountStatus_response.xml\0")), pid=0x25)

	
  9. Do the same for another reference to ECommerceSOAP: write(0x15E463, tuple(map(ord, "http://myhost.com/GetAccountStatus_response.xml\0")), pid=0x25)

	
  10. Go into system settings, and perform an update (do NOT exit system settings as you will lose your patches and will have to perform them again after restarting).

	
  11. Once the update is done, you will be prompted to restart, however because you are in NTR mode, the screen will just go black. You need to hold the power button and manually restart.


