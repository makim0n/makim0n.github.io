---
weight: 4
title: "[French FCSC] - Find Me"
date: 2020-06-27T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["anssi", "fcsc", "luks", "forensic", "testdisk"]
categories: ["Writeups"]

lightgallery: true
---

>Vous avez accès à un fichier find_me qui semble renfermer un secret bien gardé, qui n'existe peut-être même plus. Retrouvez son contenu !

## TL;DR

Cette épreuve consiste à analyser une partition EXT4. Dans cette partition, on trouve une autre partition chiffrée (LUKS). La clé de chiffrement de ce volume a été supprimée, mais à l'aide de testdisk, il est possible de la récupérer. Ainsi, déchiffrer le volume, le monter et accéder au flag.

## État des lieux

On commence cette épreuve avec un volume EXT4, système de fichier classique. Pour ce challenge, il existe au moins deux méthodes pour retrouver la clé du volume chiffré.

```bash
➜  file find_me 
find_me: Linux rev 1.0 ext4 filesystem data, UUID=9c0d2dc5-184c-496a-ba8e-477309e521d9, volume name "find_me" (needs journal recovery) (extents) (64bit) (large files) (huge files)
```

## Solution 1 - Testdisk

Le premier réflexe en tant que CTFer est de le passer dans `testdisk`, d'autant plus que le volume est plutôt léger.

```bash
➜  sudo testdisk find_me
# Entrer -> None -> Advanced -> List
```

<center>

![](https://i.imgur.com/QrHBHMJ.png)

</center>

On extrait tous les fichiers et un simple `cat` de tous les fichiers `part**` montre une chaine de caractère rigolote:

```bash
➜  sudo cat part*
TWYtOVkyb01OWm5IWEtzak04cThuUlRUOHgzVWRZ
➜  sudo cat part* | base64 -d
Mf-9Y2oMNZnHXKsjM8q8nRTT8x3UdY
```

Il semblerait que la clé soit:

> Mf-9Y2oMNZnHXKsjM8q8nRTT8x3UdY

## Solution 2 - The Sleuth Kit

Cette solution n'utilise pas l'outil `testdisk`, mais la suite __The Sleuth Kit__ (TSK). Cette suite d'outils est très utilisée dans les investigations numériques. Dans les faits, les outils les plus utilisés de cette suite doivent être:

* mmls : Permets de lister les partitions d'un volume;
* fls : Liste les fichiers et répertoires d'un volume;
* istat : Affiche les méta données d'une structure, comme un inode;
* icat : Affiche le contenu d'un inode.

Avec la commande `fls` il est possible de lister aussi les fichiers supprimés:

```bash
➜  mkdir a
➜  fls -a -r -m a find_me 
0|a/.|2|d/drwxr-xr-x|0|0|1024|1585770980|1585770852|1585770852|0
0|a/..|2|d/drwxr-xr-x|0|0|1024|1585770980|1585770852|1585770852|0
0|a/lost+found|11|d/drwx------|0|0|12288|1585770852|1585770844|1585770844|0
0|a/lost+found/.|11|d/drwx------|0|0|12288|1585770852|1585770844|1585770844|0
0|a/lost+found/..|2|d/drwxr-xr-x|0|0|1024|1585770980|1585770852|1585770852|0
0|a/unlock_me|12|r/rrw-r--r--|0|0|26214400|1585771103|1585771236|1585771236|0
0|a/pass.b64|13|r/rrw-r--r--|0|0|32|1585770998|1585770852|1585770852|0
0|a/part00 (deleted)|14|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part01 (deleted)|15|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part02 (deleted)|16|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part03 (deleted)|17|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part04 (deleted)|18|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part05 (deleted)|19|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part06 (deleted)|20|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part07 (deleted)|21|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part08 (deleted)|22|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part09 (deleted)|23|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part0a (deleted)|24|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part0b (deleted)|25|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part0c (deleted)|26|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part0d (deleted)|27|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part0e (deleted)|28|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part0f (deleted)|29|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part10 (deleted)|30|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part11 (deleted)|31|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part12 (deleted)|32|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part13 (deleted)|33|r/rr--------|0|0|2|1585770852|1585770852|1585770852|0
0|a/part14 (deleted)|34|r/rr--------|0|0|1|1585770852|1585770852|1585770852|0
0|a/$OrphanFiles|7681|V/V---------|0|0|0|0|0|0|0
0|a/$OrphanFiles/OrphanFile-1921 (deleted)|1921|-/drwxr-xr-x|0|0|0|1585770852|1585770852|1585770852|0
```

On peut voir qu'il y a plusieurs fichiers supprimés de l'inode 14 à 34.

### Récupérer la clé 

Avec `icat`, il est possible d'afficher directement le contenu et décoder la clé:

```bash
➜  for i in $(seq 14 34); do icat find_me $i; done | base64 -d
Mf-9Y2oMNZnHXKsjM8q8nRTT8x3UdY
```

> Mf-9Y2oMNZnHXKsjM8q8nRTT8x3UdY

## Déchiffrement et montage du volume LUKS

L'utilitaire `cryptsetup` permet de déchiffrer un volume:

```bash
➜  mkdir a
➜  sudo cryptsetup luksOpen unlock_me a 
Saisissez la phrase secrète pour unlock_me : Mf-9Y2oMNZnHXKsjM8q8nRTT8x3UdY
➜  lsblk
NAME                    MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
loop29                    7:29   0    25M  0 loop  
└─a                     253:3    0    23M  0 crypt 
```

Maintenant que le volume déchiffré a été associé à un mapper, il ne reste qu'à le monter:

```bash
➜  sudo mount /dev/mapper/a b 
➜  ls -la b 
total 4
drwxr-xr-x 2 root root   92 avril  1 21:54 .
drwxrwxr-x 4 maki maki 4096 avril 24 16:41 ..
-r-------- 1 root root   70 avril  1 21:54 .you_found_me
```

Le flag se trouve dans le fichier __.you_found_me__

## Flag

> FCSC{750322d61518672328c856ff72fac0a80220835b9864f60451c771ce6f9aeca1}