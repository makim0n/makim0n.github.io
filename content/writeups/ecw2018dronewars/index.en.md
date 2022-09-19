---
weight: 4
title: "[European Cyber Week] - Drone wars"
date: 2018-10-22T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["ecw", "thales", "airbus", "rf", "radiofrequency"]
categories: ["Writeups"]

lightgallery: true
---

## Drone Wars 1

### State of the art

We have 2 files: `Capture.zip` and `capture.wav`. There is a binary file in the zip archive, don't worry about it for now.

After several hours of crawling the internet searching for datas about the .wav files, I found a stego technic: SSTV.

> https://medium.com/@sumit.arora/audio-steganography-the-art-of-hiding-secrets-within-earshot-part-2-of-2-c76b1be719b3

I found a tool for linux: `qsstv`.

### Decoding the .wav

I run `QSSTV` with `VLC` in background, set my audio output into QSSTV, and:


![](/lib/images/writeups/2018_ecw/dronewars/drone1.png)


### Flag

> ECW{da553166e44a3151dfe422c34f693fe6}

---

## Drone Wars Hint 1


Like the first Drone Wars challenge, we have a horrible `.wav` file. Let's try with QSSTV:


![](/lib/images/writeups/2018_ecw/dronewars/drone2.png)


Same thing :)

### Flag

> ECW{SHELLCODES}

---

## Drone wars 2

In the first challenge, we got a QR code. It contained the following data:

> 6xVeMcAx2zHJs0WwKjEEDkE5y3X4/+bo5v///xvqG/Eb4xv4mi6ZK8Emc5gAPueqG+pqG/HnqsLF1dXVbwBpfVFLHhtPT05LGxNOThlJABlMThJITxIbGEgdEkxMHBsASE9IVyA=

Decoded:

> \xeb\x15^1\xc01\xdb1\xc9\xb3E\xb0*1\x04\x0eA9\xcbu\xf8\xff\xe6\xe8\xe6\xff\xff\xff\x1b\xea\x1b\xf1\x1b\xe3\x1b\xf8\x9a.\x99+\xc1&s\x98\x00>\xe7\xaa\x1b\xeaj\x1b\xf1\xe7\xaa\xc2\xc5\xd5\xd5\xd5o\x00i}QK\x1e\x1bOONK\x1b\x13NN\x19I\x00\x19LN\x12HO\x12\x1b\x18H\x1d\x12LL\x1c\x1b\x00HOHW 

Wow, cool, bullshit. Let's try with le old bruteforce technique. Rotation? Nope. XOR ? Yes! It was a one byte key.

```python
#!/usr/bin/python2

import base64

msg = base64.b64decode('6xVeMcAx2zHJs0WwKjEEDkE5y3X4/+bo5v///xvqG/Eb4xv4mi6ZK8Emc5gAPueqG+pqG/HnqsLF1dXVbwBpfVFLHhtPT05LGxNOThlJABlMThJITxIbGEgdEkxMHBsASE9IVyA=')
key = chr(42)
s = ""

for j in range(256):
    j = chr(j)
    for i in range(len(msg)):
        s += chr(ord(msg[i])^ord(j[i%len(j)]))
        if 'W{' in s:
            print(s)
    s = ""
```

> �?to��_��������1�1�1�1Ұ��
                         Y�*̀1�@1�̀�����E*CW{a41eeda19dd3c*3fd8be812b78ff61*beb}
 
### Flag

> ECW{a41eeda19dd3c3fd8be812b78ff61beb}

---

## Drone wars hint 2

### State of the art

We have a JPG picture, a lot of steg stuff can be effective.


![](/lib/images/writeups/2018_ecw/dronewars/dwhint2.jpg)


### Guessing of the year

I went with the `steghide` tool... And the steghide passphrase: `ECW`.

```bash
$ steghide extract -sf DSC20181007160312834378.jpg
Entrez la passphrase: (ECW)
�criture des donn�es extraites dans "secret.txt".

$ cat secret.txt 
..-. .. .-.. . / ... --- ..- .-. -.-. . / -....- # / --. ..-. ... -.- / -.. . -- --- -.. / -....- # / .--. .- -.-. -.- . - / -.. . -.-. --- -.. . .-. / -....- # / ..-. .. .-.. . / ... .. -. -.-
```

### Morse to binary

> https://www.dcode.fr/code-morse

Input:

```
..-. .. .-.. . / ... --- ..- .-. -.-. . / -....- / --. ..-. ... -.- / -.. . -- --- -.. / -....- / .--. .- -.-. -.- . - / -.. . -.-. --- -.. . .-. / -....- / ..-. .. .-.. . / ... .. -. -.-
```

Output and flag:

> FILE SOURCE - GFSK DEMOD - PACKET DECODER - FILE SINK

---

## Drone wars 3


### State of the art

Do you remember the binary file in the zip archive in the Drone Wars 1 challenge? The `Capture.bin` one. Thanks to the Drone Wars 2 hint, we now know that we have to use `GNU Radio`.

File source, GFSK demode, etc... Are GNU Radio blocks to decode a source file.

### GNU Radio


![](/lib/images/writeups/2018_ecw/dronewars/drone3_1.png)


I choose this configuration:

* File source -> GFSK Demod: Complex mode
* GFSK Demod -> Packet decoder -> File Sink: Byte mode

Let's run this, and see what kind of data there is in our `toto.bin` file:


![](/lib/images/writeups/2018_ecw/dronewars/drone3_2.png)


### GPS to ASCII

By googling `gps to ascii`, I found this website:

> http://www.gpsvisualizer.com/convert_input

I format my `toto.bin` into a valid CSV file for this website:


![](/lib/images/writeups/2018_ecw/dronewars/drone3_3.png)


### Flag

And then (when zooming):


![](/lib/images/writeups/2018_ecw/dronewars/drone3_4.png)
