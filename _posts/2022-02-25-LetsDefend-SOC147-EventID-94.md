---
title: "LetsDefend SOC147 - SSH Scan Activity"
date: 2022-02-25 14:21 +0800
categories: [LetsDefend, Soc Analyst]
tags: [SOC, ssh]
author: CEngover
---

### Event Details

**EventID:** : 94

**Event Time:** : June 13, 2021, 4:23 p.m.

**Rule:** : SOC147 - SSH Scan Activity

**Level:** : Security Analyst

**Source Address** : 172.16.20.5

**Source Hostname** : PentestMachine

**File Name**  : nmap

**File Hash** : 3361bf0051cc657ba90b46be53fe5b36

**File Size** : 2.82 MB

**Device Action** : Allowed

Let's take a look the activity and investigate what happened.

After download the file that is given in case details, we can get `nmap` ELF file. It's executable and I thought that I had to find out what happend in background. In Endpoint Security pane, I found the host and checked the network connections. There is an nmap scanning command which is

```bash
nmap -sC -sP 172.16.20.0/24
```

After seeing that, my answer to the first question is `Other`.

The second is that malware quarantined/cleaned?

I invastegated the activity in LogManagement panel and saw that there are several ssh scanning activity that comes from PentestMachine host, so the answer can be `not quarantined`.

By uploading file to Virustotal, we can check that the file is malicious or not. Virustotal flagged the file as `non-malicious`.

That's all.
