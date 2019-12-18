---
layout: post
title:  "Intercepting Android HTTP Requests"
author: pasan
categories: [ Android, Wireshark, BurpSuite, Packet Sniffing, tutorial ]
tags: [Android, Wireshark, BurpSuite, PacketSniffing]
image: assets/images/20191218/purplephone.jpeg
description: "Step by step guide on packet sniffing Android devices using Wireshark or Burp Proxy."
featured: false
hidden: false
---

For newbies of mobile application development and testing, it is a little difficult task to see HTTP requests going out and coming in from/to the device. Specially, when the application is not something under your control, and if still you want to see HTTP requests, you will have to use a proxy to see the traffic. Here, you can find a step by step guide on intercepting HTTP traffic for **Android mobile devices** and **emulated devices** with host machine as **Linux**. However, steps to follow on a Windows system will not be much different.

#### Tools Needed:

There are multiple tools to sniff packets. Among them, you can use one of the below, which will be discussed here:

- Wireshark OR
- Burp Suite - Community edition is sufficient

The difference is, in Wireshark, you can see traffic in both directions. In Burp Suite, you can see, as well as you can change the request content so that outgoing request is altered from the original at a middle layer of the network path.

#### Prerequisites:

- Device must be able to connect to the internet through your machine's WiFi network

This can be achieved by creating a WiFi hotspot in your machine (host machine). Your machine connects to internet using an Ethernet cable (this can again be a WiFi network). That connectivity is shared using a WiFi hostspot. Steps to follow on creating a WiFi hotspot varies drastically depending on the host device's hardware, OS, etc. and will not be covered here. Then choose your Android device to connect to that WiFi hotspot from WiFi settings.

Once everything is connected, you should be able to see two (or more) network interfaces in your host machine. One is to connect the host machine with internet, and the other to allow other devices to connect with that host machine.

***

#### How To: Using Wireshark

You need to install Wireshark in order to see packets. You can use a suitable source from list available at the bottom of this page according to your host operating system. If you are on a Windows machine, you can use Windows installer.

Once Wireshark is installed, you can start it and you should be able to see the network interfaces appearing under 'Capture' section in Welcome page.

Since your WiFi hotspot is the interface you are using for your Android device to connect with the internet, choose that interface to capture packets. (In my case, it is 'wlo1' interface)

<p align="center">
  <img src="/assets/images/20191218/wireshark.png">
</p>

Now you should be able to see the traffic coming out and going in from your device. You may filter to show only HTTP traffic to see a more cleaner output.

***

#### How To: Using Burp Suite

First you need to download and install Burp Suite. There is a paid version, but features in free community edition are sufficient for most of our regular work, including this. You can download the community edition here.

There are multiple download options. This article will be based on Linux plain JAR file option. It is simple and no set up needed if you already have Java installed. The JAR should work in Windows environment as well.

Run the downloaded JAR file to start Burp Suite:

```shell
$ java -jar burpsuite_community_vx.x.xx.jar
```

You should see the Welcome page of Burp Suite. Choose a 'Temporary Project' and then start Burp.

Once the Burp Suite is started, choose 'Proxy' tab appearing under menu bar.
