---
layout: post
title:  "Intercepting Android HTTPS Traffic when a Certificate is Pinned"
author: pasan
categories: [ Android, Wireshark, BurpSuite, Packet Sniffing, tutorial ]
tags: [Android, Wireshark, BurpSuite, PacketSniffing]
image: assets/images/20200101/nailed.jpg
imagecredit: <a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@junio_006?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from June Wong"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-2px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M10 9V0h12v9H10zm12 5h10v18H0V14h10v9h12v-9z"></path></svg></span><span style="display:inline-block;padding:2px 3px">June Wong</span></a>
description: "Decoding Android HTTPS traffic when the application is using a pinned certificate"
featured: false
hidden: false
---

As discussed in [previous blog]({% post_url 2019-12-19-Intercepting-Android-HTTPS-Traffic %}), decrypting HTTPS traffic in certain applications can be done by adding a root certificate to the mobile device or the emulated device. But some applications may not use system CAs at all and will stick to custom CAs added to the application by the developer. With such applications, it will not be possible to decrypt HTTPS traffic simply by adding a root certificate. Since the pinned certificate does not match with the certificate received through proxy, it will prevent making any requests via HTTPS and will not function properly.

<p align="center">
  <img src="/assets/images/20200101/unsec_warn1.png" width="40%">
  <br><br>
  <img src="/assets/images/20200101/unsec_warn2.png" width="40%">
</p>

In this experiment, I could successfully decrypt HTTPS traffic from the application by replacing the embedded (pinned) certificate with Burp Certificate.

### Here is what I did:

#### 1. Extract the APK file

Using ```apktool```, the APK file could be extracted

```shell
./apktool d <path-to-apk-file.apk>
I: Using Apktool 2.4.1 on com.xxxxxxxxxxxxx.xxxx.xxx.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /home/xxxx/.local/share/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
W: Cant find 9patch chunk in file: "drawable/xxxxxxx.png". Renaming it to *.png.
W: Cant find 9patch chunk in file: "drawable/xxxxxxx.png". Renaming it to *.png.
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
```

#### 2. Replace the certificate file

Luckily, in my test app, the certificate file was available in 'assets' directory and replacing that was straightforward. I just removed the original ```.cer``` file with Burp Certificate.

<p align="center">
  <img src="/assets/images/20200101/filesInAssetsDir.png">
</p>

#### 3. Repack and sign the APK

Using ```apktool```, repacked the APK file. Using ```jarsigner```, signed it.

```shell
./apktool b <path-to-extracted-apk-directory>
I: Using Apktool 2.4.1
I: Checking whether sources has changed...
I: Checking whether resources has changed...
I: Building apk file...
I: Copying unknown files/dir...
I: Built apk...
```

New APK file is created inside ```dist``` directory. Then we have to sign it using ```jarsigner```. To do that, it is necessary to have a key. Since I do not have a key, I created a self-signed key.

To create a keystore together with a self-signed key valid for 10,000 days, following command can be used:

```shell
keytool -genkey -v -keystore <keystore-name> -storepass <keystore-password> -alias <alias-name> -keypass <key-password> -keyalg RSA -keysize 2048 -validity 10000
```

When above command is executed, ```keytool``` prompts to enter user name, organizational unit, organization, locality, state, and country code. Fill them appropriately. You can create the key with default values as well.

```shell
What is your first and last name?
  [Unknown]:  xxxx xxxxxx
What is the name of your organizational unit?
  [Unknown]:  xxxx
What is the name of your organization?
  [Unknown]:  xxxx
What is the name of your City or Locality?
  [Unknown]:  xxxx
What is the name of your State or Province?
  [Unknown]: xxxxx
What is the two-letter country code for this unit?
  [Unknown]: xx 
Is CN=xxxx xxxxxx, OU=xxxx, O=xxxx, L=xxxx, ST=xxxxx, C=xx correct?
  [no]:  yes

```

And then signed the APK using that key:

```shell
jarsigner -keystore <keystore-name> -storepass <keystore-password> -keypass <key-password> <path-to-unsigned-apk-file> <alias-name>
```

#### Install the APK in device

Once the signed APK is created, install it in device.

```shell
./adb install <path-to-signed-apk.apk>
```

Then the application started communicating properly without SSL errors and proxy could decrypt them.


### References:

1. https://blog.ropnop.com/configuring-burp-suite-with-android-nougat/
2. https://docs.oracle.com/cd/E19798-01/821-1841/gjrgy/
3. https://ibotpeaches.github.io/Apktool/
