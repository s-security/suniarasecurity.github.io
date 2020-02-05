---
layout: post
title: HTB - Legacy Writeup
date: 2019-01-05
image: '/images/posts/htb/legacy/legacy.jpg'
categories: ["writeups"]
tags: ["writeups", "legacy", "htb"]
---

* Box: Legacy
  - Difficulty: Easy
  - Points: 20
  - Release: 15 Mar 2017
  - IP: 10.10.10.4

### Initial Enumeration

#### 1.Nmap Scanning

Starting with a scan of the target ip address:

`nmap -sC -sV -oA legacy.nmap 10.10.10.4`

Based on the output of the nmap scan we can see that SMB port is open and the operating system is Windows XP.

<img src="/images/posts/htb/legacy/legacy1.jpg">


### Exploitation

#### 2.SMB Exploits

Now that we know the OS is Windows XP and SMB is open, lets do a quick google search.
One of the first exploits we see is named "CVE-2008-4250" and there looks to be a Metasploit module for this. Lets give that a try.

More information on [CVE-2008-4250](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250)

<img src="/images/posts/htb/legacy/legacy2.jpg">

User flag is at **C:\Documents and Settings\john\Desktop\user.txt**

<img src="/images/posts/htb/legacy/legacy3.jpg">

Root flag is at **C:\Documents and Settings\Administrator\Desktop\root.txt**

<img src="/images/posts/htb/legacy/legacy4.jpg">
