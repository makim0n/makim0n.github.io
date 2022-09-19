---
weight: 4
title:  "[Santhacklaus 2018] - Bret Stiles"
date: 2018-12-19T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["santhacklaus", "pinkflood", "forensic", "memory", "gimp"]
categories: ["Writeups"]

lightgallery: true
---


## Statement


![](/lib/images/writeups/2018_santhacklaus/bretstiles/statement_bretstiles.png)


## State of the art

A memory dump <3


![](https://media.giphy.com/media/8g63zqQ5RPt60/giphy.gif)


```bash
$ volatility -f challenge.dmp imageinfo
[Some garbage errors]
Suggested Profile(s) : Win10x64_17134, Win10x64_10240_17770, Win10x64_14393, Win10x64_10586, Win10x64, Win2016x64_14393, Win10x64_16299, Win10x64_15063 (Instantiated with Win10x64_15063)
                     AS Layer1 : SkipDuplicatesAMD64PagedMemory (Kernel AS)
                     AS Layer2 : WindowsCrashDumpSpace64 (Unnamed AS)
                     AS Layer3 : FileAddressSpace (/home/monique/Téléchargements/bretstiles/challenge.dmp)
                      PAE type : No PAE
                           DTB : 0x1aa000L
                          KDBG : 0xf80150ad4a60L
          Number of Processors : 1
     Image Type (Service Pack) : 0
                KPCR for CPU 0 : 0xfffff80150b2d000L
             KUSER_SHARED_DATA : 0xfffff78000000000L
           Image date and time : 2018-11-05 20:50:14 UTC+0000
     Image local date and time : 2018-11-05 12:50:14 -0800
```

Yay Windows 10... :'(

## Find the right profile

First things to do is finding the correct Windows 10 profile for volatility. After some test, I finally find: `Win10x64_10586`

```bash
$ volatility -f challenge.dmp --profile=Win10x64_10586 pstree

Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0xffffe0009342b680:System                              4      0    103      0 2018-11-05 20:47:01 UTC+0000
. 0xffffe00094897040:smss.exe                         272      4      3      0 2018-11-05 20:47:01 UTC+0000
.. 0xffffe000952c6080:smss.exe                        412    272      0 ------ 2018-11-05 20:47:08 UTC+0000
... 0xffffe000951e8540:csrss.exe                      432    412     10      0 2018-11-05 20:47:08 UTC+0000
... 0xffffe00095395080:winlogon.exe                   484    412      5      0 2018-11-05 20:47:08 UTC+0000
.... 0xffffe0009566d640:dwm.exe                       772    484     12      0 2018-11-05 20:47:09 UTC+0000
.... 0xffffe00095ddc680:userinit.exe                 2332    484      0 ------ 2018-11-05 20:47:33 UTC+0000
..... 0xffffe00095dda500:explorer.exe                2348   2332     58      0 2018-11-05 20:47:33 UTC+0000
...... 0xffffe00096164080:OneDrive.exe               3328   2348     18      0 2018-11-05 20:47:53 UTC+0000
...... 0xffffe000961ae3c0:mspaint.exe                3372   2348      7      0 2018-11-05 20:47:56 UTC+0000
...... 0xffffe000961257c0:VBoxTray.exe               3252   2348     13      0 2018-11-05 20:47:52 UTC+0000
...... 0xffffe0009474f080:cmd.exe                    2144   2348      5      0 2018-11-05 20:50:05 UTC+0000
....... 0xffffe00094841080:conhost.exe               3352   2144      3      0 2018-11-05 20:50:05 UTC+0000
 0xffffe00095df8080:NisSrv.exe                       2112    524      7      0 2018-11-05 20:47:30 UTC+0000
[...]
```

It looks to work, let's continue the analysis.

## File list

In a forensic challenge, I immediately list all opened file in memory, using `filescan` plugin:

```bash
$ volatility -f challenge.dmp --profile=Win10x64_10586 filescan > filescan.txt 

$ cat filescan.txt| grep John | grep Desktop
0x0000e000948df780  32768      1 R--rwd \Device\HarddiskVolume2\Users\John\Desktop
0x0000e000956917a0      2      0 R--rwd \Device\HarddiskVolume2\Users\John\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\System Tools\Desktop.ini
0x0000e00095e65320  32768      1 R--rwd \Device\HarddiskVolume2\Users\John\Desktop
0x0000e00095f03090     16      0 R--rwd \Device\HarddiskVolume2\Users\John\Desktop\desktop.ini
0x0000e0009600df20      2      0 R--rwd \Device\HarddiskVolume2\Users\John\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Accessories\Desktop.ini
0x0000e00096087a20      2      0 R--rwd \Device\HarddiskVolume2\Users\John\AppData\Roaming\Microsoft\Windows\SendTo\Desktop.ini
0x0000e0009608abe0      8      0 R--r-d \Device\HarddiskVolume2\Users\John\Desktop\bob.png
0x0000e000960c0700  32768      1 R--rw- \Device\HarddiskVolume2\Users\John\Desktop
```

We can notice "bob.png" on the Desktop, let's extract it:

```bash
$ volatility -f challenge.dmp --profile=Win10x64_10586 dumpfiles -Q 0x0000e000960c0700 -D .
```

Aaaaaaaaaand... Nothing. Ok, not a problem, I got more than one trick in my hat :D

## Process memory dump in GIMP

During my search for a correct profile, I noticed the `mspaint.exe` process. According to this website: https://w00tsec.blogspot.com/2015/02/extracting-raw-pictures-from-memory.html

I tried to dump the process memory and open it in GIMP as raw picture.

```bash
$ volatility -f challenge.dmp --profile=Win10x64_10586 memdump -p 3372 -D .
************************************************************************
Writing mspaint.exe [  3372] to 3372.dmp

$ mv 3372.dmp mspaint.data
```

After few minutes burning my eyes, I finally found the Graal:


![](/lib/images/writeups/2018_santhacklaus/bretstiles/bret_1.png)


## Flag

> IMTLD{1m4gin4ti0N}

## Fun fact

You can use `strings` and `grep`, best friends of forensic analysts:


![](/lib/images/writeups/2018_santhacklaus/bretstiles/bret_2.png)
