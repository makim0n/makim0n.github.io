# [German FCSC] - Intro to reversing


# Intro to reversing 1

![](/lib/images/writeups/german_fcsc/introrev/upload_f0ef66205c9dab8a8195e0954ecf4ade.png)



## TL;DR

Le mot de passe est hardcodé dans le code. Un simple "strings" ou une inspection des appels systèmes permet d'avoir le mot de passe permettant d'obtenir le flag.

## Etat des lieux

L'épreuve commence avec un binaire Linux (ELF) demandans un mot de passe:

![](/lib/images/writeups/german_fcsc/introrev/upload_1361c5fb2b4eee23a05faa0e0013331d.png)


## Analyse dynamique

Un des outils les plus utilisés en analyse dynamique est "ltrace". Cette commande permet à un analyste de suivre les appels aux libraries au fur et à mesure de l'exécution du binaire.

Etant donné que le programme demande un mot de passe, on peut se douter qu'il utilise la fonction "strcmp":

![](/lib/images/writeups/german_fcsc/introrev/upload_16e37b51122e4bfa045fd5da12044731.png)

Finalement, les arguments du strcmp sont affichés et permet d'obtenir le flag de validation.


    
![](/lib/images/writeups/german_fcsc/introrev/upload_8f08288ffec3d48c409948ac0c4752e8.png)


## Flag

> CSCG{ez_pz_reversing_squ33zy}

# Intro to reversing 2



![](/lib/images/writeups/german_fcsc/introrev/upload_f01fa204f76c046261db8cdb646c37d6.png)



## TL;DR

Ce challenge de reverse porte sur le reverse d'un césar. C'est à dire que chaque caractère doit être décalé d'un offset de "119", ce résultat doit être mis au modulo 256 (nombre de caractère total) et finalement le flag apparait.

## Etat des lieux

On commence cette épreuve avec un binaire Linux 64 bits. Il n'est pas très gros et ne comporte pas énormément de fonctions. 



![](/lib/images/writeups/german_fcsc/introrev/upload_1683df2400f36564b279300f86fe8d03.png)


Comme dans le précédent challenge, le traitement doit se faire dans la fonction main. Pour s'en assurer, il est de bon de passer ce binaire dans un désassembleur (ida ftw).

Au premier abort on voit une boucle qui fait un "sub" de 0x77:



![](/lib/images/writeups/german_fcsc/introrev/upload_3ecd64849765305cd1e2c677d5779b06.png)


N'étant pas fan de l'assembleur, regardons ce qu'il se passe en pseudo code.

## Cesar

Grâce au décompilateur "Hexrays" de IDA, la boucle est bien plus clair et il est possible de voir qu'il s'agit d'un chiffrement de césar avec un décalage de 119 ou 0x77.



![](/lib/images/writeups/german_fcsc/introrev/upload_0b6ccb89b084ec3540398564ef776af3.png)


La chaine à retrouvé se trouve dans la section ".data" sous le doux nom de "s2".


    
![](/lib/images/writeups/german_fcsc/introrev/upload_2a59de91dc2d4da19b407fb6a3827073.png)


On a le chiffré, on a le décalage, il ne reste qu'à scripter.

## Décodage

Pour ça, le python est à mon sens le langage le plus approprié.

```python
res = ""
a = ["\xFC","\xFD","\xEA","\xC0","\xBA","\xEC","\xE8","\xFD","\xFB","\xBD","\xF7","\xBE","\xEF","\xB9","\xFB","\xF6","\xBD","\xC0","\xBA","\xB9","\xF7","\xE8","\xF2","\xFD","\xE8","\xF2","\xFC"]
for i in a:
    res += chr((ord(i)+119)%256)

print(res)
```

La liste "a" est parcouru et chaque caractère est converti en décimal. Ensuite l'opération inverse de celle présente dans le binaire est effectuée (on additionne au lieu de soustraire). Finalement, un modulo 256 est appliqué à ces nombres, car il existe 256 symboles dans la table ascii. 

## Flag

> CSCG{sta71c_tr4n5f0rm4710n_it_is}

# Intro to reversing 3



![](/lib/images/writeups/german_fcsc/introrev/upload_c4b27d3cb597a7de4e7953b64da86ecb.png)



## TL;DR

Comme les deux premiers, il s'agit d'une petite routine "crypto". En l'occurence un xor suivi d'un décalage. Même logique que précédemment, comprendre la routine, implémenter l'inverse et récupérer le flag.

## Etat des lieux

Encore une fois le binaire est très petit et ne contient pas beaucoup de fonctions. Le traitement se fait dans le main.

Avec la vue bloc de IDA, on peut identifier deux opérations:



![](/lib/images/writeups/german_fcsc/introrev/upload_9d66682b95bea539d79d7c37cf57df16.png)


Encore une fois, le décompilateur va nous être utile pour comprendre plus facilement ce qu'il se passe:


    
![](/lib/images/writeups/german_fcsc/introrev/upload_6cb1b91a034d191bc27def526c4bb3be.png)


La chaine chiffré se situé à nouveau dans ".data" sous le nom de "s2".



    
![](/lib/images/writeups/german_fcsc/introrev/upload_695271ca52fca941682a5d8d9e50556a.png)


Comme il y a des caractères imprimable, IDA va en faire une chaine de caractère, pour récupérer tout l'hex comme sur la capture précédente, il suffit d'appuyer sur "u", permettant à IDA de supprimer les traitements automatiques et revenir à un tableau d'octets brut.

## Déchiffrement du XOR

Comme vu précédemment, il y a un xor puis un décalage. Il suffit d'implémenter l'inverse pour récupérer le flag de validation:

```python
res = ""
a = ["\x6C","\x70","\x60","\x37","\x61","\x3C","\x71","\x4C","\x77","\x1E","\x6B","\x48","\x6F","\x70","\x74","\x28","\x66","\x2D","\x66","\x2A","\x2C","\x6F","\x7D","\x56","\x0F","\x15","\x4A"]
for i in range(len(a)):
    tmp = ord(a[i])+2
    res += chr(tmp^(i+10))

print(res)
``` 

## Flag

> CSCG{dyn4m1c_k3y_gen3r4t10n_y34h}
