---
title: "LetsDefend SOC168 - Whoami Command Detected in Request Body"
date: 2022-03-16 11:21 +0800
categories: [LetsDefend, Soc Analyst]
tags: [SOC, Web Attacks]
author: CEngover
---

## Event Details

**EventID:** 118

**Event Time:** Feb. 28, 2022, 4:12 a.m.

**Rule:** SOC168 - Whoami Command Detected in Request Body

**Level:** Security Analyst

**Hostname** WebServer1004

**Destination IP Address** 172.16.17.16

**Source IP Address** 61.177.172.87

**HTTP Request Method** POST

**Requested URL** https://172.16.17.16/video/

**User-Agent** Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)

**Alert Trigger Reason** Request Body Contains whoami string

**Device Action** Allowed

### Investigating the Case

So, once we take ownership of the case, we can start the investigating phase. After I took ownership, I looked at Log management section to find out that what kind of requests attacker made. Here is the sample of the attacker's request.

```bash
**Request URL:** https://172.16.17.16/video/

**User-Agent:** Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1)

**Request Method:** POST

**Device Action:** Permitted

**HTTP Response Size::** 1501

**HTTP Response Status:** 200

**POST Parameters:** ?c=cat /etc/shadow
```

It's clear that the attacker made a successful code execution attack. 

`Q1 - Is Traffic Malicious?`
`A1 - Malicious`

`Q2 - What Is The Attack Type?`
`A2 - Command Injection`

`Q3 - What Is the Direction of Traffic?`
`Internet -> Local Network`

`Q4 - Was the Attack Successful?`
`Yes`

`Do You Need Tier 2 Escalation?`
`Yes`

Since, the attacker was success to execute system commands on the server there could be a web-shell or any tool that allows to execute code on system. So, Tier 2 deep dive into the case.
