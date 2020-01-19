---
layout: post
title: PrivilegeEsc-Linux
date: 2018-05-20
image: '/images/posts/linuxprivesc.jpg'
categories: ["tools"]
tags: ["tools"]
---

PrivilegeEsc-Linux , The Linux Enumeration Script
<!--more-->
This is a quick post about a script called "PrivilegeEsc-Linux"
This simple script checks the security on a Linux machine and will enumerate everything it can about the machine.
It will enumerate details such as the OS version, environment, and the apps/services. This information will give you a look into possible attack vectors and how to gain root.
<br>
I find this tool especially useful on Linux CTF boxes.
<br>
<h1>Installing PrivilegeEsc-Linux</h1>
<br>
You can download the script by running the below **git clone** command:
```
git clone https://github.com/J4c3kRz3znik/PrivilegeEsc-Linux.git
```

<h1>Running PrivilegeEsc-Linux</h1>
Type the below commands to run the script:

`cd PrivilegeEsc-Linux/`

`chmod +x Priv_enum.sh`

`./Priv_enum.sh`

<br>
<img src="/images/posts/linuxprivescrun.jpg">

<br>

PrivilegeEsc-Linux gives you multiple choices to what you would like to enumerate. Typing **All** will enumerate everything on the victim's machine. After the scan is complete, it will print all the information it has gathered on the victim's machine. This tool does not suggest exploits, however, you may see out of date software or user/group misconfigurations.

<br>
**This tool also presumes you have a foothold in the system and have sufficient privileges to run scripts.**

<br>
Thanks for reading!
