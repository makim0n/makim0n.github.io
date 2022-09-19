---
weight: 4
title: "[ECSC] - Exfiltration"
date: 2019-06-27T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["anssi", "ecsc", "wireshark", "crypto", "network"]
categories: ["Writeups"]

lightgallery: true
---

## Description

> Notre SoC a détecté qu'un document confidentiel avait été exfiltré ! La méthode utilisée ne semble pas avancée et heureusement, une capture réseau a pu être faite au bon moment... Retrouvez ce document.

## Trouver la configuration de l'extraction

On commence cette épreuve par une capture réseau. Comme à chaque fois en matière d'exfiltration, il suffit de trouver un bout du filon pour le remonter complètement.



![](https://i.imgur.com/LKv4Fsv.png)

_Fig 6_: Configuration de l'exfiltration



```bash
➜  exfil tshark -r exfiltration.pcap -Y 'icmp.resp_to' -Tfields -e 'data.data' | xxd -r -p
config : exfiltered_file_size=4193bytes
config : file_type=DOCX
config : data_len_for_each_packet=random
config : encryption=XOR
```

Maintenant, on sait qu'on cherche un DOCX xoré de 4193 octets.

## Récupérer la donnée chiffrée

On filtre donc sur les IP qui se sont échangées la configuration:



![](https://i.imgur.com/ijTPbSF.png)

![](https://i.imgur.com/QVlsLv5.png)

_Fig 7_: Exfiltration de la donnée



* __data__ étant la donnée chiffrée.
* __uuid__ l'identifiant du fichier extrait.

```bash
➜ tshark -r exfiltration.pcap -Y '((urlencoded-form.key == "data")) and (ip.src == 192.168.1.26)' -Tfields -e 'urlencoded-form.value' | cut -d',' -f1 | tr -d ' \n' | xxd -r -p > docx.xor
```

On a bien récupéré notre bouilli.

## XOR time

On sait que c'est xoré, on sait aussi que c'est un DOCX donc en principe on peut faire du clair connu dessus. Maintenant, est ce que `xortool` ne peux pas le faire tout seul comme un grand :

```bash
(exfil) ➜  exfil xortool/xortool/xortool docx.xor
The most probable key lengths:
   2:   12.8%
   4:   15.4%
   6:   10.8%
   8:   12.7%
  10:   8.9%
  12:   10.4%
  14:   7.4%
  16:   8.5%
  18:   6.0%
  20:   7.0%
Key-length can be 4*n
Most possible char is needed to guess the key!

(exfil) ➜  exfil xortool/xortool/xortool docx.xor -l 4 -c 00
1 possible key(s) of length 4:
ecsc
Found 0 plaintexts with 95.0%+ valid characters
See files filename-key.csv, filename-char_used-perc_valid.csv

(exfil) ➜  exfil xortool/xortool/xortool-xor -f docx.xor -s "ecsc" > clear.docx

(exfil) ➜  exfil file clear.docx
clear.docx: Microsoft Word 2007+
```



![](https://i.imgur.com/UnXmAjV.png)

_Fig 8_: Donnée confidentielle récupérée



## Flag

> ECSC{v3ry_n01sy_3xf1ltr4t10n}