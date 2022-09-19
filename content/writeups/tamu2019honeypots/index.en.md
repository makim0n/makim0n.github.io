---
weight: 4
title: "[TamuCTF 2019] - Honeypots"
date: 2019-02-28T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["tamuctf", "forensic", "honeypots", "suricata", "malware"]
categories: ["Writeups"]

lightgallery: true
---

This part of the writeup will go pretty fast, it's just parsing in the end. All challenges are in the following archive:

Password for the archive: tamuctf

| Filename | MD5 Hash | Download link |
|----------|----------|---------------|
| honeypot2.7z | b08992d50e5885f6db8cf50f22eefab4 | https://drive.google.com/uc?id=1lhYsk97AgYDMxzfz1r6FzUs28sugZUR0&export=download |

__Warning this challenge contains some malware samples.__

## Cowrie

- What was the most common src ip (telnet & ssh)?
- What was the most common telnet username?
- What was the most common ssh username?
- What is the url and channel of the IRC server that the one downloaded script tried to connect to? (url, channel)

In order to find the most used IP address for telnet and ssh, I just count, sort and print the first line:

```bash
▶ cat cowrie.json.2018*| jq | grep "src_ip" | sort | uniq -c | sort -nr | head -n 1
  21011   "src_ip": "211.143.198.161",
```

> Flag 1: 211.143.198.161

Some things for flag 2 and 3:

```bash
▶ cat cowrie.json.2018* | jq | grep username | sort | uniq -c | sort -nr | head -n 2
  12998   "username": "root",
   9626   "username": "admin",
```

> Flag 2: root
> Flag 3: admin

Cowrie is saving all binaries / scripts or whatever dropped by the attacker, I'm able to find it in download folder:

```bash
▶ file cowrie/downloads/d3f074230f4b62a4d2a8d50a5df9a51d6fe20a8d3b27c1ff9459cdbc531f489d 
cowrie/downloads/d3f074230f4b62a4d2a8d50a5df9a51d6fe20a8d3b27c1ff9459cdbc531f489d: a /usr/bin/perl script executable (binary data)

▶ cat d3f074230f4b62a4d2a8d50a5df9a51d6fe20a8d3b27c1ff9459cdbc531f489d | sed '/^[[:space:]]*$/d'
```

```perl
[...]
$server = 'irc.quakenet.org' unless $server;
my $port = '6667';
my $linas_max='8';
my $sleep='5';
my $homedir = "/tmp";
my $version = 'Undernet Perl Bot v1.0';
my @admins = ("gov","gov-","fucker-","fucker","op");
my @hostauth = ("fucker.users.quakenet.org","gov.users.quakenet.org","cker.pro");
my @channels = ("#bookz");
[...]
``` 

> Flag 4: irc.quakenet.org, bookz


## Dionaea

- What was the most common src ip?
- What is the common name for the most commonly downloaded malware?

```bash
▶ cat dionaea/log/dionaea.json.* | jq | grep 'src_ip' | sort | uniq -c | sort -nr | head -n 1
    128   "src_ip": "::ffff:193.56.29.24",
```

> Flag 1: 193.56.29.24

Lots of binaries are stored by the honeypot. I just generate md5sum for all of them and check the first one on [VirusTotal](https://virustotal.com):

```bash
▶ md5sum dionaea/binaries/data/dionaea/binaries/*
0ab2aeda90221832167e5127332dd702  dionaea/binaries/data/dionaea/binaries/0ab2aeda90221832167e5127332dd702
1533a4e55cee10a9487e4b13abff4688  dionaea/binaries/data/dionaea/binaries/1533a4e55cee10a9487e4b13abff4688
1a400481251fac98bc574c0aed7beca8  dionaea/binaries/data/dionaea/binaries/1a400481251fac98bc574c0aed7beca8
20b431c101855960614b21e4c1b26451  dionaea/binaries/data/dionaea/binaries/20b431c101855960614b21e4c1b26451
2622e5c9ac05ed71ab35606493627c13  dionaea/binaries/data/dionaea/binaries/2622e5c9ac05ed71ab35606493627c13
2de98404eb4ac4a525ed1884f4ea445b  dionaea/binaries/data/dionaea/binaries/2de98404eb4ac4a525ed1884f4ea445b
[...]
```

![](/lib/images/writeups/2019_tamuctf/honeypot/dionaea/chall21.png)

According to Avira:

> Flag 3: wannacry

## Glastopf

- What was the most common src ip?
- What are the three most commonly requested url besides / get or post? (no slashes, all lowercase, alphabetical (1.ext, a.ext, b.ext))

In this honeypot logs are not stored in JSON format... So I had to parse them with `cut`.

```bash
▶ cat glastopf/log/glastopf.log.* | cut -d" " -f4 | sort | uniq -c | sort -nr | head -n 1
    274 85.121.16.8
```
    
> Flag 1: 85.121.16.8

```bash
▶ cat glastopf/log/glastopf.log.* | cut -d" " -f7 | sort | uniq -c | sort -nr | head -n 4
     96 /
     20 /qq.php
     20 /confg.php
     20 /1.php
```

> Flag 2: 1.php, confg.php, qq.php

## Honeytrap

- What was the most common src ip?
- What was the most common user agent?
- What was the second most common user agent?

As the previous honeypot, logs are not stored as json file, so I `cut` them and doing some `sed` stuff in order to remove all associate ports, I only need IP address:

```bash
▶ cat honeytrap/log/attacker.log| cut -d" " -f5 | sed 's/:.*//' | uniq | sort | uniq -c | sort -nr | head -n 1
      9 5.188.210.12
```

> Flag 1: 5.188.210.12

For two most common user-agent:

```bash
▶ cat honeytrap/attacks/* | grep -a 'User-Agent' | sort | uniq -c | sort -nr | head -n 2
     28 User-Agent: python-requests/2.6.0 CPython/2.6.6 Linux/2.6.32-696.30.1.el6.x86_64
     11 User-Agent: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.77 Safari/537.36
```

> Flag 2: python-requests/2.6.0 CPython/2.6.6 Linux/2.6.32-696.30.1.el6.x86_64
> Flag 3: Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.77 Safari/537.36

## Suricata

- What CVE was alerted for the most?
- What was the most common signature?

JSON logs, great.

```bash
▶ cat suricata/log/suricata_ews.log.* | jq | grep "cve_id" | sort | uniq -c | sort -nr | head -n 1
   1527     "cve_id": "CVE-2006-2369",
```

> Flag 1: CVE-2006-2369

There are __signature__ pattern in both log files (eve.json and suricata_ews.log), then:

```bash
▶ cat suricata/log/* | jq | grep 'signature"' | sort | uniq -c | sort -nr | head -n 1
1426173     "signature": "ET EXPLOIT [PTsecurity] DoublePulsar Backdoor installation communication",
```

> Flag 2: ET EXPLOIT [PTsecurity] DoublePulsar Backdoor installation communication