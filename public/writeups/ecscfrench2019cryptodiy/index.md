# [ECSC] - CryptoDIY


## Description

> Dumby possède un secret qui en fait une personne exceptionnelle, en tout cas c’est ce qu’il dit. Dumb, un ami de Dumby, a échangé avec lui sur le sujet et n’a pas réussi à révéler ce mystère. Il vous confie une archive de cette conversation et compte sur vous pour résoudre l’énigme, surtout qu’il vient d’effacer la clé USB qui contenait ses clés secrètes et les fichiers déchiffrés, il porte bien son nom celui-là…

## Mail

On commence cette épreuve avec un dump de boit gmail. Dedans on retrouve un échange entre deux personnes parlant d'un super algo de crypto de la mort qui tue et un fichier chiffré.



![](https://i.imgur.com/2Ttrcps.png)




Et on retrouve le script implémentant cet algo:

```python
#Public keys
N=53631231719770388398296099992823384509917463282369573510894245774887056120440201731735752915096736722856082884548917654078388282501542293219713052500317361 #N=p*q with primes p and q that are part of my secret key

g1=27888419610931008932601664194635863362934795268815711191831564335481311281875775885782976960387128801701810764586184606874328259752328953489610371824189861 #g1=g^(r1*(p-1)) mod N where r1 is a secret random
g2=48099264739307394087061906063506998841178675587231635606136922987485103900358857801144199074460106065443613588490877251721439499846448566220473205526148817 #g2=g^(r2*(q-1)) mod N where r2 is a secret random

#To encipher the message m of same size as N
#Choose two nounces s1 and s2 (plz keep them secret) and send me c1 and c2 given by:

import random
def encipher(m,g1,g2,N):
    s1 = random.randrange(2**127,2**128)
    s2 = random.randrange(2**127,2**128)
    return (m*pow(g1,s1,N))%N, (m*pow(g2,s2,N))%N

#To encipher a data use this code and send me the cipher file

nb_car_block=64

def BlobToIntSeq(m,nb_car_block):
    m = m + bytes([71,82,114])
    l = len(m)
    r = l % nb_car_block
    
    if not(r == 0):
        m = m + (nb_car_block-r)*bytes([114])
        l = len(m)

    out = []
    i = 0
    j = nb_car_block

    while j<=l:
        tmp = int.from_bytes(m[i:j], "big")
        out = out + [tmp]
        i,j = i+nb_car_block,j+nb_car_block

    return out

import sys

plainbin = open(sys.argv[1], 'rb')
plain = plainbin.read()
plainbin.close()

l = len(plain)
plainint = BlobToIntSeq(plain,nb_car_block)
cipher = open(sys.argv[1]+'.cipher', 'w')

i = 0
for m in plainint:
    c1,c2=encipher(m,g1,g2,N)
    cipher.write((str(c1)) + '\n')
    cipher.write((str(c2)) + '\n')
    i=i+1

cipher.close()
```

Avant de commencer à attaquer le vif du sujet, il y a de mentions intéressantes dans les mails :

* "in particular factorizzation of the modulus"
* "the asiatic theorem... I do not remember the exact name"

Enfin, on sait que l'on cherche un fichier MP4, qui a la signature suivante :



![](https://www.file-recovery.com/signatures/mp4.png)




Maintenant, on sait pour où commencer et ce qu'on cherche.

## Factorisation

On a un gros N, donc essayons de le factoriser. D'habitude factordb fonctionne bien, mais dans ce cas précis pas du tout, je me suis rabattu sur : https://www.alpertron.com.ar/ECM.HTM



![](https://i.imgur.com/kIt1lVz.png)



Nous avons donc:

> p = 115792089237316195423570985008687907853269984665640564039457584007913129640233
> q = 463168356949264781694283940034751631413079938662562256157830336031652518559817

## "Simplification"

Les équations qui nous intéressent sont :

> g1=g^(r1*(p-1)) mod N
> g2=g^(r2*(q-1)) mod N

Au début le fait de ne pas avoir `g` m'a perturbé et je comprennais pas comment faire.
Alors je me suis mis à jouer avec les modulo `p` et `q`. Et quelques chose de rigolo arriva:

```python
>>> import random
>>> g1=27888419610931008932601664194635863362934795268815711191831564335481311281875775885782976960387128801701810764586184606874328259752328953489610371824189861 #g1=g^(r1*(p-1)) mod N where r1 is a secret random
>>> g2=48099264739307394087061906063506998841178675587231635606136922987485103900358857801144199074460106065443613588490877251721439499846448566220473205526148817 #g2=g^(r2*(q-1)) mod N where r2 is a secret random
>>> N=53631231719770388398296099992823384509917463282369573510894245774887056120440201731735752915096736722856082884548917654078388282501542293219713052500317361 #N=p*q with primes p and q that are part of my secret key

>>> p = 115792089237316195423570985008687907853269984665640564039457584007913129640233
>>> q = 463168356949264781694283940034751631413079938662562256157830336031652518559817
>>> m = p+5000000000000000000000000000000000000000000000000000 # Pour prendre un m bien plus grand que p

>>> def encipher(m,g1,g2,N):
...     s1 = random.randrange(2**127,2**128)
...     s2 = random.randrange(2**127,2**128)
...     return (m*pow(g1,s1,N))%N, (m*pow(g2,s2,N))%N # m x
...
>>> i,j = encipher(m,g1,g2,N)
>>> j%q
115792089237316195423570990008687907853269984665640564039457584007913129640233L
>>> m%q
115792089237316195423570990008687907853269984665640564039457584007913129640233L
>>> i%p
5000000000000000000000000000000000000000000000000000L
>>> m%p
5000000000000000000000000000000000000000000000000000L
```

On peut donc appliquer le CRT.
Lors du challenge j'ai fuzzé pour trouver ces résultats. Cependant, avec un peu de recherche, je suis tombé sur le __petit théroème de Fermat__, qui explique cette propriété :

> http://villemin.gerard.free.fr/ThNbDemo/PtThFerm.htm

## Chinese Remainder Theorem

On cherche `m%n` avec `n = p*q`, grace à la simplification précédente on connait `m = i%p` et `m = j%q`.

Comme je suis une buse en math et une buse en prog, mais que je suis en bon en recherche Google, je suis tombé sur ce site :

> https://www.geeksforgeeks.org/using-chinese-remainder-theorem-combine-modular-equations/

Ce site explique ce qu'est le CRT et propose une implémentation en python. Et enfin quelques chose de prometteur :

```python
>>> p = 115792089237316195423570985008687907853269984665640564039457584007913129640233
>>> q = 463168356949264781694283940034751631413079938662562256157830336031652518559817
>>> j = 0
>>> k = 1
>>>
>>> f = open('/Users/maki/Documents/ctf/ecsc/cryptodiy/Takeout/Mail/WIM.mp4.cipher','r')
>>> data = f.read().split('\n')
>>> f.close()
>>> def extended_euclidean(a, b):
...     if a == 0:
...             return (b, 0, 1)
...     else:
...             g, y, x = extended_euclidean(b % a, a)
...             return (g, x - (b // a) * y, y)
...
>>> def modinv(a, m):
...     g, x, y = extended_euclidean(a, m)
...     return x % m
...
>>> def crt(m, x):
...     while True:
...             temp1 = modinv(m[1],m[0]) * x[0] * m[1] + modinv(m[0],m[1]) * x[1] * m[0]
...             temp2 = m[0] * m[1]
...             x.remove(x[0])
...             x.remove(x[0])
...             x = [temp1 % temp2] + x
...             m.remove(m[0])
...             m.remove(m[0])
...             m = [temp2] + m
...             if len(x) == 1:
...                     break
...     return x[0]
...
>>> m = [p,q]
>>> x = [int(data[0])%p, int(data[1])%q]
>>> tmp = crt(m,x)
>>> hex(tmp)
'0x1c667479706d703432000000016d7034316d70343269736f6d00016a8b6d6f6f760000006c6d76686400000000d8f7caa8d8f88d90000003e800009745L'
>>> hex(tmp)[2:-1].decode('hex')
'\x1cftypmp42\x00\x00\x00\x01mp41mp42isom\x00\x01j\x8bmoov\x00\x00\x00lmvhd\x00\x00\x00\x00\xd8\xf7\xca\xa8\xd8\xf8\x8d\x90\x00\x00\x03\xe8\x00\x00\x97E'
```



![](https://media.giphy.com/media/VarW8M9eEha1y/giphy.gif)



La signature d'un MP4 ! Maintenant il faut jouer un peu avec la donnée. On remarque qu'il manque des `00` au début du fichier. Il faut donc padder avec des `00` les blocs faisant une taille inférieure à 64.

## Scripting time

```python
#!/usr/bin/python2

# Factor N : https://www.alpertron.com.ar/ECM.HTM
p = 115792089237316195423570985008687907853269984665640564039457584007913129640233
q = 463168356949264781694283940034751631413079938662562256157830336031652518559817
j = 0
k = 1

f = open('/Users/maki/Documents/ctf/ecsc/cryptodiy/Takeout/Mail/WIM.mp4.cipher','r')
data = f.read().split('\n')
f.close()

# function that implements Extended euclidean 
# algorithm 
def extended_euclidean(a, b): 
	if a == 0: 
		return (b, 0, 1) 
	else: 
		g, y, x = extended_euclidean(b % a, a) 
		return (g, x - (b // a) * y, y) 

# modular inverse driver function 
def modinv(a, m): 
	g, x, y = extended_euclidean(a, m) 
	return x % m 

# function implementing Chinese remainder theorem 
# list m contains all the modulii 
# list x contains the remainders of the equations 
def crt(m, x): 
	while True: 
		temp1 = modinv(m[1],m[0]) * x[0] * m[1] + modinv(m[0],m[1]) * x[1] * m[0] 
		temp2 = m[0] * m[1] 
		x.remove(x[0]) 
		x.remove(x[0]) 
		x = [temp1 % temp2] + x 
		m.remove(m[0]) 
		m.remove(m[0]) 
		m = [temp2] + m 
		if len(x) == 1: 
			break
	return x[0] 

g = open('clear_dat','wb+')

for i in range(0, len(data)):
	x = [int(data[j])%p, int(data[k])%q]
	j = j+2
	k = k+2
	m = [p,q]
	tmp_var = crt(m,x)
	tmp_var = hex(tmp_var).lstrip('0x').rstrip('L')
	if len(tmp_var)%2 == 0:
		pass
	else:
		tmp_var = '0'+tmp_var
	dec = tmp_var.decode("hex").rjust(64,'\x00')
	g.write(dec)

g.close()
```

## SHAZAAAAAAAAM

Nous voilà avec une vidéo... De K-pop... M'enfin, regardons les exifs:

```bash
➜  cryptodiy exiftool clear_dat
[...]
Handler Description             : CLUE->JACKET ;-)
[...]
Description                     : LSooRkxBRyA6IEVDU0N7c2hhMjU2KHNvbmcncyB0aXRsZSBpbiBsb3dlciBjYXNlIHdpdGhvdXQgc3BhY2UpfSkqLQo=
[...]

➜  cryptodiy echo -n 'LSooRkxBRyA6IEVDU0N7c2hhMjU2KHNvbmcncyB0aXRsZSBpbiBsb3dlciBjYXNlIHdpdGhvdXQgc3BhY2UpfSkqLQo=' | base64 -D
-*(FLAG : ECSC{sha256(song's title in lower case without space)})*-
```

Pour les plus vieux des personnes qui liront ce writeup, l'application `Snapchat` permet de reconnaitre une musique vu que Shazam est embarqué:



<img src="https://i.imgur.com/kdX0kwr.jpg" />




```bash
➜  cryptodiy echo -n 'iamthebest' | sha256sum
6693efcd0503059fc0561450033ac9fe712aff09e31e460f4f321f4324585188  -
```

## Flag

> ECSC{6693efcd0503059fc0561450033ac9fe712aff09e31e460f4f321f4324585188}
