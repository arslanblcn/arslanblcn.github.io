---
title: "LetsDefend SOC141 - Phishing URL Detected"
date: 2022-02-25 16:21 +0800
categories: [LetsDefend, Soc Analyst]
tags: [SOC, Phishing]
author: CEngover
---

### Event Details

**EventID:** : 88

**Event Time:** : April 4, 2021, 11:10 p.m.

**Rule:**  SOC141 - Phishing URL Detected

**Level:** Security Analyst

**Source Address** 172.16.17.88

**Source Hostname** MarkPRD

**Destination Address** : 192.64.119.190

**Destination Hostname** : nuangaybantiep.xyz

**Username** : Mark

**Request URL** : http://nuangaybantiep.xyz

**User Agent** : Mozilla - Windows

**Device Action** : Allowed

Let's begin to investigate the case. 

By filtering destination IP address in the Log Management section, there is a `HTTP GET` request to mentioned URL in the above. The URL looks like harmless. However, destination IP address is flagged as `malicious` by Comodo Valkyrie.

More details about the action;

When was it accessed?
`Apr, 04, 2021, 11:10 PM`

What is the source address?
`172.16.17.88`

What is the destination address?
`192.64.119.190`

Which user tried to access?
`MarkPRD`

What is User Agent?
`Chrome - Windows` (It's defined as Mozilla but process list shows that there is a chrome.exe )

Is the request blocked?
`Nope`
