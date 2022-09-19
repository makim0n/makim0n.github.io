---
weight: 4
title: "[TamuCTF 2019] - Reading rainbow"
date: 2019-02-28T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["tamuctf", "forensic", "network", "wireshark", "pcap", "cryptography", "det"]
categories: ["Writeups"]

lightgallery: true
---


> Recently, the office put up a private webserver to store important information about the newest research project for the company. This information was to be kept confidential, as it's release could mean a large loss for everyone in the office.
> Just as the research was about to be published, a competing firm published information eerily similar. Too similar...
> Time to take a look through the office network logs to figure out what happened.

- What is the IP address of the private webserver?
- How many hosts made contact with the private webserver that day?

| Filename     | MD5 Hash                         | Download link                                                          |
|--------------|----------------------------------|------------------------------------------------------------------------|
| capture.pcap | e36ff23c6995e3595035982cced6c6a9 | https://mega.nz/#!aihUgK6I!a9Lvt6R1bBKit_bE8oOSQAUCdnl9LX_5egidhK5veRM |

In this task, the challenge deal with a PCAP file, I let you check my article about [PCAP analysis](/articles/wiresharkhowtobasic/).

The first flag is to find the internal IP address of a web server. Since PCAP is quite large, I just have to load it into `Capanalysis` and filter on the __SSL__ and __HTTP__ protocols, then filter on the IP that receives the most data:


![](/lib/images/writeups/2019_tamuctf/readingrainbow/0_netenum_chall11.png)


It was the web server we were looking for:

> Flag 1: 192.168.11.4

Now the second challenge is to find the number of IP addresses that have connected to this webserver. Since we know his IP address, with `tshark` it's pretty easy:

```bash
▶ tshark -r capture.pcap -Y "ip.dst == 192.168.11.4" -Tfields -e 'ip.src' | sort | uniq
128.194.165.200
172.217.6.138
172.226.209.130
192.168.1.1
192.168.11.5
192.168.11.7
192.168.11.8
192.168.11.9
35.222.85.5
35.224.99.156
52.43.40.243
54.213.168.194
91.189.92.38

▶ tshark -r capture.pcap -Y "ip.dst == 192.168.11.4" -Tfields -e 'ip.src' | sort | uniq | wc -l
13
```

Fortunately, all connections are done on the same day::

> Flag 2: 13

## 1_Discovery

- What is the IP address of the host exfiltrating data?
- For how long did the exfiltration happen? (Round to the nearest second. Format: MM:SS)
- What protocol/s was used to exfiltrate data? (Alphabetical order, all caps, comma separated, with spaces - ex: ABCD, BBCD)

It's time to open this big PCAP file in Wireshark. Thanks to the first question, we know that the attack exfiltrates some data. The best filter for that is still: `data.data`:


![](/lib/images/writeups/2019_tamuctf/readingrainbow/1_disco_chall11.png)


We can see that our web server (192.168.11.4) is chatting with another host (192.168.11.7) via weird ICMP requests. Let's see what these requests contain:


![](/lib/images/writeups/2019_tamuctf/readingrainbow/1_disco_chall12.png)


If we get the first request with a `tshark`:

```bash
▶ tshark -r capture.pcap -Y "data.data && ip.dst == 192.168.11.4" -Tfields -e 'data.text' | head -n 1 | xxd -r -p
SEx4IRV.746f74616c6c795f6e6f7468696e672e706466.REGISTER.6156eab6691f32b8350c45b3fc4aadc1               
```

The data formatted like this looks a lot like the [DET (Data Exfiltration Toolkit)](https://github.com/sensepost/DET) framework. I had already talked about it in a writeup at the [SantHackLause 2018](/walkthrough/santhacklaus2018/#mission-impossible-2).

> Flag 1: 192.168.11.7

Now we have to determine the duration of the exfiltration. With a small filter on IPs, we see some interesting things in the `DNS`:


![](/lib/images/writeups/2019_tamuctf/readingrainbow/1_disco_chall21.png)


If we extract the last DNS request and this is indeed the last DET request, we should find a "DONE":

```bash
▶ echo -n '534578344952562e35312e444f4e45' | xxd -r -p                          
SEx4IRV.51.DONE
```

Perfect, we know the first request with the "REGISTER" and the last one with the "DONE".


![](/lib/images/writeups/2019_tamuctf/readingrainbow/1_disco_chall23.png)
![](/lib/images/writeups/2019_tamuctf/readingrainbow/1_disco_chall22.png)


```bash
▶ echo $((35.49-24.40))                               
11.090000000000003
```

The exfiltration lasted 11 minutes and 9 seconds:

> Flag 2: 11.09

The last step will be the simplest, we already have all the information, to find the protocols, we will do a little tshark trick:

```bash
▶ tshark -r capture.pcap -Y "ip.src == 192.168.11.4 && ip.dst == 192.168.11.7" -Tfields -e '_ws.col.Protocol' | sort | uniq 
DNS
HTTP
ICMP
TCP
```

The "TCP" protocol is not counted since it is a transport protocol (see OSI Model).

> Flag 3: DNS, HTTP, ICMP

## 2_Exfiltration


- What is the name of the stolen file?
- What is the md5sum of the stolen file?

These questions are a little bit easy since we know that the attacker used DET. First request contains both flags:

```
SEx4IRV.746f74616c6c795f6e6f7468696e672e706466.REGISTER.6156eab6691f32b8350c45b3fc4aadc1
```

* SEx4IRV
* 746f74616c6c795f6e6f7468696e672e706466
* REGISTER
* 6156eab6691f32b8350c45b3fc4aadc1

The first hexadecimal sequence is the name of the encoded extracted file:

```bash
▶ echo -n '746f74616c6c795f6e6f7468696e672e706466' | xxd -r -p                              
totally_nothing.pdf
```

> Flag 1: totally_nothing.pdf

The second hexadecimal sequence is the MD5 hash of the encoded extracted file:

> Flag 2: 6156eab6691f32b8350c45b3fc4aadc1

## 3_Data

- What compression encoding was used for the data?
- What is the name and type of the decompressed file? (Format: NAME.TYPE e.g. tamuctf.txt)

Before answering the questions, it will be necessary to find a way to recover the original file. We know that the attacker used HTTP, DNS and ICMP protocols between IP 192.168.11.7 (attacker) and 192.168.11.4 (webserver) to extract his file. We will, therefore, use the following filter and save only the displayed packets to make a lightened PCAP file:

```bash
ip.src == 192.168.11.4 && ip.dst == 192.168.11.7 && (http || dns || icmp
```

![](/lib/images/writeups/2019_tamuctf/readingrainbow/3_exfil_chall11.png)


We go from 15k packets to 117, it's still more pleasant to analyze.

### Get data from ICMP

ICMP data is sent in hexadecimal when decoding on the fly with `xxd` piped to `tshark` there is no more line break and the data becomes difficult to analyze. For that, there is `sed` which will add more returns to the line:

```bash
▶ tshark -r exfil.pcap -Y "icmp" -Tfields -e "data.text" | xxd -r -p | sed 's/SEx4IRV/\nSEx4IRV/g'
SEx4IRV.746f74616c6c795f6e6f7468696e672e706466.REGISTER.6156eab6691f32b8350c45b3fc4aadc1
SEx4IRV.2.85a846255178c4cbbd77ee999d7b7736892afaa392cf6ae7ccf9ee39f79efb9c3367325a767c1c7db414c0d4dadc4c78b0b5
SEx4IRV.12.2bb53aaf40c5354868c984db4df8b209379f172b26dcbc5f6e99f04a130ef3e234f944e875a64f746d26fc920977987079ee
[...]

▶ tshark -r exfil.pcap -Y "icmp" -Tfields -e "data.text" | xxd -r -p | sed 's/SEx4IRV/\nSEx4IRV/g' > clear_icmp
```

These are the ICMP data extracted from a file.

### Get data from HTTP

HTTP data are sent as POST data:


![](/lib/images/writeups/2019_tamuctf/readingrainbow/3_exfil_chall12.png)


Same process as for ICMP:

```bash
▶ tshark -r exfil.pcap -Y "http" -Tfields -e urlencoded-form.value | xxd -r -p | sed 's/SEx4IRV/\nSEx4IRV/g'               
SEx4IRV.0.1f8b080094e16c5c0003ed596b6c1c5715beb30f7b9dd8eb4dea249b07cdb64d8493ca9b5d3b7ea4698877fd1a83ed98d40e
SEx4IRV.1.01ea4cd7deb1bdb00f6b77b6d8018125a755b7a94390a0ca1f50a5a20a103f5ca844040236b82a25bf1250451005b9555339
SEx4IRV.3.3960e6981a034d81000a36ed6f6a0ab6049a5b5a5120d818686c41bec047ec17a56c468ba47d3ea44512d931adfc50dcadfc

▶ tshark -r exfil.pcap -Y "http" -Tfields -e urlencoded-form.value | xxd -r -p | sed 's/SEx4IRV/\nSEx4IRV/g' > clear_http
```

### Get data from DNS

The functioning of the DNS is a bit more different, because each request will be encoded in another.

```bash
▶ tshark -r exfil.pcap -Y "dns" -Tfields -e dns.qry.name | cut -d'.' -f2 | xxd -r -p | sed 's/SEx4IRV/\nSEx4IRV/g'
SEx4IRV.5.a6cb22df81782e99b8f30efd5976f11c219f61477c5da8d1d1851a1fc79f60ed4ed9783b1bb3cb33bb3cd307bec21c5b11fa
SEx4IRV.7.d868cbfe7df168433c96cc4e374cb7b534b4ecf76752fe46ea9387e9f60c0c537d32df30b4a4bc8e61a4fcedfa9fff74f35f
SEx4IRV.8.af6baf4c5fb9f1cbf7479e0bfce2f22b44d7858af346e66396d9137c6238025db0a4cf2098419e36e0ff460bbca10cee2d83

▶ tshark -r exfil.pcap -Y "dns" -Tfields -e dns.qry.name | cut -d'.' -f2 | xxd -r -p | sed 's/SEx4IRV/\nSEx4IRV/g' > clear_dns
```

### Ordering each request

All requests must be put back in the right order. This is how DET works:

1. File ID
2. Packet number
3. Data

This query construction is not valid for the first (REGISTER) and last (DONE) requests.
To put all this in order, python will do it for us. I will put all the lines in a dictionary with the packet number as the key:

```python
f = open('clear_data')
a = f.read()
f.close()

final = ""
tmp = {}

for i in range(0,len(a)):
	tmp[int(a[i].split('.')[1])] = a[i].split('.')[2]

for j in range(0,len(tmp)):
	final = tmp[j]

g = open('result','wb')
g.write(final)
g.close()
```

Here is what we obtain:

```bash
▶ file result 
result: ASCII text, with very long lines, with no line terminators

▶ cat result                    
1f8b080094e16c5c0003ed596b6c1c5715beb30f7b9dd8eb4dea249b07cdb64d8493ca9b5d3b7ea4698877fd1a83ed98d40e01ea4cd7deb1bdb00f6b7
[...]

▶ cat result | xxd -r -p > test 

▶ file test    
test: gzip compressed data, last modified: Wed Feb 20 05:11:48 2019, from Unix, original size 10240
```

Remember the first question of the challenge is the type of compression used:

> Flag 1: gzip

Now, let's uncompress the archive and got the original file:

```bash
▶ mv test test.gz                                                             
                                                                        
▶ gzip -d test.gz            
                                                                        
▶ file test 
test: POSIX tar archive (GNU)
                                                                          
▶ tar xvf test    
stuff                                                                            
▶ file stuff 
stuff: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=e228bab316deed74b478d8f5bdef5d8c30bbd1b4, not stripped
```

And now, let's validate the last flag: 

> Flag 2: stuff.elf
