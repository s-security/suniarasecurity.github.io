---
layout: post
title: Network Reset Commands
date: 2017-02-20
image: '/images/posts/network-cable.jpg'
categories: ["sysadmin"]
tags: ["sysadmin"]
---

Type the following at a command prompt to fully reset the TCP/IP Stack

<!--more-->
If you're having connection issues and you believe the hardware is good use the below to reset the TCP/IP Stack:

```
netsh winsock reset
netsh int ip reset
netsh interface ipv4 reset
netsh interface ipv6 reset
netsh interface tcp reset
netsh int reset all
ipconfig /flushdns
nbtstat -R
nbtstat -RR
netsh interface tcp set global autotuninglevel=disabled
netsh advfirewall reset
```

You can also use PowerShell (as an administrator) and type:

<**Get-NetAdapter | Restart-NetAdapter**>
