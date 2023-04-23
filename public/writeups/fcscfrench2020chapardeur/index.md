# [French FCSC] - Chapardeur de mots de passe


>Un ami vous demande de l'aide pour déterminer si l'email qu'il vient d'ouvrir au sujet du Covid-19 était malveillant et si c'était le cas, ce qu'il risque.
>Il prétend avoir essayé d'ouvrir le fichier joint à cet mail sans y parvenir. Peu de temps après, une fenêtre liée à l'anti-virus a indiqué, entre autre, le mot KPOT v2.0 mais rien d'apparent n'est arrivé en dehors de cela.
>
>Après une analyse préliminaire, votre ami vous informe qu'il est probable que ce malware ait été légèrement modifié, étant donné que le contenu potentiellement exfiltré (des parties du format de texte et de fichier avant chiffrement) ne semble plus prédictible. Il vous recommande donc de chercher d'autres éléments pour parvenir à l'aider.
>
>Vous disposez d'une capture réseau de son trafic pour l'aider à déterminer si des données ont bien été volées et lui dire s'il doit rapidement changer ses mots de passe !
>SHA256(pws.pcap) = 98e3b5f1fa4105ecdad4880cab6a7216c5bb3275d7367d1309ab0a0d7411475d - 463MB

## TL;DR

Un challenge de PCAP assez lourd (~500 Mo). Il faut trouver la connexion du malware au panel de C2, récupérer les bytes chiffrés. En se documentant on sait qu'il est possible de bruteforce la clé assez rapidement et de récupérer le flag.

## État de l'art

On commence donc cette épreuve avec un gros PCAP. Le souci des gros PCAP est qu'il est facile de se perdre dedans en suivant de fausses pistes. Pour éviter ça, il faut savoir ce que l'on cherche avant de plonger tête baissée dedans.

Ce qu'on sait:

* Le malware est un KPOT v2.0 modifié.

Ce qu'on cherche:

* Des indices de compromission d'un KPOT v2.0.

## Analyse du malware

Dans ma méthodologie, il est possible (et c'est souvent le cas), que le "ce qu'on cherche" demande des connaissances pas encore acquises. En l'occurrence, on connait le malware utilisé et il existe des analyses détaillées:

* https://www.proofpoint.com/us/threat-insight/post/new-kpot-v20-stealer-brings-zero-persistence-and-memory-features-silently-steal

__Proofpoint__ propose une analyse complète du malware. On peut maintenant affiner le "ce qu'on cherche":

* Le RTF ou document de macro utilisé pour drop le malware ;
* Des connexions vers "past.ee";
* Des connexions vers "gate.php";
* Trouver la clé de chiffrement du XOR;
* Trouver le C2 avec une requête GET puis une requête POST.

Cependant, il ne faut pas oublier que l'énoncé nous dit: 

> il est probable que ce malware ait été légèrement modifié, étant donné que le contenu potentiellement exfiltré (des parties du format de texte et de fichier avant chiffrement) ne semble plus prédictible

## Trouver la connexion suspecte

Le fait de chercher "gate.php" est une bonne piste:



![](https://i.imgur.com/PBZFGxE.png)



Il y a quelque chose de rigolo: toutes les connexions sont uniques sauf la dernière. Toutes les connexions sont de simple GET avec de la data chiffrées et encodées en b64, mais pas la dernière:



![](https://i.imgur.com/YXdGYLY.png)



C'est une requête POST avec de la donnée chiffrée brute, serait-ce notre flag tant attendu? On verra ça plus tard.

L'article de Proofpoint nous dit que la clé XOR est hardcodée dans le binaire. Mais il n'y a pas de binaire apparent dans ce PCAP (pour vérifier, un binwalk et "Fichiers -> Exporter objet -> HTTP").

Donc il faut trouver un autre moyen. On sait que c'est du XOR et on connait le format de la configuration envoyée à "gate.php". Toujours dans l'exemple de proofpoint:

* Configuration

```
1111111111111100__DELIMM__A.B.C.D__DELIMM__appdata__GRABBER__*.log,*.txt,__GRABBER__%appdata%__GRABBER__0__GRABBER__1024__DELIMM__desktop_txt__GRABBER__*.txt,__GRABBER__%userprofile%\Desktop__GRABBER__0__GRABBER__150__DELIMM____DELIMM____DELIMM__
```

* Clé du XOR

> 4p81GSwBwRrAhCYK

La clé fait 16 bytes, l'espèce de binaire du début de la configuration fait 16 bytes. De plus, la clé a un charset alphanumérique, soit: [A-Za-z0-9]. Il ne reste qu'à bruteforce tout ça pour trouver la clé qui nous intéresse.

## Bruteforce de la clé

Le fichier __enc_dat__ est le base64 décodé de la connexion GET au C2.

```python
#!/usr/bin/python3

from ENO import *
import itertools

enc_dat = open('enc_dat','rb').read()

# Génération de toutes les possibilités du binaire
lst_bin = []
for i in itertools.product(range(2),repeat=16):
    lst_bin.append("".join(map(str,i)))

# On filtre sur les clés potentielles (le filtrage aurait pu être mieux)
pot_key = []
for l in lst_bin:
    tmp = cribDrag(enc_dat,l.encode())
    for j in tmp:
        if j.decode().isprintable():
            pot_key.append(j)

# On bruteforce jusqu'à trouver la configuration déchiffrée
for k in pot_key:
    decrypt_dat = strxor(enc_dat,k)
    if decrypt_dat.decode()[:16].isprintable():
        if "__DELIMM____DELIMM__" in decrypt_dat.decode():
            print(f"Key : {k}\nData : {decrypt_dat}\n\n")
```

Après environ 1 petite minute, on trouve:

```bash
➜  chapardeyr python3 ./bf.py 
Key : b'tDlsdL5dv25c1Rhv'
Data : b'0110101110111110__DELIMM__218.108.149.373__DELIMM__appdata__GRABBER__*.log,*.txt,__GRABBER__%appdata%__GRABBER__0__GRABBER__1024__DELIMM__desktop_txt__GRABBER__*.txt,__GRABBER__%userprofile%\\Desktop__GRABBER__0__GRABBER__0__DELIMM____DELIMM____DELIMM__'
```

## Déchiffrement du message

Yapluka :)

```python
from ENO import *

key = b'tDlsdL5dv25c1Rhv'
enc = htos("2b003e3234097431296249164260181301363d06017e78501a13154363661b0501365f09491a5a10045718225c63453300691a1c552f04321946470655205c061125192c220f66277c49015508375047427c5b425c750c5213510d50506b5a1717205a15017a575d1502060007310d1246255f125329020544020d5a53675b4216250d165d7b54530b386a2763133833351133")

print(strxor(enc,key))
```

> b'_DRAPEAU_P|us2peurQue2M4l!  R4ssur3z-Votre-Am1-Et-vo1c1Votredr4peau_FCSC\n{469e8168718996ec83a92acd6fe6b9c03c6ced2a3a7e7a2089b534baae97a7}\n_DRAPEAU_'

En l'occurrence, comme l'énoncé le disait, le code du malware a été modifié, ainsi la structure de la connexion POST change et ne propose plus "\_FFFILEE_" ou "\_SYSINFORMATION_". Il faut lire avait de râler :D

## Flag

> FCSC{469e8168718996ec83a92acd6fe6b9c03c6ced2a3a7e7a2089b534baae97a7}
