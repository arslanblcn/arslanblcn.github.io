---
title: "LetsDefend SOC137 - Malicious File/Script Download Attempt"
date: 2022-02-28 11:21 +0800
categories: [LetsDefend, Soc Analyst]
tags: [SOC]
author: CEngover
---


### Event Details

**EventID:** 76

**Event Time:** March 14, 2021, 7:15 p.m.

**Rule:** SOC137 - Malicious File/Script Download Attempt

**Level:** Security Analyst

**Source Address:** 172.16.17.37

**Source Hostname:** NicolasPRD

**File Name:** INVOICE PACKAGE LINK TO DOWNLOAD.docm

**File Hash:** f2d0c66b801244c059f636d08a474079

**File Size:** 16.66 Kb

**Device Action:** Blocked

Getting into case made me little bit confused but after, I handled it.  First, I investegated the case that what happened in the background. I installed the zip file which is given at the beginning and extracted it. We've a document file which is flagged as malicious by 28 security vendor on Virustotal. I decided to look into endpoint management and saw that there are suspicious powershell commands. Also, the Virustotal result shows us the file behavior, as well. It tries to download a file from remote file server. The encrypted command I saw in the command line history could be this one. So, I answered the first question as `Other` . 

There is a process that is created via powershell and log management says the IP address makes a succesful request to the remote server. The second question is `Not quarantied`.

We can see that there are different ip addresses which communicate with malicious file. Unless, there is no request to them. The 3rd question is `Not Accessed` 
