---
layout: post
title: HTB - Nibbles Writeup
date: 2019-02-07
image: '/images/posts/htb/nibbles/nibbles.jpg'
categories: ["writeups"]
tags: ["writeups", "nibbles", "htb"]
---

* Box: Nibbles
  - Difficulty: Easy
  - Points: 20
  - Release: 13 Jan 2018
  - IP: 10.10.10.75

### Initial Enumeration

#### 1.Nmap Scanning

Starting with a scan of the target ip address:

`nmap -sC -sV -oA nibbles.nmap 10.10.10.75`

We can see 22 and 80 are open. Let's navigate to the web browser and access the webpage on port 80.

<img src="/images/posts/htb/nibbles/nibbles1.jpg">

#### Directory Enumeration

Looking at the source code of the index.html, we can see a command referencing a directory named "**/nibbleblog/**".

<img src="/images/posts/htb/nibbles/nibbles2.jpg">

So let's run gobuster in that directory to enumerate further.
The following command will do the trick:

`gobuster dir -u http://10.10.10.75/nibbleblog -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`

<img src="/images/posts/htb/nibbles/nibbles3.jpg">

Note the **/admin.php** file. Accessing it, it appears to be an admin login page. Looking up default nibbleblog credentials and trying them manually actually works. The username admin:nibbles worked for me.

<img src="/images/posts/htb/nibbles/nibbles4.jpg">

Let's enumerate some more and look into possible exploits available for nibbleblog. There is a file named **README** that our gobuster found.

<img src="/images/posts/htb/nibbles/nibbles5.jpg">

Visiting that shows us the current nibble version is 4.0.3.

Next, we run `searchsploit` to look for any nibbleblog exploits. That turns up a shell upload vulnerability.
Meaning nibbleblog's upload feature doesn't actually check the file extension of image uploads. We can exploit this by uploading a reverse php shell using the upload plugin instead of an image.

<img src="/images/posts/htb/nibbles/nibbles6.jpg">

### Exploitation

#### Payload Generation

I'm going to use the default php-reverse-shell.php script that comes with Kali and edit the **$ip** and **$port**.

Let's also navigate to the image upload plugin and have a look at where we'll be uploading this script.

<img src="/images/posts/htb/nibbles/nibbles7.jpg">

#### Uploading Payload

I set up a netcat listener and upload the malicious file.

From there, I will want to browse to the specific file at:

**http://10.10.10.75/nibbleblog/content/private/plugins/image.php**

<img src="/images/posts/htb/nibbles/nibbles8.jpg">

And now we have a reverse shell to the target machine.

To make the reverse connection fully interactive you follow the below steps:

[obtaining-a-fully-interactive-shell](https://forum.hackthebox.eu/discussion/142/obtaining-a-fully-interactive-shell)

We can then see we're under the nibbler account and the user flag is under `/home/nibbler/user.txt`

<img src="/images/posts/htb/nibbles/nibbles9.jpg">

<img src="/images/posts/htb/nibbles/nibbles10.jpg">

### Privilege Escalation

Running **sudo -l** reveals an entry for **/home/nibbler/personal/stuff/monitor.sh**. However, this file does not exist so it should be possible for us to create a bash script and run it as root.

<img src="/images/posts/htb/nibbles/nibbles11.jpg">

The script looks something like this. Super simple.

<img src="/images/posts/htb/nibbles/nibbles12.jpg">

Change it into an executable and run it with sudo.

And we have a root terminal from here we can simply find the root.txt file and finish the box.

<img src="/images/posts/htb/nibbles/nibbles13.jpg">

root.txt can be found in the **/root** directory.

<img src="/images/posts/htb/nibbles/nibbles14.jpg">
