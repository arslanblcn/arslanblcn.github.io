---
title: "SOC119 - Proxy - Malicious Executable File Detected"
date: 2022-02-25 14:21 +0800
categories: [LetsDefend, Soc Analyst]
tags: [SOC]
author: CEngover
---

### Event Details

**EventID:**
83

**Event Time:**
March 21, 2021, 1:02 p.m.

**Rule:**
SOC119 - Proxy - Malicious Executable File Detected

**Level:**
Security Analyst

**Source Address**
172.16.17.5

**Source Hostname**
SusieHost

**Destination Address**
51.195.68.163

**Destination Hostname**
win-rar.com

**Username**
Susie

**Request URL**
https://www.win-rar.com/postdownload.html?&L=0&Version=32bit

**User Agent**
Chrome - Windows

**Device Action**
Allowed

Here are some notes about the event. 

```bash
**Request URL:** https://www.win-rar.com/postdownload.html?&L=0&Version=32bit

**Request Method:** GET

**Device Action:** Allowed

**Process:** chrome.exe

**Parent Process:** explorer.exe

**Parent Process MD5:** 8b88ebbb05a0e56b7dcc708498c02b3e
```

We've one `GET` request to phishing URL. Request details are;

Date : Mar, 21, 2021, 01:02 PM
Type : Proxy
Source IP : 172.16.17.5
Source Port : 43213
Destination IP : 51.195.68.163
Destination Port : 443

After checking the url if that is malicious or not, Virustotal returns that the IP and domain are non-malicious.

`Q1 -- Not Malicious`

That's it :)

