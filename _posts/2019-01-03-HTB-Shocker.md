---
layout: post
title: HTB - Shocker Writeup
date: 2019-01-03
image: '/images/posts/htb/shocker/shocker.jpg'
categories: ["writeups"]
tags: ["writeups", "shocker", "htb"]
---

* Box: Shocker
  - Difficulty: Easy
  - Points: 20
  - Release: 30 Sep 2017
  - IP: 10.10.10.56

###Initial Enumeration

####Nmap Scanning

Starting with a scan of the target ip address:

`nmap -sC -sV -oA shocker.nmap 10.10.10.56`

We see 2 services are open. Apache/2.4.18 and Openssh.
OpenSSH is on a non-standard port but that does not come into affect during exploitation.

 This is also a glimpse at the "Security by Obscurity" model some administrators implement and how obscuring open port numbers doesn't generally deter an attacker.

<img src="/images/posts/htb/shocker/shocker1.jpg">

####Dirbuster enumeration

Running Dirbuster with the lowercase medium directory list shows us some promising results.

Based on the name of the box and the directory names we are seeing, it is safe to presume this box is designed to outline the **ShellShock** exploit.

More information regarding this exploit can be found [here:](https://blog.cloudflare.com/inside-shellshock/)

With that in mind, let's focus our attention on the **/cgi-bin/** directory.

We see a file under **/cgi-bin/** named **"user.sh"**. This confirms that we are likely going to be exploiting ShellShock.

<img src="/images/posts/htb/shocker/shocker2.jpg">

###Exploitation

Once we know which exploit to start with, lets fire up Metasploit and use the built-in Metasploit module for this vulnerability.

Module: `exploit/multi/http/apache_mod_cgi_bash_env_exec`

We set the **RHOST** and **TARGETURI** accordingly.
The **TARGETURI** will be **/cgi-bin/user.sh**.

After running that, we have a pretty basic user shell.

We can simply run the following to get a working root shell:

`sudo /usr/bin/perl -e 'exec "/bin/sh"'`

<img src="/images/posts/htb/shocker/shocker3.jpg">

And we can find the user flag in **/home/shelly/user.txt**
<img src="/images/posts/htb/shocker/shocker4.jpg">
