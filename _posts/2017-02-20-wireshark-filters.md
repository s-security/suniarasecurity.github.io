---
layout: post
title: Wireshark Filters
date: 2017-06-01
image: '/images/posts/network-cable2.jpg'
categories: ["sysadmin"]
tags: ["sysadmin"]

---

Below are some WireShark commands that I find useful.
Wireshark is an excellent tool for system administrators and penetration testers to capture traffic on a network.

<!--more-->

**Is your traffic efficient?**
Use the following filter to identify problems in your traffic:

```
tcp.analysis.flags
```

This flag helps to look at problems you may have in a trace file.  By using this filter, you can see re-transmissions, acknowledgement problems and more.

**Are you seeing latency in traffic to a server or do you believe you're being SYN attacked?**

```
tcp.flags.syn==1
```
This filter will help you identify a SYN attack by recognizing the pattern of traffic that gets outputted.

**Is an application operating? Do you have the port # for the application?**

```
tcp.port==443
```
This will print you the stream of traffic on a specific port.


**This built-in filter will show you packets that have some kind of expert message from Wireshark**

```
tcp.analysis.flags
```
