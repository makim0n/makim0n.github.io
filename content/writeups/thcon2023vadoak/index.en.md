---
weight: 4
title: "[THCon 2023] - Vadoak"
date: 2023-04-23T09:20:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["thcon", "forensic", "memory", "braille", "windows", "volatility", "vad"]
categories: ["Writeups"]

lightgallery: true
---

## Description

> memory, volatility, hard

James Vadoak is a blind soldier who has enrolled in the Rebel Alliance (his eyes were taken by Darth Vader). During a mission to compromise the Death Star, he managed to hack one of the central computers. He also left a message addressed to the Empire in a virtual machine which was on the computer. Unfortunately he did not have the time to save the file containing the message.

A VMware Fusion snapshot and memory files are being given to you. As a digital forensics analyst of the Empire your goal is to find the message left by James Vadoak in this VM.

Important note : the flag is the part of the message between the symbols. ”THCon23{}” is not written in the message.

Example: THCon23{secretmessage}

Author : Hugo Courtadon (Hugo Courtadon#6552)

## Prise d'informations

On commence donc cette épreuve avec deux fichiers VMWare, comme indiqué dans l'énoncé :
* Death_Star.vmem
* Death_Star.vmsn

Le fichier "vmsn" et le dossier "_MACOSX" ne sont pas utiles pour cette épreuve.

VMWare n'utilise pas de format de fichier particulier pour la RAM (à l'instar de VirtualBox...). Il est donc possible de l'utiliser directement avec Volatility3 (le 2 aussi, mais nous sommes en 2023, il faut évoluer avec son temps).

```bash
➜  vadoak vol3 -f ./Death_Star.vmem info.Info                                 
Volatility 3 Framework 2.4.2
Progress:  100.00		PDB scanning finished                        
[...]
NtSystemRoot	C:\WINDOWS
NtProductType	NtProductWinNt
NtMajorVersion	5
NtMinorVersion	2
[...]
```

On sait qu'il faut trouver une note qui n'a pas eu le temps d'être enregistrée, donc en regardant les process :

```bash
➜  vadoak vol3 -f ./Death_Star.vmem windows.pstree.PsTree
Volatility 3 Framework 2.4.2
Progress:  100.00		PDB scanning finished                        
PID	PPID	ImageFileName	Offset(V)	Threads	Handles	SessionId	Wow64	CreateTime	ExitTime

4	0	System	0xfadfce8dbc20	59	413	N/A	False	N/A	N/A
[...]
1328	1248	explorer.exe	0xfadfcc04f4b0	12	291	0	False	2023-02-08 15:07:58.000000 	N/A
* 1536	1328	rundll32.exe	0xfadfccc26c20	4	53	0	False	2023-02-08 15:07:59.000000 	N/A
* 1544	1328	vmtoolsd.exe	0xfadfcdcbcb60	6	196	0	False	2023-02-08 15:07:59.000000 	N/A
* 2076	1328	notepad.exe	0xfadfcc05a540	1	19	0	False	2023-02-28 08:44:45.000000 	N/A
```

Le process "notepad.exe" parait être un bon candidat (toute façon, y a rien d'autres dans ce dump :))

## Notepad.exe ? Piece of cake

Si on retourne un peu lors de la préhistoire, il existe 2 plugins volatility2 peut connus qui permettent de récupérer ce qui a été saisi dans "notepad.exe" en mémoire :

* editbox
* notepad

```bash
➜  vadoak sudo docker run --rm --name volatility -v /home/maki/Tools/plugin_vol:/opt/plug_vol -v ${PWD}:/opt/usr_land -ti volatility

root@c9af9518f7f1:/opt/usr_land# vol.py -f Death_Star.vmem --profile=WinXPSP1x64 editbox
Volatility Foundation Volatility Framework 2.6.1
[...]
TypeError: unsupported operand type(s) for +: 'long' and 'NoneType'


root@c9af9518f7f1:/opt/usr_land# vol.py -f Death_Star.vmem --profile=WinXPSP1x64 notepad
Volatility Foundation Volatility Framework 2.6.1
Process: 2076
Traceback (most recent call last):
[...]
AttributeError: Struct _HEAP has no member segments
```

(Pour utiliser volatility2, j'ai un docker : https://gitlab.com/makimitz/infosec-docker/-/tree/master/forensic/volatility (c'est pas maintenu, ça ne le sera, pas s'il y a des bugs, tant pis :D))

Donc à ce stade du challenge, nous avons :

* On sait que c'est dans le "notepad.exe"
* On sait que c'est pas forcément quelque chose de printable ou un bug lors de l'acquisition (sinon les plugins auraient fonctionnés)

Maintenant il faut répondre à ces questions :

* Le format de l'encoding -> Savoir quoi chercher dans le dump
* Où -> Eviter de se taper toute la mémoire de notepad.exe

## Guessing

Le moment le plus fun du challenge était donc celui ci : guess et lire entre les lignes de l'énoncé :D

L'énoncé nous dit : "James Vadoak is a blind soldier [...] He also left a message"

Comment donc un aveugle peut laisser un message sur un ordinateur ? Et oui, en braille.

![](https://media3.giphy.com/media/K0Hy2NwI8IXZK/giphy.gif?cid=ecf05e47pu31spvsnl8bz2sv1hn71twm3x5ti74tvvipmukd&rid=giphy.gif)    

N'étant pas un expert du braille et encore moins de sa représentation en mémoire, un coup de google Fu (wikipedia quoi), nous donne la solution : https://en.wikipedia.org/wiki/Braille_Patterns

![](/lib/images/writeups/2023_thcon/braille_encoded.png)

On remarque que le braille est sous forme d'unicode et qu'il y le pattern "0x28 0x?? 0x28 0x??" etc. On a répondu à la première question sur le format à chercher.

## Yara + VADTree

Pour chercher un pattern en mémoire, il est possible de s'amuser avec hexdump, strings et grep, mais n'étant pas spécialement masochiste, je préfère utiliser "yarascan".

```bash
➜  vadoak vol3 -f ./Death_Star.vmem --yara-rules "{28 ?? 28 ?? 28 ?? 28 ?? 28 ?? 28 ?? 28 ?? 28 ?? 28 ?? 28 ?? 28 ?? 28 ??}" --pid 2076
Volatility 3 Framework 2.4.2
Progress:  100.00		PDB scanning finished                        
Offset	PID	Rule	Component	Value

0xd6eab	2076	r1	$a	28 0b 28 0a 28 1d 28 19 28 3d 28 15 28 25 28 17 28 07 28 01 28 09 28 05
0xd6ead	2076	r1	$a	28 0a 28 1d 28 19 28 3d 28 15 28 25 28 17 28 07 28 01 28 09 28 05 28 15
0xd6eaf	2076	r1	$a	28 1d 28 19 28 3d 28 15 28 25 28 17 28 07 28 01 28 09 28 05 28 15 28 0b
0xd6eb1	2076	r1	$a	28 19 28 3d 28 15 28 25 28 17 28 07 28 01 28 09 28 05 28 15 28 0b 28 0e
[...]
```

Au final on obtient environ 2 offsets correspondant à notre recherche dans le process de notepad.exe :

* 0xd6eab
* 0xdc47b

Le portage des plugins volatility2 vers le 3 n'étant pas terminé, on refaire un petit "time travel" pour utiliser vadtree et la fonctionnalité graphviz :

```bash
root@cd7a0d615d9c:/opt/usr_land# vol.py -f Death_Star.vmem --profile=WinXPSP1x64 vadtree -p 2076 --output=dot --output-file=graph.dot
Volatility Foundation Volatility Framework 2.6.1
Outputting to: graph.dot
```

Ca a donc cette tête là : 

![](/lib/images/writeups/2023_thcon/graphviz.svg)

Le fichier est téléchargeable ici : [graph.dot](/lib/attachments/thcon2023_graph.dot)

On voit pas grand chose sur le graph, mais on s'en fiche parce qu'on a déjà des offsets à trouver. On voit donc que nos marqueurs ont match avec un bout de mémoire dans la heap (rouge dans vadtree c'est la heap) :

![](/lib/images/writeups/2023_thcon/vol_identifier_heap.png)

Avec cette information, nous venons de répondre à notre "où".

## Extraction et décodage

Maintenant, qu'on sait où et quoi chercher, c'est tout de suite plus simple :

```bash
➜  vad_notepad vol3 -f ../Death_Star.vmem windows.vadinfo.VadInfo --dump --pid 2076
Volatility 3 Framework 2.4.2
Progress:  100.00		PDB scanning finished                        
PID	Process	Offset	Start VPN	End VPN	Tag	Protection	CommitCharge	PrivateMemory	Parent	File	File output

2076	notepad.exe	0xfadfcc465090	0x78ec0000	0x78ff8fff	Vad 	PAGE_EXECUTE_WRITECOPY	8	0	0xfadfcc05a8d8	\WINDOWS\system32\ntdll.dll	pid.2076.vad.0x78ec0000-0x78ff8fff.dmp
2076	notepad.exe	0xfadfcbfabf50	0x2b0000	0x3affff	VadS	PAGE_READWRITE	50	1	0xfadfcc465090	N/A	pid.2076.vad.0x2b0000-0x3affff.dmp
2076	notepad.exe	0xfadfce23e5d0	0xd0000	0x1cffff	VadS	PAGE_READWRITE	13	1	0xfadfcbfabf50	N/A	pid.2076.vad.0xd0000-0x1cffff.dmp
[...]
```

Il ne reste plus qu'à retrouver nos petits :

```bash
➜  vad_notepad hexdump -C pid.2076.vad.0xd0000-0x1cffff.dmp | grep '28 .. 28 ..'   
00006e90  0b 28 07 28 08 28 1b 28  00 28 0a 28 0e 28 00 28  |.(.(.(.(.(.(.(.(|
00006ea0  00 28 31 28 0d 00 0a 00  7b 00 0a 28 0b 28 0a 28  |.(1(....{..(.(.(|
00006eb0  1d 28 19 28 3d 28 15 28  25 28 17 28 07 28 01 28  |.(.(=(.(%(.(.(.(|
00006ec0  09 28 05 28 15 28 0b 28  0e 28 0f 28 11 28 11 28  |.(.(.(.(.(.(.(.(|
00006ed0  19 28 19 28 0a 28 0e 28  1e 28 25 28 17 28 03 28  |.(.(.(.(.(%(.(.(|
00006ee0  0a 28 1d 28 1b 28 7d 00  00 00 00 00 00 00 00 00  |.(.(.(}.........|

0000c460  0b 28 07 28 08 28 1b 28  00 28 0a 28 0e 28 00 28  |.(.(.(.(.(.(.(.(|
0000c470  00 28 31 28 0d 00 0a 00  7b 00 0a 28 0b 28 0a 28  |.(1(....{..(.(.(|
0000c480  1d 28 19 28 3d 28 15 28  25 28 17 28 07 28 01 28  |.(.(=(.(%(.(.(.(|
0000c490  09 28 05 28 15 28 0b 28  0e 28 0f 28 11 28 11 28  |.(.(.(.(.(.(.(.(|
0000c4a0  19 28 19 28 0a 28 0e 28  1e 28 25 28 17 28 03 28  |.(.(.(.(.(%(.(.(|
0000c4b0  0a 28 1d 28 1b 28 7d 00  00 00 00 00 00 00 00 00  |.(.(.(}.........|
```

C'est pas vraiment aligné pareil, mais on a bien un pattern. Un peu de bash fu pour ré encoder tout ça et normalement il sera possible d'en tirer quelque chose :

![](/lib/images/writeups/2023_thcon/braille_decode.png)

> ⠇⠈⠛⠀⠊⠎⠀⠀⠱⠍⠊⡻⠊⠋⠊⠝⠙⠽⠕⠥⠗⠇⠁⠉⠅⠕⠋⠎⠏⠑⠑⠙⠙⠊⠎⠞⠥⠗⠃⠊⠝⠛⡽

Un tour sur le meilleur site pour les challenges stégo guess : https://www.dcode.fr/alphabet-braille

![](/lib/images/writeups/2023_thcon/flag.png)

## Flag

> THCon23{IFINDYOURLACKOFSPEEDDISTURBING}

Pour conclure sur cette épreuve, la méthode d'analyse est intéressante, être obligé d'aller fouiller dans les VAD, faire un peu de Yara etc. Je pense que c'est un bon challenge pour prendre en main les outils d'analyse mémoire et mettre en place une méthodologie pertinente.
Le bémol se situe sur le use case étudié lors de cette épreuve, il aurait été plus intéressant de le faire sous forme de réelles investigations, avec des malwares etc.

En tout cas GG à toute l'orga de la THcon, c'était très sympa :)
A l'année prochaine \o/

