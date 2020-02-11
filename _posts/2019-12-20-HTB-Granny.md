---
layout: post
title: HTB - Granny Writeup
date: 2019-12-20
image: '/images/posts/htb/granny/granny.jpg'
categories: ["writeups"]
tags: ["writeups", "granny", "htb"]
---

* Box: Granny
  - Difficulty: Easy
  - Points: 20
  - Release: 12 Apr 2017
  - IP: 10.10.10.15


### Initial Enumeration

#### 1. Nmap Scanning

Starting with a scan of the target ip address:

`nmap -sC -sV -oA granny.nmap 10.10.10.15`

<img src="/images/posts/htb/granny/granny1.jpg">

TCP/UDP ports we found nothing just single TCP 80 Port opened.
And it states it’s IIS httpd 6.0

After doing some research we can find an remote code execution vulnerability.

https://www.rapid7.com/db/modules/exploit/windows/iis/iis_webdav_upload_asp


### Exploitation

Let's load up metasploit and try the exploit:

``use exploit/windows/iis/iis_webdav_upload_asp``

We simply need to set the RHOST and the LHOST then run exploit.

<img src="/images/posts/htb/granny/granny2.jpg">

We can see that we instantly got the shell.
<img src="/images/posts/htb/granny/granny3.jpg">
Next the privilege escalation:

##Privilege escalation

The exploit gave us user access as NT Authority.
We're going to need to background our shell and use the msf post/windows/manage/migrate module.

``use post/windows/manage/migrate``

So here we see that this module will spawn a notepad.exe process and migrate our shell to run within that process.


<img src="/images/posts/htb/granny/granny4.jpg">

At this point it is a good idea to migrate to a process running under NT AUTHORITY\NETWORK SERVICE​.
In this case davcdata.exe ​seemed to be the only stable process available.
The intended exploit in this case is ms15_051_client_copy_image​, which immediately grants a root shell.

<img src="/images/posts/htb/granny/granny5.jpg">


After running this module we started to get some suggestions that this machine is vulnerable to this vulnerability.

Testing through them, the most stable seems to be ``exploit/windows/local/ms14_070_tcpip_ioctl``

After that has run, we can check who we are by running ``getuid`` and we can see we're NT AUTHORITY\SYSTEM
```
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

<img src="/images/posts/htb/granny/granny6.jpg">

The root flag is located:
**C:\Documents and Settings\Administrator\Desktop\root.txt**

User flag is:
**C:\Documents and Settings\Lakis\Desktop\user.txt**
