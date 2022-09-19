---
weight: 4
title: "[French FCSC] - Académie"
date: 2020-06-27T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["anssi", "fcsc", "forensic", "volatility", "memory analysis", "keepass", "chrome"]
categories: ["Writeups"]

lightgallery: true
---


## C'est la rentrée

>Bienvenue à l'académie de l'investigation numérique ! Votre mission, valider un maximum d'étapes de cette série afin de démontrer votre dextérité en analyse mémoire GNU/Linux.
>
>Première étape : retrouvez le HOSTNAME, le nom de l'utilisateur authentifié lors du dump et la version de Linux sur lequel le dump a été fait.
>
>Format du flag : FCSC{hostname:user:x.x.x-x-amdxx}

### TL;DR

Cette suite de challenge se base sur le même dump mémoire. Le premier consiste à se rendre compte que c'est un dump Linux.

### État des lieux

La première chose que je fais sur un dump mémoire pour un challenge, c'est de faire deux `strings`:

```bash
➜  strings dmp.mem > str_dmp.txt 
➜  strings -e l dmp.mem > str_el_dmp.txt
```

Pour ce challenge, on n’aura pas besoin de plus.

### Hostname

```bash
➜  cat str_* | grep "HOSTNAME="
XAUTHLOCALHOSTNAME=
_HOSTNAME=challenge.fcsc
```

> challenge.fcsc

### User

```bash
➜  cat str_* | grep -Eo "^/home/.*/$"
/home/Lesage/.mozilla/firefox/peyjyk3f.default-esr/extensions/
/home/Lesage/.local/share/mime/
/home/Lesage/.mozilla/firefox/peyjyk3f.default-esr/extensions/
/home/Lesage/.mozilla/firefox/peyjyk3f.default-esr/extensions/
/home/Lesage/.local/share/mime/
/home/Lesage/.local/share/mime/
/home/Lesage/Documents/temp/

➜  cat str_* | grep "USER=" | sort | uniq
USER=Lesage
[...]
```

> Lesage

### Version kernel

```bash
➜  cat str_* | grep 'Linux version' | uniq      
Linux version 5.4.0-4-amd64 (debian-kernel@lists.debian.org) (gcc version 9.2.1 20200203 (Debian 9.2.1-28)) #1 SMP Debian 5.4.19-1 (2020-02-13)

➜  cat str_* | grep "BOOT_IMAGE=" 
BOOT_IMAGE=/boot/vmlinuz-5.4.0-4-amd64 root=UUID=536c82dd-f1c5-43ce-b65d-c94e5c4a5031 ro quiet
[...]
```

> 5.4.0-4-amd64

### Flag

> FCSC{challenge.fcsc:Lesage:5.4.0-4-amd64}

## Administration

>Ce poste administre un serveur distant avec le protocole SSH à l'aide d'une authentification par clé (clé protégée par mot de passe). La clé publique a été utilisée pour chiffrer le message ci-joint (flag.txt.enc).
>
>Retrouvez et reconstituez la clé en mémoire qui permettra de déchiffrer ce message.

### TL;DR

Pour ce challenge il n'y a pas non plus besoin de générer un profil volatility. En effet, le fichier chiffré a une taille de 2048 bits et l'énoncé parle de SSH. Donc on cherche une clé privée RSA, pour cela il existe `rsakeyfind` qui nous donne "d" et "n". Ces informations permettent de déchiffrer le flag.

### État de l'art

On commence le challenge avec un fichier chiffré "SSH". Intuitivement, on peut penser à du RSA, mais il existe plusieurs algorithmes disponibles. Pour s'en assurer, il suffit de vérifier la taille du fichier chiffré:

```bash
➜  cat flag.txt.enc| wc -c
256 # 256*8 = 2048
```

On peut se dire qu'on cherche une clé privée RSA.

### Récupérer la clé privée

L'outil `rsakeyfind` : https://github.com/congwang/rsakeyfind

```bash
➜  ~/tools/forensic/rsakeyfind/rsakeyfind dmp.mem 
FOUND PRIVATE KEY AT c64ac50
version = 
00 
modulus = 
00 d7 1e 77 82 8c 92 31 e7 69 02 a2 d5 5c 78 de 
a2 0c 8f fe 28 59 31 df 40 9c 60 61 06 b9 2f 62 
40 80 76 cb 67 4a b5 59 56 69 17 07 fa f9 4c bd 
6c 37 7a 46 7d 70 a7 67 22 b3 4d 7a 94 c3 ba 4b 
7c 4b a9 32 7c b7 38 95 45 64 a4 05 a8 9f 12 7c 
4e c6 c8 2d 40 06 30 f4 60 a6 91 bb 9b ca 04 79 
11 13 75 f0 ae d3 51 89 c5 74 b9 aa 3f b6 83 e4 
78 6b cd f9 5c 4c 85 ea 52 3b 51 93 fc 14 6b 33 
5d 30 70 fa 50 1b 1b 38 81 13 8d f7 a5 0c c0 8e 
f9 63 52 18 4e a9 f9 f8 5c 5d cd 7a 0d d4 8e 7b 
ee 91 7b ad 7d b4 92 d5 ab 16 3b 0a 8a ce 8e de 
47 1a 17 01 86 7b ab 99 f1 4b 0c 3a 0d 82 47 c1 
91 8c bb 2e 22 9e 49 63 6e 02 c1 c9 3a 9b a5 22 
1b 07 95 d6 10 02 50 fd fd d1 9b be ab c2 c0 74 
d7 ec 00 fb 11 71 cb 7a dc 81 79 9f 86 68 46 63 
82 4d b7 f1 e6 16 6f 42 63 f4 94 a0 ca 33 cc 75 
13 
publicExponent = 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 00 00 00 00 00 01 00 01 
privateExponent = 
62 b5 60 31 4f 3f 66 16 c1 60 ac 47 2a ff 6b 69 
00 4a b2 5c e1 50 b9 18 74 a8 e4 dc a8 ec cd 30 
bb c1 c6 e3 c6 ac 20 2a 3e 5e 8b 12 e6 82 08 09 
38 0b ab 7c b3 cc 9c ce 97 67 dd ef 95 40 4e 92 
e2 44 e9 1d c1 14 fd a9 b1 dc 71 9c 46 21 bd 58 
88 6e 22 15 56 c1 ef e0 c9 8d e5 80 3e da 7e 93 
0f 52 f6 f5 c1 91 90 9e 42 49 4f 8d 9c ba 38 83 
e9 33 c2 50 4f ec c2 f0 a8 b7 6e 28 25 56 6b 62 
67 fe 08 f1 56 e5 6f 0e 99 f1 e5 95 7b ef eb 0a 
2c 92 97 57 23 33 36 07 dd fb ae f1 b1 d8 33 b7 
96 71 42 36 c5 a4 a9 19 4b 1b 52 4c 50 69 91 f0 
0e fa 80 37 4b b5 d0 2f b7 44 0d d4 f8 39 8d ab 
71 67 59 05 88 3d eb 48 48 33 88 4e fe f8 27 1b 
d6 55 60 5e 48 b7 6d 9a a8 37 f9 7a de 1b cd 5d 
1a 30 d4 e9 9e 5b 3c 15 f8 9c 1f da d1 86 48 55 
ce 83 ee 8e 51 c7 de 32 12 47 7d 46 b8 35 df 41 
prime1 = 
00 
prime2 = 
00 
exponent1 = 
00 
exponent2 = 
00 
coefficient = 
00 
```

### Déchiffrer le fichier

Il suffit d'utiliser 'd', 'n' et le fichier "flag.txt.enc" pour récupérer le flag:

```bash
➜  cat flag.txt.enc | xxd -p | tr -d '\n'
6da249ada8a979ed289b249d9ed4af472bda2ebbd1d9ea8b63f927f9bf91ba1bb76c378e6e62f07b7717f1e1def03dd65a8c6ee3ce40447d12f7b07ad2d18256a3997bd5786cccfaa678b1f60b069e6b8ccc70eb6948c3624eaf7832727bd034c9964c7dd863830d3b07bc4b242c4fe0ead36ae7a81ff1419b658b86dd30d78acc797f5b7a2b5c08d3fc807fbcd9e61e7f46071cdc3906882b9084e9af9231c4b343fbac2349232d368b0d343f6e9e2c7ff32242e1885590fdfe762ce8658c684c04ea19b8bf7b5153c4a4cdfbd5847da3b53fc285469b7761a5da6f1778b8972da6006d279a8042bbfaa7045ec72bf778f24d6da5fe1fde422667bc9c53f77d
```

```python
d = int(0xd71e77828c9231e76902a2d55c78dea20c8ffe285931df409c606106b92f62408076cb674ab55956691707faf94cbd6c377a467d70a76722b34d7a94c3ba4b7c4ba9327cb738954564a405a89f127c4ec6c82d400630f460a691bb9bca0479111375f0aed35189c574b9aa3fb683e4786bcdf95c4c85ea523b5193fc146b335d3070fa501b1b3881138df7a50cc08ef96352184ea9f9f85c5dcd7a0dd48e7bee917bad7db492d5ab163b0a8ace8ede471a1701867bab99f14b0c3a0d8247c1918cbb2e229e49636e02c1c93a9ba5221b0795d6100250fdfdd19bbeabc2c074d7ec00fb1171cb7adc81799f86684663824db7f1e6166f4263f494a0ca33cc7513)

n = int(0x62b560314f3f6616c160ac472aff6b69004ab25ce150b91874a8e4dca8eccd30bbc1c6e3c6ac202a3e5e8b12e6820809380bab7cb3cc9cce9767ddef95404e92e244e91dc114fda9b1dc719c4621bd58886e221556c1efe0c98de5803eda7e930f52f6f5c191909e42494f8d9cba3883e933c2504fecc2f0a8b76e2825566b6267fe08f156e56f0e99f1e5957befeb0a2c92975723333607ddfbaef1b1d833b796714236c5a4a9194b1b524c506991f00efa80374bb5d02fb7440dd4f8398dab71675905883deb484833884efef8271bd655605e48b76d9aa837f97ade1bcd5d1a30d4e99e5b3c15f89c1fdad1864855ce83ee8e51c7de3212477d46b835df41)

c = int(0x6da249ada8a979ed289b249d9ed4af472bda2ebbd1d9ea8b63f927f9bf91ba1bb76c378e6e62f07b7717f1e1def03dd65a8c6ee3ce40447d12f7b07ad2d18256a3997bd5786cccfaa678b1f60b069e6b8ccc70eb6948c3624eaf7832727bd034c9964c7dd863830d3b07bc4b242c4fe0ead36ae7a81ff1419b658b86dd30d78acc797f5b7a2b5c08d3fc807fbcd9e61e7f46071cdc3906882b9084e9af9231c4b343fbac2349232d368b0d343f6e9e2c7ff32242e1885590fdfe762ce8658c684c04ea19b8bf7b5153c4a4cdfbd5847da3b53fc285469b7761a5da6f1778b8972da6006d279a8042bbfaa7045ec72bf778f24d6da5fe1fde422667bc9c53f77d)

t = pow(c,d,n)
print(hex(t))

# Output : 0x22681c036dd635f46c2853e08b7359b196e5db09f3fa40a801704b36f7408c8ed931f29ff4319a7ba0be6e107624a43471d49cc14bb0e71488c76230de112bb05f7821bc81f67db4c80fd038938bfc7fed1e963b3e9adb27bd127948b1d9bb3874b1bc320e90628a991bc066dc4406830133627cb148370d14d9538ff238d12f002bc89233daa0beda1ca2c5bc714087ef808624e5b4ed1e9b62bb6f6a34c88907381f1606de98951ba3733bf8d426e310417b61e394300464353437b616335636164363631313464343836366134623535653433636238383936636334393437383535323431623561663864326638613132336333363038336439387d0a
```

```bash
➜  echo -n '022681c036dd635f46c2853e08b7359b196e5db09f3fa40a801704b36f7408c8ed931f29ff4319a7ba0be6e107624a43471d49cc14bb0e71488c76230de112bb05f7821bc81f67db4c80fd038938bfc7fed1e963b3e9adb27bd127948b1d9bb3874b1bc320e90628a991bc066dc4406830133627cb148370d14d9538ff238d12f002bc89233daa0beda1ca2c5bc714087ef808624e5b4ed1e9b62bb6f6a34c88907381f1606de98951ba3733bf8d426e310417b61e394300464353437b616335636164363631313464343836366134623535653433636238383936636334393437383535323431623561663864326638613132336333363038336439387d0a' | xxd -r -p | strings -n 10
FCSC{ac5cad66114d4866a4b55e43cb8896cc4947855241b5af8d2f8a123c36083d98}
```

### Flag

> FCSC{ac5cad66114d4866a4b55e43cb8896cc4947855241b5af8d2f8a123c36083d98}

---

## Premiers artéfacts

>Pour avancer dans l'analyse, vous devez retrouver : 
> 
> * Le nom de processus ayant le PID 1254.
> * La commande exacte qui a été exécutée le 2020-03-26 23:29:19 UTC.
> * Le nombre d'IP-DST unique en communications TCP établies (état ESTABLISHED) lors du dump.
> 
>Format du flag : FCSC{nom_du_processus:une_commande:n}

### TL;DR

Ce challenge va nécessiter le profil Linux associé. Pour cela on se sert des snapshot Debian de Sid à la bonne date pour récupérer le bon kernel. Une fois le profil généré, il suffit d'utiliser les plugins linux_psscan, linux_bash et linux_netscan. 

### État de l'art

Ce challenge annonce le début des ennuis. En effet, il va falloir créer un profil volatility pour pouvoir récupérer ces informations. En temps normal, créer un profil sur un Debian, c'est plutôt très simple (cf. Un poc en bash d'automatisation que j'avais fait : https://github.com/Zeecka/Auto_vol).

Le souci principal va être de trouver la bonne version de Kernel. Grâce au challenge précédent, on sait que le kernel est de version : __5.4.0-4-amd64__. Qui dit très récent, va dire très casse-pied.

### Génération du profil

La version est:
```bash
➜  cat str_* | grep 'Linux version' | uniq      
Linux version 5.4.0-4-amd64 (debian-kernel@lists.debian.org) (gcc version 9.2.1 20200203 (Debian 9.2.1-28)) #1 SMP Debian 5.4.19-1 (2020-02-13)
```

Deux informations intéressantes:

* Kernel : 5.4.0-4-amd64
* Date : 2020-02-13

Avec cette date, il est possible de trouver un snapshot précis des dépôts Debian sur : https://snapshot.debian.org/

La première étape est de créer une VM Debian: https://cdimage.debian.org/debian-cd/current/amd64/bt-cd/debian-10.3.0-amd64-netinst.iso.torrent

Lorsque la machine est installée, un simple `uname -a` montre qu'on a pas du tout la bonne version de kernel.  C'est là que les snapshots Debian seront utiles. Avec un peu de recherche, on trouve ces dépôts:

```bash
deb [check-valid-until=no] https://snapshot.debian.org/archive/debian/20200213T231921Z/ sid main contrib non-free
deb-src [check-valid-until=no] https://snapshot.debian.org/archive/debian/20200213T231921Z/ sid main contrib non-free
```

Donc en l'ajoutant dans notre __/etc/apt/sources.list__, on peut faire une recherche, et là:

```bash
user@debian:~$ apt search linux-headers
En train de trier... Fait
Recherche en texte intégral... Fait
[...]
linux-headers-5.4.0-4-amd64/unstable,now 5.4.19-1 amd64  
  Header files for Linux 5.4.0-4-amd64
[...]

user@debian:~$ apt search linux-image
En train de trier... Fait
Recherche en texte intégral... Fait
[...]
linux-image-5.4.0-4-amd64-unsigned/unstable,now 5.4.19-1 amd64  
  Linux 5.4 for 64-bit PCs
[...]
```

Oook boy, il est temps d'installer ce qu'il nous faut:

```bash
root@debian:/home/user# apt install linux-headers-5.4.0-4-amd64 linux-image-5.4.0-4-amd64-unsigned volatility-tools
```

Une fois le tout installé, il faut redémarrer la machine et boot sur le bon kernel (grub est plutôt intuitif pour ça):

```bash
user@debian:~$ uname -a
Linux debian 5.4.0-4-amd64 #1 SMP Debian 5.4.19-1 (2020-02-13) x86_64 GNU/Linux
```

Ca a une bonne tête. Il est temps de créer le profil (cf. https://www.maki.bzh/walkthrough/santhacklaus2018/#mission-impossible-1).

```bash
root@debian:/home/user# cd /usr/src/volatility-tools/linux/

root@debian:/usr/src/volatility-tools/linux# make
[...BLABLABLA MAKE BLABLALBA]
  
root@debian:/usr/src/volatility-tools/linux# zip ecsc_deb540.zip /usr/src/volatility-tools/linux/module.dwarf /boot/System.map-5.4.0-4-amd64 
  adding: usr/src/volatility-tools/linux/module.dwarf (deflated 91%)
  adding: boot/System.map-5.4.0-4-amd64 (deflated 79%)
```

Aller maintenant on le met dans un dossier accessible par volatility (j'utilise volatility en docker, pour être sur que les dépendances pètent pas, genre yara: https://github.com/makim0n/infosec-docker/tree/master/forensic/volatility).

Donc j'ai mis le profil __ecsc_deb540.zip__ dans le dossier de __/opt/plug_vol__ de mon docker:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --info | grep ecsc
Linuxecsc_deb540x64                      - A Profile for Linux ecsc_deb540 x64
```

> Linuxecsc_deb540x64

Aller, c'est le moment le plus stressant: faut tester que ça marche.

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_banner
Linux version 5.4.0-4-amd64 (debian-kernel@lists.debian.org) (gcc version 9.2.1 20200203 (Debian 9.2.1-28)) #1 SMP Debian 5.4.19-1 (2020-02-13)
```


<img src="https://media.giphy.com/media/S5fMUcgKUs04E/giphy.gif" />


### Nom du processus 1254

Maintenant que le profil est créé, en temps normal je fais un `linux_pstree`, mais ici ça ne semble pas aider beaucoup:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_pstree > /opt/usr_land/linux_pstree 

➜  cat linux_pstree| grep 1254
# Pas de retour, c'est fâcheux.
```

Donc là, pas le choix, on sort la doc: https://github.com/volatilityfoundation/volatility/wiki/Linux-Command-Reference

On voit le plugin `linux_psslit`:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_pslist > /opt/usr_land/pslist 

➜  cat pslist| grep 1254
# Pas de retour encore une fois, c'est fâcheux.
```

Je me rappelle ce qu'un ancien collègue et sensei du forensic ([@AZobec](https://twitter.com/AZobec) <3) m'avais dit:

> Volatility c'est bien, mais la doc c'est le code, donc creuse quand tu trouves pas

Donc on part pour le github de volatility dans le dossier des plugins: https://github.com/volatilityfoundation/volatility/tree/master/volatility/plugins/linux

![](https://i.imgur.com/27gbkDA.png)

Ce plugin n'est pas du tout présent dans la command reference.

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_psscan > /opt/usr_land/psscan 

➜  cat psscan| grep 1254
0x000000003fdccd80 pool-xfconfd         1254            -               -1              -1     0x0fd08ee88ee08ec0 -
```

Enfin:

> pool-xfconfd

### La commande exécutée

Les commandes exécutées par l'utilisateur sont plus simples à trouver:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_bash > /opt/usr_land/linux_bash

➜  cat linux_bash 
Pid      Name                 Command Time                   Command
-------- -------------------- ------------------------------ -------
[...]
1523 bash                 2020-03-26 23:29:19 UTC+0000   nmap -sS -sV 10.42.42.0/24
[...]
```

> nmap -sS -sV 10.42.42.0/24

D'ailleurs, quitte à rester dans "la doc c'est le code", il existe des arguments pour les plugins volatility, qui ne sont pas forcément documentés:

![](https://i.imgur.com/jrVDpiG.png)

> https://github.com/volatilityfoundation/volatility/blob/master/volatility/plugins/linux/bash.py

Ici, ça ne sert à rien, mais c'est bon à savoir.

### Nombre de connexion

Pour le nombre de connexions, on sait que ce sont les connexions "établies". Avec mon super niveau en anglais, "Peer connections" j'étais pas sur de ce que c'était, mais la commande `ss` de mon système m'a aidée:

```bash
➜  ss | head -n 2
Netid            State                  Recv-Q              Send-Q                                                                                Local Address:Port                                         Peer Address:Port                  
u_str            ESTAB                  0                   0                                                                                                 * 178269                                                  * 180308                
```

Ce sont donc les adresses de destinations. Il existe un plugin pour lister les connexions lors du dump: `linux_netscan`.

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_netscan > /opt/usr_land/netscan

➜  cat netscan | grep ESTA
9d72830a8000 TCP      10.42.42.131    :58772 185.199.111.154 :  443 ESTABLISHED    
9d72830a88c0 TCP      10.42.42.131    :45652 35.190.72.21    :  443 ESTABLISHED    
9d72830a9a40 TCP      10.42.42.131    :53190 104.124.192.89  :  443 ESTABLISHED    
9d72830abd40 TCP      10.42.42.131    :55226 151.101.121.140 :  443 ESTABLISHED    
9d72830ad780 TCP      10.42.42.131    :50612 104.93.255.199  :  443 ESTABLISHED    
9d72830af1c0 TCP      10.42.42.131    :38184 216.58.213.142  :  443 ESTABLISHED    
9d7284eba300 TCP      10.42.42.131    :37252 163.172.182.147 :  443 ESTABLISHED    
9d7284fe9180 TCP      127.0.0.1       :38498 127.0.0.1       :34243 ESTABLISHED    
9d7284fe9a40 TCP      10.42.42.131    :57000 10.42.42.134    :   22 ESTABLISHED    
9d7284feb480 TCP      10.42.42.131    :51858 10.42.42.128    :  445 ESTABLISHED    
9d7284fef1c0 TCP      10.42.42.131    :55224 151.101.121.140 :  443 ESTABLISHED    
9d7293778000 TCP      10.42.42.131    :47100 216.58.206.226  :  443 ESTABLISHED    
9d729377cec0 TCP      10.42.42.131    :47106 216.58.206.226  :  443 ESTABLISHED    
9d72c0acb480 TCP      10.42.42.131    :36970 116.203.52.118  :  443 ESTABLISHED    
9d72c1503d40 TCP      127.0.0.1       :34243 127.0.0.1       :38498 ESTABLISHED    
9d72c1bc1280 TCP      fd:6663:7363:1000:c10b:6374:25f:dc37:36280 fd:6663:7363:1000:55cf:b9c6:f41d:cc24:58014 ESTABLISHED    
9d72c23fcec0 TCP      10.42.42.131    :38186 216.58.213.142  :  443 ESTABLISHED    
9d72c23fe040 TCP      10.42.42.131    :47104 216.58.206.226  :  443 ESTABLISHED    
9d72c23fe900 TCP      10.42.42.131    :47102 216.58.206.226  :  443 ESTABLISHED    
```

Avec un peu de formating pour garder uniquement les IPs sortantes, on tombe sur:

```bash
➜  echo -n '185.199.111.154     
35.190.72.21        
104.124.192.89      
151.101.121.140     
104.93.255.199      
216.58.213.142      
163.172.182.147     
127.0.0.1           
10.42.42.134        
10.42.42.128        
151.101.121.140     
216.58.206.226      
216.58.206.226      
116.203.52.118      
127.0.0.1           
216.58.213.142      
216.58.206.226      
216.58.206.226      
fd:6663:7363:1000:55cf:b9c6:f41d:cc24:58014    
' | tr -d ' ' | sort | uniq | wc -l
13
```

> 13

### Flag

Challenge enfin terminé.

> FCSC{pool-xfconfd:nmap -sS -sV 10.42.42.0/24:13}

---

## Porte dérobée

>Un poste distant est connecté au poste en cours d'analyse via une porte dérobée avec la capacité d'exécuter des commandes.
>
> * Quel est le numéro de port à l'écoute de cette connexion ?
> * Quelle est l'adresse IP distante connectée au moment du dump ?
> * Quel est l'horodatage de la création du processus en UTC de cette porte dérobée ?
>
>Format du flag : FCSC{port:IP:YYYY-MM-DD HH:MM:SS}

### TL;DR

Ce challenge est dans la continuité du précédent sur l'analyse du dump. Grâce à `linux_netscan`, il est possible de voir des processus suspects. Ensuite, `linux_pstree` permet de s'en assurer. Enfin, l'horodatage est trouvable grâce à `linux_pslist`.

### État des lieux

Ce challenge est aussi de l'analyse plutôt basique d'un dump mémoire. Dans le challenge précédant, on commence par utiliser le plugin `linux_netstat`, par défaut il va aussi afficher les sockets unix, on n’y reprendra pas, allons voir le code:

![](https://i.imgur.com/IUhlkTX.png)

> https://github.com/volatilityfoundation/volatility/blob/master/volatility/plugins/linux/netstat.py

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_netstat -U > /opt/usr_land/netstat_wo_socketunix

➜  cat netstat_wo_socketunix
➜  cestlarentree cat netstat_wo_socketunix | grep ESTA
[...]
TCP      fd:6663:7363:1000:c10b:6374:25f:dc37:36280 fd:6663:7363:1000:55cf:b9c6:f41d:cc24:58014 ESTABLISHED                  ncat/1515 
[...]
TCP      fd:6663:7363:1000:c10b:6374:25f:dc37:36280 fd:6663:7363:1000:55cf:b9c6:f41d:cc24:58014 ESTABLISHED                    sh/119511
[...]
```

En vrai, je suis pas un expert, mais un __ncat__ et un __sh__... Mon petit passif de pentester me dit que ça sent pas bon.



![](https://media.giphy.com/media/3oEdv8EdOATmWJDCNO/giphy.gif)



### Port en écoute

En remontant cette piste, il est simple de trouver les informations que l'on cherche:

> 36280

### IP distante connectée

L'IP distante aussi est donnée:

> fd:6663:7363:1000:55cf:b9c6:f41d:cc24

### Horodatage

Pour l'horodatage, on a le PID du __ncat__ et du __sh__. Avec `linux_pstree` il est possible de voir le processus parent facilement:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_pstree > /opt/usr_land/linux_pstree

➜  cat linux_pstree 
Name                 Pid             Uid            
systemd              1                              
[...]
.x-terminal-emul     1503            1001           
..bash               1513            1001           
...ncat              1515            1001           
....sh               119511          1001           
```

En remontant sur le processus parent __bash__, la date est donc:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_pslist -p 1513
Offset             Name                 Pid             PPid            Uid             Gid    DTB                Start Time
------------------ -------------------- --------------- --------------- --------------- ------ ------------------ ----------
0xffff9d72c014dd00 bash                 1513            1503            1001            1001   0x000000004012c000 2020-03-26 23:24:20 UTC+0000
```

> 2020-03-26 23:24:20

### Flag

> FCSC{36280:fd:6663:7363:1000:55cf:b9c6:f41d:cc24:2020-03-26 23:24:20}

---

## Rédaction

>Le document note.docx vient d'être créé avec LibreOffice et enregistré avant le dump de la mémoire. Retrouvez son contenu !
>
> Format du flag : FCSC{xxx}, où xxx est la chaîne qui vous sera indiquée à la lecture du contenu du document.

### TL;DR

Ce challenge consiste à récupérer le contenu d'un fichier "docx". En sachant que c'est un docx, il est possible d'en créer un local, le décompresser et chercher la structure de "document.xml" en mémoire et ainsi trouver le flag.

### État des lieux

Pour ce challenge, on sait qu'on cherche un fichier __docx__ et que l'utilisateur a utilisé __LibreOffice__. La grosse erreur est que je n'ai pas respecté ma méthodologie et je suis parti bille en tête "lol je ve trouvé 1 zip". Après avoir vu que ça ne servait à rien d'y aller comme un débile, il est temps de se poser et de réfléchir.

Pour mes tests, j'ai installé LibreOffice sur ma machine et lancé le "writer" (le Word du pauvre):

```bash
➜  ~ ps aux | grep 'office'
maki     29586  0.0  0.0 176844  5912 ?        Sl   15:59   0:00 /usr/lib/libreoffice/program/oosplash --writer
maki     29601  7.0  1.3 7689752 219172 ?      Sl   15:59   0:03 /usr/lib/libreoffice/program/soffice.bin --writer --splash-pipe=5
```

Visiblement ces processus sont présents aussi dans le dump:

```bash
➜  cat linux_pstree| grep -E "soffice|oosplash"
.oosplash            119599          1001           
soffice.bin          119615          1001           
```

En continuant à piger le fonctionnement de libreOffice, j'ai remarqué qu'un dossier est apparu dans mon __/tmp__:

```bash
➜  /tmp ls -la | grep lu
drwx------  2 maki maki  4096 mai    3 15:59 lu29601r4fv4s.tmp

➜  /tmp ls -la lu29601r4fv4s.tmp 
total 44
drwx------  2 maki maki  4096 mai    3 15:59 .
drwxrwxrwt 26 root root 40960 mai    3 15:59 ..
-rw-------  1 maki maki     0 mai    3 15:59 lu29601r4fv4t.tmp
```

Un fichier vide, ce dossier est surement utilisé pour la sauvegarde automatique des documents ou ce genre de choses. En théorie, si le fichier est sauvegardé, une copie est présente ici.

Voyons si ce dossier est présent dans le dump:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_find_file -L > /opt/usr_land/linuxff.txt

➜  cat linuxff.txt| grep "/tmp/lu"
          923875 0xffff9d72891fe628 /tmp/lu1196159e8v1r.tmp
          923893 0xffff9d72c2cc3bf8 /tmp/lu1196159e8v1r.tmp/lu1196159e8v23.tmp
          923884 0xffff9d72c2cc0d90 /tmp/lu1196159e8v1r.tmp/lu1196159e8v22.tmp
          923876 0xffff9d72891ce628 /tmp/lu1196159e8v1r.tmp/lu1196159e8v1t.tmp

```

### Mécanisme de sauvegarde de LibreOffice

Après avoir créé et sauvegardé un fichier __ODT__, le dossier à bien bougé:

```bash
➜  ~ ls -la /tmp/lu29601r4fv4s.tmp 
total 56
drwx------  2 maki maki  4096 mai    3 16:06 .
drwxrwxrwt 27 root root 40960 mai    3 16:06 ..
-rw-------  1 maki maki     0 mai    3 16:06 lu29601r4fv4u.tmp
-rw-r--r--  1 maki maki  8504 mai    3 16:06 lu29601r4fv4y.tmp
```



![](https://i.imgur.com/xahfjyr.png)



Il s'agit bien de notre fichier __ODT__, c'est le même hash:

```bash
➜  ~ file /tmp/lu29601r4fv4s.tmp/lu29601r4fv4y.tmp
/tmp/lu29601r4fv4s.tmp/lu29601r4fv4y.tmp: OpenDocument Text
➜  ~ md5sum /tmp/lu29601r4fv4s.tmp/lu29601r4fv4y.tmp ~/Documents/ctf/ecsc2020/cestlarentree/Wu/test_odt.odt 
aba6138e65c1dc360920d63772e5c9c0  /tmp/lu29601r4fv4s.tmp/lu29601r4fv4y.tmp
aba6138e65c1dc360920d63772e5c9c0  /home/maki/Documents/ctf/ecsc2020/cestlarentree/Wu/test_odt.odt
```

Bon, par contre, on cherche un __DOCX__ et pas un ODT. On va retenter l'expérience avec un __DOCX__. D'ailleurs, les dossiers temporaires se suppriment lorsque que le processus __soffibe.bin__ se termine.

Après la création et la sauvegarde d'un fichier __DOCX__, un phénomène intéressant apparait sur les fichiers temporaires... Ou plutot n'apparait pas:

```bash
➜  ~ ls -la /tmp/lu30353r4wuxv.tmp/
total 44
drwx------  2 maki maki  4096 mai    3 16:13 .
drwxrwxrwt 26 root root 40960 mai    3 16:13 ..
-rw-------  1 maki maki     0 mai    3 16:12 lu30353r4wuxw.tmp
-rw-------  1 maki maki     0 mai    3 16:13 lu30353r4wuxx.tmp
```

Les fichiers temporaires sont vides, c'est pour cela qu'extraire les fichiers temporaires du dump mémoire ne donnent rien, ce n'est pas le profile volatility ou encore volatility lui même qui ne fonctionne pas correctement, c'est juste que les fichiers existent, mais ne contiennent rien.

### La structure d'un Docx

Si ces fichiers n'existent pas dans les fichiers temporaires, alors la structure du __DOCX__ doit exister en mémoire. Mais avant, comment ça fonctionne un DOCX.
On vient de créer le fichier suivant: `test_docx.docx`



![](https://i.imgur.com/dW84uui.png)



Si on prend les premiers octets:

```bash
➜  Wu file test_docx.docx 
test_docx.docx: Microsoft Word 2007+
➜  Wu hexdump -C test_docx.docx| head -n 5
00000000  50 4b 03 04 14 00 08 08  08 00 b3 71 a3 50 00 00  |PK.........q.P..|
00000010  00 00 00 00 00 00 00 00  00 00 0b 00 00 00 5f 72  |.............._r|
00000020  65 6c 73 2f 2e 72 65 6c  73 ad 92 4d 4b 03 41 0c  |els/.rels..MK.A.|
00000030  86 ef fd 15 43 ee dd 6c  2b 88 c8 ce f6 22 42 6f  |....C..l+...."Bo|
00000040  22 f5 07 84 99 ec ee d0  ce 07 33 69 ad ff de 41  |".........3i...À|
```

![](https://i.imgur.com/brp7R3a.png)

> https://en.wikipedia.org/wiki/List_of_file_signatures

En théorie, il est possible de décompresser un fichier __DOCX__:

```bash
➜  unzip test_docx.docx 

➜  tree .
.
├── [Content_Types].xml
├── docProps
│   ├── app.xml
│   └── core.xml
├── _rels
├── test_docx.docx
└── word
    ├── document.xml
    ├── fontTable.xml
    ├── _rels
    │   └── document.xml.rels
    ├── settings.xml
    └── styles.xml

4 directories, 9 files
```

Il y a donc tout un tas de fichiers __XML__. Avec notre DOCX, on sait que le contenu est "TEST D’UN FICHIER DOCX". Voyons où il apparait:



![](https://i.imgur.com/gsgbF1O.png)



Good, maintenant on sait qu'on cherche le fichier "document.xml" probablement ouvert en mémoire de __soffice.bin__. Pour s'en assurer et éviter de faire un grep comme un débile sur tout le dump, on peut faire une yararule grâce à volatility, en cherchant à partir de `<w:document`:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_yarascan -Y "<w:document"

Task: soffice.bin pid 119615 rule r1 addr 0x5636239689e0
0x5636239689e0  3c 77 3a 64 6f 63 75 6d 65 6e 74 20 78 6d 6c 6e   <w:document.xmln
0x5636239689f0  73 3a 6f 3d 22 75 72 6e 3a 73 63 68 65 6d 61 73   s:o="urn:schemas
0x563623968a00  2d 6d 69 63 72 6f 73 6f 66 74 2d 63 6f 6d 3a 6f   -microsoft-com:o
0x563623968a10  66 66 69 63 65 3a 6f 66 66 69 63 65 22 20 78 6d   ffice:office".xm
0x563623968a20  6c 6e 73 3a 72 3d 22 68 74 74 70 3a 2f 2f 73 63   lns:r="http://sc
0x563623968a30  68 65 6d 61 73 2e 6f 70 65 6e 78 6d 6c 66 6f 72   hemas.openxmlfor
0x563623968a40  6d 61 74 73 2e 6f 72 67 2f 6f 66 66 69 63 65 44   mats.org/officeD
0x563623968a50  6f 63 75 6d 65 6e 74 2f 32 30 30 36 2f 72 65 6c   ocument/2006/rel
0x563623968a60  61 74 69 6f 6e 73 68 69 70 73 22 20 78 6d 6c 6e   ationships".xmln
0x563623968a70  73 3a 76 3d 22 75 72 6e 3a 73 63 68 65 6d 61 73   s:v="urn:schemas
0x563623968a80  2d 6d 69 63 72 6f 73 6f 66 74 2d 63 6f 6d 3a 76   -microsoft-com:v
0x563623968a90  6d 6c 22 20 78 6d 6c 6e 73 3a 77 3d 22 68 74 74   ml".xmlns:w="htt
0x563623968aa0  70 3a 2f 2f 73 63 68 65 6d 61 73 2e 6f 70 65 6e   p://schemas.open
0x563623968ab0  78 6d 6c 66 6f 72 6d 61 74 73 2e 6f 72 67 2f 77   xmlformats.org/w
0x563623968ac0  6f 72 64 70 72 6f 63 65 73 73 69 6e 67 6d 6c 2f   ordprocessingml/
0x563623968ad0  32 30 30 36 2f 6d 61 69 6e 22 20 78 6d 6c 6e 73   2006/main".xmlns
```

Bon et bien on dirait que le __document.xml__ a été retrouvé à l'adresse __0x5636239689e0__.

### Extraction de la zone mémoire de soffice.bin

Il ne reste plus qu'à extraire cette portion de la mémoire. D'abord localiser notre adresse trouvée précédemment:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_proc_maps -p 119615
Offset             Pid      Name                 Start              End                Flags               Pgoff Major  Minor  Inode      File Path
------------------ -------- -------------------- ------------------ ------------------ ------ ------------------ ------ ------ ---------- ---------
[...]
0xffff9d72848edd00   119615 soffice.bin          0x000056361c0cb000 0x0000563623f25000 rw-                   0x0      0      0          0 [heap]
[...]
```

On dirait bien que notre adresse se trouve dans la heap du processus __soffice.bin__. Il ne reste qu'à l'extraire:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_dump_map -p 119615 -s 0x000056361c0cb000 -D /opt/usr_land/wu/
Task       VM Start           VM End                         Length Path
---------- ------------------ ------------------ ------------------ ----
    119615 0x000056361c0cb000 0x0000563623f25000          0x7e5a000 /opt/usr_land/Wu/task.119615.0x56361c0cb000.vma
```

### Chercher la structure du XML

Maintenant, l'outil indispensable:



![](https://i.imgur.com/CNqJpb3.png)



### Flag

> FCSC{PQHJRTSFYH-3467024-LSHRFLDFGA}

---

## Académie - Dans les nuages

> Le poste en cours d'analyse est connecté à un serveur web à l'adresse 10.42.42.132. Le serveur web est protégé par une authentification.
> 
> Retrouvez le nom d'utilisateur et le mot de passe de cette connexion.
> Format du flag : 
> 
> FCSC{utilisateur:mot_de_passe}. Ce flag est sensible à la casse.

### TL;DR

Pour ce challenge, il faut recréer l'infrastructure de l'authentification afin de voir où se trouvent en mémoire les identifiants. Au final, on se retrouve dans une section mémoire de chromium avec les identifiants à la suite encodés en utf16.

### État des lieux

Alors pour ce dernier challenge d'une série très intéressante, c'est vraiment le boss final. Si on devait faire l'analogie de ce challenge avec autre chose, on pourrait dire que c'est comme chercher une aiguille, dans un tas d'autres aiguilles et le tout dans un gros tas de foin.

Donc là, on va essayer de ne pas partir comme un débile, parce que déjà que dans les challenges de forensic c'est facile de se perdre, mais alors ici, c'est presque sûr.

Ce qu'on sait:

* Adresse IP du serveur web : 10.42.42.132;
* Authentification par login / mot de passe (basic auth ? form html ?).

Ce qu'on cherche:

* Des identifiants (user/password).

La première chose qui peut être utile est de faire un yarascan sur l'adresse IP pour essayer d'avoir des pistes de recherches:

```bash
➜  vol.py --plugins=/opt/plug_vol/ -f usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_yarascan -Y "10.42.42.132" | tee /opt/usr_land/yarascan_10.42.42.132.txt

➜  yarascan_10.42.42.132.txt | grep -B 2 Task
[.. L'output est suuuuper verbeuse, donc j'vais trier pour le WU :) ..]
Task: chromium pid 119148 rule r1 addr 0x55f4346630eb
0x55f4346630eb  31 30 2e 34 32 2e 34 32 2e 31 33 32 2f 70 61 6e   10.42.42.132/pan
0x55f4346630fb  65 6c 2f 00 00 00 00 00 00 00 00 00 00 04 00 00   el/.............
[...]
Task: chromium pid 119187 rule r1 addr 0x7fb1cc18dfa7
0x7fb1cc18dfa7  31 30 2e 34 32 2e 34 32 2e 31 33 32 2f 66 61 76   10.42.42.132/fav
0x7fb1cc18dfb7  69 63 6f 6e 2e 69 63 6f 00 10 00 00 00 00 00 00   icon.ico........
[...]
Task: chromium pid 119218 rule r1 addr 0x7efedc023373
0x7efedc023373  31 30 2e 34 32 2e 34 32 2e 31 33 32 2f 70 61 6e   10.42.42.132/pan
0x7efedc023383  65 6c 2f 00 00 00 00 00 00 00 00 00 00 03 00 00   el/.............
--
Task: chromium pid 119233 rule r1 addr 0x55f4febd60eb
0x55f4febd60eb  31 30 2e 34 32 2e 34 32 2e 31 33 32 2f 70 61 6e   10.42.42.132/pan
0x55f4febd60fb  65 6c 2f 00 00 00 00 00 00 00 00 00 00 04 00 00   el/.............
[...]
Task: chromium pid 119274 rule r1 addr 0x55f504bf68b9
0x55f504bf68b9  31 30 2e 34 32 2e 34 32 2e 31 33 32 3a 38 30 2c   10.42.42.132:80,
0x55f504bf68c9  2a 22 3a 7b 22 6c 61 73 74 5f 6d 6f 64 69 66 69   *":{"last_modifi
--
Task: chromium pid 119302 rule r1 addr 0x55f4febd60eb
0x55f4febd60eb  31 30 2e 34 32 2e 34 32 2e 31 33 32 2f 70 61 6e   10.42.42.132/pan
0x55f4febd60fb  65 6c 2f 00 00 00 00 00 00 00 00 00 00 04 00 00   el/.............
[...]
Task: chromium pid 119364 rule r1 addr 0x16a55ae239d3
0x16a55ae239d3  31 30 2e 34 32 2e 34 32 2e 31 33 32 65 02 00 00   10.42.42.132e...
0x16a55ae239e3  00 0e 00 00 00 00 00 00 4a 41 63 63 65 73 73 20   ........JAccess.
--
Task: soffice.bin pid 119615 rule r1 addr 0x7fee90d25b1b
0x7fee90d25b1b  31 30 2e 34 32 2e 34 32 2e 31 33 32 44 43 46 37   10.42.42.132DCF7
0x7fee90d25b2b  41 35 38 39 35 42 46 37 41 37 45 43 31 37 41 39   A5895BF7A7EC17A9
--
Task: chromium pid 119757 rule r1 addr 0x55f4febd60eb
0x55f4febd60eb  31 30 2e 34 32 2e 34 32 2e 31 33 32 2f 70 61 6e   10.42.42.132/pan
0x55f4febd60fb  65 6c 2f 00 00 00 00 00 00 00 00 00 00 04 00 00   el/.............
```

Bon, cet énorme output nous apprend plusieurs choses:

* Probablement que Chromium a été utilisé pour la connexion ;
* Il existe un dossier `/panel/`.

Il existe d'autres outils que __volatility__ pour analyser la mémoire, par exemple __bulk_extractor__ permet notamment de créer un pcap avec les traces laissées dans le dump:

```bash
➜  bulk_extractor -o bulk dmp.mem
```

Le pcap généré nous donne quelques informations supplémentaires:



![](https://i.imgur.com/jPpypD8.png)

![](https://i.imgur.com/P0VtaMy.png)




On répond donc à une question du début sur le type d'authentification, c'est une __Basic authentication__, le realm est __Panel \o/__ et le serveur web utilisé est un __nginx 1.14.2__. Ce qui est un marqueur supplémentaire pour notre recherche.

Avant d'aller plus loin, une connexion via Basic authentication, se fait avec une pop qui demande les identifiants. Ces identifiants sont placés dans un header HTTP __Authorization: Basic base64(username:password)__.

Après quelques recherches non concluantes autour de ça, il est temps de revoir la méthode de travail. Le but va être de recréer une connexion entre un client Chromium et un serveur web.

### Proof of concept

#### Mise en place de l'infrastructure

Pour la partie __client__, on a déjà la VM de test: celle qui nous a permis de faire le profil volatility dans les challenges précédents. Il ne reste qu'à installer une interface graphique (comme xfce4) et __chromium__.

![](https://i.imgur.com/OMEUTfe.png)

Pour le serveur nginx, un bon vieux Docker Debian fera l'affaire en suivant un bon p'tit tuto des familles:

> https://ahmet.im/blog/docker-http-basic-auth/

```bash
➜  ~ docker run --name basic_authent -p4242:4242 -ti debian:latest /bin/bash
➜  (docker) apt-get install nginx apache2-utils
➜  (docker) htpasswd -c /etc/nginx/.htpasswd maki
Password: JeSuisUnGrosZgueg
➜  (docker) sed -i 's/user .*;/user root;/' /etc/nginx/nginx.conf
➜  (docker) tee /etc/nginx/sites-enabled/docker <<EOF 
upstream docker {
  server unix:/var/run/docker.sock;
}

server {
  listen 4242 default_server;
  location / {
    proxy_pass http://docker;
    auth_basic_user_file /etc/nginx/.htpasswd;
    auth_basic "SecretPanelMofo";
  }
}
EOF
➜  (docker) service nginx restart
```

L'infrastructure a l'air d'être prête, on redémarre la machine cliente pour vider la mémoire et on initialise une connexion. Le prompt apparait et on entre 1 fois les mauvais identifiants et 1 fois les bons, pour avoir un statut __401__ comme dans bulk_extractor et un __200__, parce que c'est quand même les bons identifiants qu'on cherche à la base.

Les mauvais identifiants:

> USERPASBON_MARQUEUR : USERPASBON_MARQUEURPASS

Les bons identifiants:

> maki : JeSuisUnGrosZgueg

Maintenant on met la machine en pause et on récupère la mémoire. Utilisant __VMWare__, il est possible de récupérer la mémoire avec le fichier __vmem__. 

#### Analyse du PoC

Comme on a utilisé la même machine virtuelle que pour la création du profil, on s'épargne une étape fastidieuse:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/localtest/test\ ecsc\ deb10-f35aa608.vmem --profile=Linuxecsc_deb540x64 linux_banner
Linux version 5.4.0-4-amd64 (debian-kernel@lists.debian.org) (gcc version 9.2.1 20200203 (Debian 9.2.1-28)) #1 SMP Debian 5.4.19-1 (2020-02-13)
```

Il est temps de partir à la recherche de nos marqueurs, pas classe, mais efficace. Avant d'attendre 5 minutes qu'un Yarascan se termine, on va vérifier que les identifiants apparaissent bien en clair dans la mémoire:

```bash
➜  localtest strings test\ ecsc\ deb10-f35aa608.vmem| grep -E '^maki$|JeSuisUnGrosZgueg' | sort | uniq
maki

➜  strings -e l test\ ecsc\ deb10-f35aa608.vmem| grep -E 'maki|JeSuisUnGrosZgueg' | sort | uniq
mikmaki
[...]
maki
[...]
JeSuisUnGrosZgueg
```

Par contre, les identifiants en mémoire sont encodés sur 16 bytes. Il faut donc adapter la rule en conséquence:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/localtest/test\ ecsc\ deb10-f35aa608.vmem --profile=Linuxecsc_deb540x64 linux_yarascan -Y "{4a ?? 65 ?? 53 ?? 75 ?? 69 ?? 73 ?? 55 ?? 6e ?? 47}"

Task: chromium pid 861 rule r1 addr 0x7f30980e8ea0
0x7f30980e8ea0  4a 00 65 00 53 00 75 00 69 00 73 00 55 00 6e 00   J.e.S.u.i.s.U.n.
0x7f30980e8eb0  47 00 72 00 6f 00 73 00 5a 00 67 00 75 00 65 00   G.r.o.s.Z.g.u.e.
0x7f30980e8ec0  67 00 00 00 00 00 00 00 95 00 00 00 00 00 00 00   g...............
0x7f30980e8ed0  10 af 00 8c 30 7f 00 00 d0 08 00 98 30 7f 00 00   ....0.......0...
0x7f30980e8ee0  a0 dd 06 98 30 7f 00 00 00 00 00 00 00 00 00 00   ....0...........
0x7f30980e8ef0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
0x7f30980e8f00  00 00 00 00 00 00 00 00 00 01 09 98 30 7f 00 00   ............0...
0x7f30980e8f10  40 3a 02 98 30 7f 00 00 48 00 00 00 00 00 00 00   @:..0...H.......
0x7f30980e8f20  88 3a 02 98 30 7f 00 00 50 02 00 00 00 00 00 00   .:..0...P.......
0x7f30980e8f30  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00   ................
0x7f30980e8f40  00 00 00 00 00 00 00 00 e8 49 8a 9d ad c4 e5 bb   .........I......
0x7f30980e8f50  be 3d 6c 31 df 92 20 98 25 00 00 00 00 00 00 00   .=l1....%.......
0x7f30980e8f60  01 00 00 00 00 7f 00 00 00 00 00 00 00 00 00 00   ................
0x7f30980e8f70  69 64 61 74 6f 72 00 00 a5 00 00 00 00 00 00 00   idator..........
0x7f30980e8f80  02 00 01 bb d8 3a ce e3 00 00 00 00 00 00 00 00   .....:..........
0x7f30980e8f90  77 77 77 2e 67 73 74 61 74 69 63 2e 63 6f 6d 00   www.gstatic.com.
```

Sans trop de surprises, le mot de passe est présent dans la mémoire de chromium. Mais où exactement:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/localtest/test\ ecsc\ deb10-f35aa608.vmem --profile=Linuxecsc_deb540x64 linux_proc_maps -p 861

Offset             Pid      Name                 Start              End                Flags               Pgoff Major  Minor  Inode      File Path
------------------ -------- -------------------- ------------------ ------------------ ------ ------------------ ------ ------ ---------- ---------
[...]
0xffff99baf6a3cd80      861 chromium             0x000055943fbb1000 0x000055943fc9f000 rw-                   0x0      0      0          0 [heap]
0xffff99baf6a3cd80      861 chromium             0x00007f3084000000 0x00007f308416c000 rw-                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f308416c000 0x00007f3088000000 ---                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f3088000000 0x00007f3088021000 rw-                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f3088021000 0x00007f308c000000 ---                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f308c000000 0x00007f308c062000 rw-                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f308c062000 0x00007f3090000000 ---                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f3090000000 0x00007f3090021000 rw-                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f3090021000 0x00007f3094000000 ---                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f3094000000 0x00007f30941ba000 rw-                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f30941ba000 0x00007f3098000000 ---                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f3098000000 0x00007f3098118000 rw-                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f3098118000 0x00007f309c000000 ---                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f309c000000 0x00007f309c021000 rw-                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f309c021000 0x00007f30a0000000 ---                   0x0      0      0          0 
0xffff99baf6a3cd80      861 chromium             0x00007f30a16d5000 0x00007f30a16de000 r--                   0x0      8      1    1063603 /
[...]
```

Bon, on se retrouve dans une zone mémoire perdue au milieu de tout ce bazar, entre la heap et la mémoire allouée par des fichiers. Il suffit de l'extraire pour trouver ce qu'il y a dedans:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/localtest/test\ ecsc\ deb10-f35aa608.vmem --profile=Linuxecsc_deb540x64 linux_dump_map -s 0x00007f3098000000 -p 861 -D /opt/usr_land/wu/

Task       VM Start           VM End                         Length Path
---------- ------------------ ------------------ ------------------ ----
       861 0x00007f3098000000 0x00007f3098118000           0x118000 /opt/usr_land/Wu/task.861.0x7f3098000000.vma
```

Et dedans on retrouve bien nos identifiants:

```bash
➜  strings -e l task.861.0x7f3098000000.vma
maki
maki
f us
kN6U
_MAR
JeSuisUnGrosZgueg
4N6U
	dgknorst}
46]_cfilv{
').01238[hjmuy
```

A première vue, il n'y a pas de marqueurs efficaces pour localiser les identifiants dans le dump du challenge. Par contre, il peut être intéressant de regarder tous ça d'un point de vu hexa:

```bash
000e8e80  e0 c1 0d 98 30 7f 00 00  60 86 0e 98 30 7f 00 00  |....0...`...0...|
000e8e90  70 6e 0e 98 30 7f 00 00  35 00 00 00 00 00 00 00  |pn..0...5.......|
000e8ea0  4a 00 65 00 53 00 75 00  69 00 73 00 55 00 6e 00  |J.e.S.u.i.s.U.n.|
000e8eb0  47 00 72 00 6f 00 73 00  5a 00 67 00 75 00 65 00  |G.r.o.s.Z.g.u.e.|
000e8ec0  67 00 00 00 00 00 00 00  95 00 00 00 00 00 00 00  |g...............|
000e8ed0  10 af 00 8c 30 7f 00 00  d0 08 00 98 30 7f 00 00  |....0.......0...|
000e8ee0  a0 dd 06 98 30 7f 00 00  00 00 00 00 00 00 00 00  |....0...........|
000e8ef0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
000e8f00  00 00 00 00 00 00 00 00  00 01 09 98 30 7f 00 00  |............0...|
000e8f10  40 3a 02 98 30 7f 00 00  48 00 00 00 00 00 00 00  |@:..0...H.......|
```

Quelque chose qui pourrait être intéressant est de chercher l'adresse de la chaine de caractère: `0x7f30980e8ea0`. Et là, un alignement de planète:



![](https://i.imgur.com/YshCwgF.png)



* Vert: Adresse de la chaine pour le "realm"
* Le second pointeur après la chaine du realm (rouge): Adresse pour le nom d'utilisateur
* Le troisième pointeur après la chaine du realm (jaune): Adresse pour le mot de passe

C ce n'est pas si étonnant, dans les sources de Chromium il y a ceci:



![](https://i.imgur.com/P2JFP1e.png)



> https://chromium.googlesource.com/chromium/src/net/+/master/http/http_auth_controller.cc

On commence à y voir plus clair dans cette histoire, mais faut-il encore trouver dans quel processus chercher:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/localtest/test\ ecsc\ deb10-f35aa608.vmem --profile=Linuxecsc_deb540x64 linux_pstree | grep chromium

.....chromium        821             1000           
.......chromium      836             1000           
........chromium     838             1000           
.........chromium    981             1000           
.........chromium    993             1000           
chromium             859             1000           
.chromium            940             1000           
chromium             861             1000           
```

Pour savoir correctement d'où vient notre processus, même s'il n'a pas de parent, il a été lancé par quelque chose. Les plugins `pslist` et `psscan` vont répondre à ces questions:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/test\ ecsc\ deb10-f35aa608.vmem --profile=Linuxecsc_deb540x64 linux_pslist | grep 861

Offset             Name                 Pid             PPid            Uid             Gid    DTB                Start Time
------------------ -------------------- --------------- --------------- --------------- ------ ------------------ ----------
0xffff99baf6a3cd80 chromium             861             852             1000            1000   0x000000007692e000 2020-04-28 19:19:35 UTC+0000
```

Le processus __852__ n'est autre que `ThreadPoolSingl`:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/test\ ecsc\ deb10-f35aa608.vmem --profile=Linuxecsc_deb540x64 linux_psscan | grep 852

Offset             Name                 Pid             PPid            Uid             Gid    DTB                Start Time
------------------ -------------------- --------------- --------------- --------------- ------ ------------------ ----------
0x00000000768e8000 ThreadPoolSingl      852             -               -               -      ------------------ -

➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/test\ ecsc\ deb10-f35aa608.vmem --profile=Linuxecsc_deb540x64 linux_pslist | grep 852

Offset             Name                 Pid             PPid            Uid             Gid    DTB                Start Time
------------------ -------------------- --------------- --------------- --------------- ------ ------------------ ----------
0xffff99baf6930000 chromium             859             852             1000            1000   0x0000000076af0000 2020-04-28 19:19:35 UTC+0000
0xffff99baf6a3cd80 chromium             861             852             1000            1000   0x000000007692e000 2020-04-28 19:19:35 UTC+0000
```

Le processus que l'on cherche est donc le second lancé par __ThreadPoolSingl__. 

### À la recherche des identifiants perdus

Le plugin `pstree` de volatility a déjà été exécuté dans les challenges précédents:

```bash
➜  cat linux_pstree | grep chromium
.....chromium        119148          1001           
.......chromium      119163          1001           
........chromium     119165          1001           
.........chromium    119218          1001           
.........chromium    119233          1001           
.........chromium    119274          1001           
.........chromium    119302          1001           
.........chromium    119364          1001           
.........chromium    119380          1001           
.........chromium    119394          1001           
.........chromium    119735          1001           
.........chromium    119757          1001           
chromium             119184          1001           
chromium             119187          1001           
```

En suivant la logique précédente, le processus qui nous intéresse est le __119187__:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_pslist | grep chromium
Volatility Foundation Volatility Framework 2.6.1
0xffff9d72c1848f80 chromium             119148          1378            1001            1001   0x0000000013768000 2020-03-26 23:27:39 UTC+0000
0xffff9d72940e8000 chromium             119163          119162          1001            1001   0x0000000001f0c000 2020-03-26 23:27:39 UTC+0000
0xffff9d72940eae80 chromium             119165          119163          1001            1001   0x000000004213c000 2020-03-26 23:27:39 UTC+0000
0xffff9d72bd57be00 chromium             119184          119180          1001            1001   0x0000000011360000 2020-03-26 23:27:39 UTC+0000
0xffff9d72bd57cd80 chromium             119187          119180          1001            1001   0x0000000011296000 2020-03-26 23:27:39 UTC+0000
0xffff9d729ce6dd00 chromium             119218          119165          1001            1001   0x00000000107da000 2020-03-26 23:27:40 UTC+0000
0xffff9d7283045d00 chromium             119233          119165          1001            1001   0x0000000003058000 2020-03-26 23:27:40 UTC+0000
0xffff9d72824e3e00 chromium             119274          119165          1001            1001   0x00000000025dc000 2020-03-26 23:27:46 UTC+0000
0xffff9d7282481f00 chromium             119302          119165          1001            1001   0x0000000004ece000 2020-03-26 23:27:46 UTC+0000
0xffff9d7284f86c80 chromium             119364          119165          1001            1001   0x00000000052fe000 2020-03-26 23:27:50 UTC+0000
0xffff9d7284f82e80 chromium             119380          119165          1001            1001   0x000000001a206000 2020-03-26 23:28:02 UTC+0000
0xffff9d729a269f00 chromium             119394          119165          1001            1001   0x000000001a29e000 2020-03-26 23:28:02 UTC+0000
0xffff9d72940e4d80 chromium             119735          119165          1001            1001   0x0000000011234000 2020-03-26 23:39:05 UTC+0000
0xffff9d72c1848000 chromium             119757          119165          1001            1001   0x0000000002e62000 2020-03-26 23:39:08 UTC+0000
```

On peut exclure tous les processus dont le père est __119165__, car il s'agit du broker de Chromium, mais comme vu précédemment le processus que l'on cherche ne dépend pas du broker. 

Le broker de Chromium est le processus principal, en l'occurrence: __119165__. Ensuite, chaque processus fils seront des sites, des iframes, etc... Ces processus seront sandboxés par Chromium:



![](https://i.imgur.com/RMMZ9J7.png)



> https://chromium.googlesource.com/chromium/src/+/master/docs/design/sandbox.md

Les processus __119218, 119233, 119274, 119302, 119364, 119380, 119394, 119735, 119757__, ne nous intéressent donc pas dans ce cas.

Pour coller au PoC fait dans le chapitre précédent, il faut trouver deux processus __chromium__ avec le même parent. Le processus __119180__ doit donc être _ThreadPoolSingl_.

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_pslist | grep 119180

Offset             Name                 Pid             PPid            Uid             Gid    DTB                Start Time
------------------ -------------------- --------------- --------------- --------------- ------ ------------------ ----------
0xffff9d72bd57be00 chromium             119184          119180          1001            1001   0x0000000011360000 2020-03-26 23:27:39 UTC+0000
0xffff9d72bd57cd80 chromium             119187          119180          1001            1001   0x0000000011296000 2020-03-26 23:27:39 UTC+0000
```

A partir de ce point, il existe au moins deux méthodes pour trouver les identifiants. On sait qu'il faut creuser dans le processus __119187__, car comme dans le PoC, il s'agit du __second__ processus exécuté.

### Solution 1 - Trouver la structure en mémoire

En connaissant la structure en mémoire, il est possible de commencer à chercher en partant de la chaine du "realm":



![](https://i.imgur.com/N1u2DQu.png)



On peut commencer par chercher le realm `Panel-\o/`:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_yarascan -Y "Panel-" -p 119187

[...]
Task: chromium pid 119187 rule r1 addr 0x7fb1cc163630
0x7fb1cc163630  50 61 6e 65 6c 2d 6f 2f 00 64 6a 37 33 61 65 77   Panel-o/.dj73aew
0x7fb1cc163640  00 00 00 00 33 69 64 74 70 0a 0e cc b1 7f 00 00   ....3idtp.......
0x7fb1cc163650  17 00 00 00 00 00 00 00 1e 00 00 00 00 00 00 00   ................
0x7fb1cc163660  6a 4d bb 5f 7a 34 0a 9f e0 a0 0e cc b1 7f 00 00   jM._z4..........
0x7fb1cc163670  09 00 00 00 00 00 00 00 0e 00 00 00 00 00 00 00   ................
0x7fb1cc163680  34 19 dd 3c f7 a7 5d b8 90 5f 17 cc b1 7f 00 00   4..<..].._......
0x7fb1cc163690  08 00 00 00 00 00 00 00 0e 00 00 00 00 00 00 00   ................
0x7fb1cc1636a0  88 bb 94 35 52 22 36 60 03 00 00 00 d1 d0 c5 85   ...5R"6`........
0x7fb1cc1636b0  90 73 15 cc b1 7f 00 00 90 73 15 cc b1 7f 00 00   .s.......s......
0x7fb1cc1636c0  01 00 00 00 00 00 00 00 ac 61 8d 0e 00 00 00 00   .........a......
0x7fb1cc1636d0  0d ff 2b 37 00 00 00 00 52 7b 0c 9d 60 00 2f 00   ..+7....R{..`./.
0x7fb1cc1636e0  a0 03 00 00 00 00 00 00 25 02 00 00 00 00 00 00   ........%.......
0x7fb1cc1636f0  10 93 18 cc b1 7f 00 00 e0 93 1d cc b1 7f 00 00   ................
0x7fb1cc163700  90 8d 15 cc b1 7f 00 00 18 00 00 00 00 00 00 00   ................
0x7fb1cc163710  18 00 00 00 00 00 00 00 4d 31 6a 9e 31 0a a1 0a   ........M1j.1...
0x7fb1cc163720  bb 01 7f a4 fc a1 dc f1 00 00 0b 74 24 db 8d cc   ...........t$...
```

Cette yararule ne ressort que 4 résultats, il est donc aisé de trouver le bloc qui nous intéresse. En suivant la logique définie précédemment:



![](https://i.imgur.com/WlY72cw.png)



* Second pointeur (rouge), adresse de username: `0x7fb1cc0ea0e0` 
* Troisième pointeur (vert) adresse de password: `0x7fb1cc175f90` 

Il n'y a plus qu'à extraire la __Virtual Memory Area__ (VMA) et regarder ce qu'il y à ces adresses:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_proc_maps -p 119187

[...]
0xffff9d72bd57cd80   119187 chromium             0x00007fb1cc000000 0x00007fb1cc4e2000 rw-                   0x0      0      0          0 
[...]

➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_dump_map -s 0x00007fb1cc000000 -p 119187 -D /opt/usr_land/wu/119187/
```

Le nom d'utilisateur:

```bash
➜  hexdump -C -s 0x0ea0e0 ../187/Wu/task.119187.0x7fb1cc000000.vma | head -n 4
000ea0e0  41 00 64 00 6d 00 69 00  6e 00 33 00 4b 00 7a 00  |A.d.m.i.n.3.K.z.|
000ea0f0  37 00 00 00 00 00 00 00  a0 26 18 cc b1 7f 00 00  |7........&......|
000ea100  00 00 00 00 00 00 00 00  25 00 00 00 00 00 00 00  |........%.......|
000ea110  60 ab 13 cc b1 7f 00 00  20 06 1e cc b1 7f 00 00  |`....... .......|
```

> Admin3Kz7

Le mot de passe:

```bash
➜  hexdump -C -s 0x175f90 ../187/Wu/task.119187.0x7fb1cc000000.vma | head -n 4
00175f90  35 00 73 00 64 00 74 00  59 00 68 00 36 00 38 00  |5.s.d.t.Y.h.6.8.|
00175fa0  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00175fb0  11 00 00 00 00 00 00 00  75 00 00 00 00 00 00 00  |........u.......|
00175fc0  02 00 00 00 b1 7f 00 00  c0 93 02 cc b1 7f 00 00  |................|
```

> 5sdtYh68

### Solution 2 - La débrousailleuse

Cette solution a le mérite d'etre plus rapide que la première. La première chose à faire est d'extraires tous les VMA du processus:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_dump_map -p 119187 -D /opt/usr_land/119187/all_vma/

➜  strings --print-file-name 119187/all_vma/* | grep 'http://10.42.42.132'        
task.119187.0x7fb1cc000000.vma: http://10.42.42.132/
task.119187.0x7fb1cc000000.vma: http://10.42.42.132/
task.119187.0x7fb1cc000000.vma: http://10.42.42.132/favicon.ico
task.119187.0x7fb1cc000000.vma: http://10.42.42.132/
task.119187.0x7fb1cc000000.vma: http://10.42.42.132/panel/
```

Au final, une seule portion ressort, il suffit de faire un strings UTF-16 dessus pour voir la magie opérer:

```bash
➜  strings -e l -n 6 task.119187.0x7fb1cc000000.vma
Admin3Kz7
5sdtYh68
 !&(/01=CEIORS_bcfghmvwxy}
-78;?ABDFGLMNPTUVW[]jkz{|
+234569HJKQXYZq~
```

### Flag

> FCSC{Admin3Kz7:5sdtYh68}

### Bonus 1 - Keepass

En inspectant les processus en cours d'exécution, on remarque un __Keepass__:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_psaux

Pid    Uid    Gid    Arguments
119514 1001   1001   /usr/bin/cli /usr/lib/keepass2/KeePass.exe
```

Ne connaissant pas ce processus __cli__, on demande à Google:

```bash
$ which cli
/usr/bin/cli
$ which mono
/usr/bin/mono
$ ls -l /usr/bin/mono
-rwxr-xr-x 1 root root 3054984 Jul 24  2012 /usr/bin/mono
$ ls -l /usr/bin/cli
lrwxrwxrwx 1 root root 21 Apr 17 16:16 /usr/bin/cli -> /etc/alternatives/cli
$ ls -l /etc/alternatives/cli
lrwxrwxrwx 1 root root 13 Apr 17 16:16 /etc/alternatives/cli -> /usr/bin/mono
```

> https://stackoverflow.com/questions/16426041/what-is-difference-between-cli-and-mono-in-unix
> https://fr.wikipedia.org/wiki/Common_Language_Infrastructure

#### État des lieux

Donc on peut dire que __cli__ == __mono__. Mono est un utilitaire permettant d'exécuter le framework .NET de Microsoft sur Linux. Le fait d'avoir un "Keepass.exe" prend son sens.

On sait que les extensions des containers Keepass ont l'extension: `.kdbx`

```bash
➜  cestlarentree cat str_* | grep -i kdbx | sort | uniq
/home/Lesage/Documents/pass/mdp.kdbx
kdbx
KDBX_DetailsFull_HTML.xsl
KDBX_DetailsLight_HTML.xsl
KDBX_Tabular_HTML.xsl
mdp.kdbx
mdp.kdbx - KeePass
					<Path>../../../home/Lesage/Documents/pass/mdp.kdbx</Path>
			<Path>../../../home/Lesage/Documents/pass/mdp.kdbx</Path>
```

> /home/Lesage/Documents/pass/mdp.kdbx

Les base de données Keepass semblent avoir une signature:

```bash
➜  file ./test.kdbx 
test.kdbx: Keepass password database 2.x KDBX

➜  hexdump -C ./test.kdbx | head -n 2 
00000000  03 d9 a2 9a 67 fb 4b b5  01 00 03 00 02 10 00 31  |....g.K........1|
00000010  c1 f2 e6 bf 71 43 50 be  58 05 21 6a fc 5a ff 03  |....qCP.X.!j.Z..|
```



![](https://i.imgur.com/JiKavdx.png)



> https://keepass.info/help/base/repair.html

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_yarascan -Y "{03 D9 A2 9A 67 FB 4B B5}"
# Y a wallou
```

Le yarascan n'est pas très concluant. Faisons un test en local pour voir si le contenu du container se trouve dans la mémoire.

#### Test en local

Une fois n'est pas coutume, on va procéder de la même manière que dans les challenges précédents en installant un Keepass2 sur notre VM de test.

```bash
$ apt install keepass2
```



![](https://i.imgur.com/YNFbvBN.png)



* Master key : JeSuisUnSuperPassword
* Nom de la bdd : TestFCSC_Database
* Entrée utilisateur : TESTENTREEKEEPASS_maki
* Entrée mot de passe : TESTENTREEKEEPASS_PASSWORD

La première chose à faire est donc d'essayer de retrouver la Master key:

```bash
➜  strings test\ ecsc\ deb10-f35aa608.vmem > str
➜  strings -e l test\ ecsc\ deb10-f35aa608.vmem > str_el
➜  cat str* | grep "JeSuisUnSuper"
```

Ca ne donne rien. Avant de se décourager, on peut essayer pour les entrées:

```bash
➜  cat str* | grep -E "TESTENTREEKEEPASS_maki|TESTENTREEKEEPASS_PASSWORD" | uniq | sort
[...]
TESTENTREEKEEPASS_maki
TESTENTREEKEEPASS_maki
TESTENTREEKEEPASS_PASSWORD
TESTENTREEKEEPASS_PASSWORD
 Title: TESTENTREEKEEPASS_maki, Password: ********, Creation Time: 05/05/2020 17:21:55, Last Modification Time: 
[...]
```

Bon, pas besoin de la master key s'il est possible de récupérer les informations. Alors, maintenant il faut réussir à localiser précisément où sont les entrées:

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/test\ ecsc\ deb10-f35aa608.vmem --profile=Linuxecsc_deb540x64 linux_dump_map -p 814 -D /opt/usr_land/vma/

➜  strings --print-file-name -e l * | grep -E "TESTENTREEKEEPASS_maki|TESTENTREEKEEPASS_PASSWORD" | cut -d':' -f1  | uniq
task.814.0x7f3c83800000.vma

➜  strings -e l task.814.0x7f3c83800000.vma | grep -E "TESTENTREEKEEPASS_maki|TESTENTREEKEEPASS_PASSWORD" | sort | uniq
[...]
MeGYxEBVnEsTrGroup:HPRIZOIDrzIXZ Database, MeGYxEBVnEsTrTitle:HPRIZOIDrzIXZ TESTENTREEKEEPASS_maki, MeGYxEBVnEsTrPassword:HPRIZOIDrzIXZ ********, MeGYxEBVnEsTrCreation Time:HPRIZOIDrzIXZ 05/05/2020 17:21:55, MeGYxEBVnEsTrLast Modification Time:HPRIZOIDrzIXZ 05/05/2020 17:22:56
RIZOIDrzIXZ TESTENTREEKEEPASS_maki, MeGYxEBVnEsTrPassword:HPRIZO
s16 MeGYxEBVnEsTrGroup:HPRIZOIDrzIXZ Database, MeGYxEBVnEsTrTitle:HPRIZOIDrzIXZ TESTENTREEKEEPASS_maki, MeGYxEBVnEsTrPassword:HP
TESTENTREEKEEPASS_maki
TESTENTREEKEEPASS_PASSWORD
 Title: TESTENTREEKEEPASS_maki, Password: ********, Creation Time: 05/05/2020 17:21:55, Last Modification Time: 

➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/test\ ecsc\ deb10-f35aa608.vmem --profile=Linuxecsc_deb540x64 linux_proc_maps -p 814
Volatility Foundation Volatility Framework 2.6.1
Offset             Pid      Name                 Start              End                Flags               Pgoff Major  Minor  Inode      File Path
------------------ -------- -------------------- ------------------ ------------------ ------ ------------------ ------ ------ ---------- ---------
[...]
0xffff98a5b9708f80      814 cli                  0x00007f3c83800000 0x00007f3c84401000 rw-                   0x0      0      0          0 
[...]
```

Il n'y a qu'une zone mémoire contenant les informations que l'on cherche. Maintenant, on sait que le nom d'utilisateur est relativement simple à trouver, car il faut partir d'une structure. En théorie, il y a juste à retrouver cette structure. Quant au mot de passe, c'est un peu différent, il faut réussir à voir ce qu'il y a autour et peut-être déterminer des marqueurs. 

```bash
➜  strings -e l --print-file-name * | grep -C 2 "TESTENTREEKEEPASS_PASSWORD"
[...]
--
task.814.0x7f3c83800000.vma: SWFClass0.System.Windows.Forms.CheckBox
task.814.0x7f3c83800000.vma: TESTENTREEKEEPAS
task.814.0x7f3c83800000.vma: TESTENTREEKEEPASS_PASSWORD
task.814.0x7f3c83800000.vma: S_PASSWORD
task.814.0x7f3c83800000.vma: TESTENTREEKEEPASS_PASSWORD
task.814.0x7f3c83800000.vma: KeePass.UI.SecureTextBoxEx
task.814.0x7f3c83800000.vma: SWFClass0.KeePass.UI.SecureTextBoxEx
task.814.0x7f3c83800000.vma: TESTENTREEKEEPASS_PASSWORD
task.814.0x7f3c83800000.vma: KeePass.UI.SecureTextBoxEx
task.814.0x7f3c83800000.vma: SWFClass0.KeePass.UI.SecureTextBoxEx
--
task.814.0x7f3c83800000.vma: SWFClass0.System.Windows.Forms.Button
task.814.0x7f3c83800000.vma: TESTENTREEKEEPAS
task.814.0x7f3c83800000.vma: TESTENTREEKEEPASS_PASSWORD
task.814.0x7f3c83800000.vma: S_PASSWORD
task.814.0x7f3c83800000.vma: TESTENTREEKEEPASS_PASSWORD
task.814.0x7f3c83800000.vma: KeePass.UI.SecureTextBoxEx
task.814.0x7f3c83800000.vma: SWFClass0.KeePass.UI.SecureTextBoxEx
task.814.0x7f3c83800000.vma: TESTENTREEKEEPASS_PASSWORD
task.814.0x7f3c83800000.vma: System.Windows.Forms.TabControl
```

On peut voir que le mot de passe apparait plusieurs fois autour d'une méthode ou d'une classe: __SWFClass0.KeePass.UI.SecureTextBoxEx__. Ce qui est un nom plutôt évocateur.

#### Trouver le contenu du keepass

Maintenant qu'on a trouvé un certain nombre d'informations, il est temps de tester sur le Keepass du challenge. Donc même chose que pour le test en local, dump des zones mémoires du keepass et la fameuse technique de "la débroussailleuse":

```bash
➜  python volatility/vol.py --plugins=/opt/plug_vol/ -f /opt/usr_land/dmp.mem --profile=Linuxecsc_deb540x64 linux_dump_map -p 119514 -D /opt/usr_land/keepass

➜  strings --print-file-name -e l * | grep "Title:"  
task.119514.0x7f3f46338000.vma: IyfTEvOFrjanWGroup:IdmZfHKZcQfnm 453, IyfTEvOFrjanWTitle:IdmZfHKZcQfnm 453, IyfTEvOFrjanWUser Name:IdmZfHKZcQfnm LePasSage, IyfTEvOFrjanWPassword:IdmZfHKZcQfnm ********, IyfTEvOFrjanWCreation Time:IdmZfHKZcQfnm 3/26/2020 2:36:34 PM, IyfTEvOFrjanWLast Modification Time:IdmZfHKZcQfnm 3/26/2020 2:39:59 PM
task.119514.0x7f3f46338000.vma: IyfTEvOFrjanWGroup:IdmZfHKZcQfnm 453, IyfTEvOFrjanWTitle:IdmZfHKZcQfnm 453, IyfTEvOFrjanWUser Name:IdmZfHKZcQfnm LePasSage, IyfTEvOFrjanWPassword:IdmZfHKZcQfnm ********, IyfTEvOFrjanWCreation Time:IdmZfHKZcQfnm 3/26/2020 2:36:34 PM, IyfTEvOFrjanWLast Modification Time:IdmZfHKZcQfnm 3/26/2020 2:39:59 PM
task.119514.0x7f3f46338000.vma: IyfTEvOFrjanWGroup:IdmZfHKZcQfnm 453, IyfTEvOFrjanWTitle:IdmZfHKZcQfnm 453, IyfTEvOFrjanWUser Name:IdmZfHKZcQfnm LePasSage, IyfTEvOFrjanWPassword:IdmZfHKZcQfnm ********, IyfTEvOFrjanWCreation Time:IdmZfHKZcQfnm 3/26/2020 2:36:34 PM, IyfTEvOFrjanWLast Modification Time:IdmZfHKZcQfnm 3/26/2020 2:39:59 PM
task.119514.0x7f3f51000000.vma: {\*\generator Mono RichTextBox;}\pard\f0\fs22 \ul Group:IdmZfHKZcQfnm 453, \ul Title:IdmZfHKZcQfnm 453, \ul User Name:IdmZfHKZcQfnm LePasSage, \ul Password:IdmZfHKZcQfnm ********, \ul Creation Time:IdmZfHKZcQfnm 3/26/2020 2:36:34 PM, \ul Last Modification Time:IdmZfHKZcQfnm 3/26/2020 2:39:59 PM\par
```

Si on nettoie un peu la sortie, on peut récupérer un nom d'utilisateur:

```
Group: 453, Title: 453, User Name: LePasSage, Password: ********, Creation Time: 3/26/2020 2:36:34 PM, Last Modification Time: 3/26/2020 2:39:59 PM
```

> LePasSage

```bash
➜  keepass strings --print-file-name -e l * | grep "LePasSage"
task.119514.0x7f3f46338000.vma: IyfTEvOFrjanWGroup:IdmZfHKZcQfnm 453, IyfTEvOFrjanWTitle:IdmZfHKZcQfnm 453, IyfTEvOFrjanWUser Name:IdmZfHKZcQfnm LePasSage, IyfTEvOFrjanWPassword:IdmZfHKZcQfnm ********, IyfTEvOFrjanWCreation Time:IdmZfHKZcQfnm 3/26/2020 2:36:34 PM, IyfTEvOFrjanWLast Modification Time:IdmZfHKZcQfnm 3/26/2020 2:39:59 PM
task.119514.0x7f3f46338000.vma: IyfTEvOFrjanWGroup:IdmZfHKZcQfnm 453, IyfTEvOFrjanWTitle:IdmZfHKZcQfnm 453, IyfTEvOFrjanWUser Name:IdmZfHKZcQfnm LePasSage, IyfTEvOFrjanWPassword:IdmZfHKZcQfnm ********, IyfTEvOFrjanWCreation Time:IdmZfHKZcQfnm 3/26/2020 2:36:34 PM, IyfTEvOFrjanWLast Modification Time:IdmZfHKZcQfnm 3/26/2020 2:39:59 PM
task.119514.0x7f3f46338000.vma: IyfTEvOFrjanWGroup:IdmZfHKZcQfnm 453, IyfTEvOFrjanWTitle:IdmZfHKZcQfnm 453, IyfTEvOFrjanWUser Name:IdmZfHKZcQfnm LePasSage, IyfTEvOFrjanWPassword:IdmZfHKZcQfnm ********, IyfTEvOFrjanWCreation Time:IdmZfHKZcQfnm 3/26/2020 2:36:34 PM, IyfTEvOFrjanWLast Modification Time:IdmZfHKZcQfnm 3/26/2020 2:39:59 PM
task.119514.0x7f3f47244000.vma: LePasSage
task.119514.0x7f3f51000000.vma: {\*\generator Mono RichTextBox;}\pard\f0\fs22 \ul Group:IdmZfHKZcQfnm 453, \ul Title:IdmZfHKZcQfnm 453, \ul User Name:IdmZfHKZcQfnm LePasSage, \ul Password:IdmZfHKZcQfnm ********, \ul Creation Time:IdmZfHKZcQfnm 3/26/2020 2:36:34 PM, \ul Last Modification Time:IdmZfHKZcQfnm 3/26/2020 2:39:59 PM\par
```

Dans le cadre du challenge, il n'y a pas qu'un seul espace mémoire contenant le nom d'utilisateur, il y a en 3: __task.119514.0x7f3f46338000.vma, task.119514.0x7f3f47244000.vma, task.119514.0x7f3f51000000.vma__

Il ne reste qu'à chercher notre marqueur parmi ces trois zones mémoires:

```bash
➜  strings -e l  task.119514.0x7f3f46338000.vma task.119514.0x7f3f47244000.vma task.119514.0x7f3f51000000.vma | grep -C 1 "SWFClass0.KeePass.UI.SecureTextBoxEx"
[...]
--
KeePass.UI.SecureTextBoxEx
SWFClass0.KeePass.UI.SecureTextBoxEx
_Rx9PjLIO8Z
--
KeePass.UI.SecureTextBoxEx
SWFClass0.KeePass.UI.SecureTextBoxEx
_Rx9PjLIO8Z
--
KeePass.UI.SecureTextBoxEx
SWFClass0.KeePass.UI.SecureTextBoxEx
_Rx9PjLIO8Z
```

Je pense donc le mot de passe est: 

> \_Rx9PjLIO8Z

Cependant les identifiants __LePasSage:\_Rx9PjLIO8Z__ ne sont pas ceux recherchés pour le challenge, mais peut être qu'un jour ça pourra servir.
