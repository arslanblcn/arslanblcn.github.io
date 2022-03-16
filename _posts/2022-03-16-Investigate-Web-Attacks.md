---
title: "LetsDefend Investigate Web Attacks"
date: 2022-03-16 10:21 +0800
categories: [LetsDefend, Soc Analyst]
tags: [SOC, Web Attacks]
author: CEngover
---


## Summary

A few days ago, LetsDefend released brand new challenge named `Investigate Web attack`. 

It obviously clear that there are some web attacks that we're going to investigate. First, download the file and unzip it. After extracting the zip file, it gives us a log file named `access.log` which has about 12 thousand rows. Let's answer the questions one by one.

`Which automated scan tool did attacker use for web reconnansiance?`

In this question, I cuted the fields by spaces, and then grabed User-Agent information by using following command.

```bash
awk -F " " '{print $12,$13,$14}' access.log| sort | uniq -c
	 84 "-"
     61 { _; }
   4816 "Mozilla/4.0 (compatible; MSIE
   7393 "Mozilla/5.00 (Nikto/2.1.6) (Evasions:None)
     29 "Mozilla/5.0 (Macintosh; Intel
    174 "Mozilla/5.0 (X11; Linux

```

Nikto is one of the web application vulnerability scanning tool which is used by attacker in this case. So, the answer is `Nikto/2.1.6`.

`After web reconnansiance activity, which technique did attacker use for directory listing discovery?`

By viewing logs, I saw that the attacker tried directory listing by `directory path traversal` attack. This could be second answer.

`What is the third attack type after directory listing discovery?`

We can see that the attacker tried to apply `SQL Injection `attack after directory listing. 

```bash
192.168.199.2 - - [20/Jun/2021:12:36:33 +0300] "GET /bwapp/postnuke/index.php?module=My_eGallery&do=showpic&pid=-1/**/AND/**/1=2/**/UNION/**/ALL/**/SELECT/**/0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,concat(0x3C7230783E,pn_uname,0x3a,pn_pass,0x3C7230783E),0,0,0/**/FROM/**/md_users/**/WHERE/**/pn_uid=$id/* HTTP/1.1" 404 300 "-" "Mozilla/5.00 (Nikto/2.1.6) (Evasions:None) (Test:000690)"

```

`Is the third attack success?`

The third attack is XSS. I grabbed the all xss attacks into different file. 

```bash
cat xss.log | cut -d " " -f 9 | uniq -c
      1 404
      1 302
      9 404
      1 302
     10 404
      1 403
      6 404
      2 403
      4 404
      1 200
     34 404
      1 403
      3 404
      1 400
     11 404
      1 200
     54 404
      1 400
      3 302
      1 400
      3 302
      8 404
      1 400
     13 404
      1 400
      7 404
      1 403
      1 400
      3 404
      1 400
      5 404
      1 302
      4 403
      3 404
      5 403

```

As seen in the result, there is only one `200 HTTP OK` response. We can say that the attack was successfull.

`Yes`

`What is the name of fourth atttack?`

The fourth attack is `Command Injection`. It can be seen in the below that log sample of this attack type.

```bash
192.168.199.2 - - [20/Jun/2021:12:36:34 +0300] "GET /bwapp/cgi-bin/handler/netsonar;cat /etc/passwd|?data=Download" 400 326 "-" "-"
```

`What is the first payload for 4rd attack?`

The answer can find by log sample that is given in the above.

`Is there any persistency clue for the victim machine in the log file ? If yes, what is the related payload?`

I deep dived into logs and found something suspicous which is long payload that can trigger a XSS vulnerability. 

```bash
192.168.199.2 - - [20/Jun/2021:12:36:42 +0300] "GET /bwapp/phpinfo.php?cx[]=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<script>alert(foo)</script> HTTP/1.1" 200 119572 "-" "Mozilla/5.00 (Nikto/2.1.6) (Evasions:None) (Test:003412)"

```

After some research, I came across a vulnerability that PHP version PHP 4.x/5.0/5.1 affected. The vulnerability is PHPInfo Large Input Cross-Site Scripting that 

```bash
PHP is prone to a cross-site scripting vulnerability. This issue is due to a failure in the application to properly sanitize user-supplied input. 

An attacker may leverage this issue to have arbitrary script code executed in the browser of an unsuspecting user in the context of the affected site. This may help the attacker steal cookie-based authentication credentials and launch other attacks.
```
 
The answer is long payload that given in the above.