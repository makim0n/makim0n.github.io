---
weight: 4
title: "[ECSC] - 3615 incident"
date: 2019-06-27T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: "Simple reverse shell using OpenSSL."
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["anssi", "ecsc", "forensic", "malware", "memory dump", "volatility"]
categories: ["Writeups"]

lightgallery: true
---

# 3615 incident (1)



## Description

> Une victime de plus tombée sous le coup d’un rançongiciel. Le paiement de la rançon n’est pas envisagée vu le montant demandé. Vous êtes appelé pour essayer de restaurer les fichiers chiffrés.
> Une suite d’éléments est nécessaire pour avancer dans l’investigation et constituer le rapport d’incident.
> Pour commencer, quel est le nom du fichier exécutable de ce rançongiciel, son identifiant de processus et quel est devenu le nom du fichier flag.docx une fois chiffré ? Donnez le SHA1 de ce nom avec son extension.
> Note : l’image disque fait environ 440 Mo compressée et environ 1.4 Go décompressée. Réponse attendue au format ECSC{nom_du_rançongiciel.exe:PiD:sha1}.

## Nom & PID du rançongiciel

On sait que le fichier flag.docx existe et a été chiffré par le ransomware, voyons ce qu'un simple `yarascan` nous donne :

```
root@2767878b4519:/opt# vol.py -f usr_land/mem.dmp --profile=Win10x64_10586 yarascan -Y "flag.docx"
Volatility Foundation Volatility Framework 2.6.1
Rule: r1
Owner: Process assistance.exe Pid 5208
0x1280a472  66 6c 61 67 2e 64 6f 63 78 00 6a 73 00 00 43 3a   flag.docx.js..C:
0x1280a482  5c 55 73 65 72 73 5c 54 4e 4b 4c 53 41 49 33 54   \Users\TNKLSAI3T
0x1280a492  47 54 37 4f 39 5c 44 6f 63 75 6d 65 6e 74 73 5c   GT7O9\Documents\
0x1280a4a2  66 6c 61 67 2e 64 6f 63 78 00 2e 69 6e 69 43 3a   flag.docx..iniC:
0x1280a4b2  5c 55 73 65 72 73 5c 54 4e 4b 4c 53 41 49 33 54   \Users\TNKLSAI3T
0x1280a4c2  47 54 37 4f 39 5c 4c 69 6e 6b 73 5c 5a 47 56 7a   GT7O9\Links\ZGVz
0x1280a4d2  61 33 52 76 63 43 35 70 62 6d 6b 3d 00 00 43 3a   a3RvcC5pbmk=..C:
0x1280a4e2  5c 55 73 65 72 73 5c 54 4e 4b 4c 53 41 49 33 54   \Users\TNKLSAI3T
0x1280a4f2  47 54 37 4f 39 5c 4c 69 6e 6b 73 5c 5a 47 56 7a   GT7O9\Links\ZGVz
0x1280a502  61 33 52 76 63 43 35 70 62 6d 6b 3d 6a 73 43 3a   a3RvcC5pbmk=jsC:
0x1280a512  5c 55 73 65 72 73 5c 54 4e 4b 4c 53 41 49 33 54   \Users\TNKLSAI3T
0x1280a522  47 54 37 4f 39 5c 4c 69 6e 6b 73 5c 64 65 73 6b   GT7O9\Links\desk
0x1280a532  74 6f 70 2e 69 6e 69 00 6f 70 2e 69 6e 69 43 3a   top.ini.op.iniC:
0x1280a542  5c 55 73 65 72 73 5c 54 4e 4b 4c 53 41 49 33 54   \Users\TNKLSAI3T
0x1280a552  47 54 37 4f 39 5c 4c 69 6e 6b 73 5c 64 65 73 6b   GT7O9\Links\desk
0x1280a562  74 6f 70 2e 69 6e 69 00 00 00 00 00 00 00 43 3a   top.ini.......C:
[...]
```

Un seul process en ressort : `assistance.exe`, qui a le PID 5208.

Pour voir si ce binaire est malveillant, quoi de mieux qu'un benchmark d'antivirus:

```bash
root@2767878b4519:/opt# vol.py -f usr_land/mem.dmp --profile=Win10x64_10586 procdump -p 5208 -D usr_land/wu/
Volatility Foundation Volatility Framework 2.6.1
Process(V)         ImageBase          Name                 Result
------------------ ------------------ -------------------- ------
0xffffe000106bb840 0x0000000000400000 assistance.exe       OK: executable.5208.exe
```



![](https://i.imgur.com/0fgGdxO.png)

_Fig 3_: Résultats virustotal



Un autre indice pourrait être l'emplacement de l'executable :

```bash
root@2767878b4519:/opt# vol.py -f usr_land/mem.dmp --profile=Win10x64_10586 filescan > usr_land/wu/pfilescan
Volatility Foundation Volatility Framework 2.6.1

root@2767878b4519:/opt# cat usr_land/wu/pfilescan | grep 'assistance.exe'
0x0000e00011360090      1      0 R--r-d \Device\HarddiskVolume3\Users\TNKLSAI3TGT7O9\Downloads\assistance.exe
0x0000e00011483b40     10      0 R--r-d \Device\HarddiskVolume3\Users\TNKLSAI3TGT7O9\Downloads\assistance.exe
0x0000e000121df450      6      0 R--r-d \Device\HarddiskVolume3\Users\TNKLSAI3TGT7O9\Downloads\assistance.exe
0x0000e0001256bde0     14      0 R--r-d \Device\Mup\;Z:000000000002acd3\vmware-host\Shared Folders\e\assistance.exe
```

Donc, on va dire que Michel a juste téléchargé le binaire et pas que quelqu'un l'a drop volontairement via un dossier partagé :D

## Nom de flag.docx chiffré

Restons dans les fichiers chargés en mémoire, voyons rapidement ce qu'il y a dedans :

```root@2767878b4519:/opt# cat usr_land/wu/pfilescan | grep 'Downloads'
0x0000e00010323f20  32768      1 R--rwd \Device\HarddiskVolume3\Users\TNKLSAI3TGT7O9\Downloads
0x0000e000109d5090  32768      1 R--rwd \Device\HarddiskVolume3\Users\TNKLSAI3TGT7O9\Downloads
0x0000e00011360090      1      0 R--r-d \Device\HarddiskVolume3\Users\TNKLSAI3TGT7O9\Downloads\assistance.exe
0x0000e00011483b40     10      0 R--r-d \Device\HarddiskVolume3\Users\TNKLSAI3TGT7O9\Downloads\assistance.exe
0x0000e00011e15f20  32768      1 R--rw- \Device\HarddiskVolume3\Users\TNKLSAI3TGT7O9\Downloads
0x0000e000121df450      6      0 R--r-d \Device\HarddiskVolume3\Users\TNKLSAI3TGT7O9\Downloads\assistance.exe
0x0000e000126f1650     16      0 RW-rw- \Device\HarddiskVolume3\Users\Administrateur\Downloads\ZGVza3RvcC5pbmk=.chiffré
```

Forcément, le fichier qui saute au visage est `ZGVza3RvcC5pbmk=.chiffré`. Voyons ce que le base64 signifie:

```bash
root@2767878b4519:/opt# echo 'ZGVza3RvcC5pbmk=' | base64 -d
desktop.ini
```

Intuitivement, je suppose que le nom des fichiers chiffrés est encodé en base64 et sont suivis de la charmante mention `chiffré`. Vérifions cette hypothèse :

```bash
desktop.iniroot@2767878b4519:/opt# echo -n 'flag.docx' | base64
ZmxhZy5kb2N4
root@2767878b4519:/opt# cat usr_land/wu/pfilescan | grep 'ZmxhZy5kb2N4'
0x0000e000123988d0     16      0 R--r-d \Device\HarddiskVolume4\ZmxhZy5kb2N4.chiffré
```

## Flag

> ECSC{assistance.exe:5208:c9a12b109a58361ff1381fceccdcdcade3ec595a}

# 3615 incident (2)


## Description

> Une victime de plus tombée sous le coup d’un rançongiciel. Le paiement de la rançon n’est pas envisagée vu le montant demandé. Vous êtes appelé pour essayer de restaurer les fichiers chiffrés.
> Une suite d’éléments est nécessaire pour avancer dans l’investigation et constituer le rapport d’incident.
> Retrouvez la clé de chiffrement de ce rançongiciel!
> Note : l’image disque fait environ 440 Mo compressée et environ 1.4 Go décompressée. Elle est identique au challenge 3615 Incident - 1. Réponse attendue au format ECSC{clé}.

## Etat de l'art

Bon, pour trouver la clé d'un ransomware, soit c'est hardcodé dans le binaire, soit il va falloir tailler dans le vif de la mémoire.
On a déjà récupéré le binaire, récupérons la mémoire de ce process.

En naviguant rapidement dans le binaire, on peut voir un repo GitHub : 

```bash
➜  wu strings executable.5208.exe | grep github
github.com/mattn/go-colorable
%github.com/mauri870/ransomware/client
github.com/mauri870/ransomware/cryptofs
github.com/mauri870/ransomware/build/ransomware
github.com/mauri870/ransomware
github.com/fatih/color
github.com/mattn/go-colorable
github.com/mattn/go-isatty
```

> https://github.com/mauri870/ransomware/

Un super ransomware en Go. Donc à partir de là, je sais que le reverse n'est pas une option.

## Structure de la clé - Part 1

Dans les sources du ransomware on peut voir une structure entre un identifiant et une clé. 

> https://github.com/mauri870/ransomware/blob/master/cmd/ransomware/ransomware.go

```go
func encryptFiles() {
	keys := make(map[string]string)
    [...]
	for {
        [...]
		// Generate the id and encryption key
		keys["id"], _ = utils.GenerateRandomANString(32)
		keys["enckey"], _ = utils.GenerateRandomANString(32)

		// Persist the key pair on server
		res, err := Client.AddNewKeyPair(keys["id"], keys["enckey"])
        [...]
```

> https://github.com/mauri870/ransomware/blob/master/utils/utils.go

```go
// Generate a random alphanumeric string with the given size
func GenerateRandomANString(size int) (string, error) {
	key := make([]byte, size)
	_, err := rand.Read(key)
	if err != nil {
		return "", err
	}

	return hex.EncodeToString(key)[:size], nil
}
```

Il est plutôt clair que la clé et l'identifiant vont former une chaîne de 16 octets, mais en voici la preuve :

> https://play.golang.org/

```go
package main

import (
	"crypto/rand"
	"encoding/hex"
	"fmt"
)

func GenerateRandomANString(size int) (string, error) {
	key := make([]byte, size)
	_, err := rand.Read(key)
	if err != nil {
		return "", err
	}

	return hex.EncodeToString(key)[:size], nil
}

func main() {
	for i := 0; i < 10; i++ {
    		fmt.Println(GenerateRandomANString(32))
	}
}
```

Output :

```
d6e8aec1a39e2322635dd41a8fd24cf2 <nil>
1100c622a07382ac8588fe0904cb987d <nil>
1490d561257900bbf1c91543e69c0530 <nil>
eebc86cfaf014b7b0258931b3e67870e <nil>
26e938c4f18ad7d93d0bc6cc47eccbb9 <nil>
dd712f6f01da3ed26fdf2a4bab19729c <nil>
6cf81a92b9baa84af784edb73629714e <nil>
2487a4aeab8cbc07b6f8ebe866e36ed9 <nil>
43054bf6afbbe00950dd388458e16e71 <nil>
83ce9f50f2c686873d0a05b4418696c8 <nil>
```

## Structure de la clé - Part 2

On sait quelle forme a la donnée que l'on cherche et on sait plus ou moins où elle se trouve. Mais regardons si cette structure `id / enckey` en JSON est présente en mémoire :

```bash
➜  wu strings 5208.dmp| grep -C 5 "enckey"
[/LL
95511870061fb3a2899aa6b2dc9838aa
"C:\Users\TNKLSAI3TGT7O9\Downloads\assistance.exe"
C:\Users\TNKLSAI3TGT7O9\Downloads\assistance.exe
S-1-5-21-2377780471-3200203716-3353778491-1000
{"id": "cd18c00bb476764220d05121867d62de", "enckey": "
cd18c00bb476764220d05121867d62de64e0821c53c7d161099be2188b6cac24cd18c00bb476764220d05121867d62de64e0821c53c7d161099be2188b6cac2495511870061fb3a2899aa6b2dc9838aa422d81e7e1c2aa46aa51405c13fed15b95511870061fb3a2899aa6b2dc9838aa422d81e7e1c2aa46aa51405c13fed15b
Encrypting C:\Users\Administrateur\Contacts\desktop.ini...
C:\Users\TNKLSA~1\AppData\Local\Temp\desktop.ini
C:\Users\TNKLSA~1\AppData\Local\Temp\desktop.ini
Encrypting C:\Users\Administrateur\Documents\desktop.ini...
```

On a quelque chose qui ressemble à la clé juste au-dessus de la structure. Pour nous conforter dans cette idée, regardons le pâté d'hex juste à la suite de `enckey`:

```
cd18c00bb476764220d05121867d62de
64e0821c53c7d161099be2188b6cac24
cd18c00bb476764220d05121867d62de
64e0821c53c7d161099be2188b6cac24
95511870061fb3a2899aa6b2dc9838aa
422d81e7e1c2aa46aa51405c13fed15b
95511870061fb3a2899aa6b2dc9838aa
422d81e7e1c2aa46aa51405c13fed15b
```

Pour éviter de bruteforcer comme un âne, on peut essayer de voir s'il y a d'autres chaînes pouvant être de bons candidats :

```bash
➜  wu strings 5208.dmp| grep -E '^[a-f0-9]{32}$' | grep -v -E '[0-9]{32}' | sort | uniq
15590781f1602a3ba9982b6a89cdaa83
51e07da907d2e5118729806e6f6e6963
5476d0c4a7a347909c4b8a13078d4390
857bf9aba3e92444aed7cab9ad2342ea
95511870061fb3a2899aa6b2dc9838aa
965b7fc26dad90d340d2fa0a4879039f
cd18c00bb476764220d05121867d62de
```

Il y a deux chaînes communes à nos paquets d'octets :

* __cd18c00bb476764220d05121867d62de__ : Identifiants
* __95511870061fb3a2899aa6b2dc9838aa__ : Clé ???

C'est effectivement la clé de chiffrement.

## Flag

> ECSC{95511870061fb3a2899aa6b2dc9838aa}

# 3615 incident (3)


## Description

> Une victime de plus tombée sous le coup d’un rançongiciel. Le paiement de la rançon n’est pas envisagée vu le montant demandé. Vous êtes appelé pour essayer de restaurer les fichiers chiffrés.
> Déchiffrez le fichier "flag.docx" ci-joint!

## Etat de l'art

Dans ce dernier challenge de la série _3615 incident_, le fichier chiffré nous a été donné. On sait qu'on cherche un docx, donc une signature de ZIP : `50 4B 03 04`.

On sait aussi que le fichier a été chiffré avec de l'AES CTR, donc nous avons la clé, mais il manque l'IV.

## A la recherche de l'IV perdu

> https://github.com/mauri870/ransomware/blob/master/cryptofs/file.go

```go
[...]
func (file *File) Encrypt(enckey string, dst io.Writer) error {
    [...]
    	// The IV needs to be unique, but not secure
	iv := make([]byte, aes.BlockSize)
        if _, err = io.ReadFull(rand.Reader, iv); err != nil {
            return err
        }

        // Get a stream for encrypt/decrypt in counter mode (best performance I guess)
        stream := cipher.NewCTR(block, iv)

        // Write the Initialization Vector (iv) as the first block
        // of the dst writer
        dst.Write(iv)
        [...]
```

L'IV est donc le premier bloc du fichier chiffré. Le voici :

```bash
➜  wu cat data | head -c 16 | xxd -p
b627d24fc90dfe7ce421c43312dc2f2e
```

Maintenant, soit on faire un super script python, soit on utilise CyberChef. Il est 5h53 du matin, je vais utiliser CyberChef: https://gchq.github.io/CyberChef/

Pour récupérer le fichier chiffré sans le premier bloc :

```bash
➜  wu cat data | xxd -p | tail -c+33 | tr -d ' \n'
```

Ce qui nous donne :



![](https://i.imgur.com/iCNbPt0.png)

_Fig 4_: Déchiffrement du fichier dans CyberChef



La signature du début ressemble à un ZIP et la structure qui suit avec le `_rels/.rels` et le `docProps` font penser à un DOCX.



![](https://i.imgur.com/nOvJUpi.png)

_Fig 5_: Fichier déchiffré



## Flag

> ECSC{M4ud1t3_C4mp4gn3_2_r4NC0nG1c13L}