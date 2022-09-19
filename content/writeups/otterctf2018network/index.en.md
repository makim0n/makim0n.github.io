---
weight: 4
title:  "[OtterCTF] - Network"
date: 2018-12-24T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["otterctf", "forensic", "pcap", "cryptography", "aes", "usb", "smbv2", "wireshark"]
categories: ["Writeups"]

lightgallery: true
---

# Network

There are 4 network challenges during this CTF, I only solved 3 of them, here is how.

## Birdman's Data


![](/lib/images/writeups/2018_otterctf/network/statement_net1.png)


### State of the art

The PCAP file is not really big, I decided to open it using Wireshark. There are few things to notice:

* Data in UDP protocol
* HTTP packets to `txtwizard.net`


![](/lib/images/writeups/2018_otterctf/network/net1_1.png)


There nothing interesting in UDP traffic, data does not represent much.

On the other hand, HTTP traffic is very interesting.

### "Crypto" time

There is an interesting request:

```raw
POST /crypto/AES/encrypt/CBC/PKCS5 HTTP/1.1
Host: www.txtwizard.net
Connection: keep-alive
Content-Length: 4297
Accept: application/json, text/javascript, */*; q=0.01
Origin: http://www.txtwizard.net
```

Oh, AES CBC! We're missing the encrypted data, key and IV... Just scroll a little bit ;)


![](/lib/images/writeups/2018_otterctf/network/net1_2.png)


After URL decoding data, I got the key and IV, at the end we got:

```raw
Key: XfCtxvD1yFZbxQ/+ULhAcA==
IV : sEhrZxQpnNnINixu3KQ1Tg==
Cipher text: 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
```

Quick look on: http://www.txtwizard.net/crypto


![](/lib/images/writeups/2018_otterctf/network/net1_3.png)


Here is the decoded data:

```raw
Chance Something's wrong, I can feel it (Six minutes, Slim Shady, you're on) Just a feeling I've got, like something's about
To happen, but I don't know what If that means, what I think it means, we're in trouble, big trouble, And if he is as bananas as you say, I'm not taking any chances You were just what the doctor ordered I'm beginning to
Feel like a Rap God, Rap God All my people from the front to the back nod, back nod Now who thinks their arms are long
{
Enough to slap box, slap box? They said I rap like a robot, so call
me Rapbot But for me to rap like a computer must be
in my genes I got a laptop in my back pocket My pen'll go off when I half-c*** it Got a fat knot from that rap profit Made a living and a killing off it Ever since Bill Clinton was still in office With Monica Lewinsky feeling on his
Nut-sack I'm an MC still as honest But as rude and indecent as all hell syllables, killaholic (Kill 'em all with) This slickety, gibbedy, hibbedy hip hop You don't really wanna get into a pissing match with this rappidy rap Packing a Mac in the back of the Ac, pack backpack rap, yep, yackidy-yac The
exact same time I attempt these lyrical acrobat stunts while I'm practicing That I'll still be able to break a
Motherf***in' table Over the back of a couple of
_
Faggots and crack it in half
Only
Realized it was ironic I was signed to Aftermath after the fact How could I not blow? All I do is drop F-bombs, feel my wrath of attack Rappers are having a rough time period, here's a Maxipad It's actually disastrously bad For the wack while I'm masterfully constructing this masterpiece as I'm beginning to feel
_
Like a Rap God, Rap God All my people from the front to the back nod, back nod Now who thinks their arms are long enough to slap box, slap box? Let me show you maintaining this s*** ain't that hard, that hard Everybody want the key and the secret to rap
immortality like I have got Well, to be truthful the blueprint's simply rage and youthful exuberance Everybody loves to root
for a nuisance Hit the
Earth like an asteroid, did nothing but shoot for the moon since MC's
_
get taken to school with this music Cause I use it as a vehicle to bust a rhyme Now I lead a new school full of students Me? I'm a product of Rakim, Lakim Shabazz, 2Pac N- -W.A, Cube, hey, Doc, Ren, Yella,
Eazy, thank you, they got Slim Inspired
enough to one day grow up, blow up and be in a position To meet Run DMC and induct them into the motherf***in' Rock n' Roll Hall of Fame Even though I walk in the church and burst in a ball of flames Only Hall of Fame I be inducted in is the alcohol of fame On the wall of shame You fags think it's all a game 'til I walk a flock of flames Off of planking, tell me what in the f*** are you thinking? Little gay looking boy So gay I can barely say it with a straight face looking boy You witnessing a massacre Like you watching a church gathering take place looking boy Oy vey, that boy's gay, that's all they say looking boy You get a thumbs up, pat on the back And a way to go from your label everyday looking boy Hey, looking boy, what you say looking boy? I got a "hell yeah" from Dre looking boy I'mma work for everything I have Never ask nobody for s***, get outta my face looking boy Basically boy you're never gonna be capable To keep up with the same pace looking boy 'Cause I'm beginning to feel like a Rap God, Rap God All my people from the front to the back nod, back nod The way I'm racing around the track, call me Nascar, Nascar Dale Earnhardt of the trailer park, the White Trash God Kneel before General
zorb
}
```

### Flag

Can you see the flag? Just pick the first character of each line:

> CTF{EmiNeM_FOR_LifE_gEez}

## Look at me


![](/lib/images/writeups/2018_otterctf/network/statement_net2.png)


I did this challenge after the CTF ending.


![](/lib/images/writeups/2018_otterctf/network/net2_3.png)


Wacom CTL 471 is a graphical tablet.

Based on those piece of information, I looked on the internet a CTF writeup. I finally found a writeup of "Tom and Jerry" challenge from BITSCTF: https://blogs.tunelko.com/2017/02/05/bitsctf-tom-and-jerry-50-points/

### Data extraction

Frames containing the most interesting data got the following syntax:

> 02:f1:d9:22:1c:25:4d:03:2b:00

Here is parsing of this frame:

* 02:f1 -> Header
* 50:1d -> X
* 72:1a -> Y
* 00:00 -> Pressure
* 2b:00 -> Suffix

It's possible to extract all frames with the following `tshark` command:

```bash
$ tshark -r lookatme.pcapng -Y '((usb.transfer_type == 0x01) && (frame.len == 37))' -Tfields -e usb.capdata > capdata
```

The first condition `usb.transfer_type` is used to filter the data on sending packets, not the response. It's like filtering on `echo request` for ICMP data. The second statement filter on data length.

### Data decoding

As mentioned in previous writeups, it's necessary to consider all axis, X, Y, and Z. Parsing data is possible with a little `awk` command, shamefully stolen from the BITSCTF writeup:

```bash
$ awk -F: '{x=$3$4;y=$5$6}{z=$7}$1=="02"{print x,y,z}' capdata > hex_value
```

And decoding data with a python script, still stolen from the previous write-up:

```python
#!/usr/bin/python
from pwn import *

f = open('data_plot','w')
 
for i in open('hex2').readlines():
    ii = i.strip().split(' ')
    x = int(ii[0], 16)
    y = int(ii[1], 16)
    z = int(ii[2], 16)
 
    if z > 0:
      f.write("{0} {1}\n".format(u16(struct.pack(">H", x)), u16(struct.pack(">H", y))))
```

With the last statement I ensure that data contains pressure, it prevents from some garbage during decoding. After that, just use `gnuplot` to visualize the data:

```bash
$ gnuplot
$ plot "data_plot"
```


![](/lib/images/writeups/2018_otterctf/network/net2_1.png)


### Flag

If I edit the picture a bit, it gives me:


![](/lib/images/writeups/2018_otterctf/network/net2_2.png)


> CTF{0TR_U58}

## Otter Leak


![](/lib/images/writeups/2018_otterctf/network/statement_net3.png)


For this challenge, I was really lucky. When I opened the PCAP file in my Wireshark I immediately notice some data sent through SMBv2. I configured one column of my Wireshark to display data transferred: `data.data`:


![](/lib/images/writeups/2018_otterctf/network/net3_1.png)


### Extraction and decoding time

Let's use `tshark`:

```bash
$ tshark -r OtterLeak.pcap -Y 'smb2 && data.data' -Tfields -e data.data | tr -d ' \n' | xxd -r -p
=0iLu4iLg4iLu4SLg4iLu0CIu0CIu4CIu0iLg0SLu4iLg4iLu0SLg0CIt0SLg4CIu4iLu4CIu0iLg0SLt0SLg0SL
```

It looks to be upside down base64, by putting it back in the right place, I will try to decode it:

```bash
$ cat rev_b64| rev | base64 -d
-- ----- .-. ..... . --- - --... ...-- .-. .. -. -... -.... ....-
```

Great, morse code. I should be on the right path. `dcode` website offers morse decoder:


![](/lib/images/writeups/2018_otterctf/network/net3_2.png)


### Flag

> CTF{M0R5EOT73RINB64}

---

To conclude this little paper, the OtterCTF was an amazing CTF for forensic lovers. Honestly, I really liked it!
If you have any question about this article, feel free to contact me on twitter @Maki_chaz. 
Thanks for reading, and merry Christmas guys :) 


![](https://media.giphy.com/media/fjyGsdRkYqb3j1WODz/giphy.gif)
