# [French FCSC] - Cryptolocker


>Un de nos admins nous a appelé en urgence suite à un CryptoLocker qui s'est lancé sur un serveur ultra-sensible, juste après avoir appliqué une mise à jour fournie par notre prestataire informatique.
>
>Ce malware vise spécifiquement un fichier pouvant faire perdre des millions d'euros à notre entreprise : il est très important de le retrouver !
>
>L'administrateur nous a dit que pour éviter que le logiciel ne se propage, il a mis en pause le serveur virtualisé et a récupéré sa mémoire vive dès qu'il a détecté l'attaque.
>Vous êtes notre seul espoir.
>
>SHA256(memory1.dmp.gz) = 39e31ca61067e51f9566f0b669ac1ddbd09924a6f7720dcb08c9ac46cee186f5.

## TL;DR

Un dump mémoire Windows 7 où un ransomware a été exécuté. Le processus "updater" est le ransomware. Il est possible de récupérer les fichiers de flag et de clé dans les dossiers de l'utilisateur courant. Un peu de rétro-ingénierie sur le binaire montre que c'est un xor un peu modifié.

## État des lieux

On commence cette épreuve avec un dump de la mémoire d'un Windows visiblement:

```bash
➜  file memory.dmp 
memory.dmp: MS Windows 32bit crash dump, PAE, full dump, 262030 pages
```

On sort notre plus beau volatility et on cherche le bon profil:

```bash
➜  python volatility/vol.py -f /opt/usr_land/memory.dmp imageinfo
Volatility Foundation Volatility Framework 2.6.1
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86_24000, Win7SP1x86 (Instantiated with WinXPSP2x86)
                     AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                     AS Layer2 : WindowsCrashDumpSpace32 (Unnamed AS)
                     AS Layer3 : FileAddressSpace (/opt/usr_land/memory.dmp)
                      PAE type : PAE
                           DTB : 0x185000L
             KUSER_SHARED_DATA : 0xffdf0000L
           Image date and time : 2020-04-13 18:39:35 UTC+0000
     Image local date and time : 2020-04-13 11:39:35 -0700
```

Le profile à utiliser est:

> Win7SP1x86_23418

Avant de partir comme un débile dans le dump, il faut se poser et se demander ce qu'on sait et ce qu'on cherche.
Ce qu'on sait:

* Déclenchement juste après l'exécution d'une __mise à jour__ ;
* Le malware vise un fichier (flag ?) ;
* Dump mémoire Windows 7.

Ce qu'on cherche:

* Le malware ;
* Le fichier chiffré ;
* Le mécanisme de chiffrement.

Aller, yapluka.

## Localiser le malware

Comme le malware a été exécuté après une mise à jour, voyons ce que les processus donnent:

```bash
➜  python volatility/vol.py -f /opt/usr_land/memory.dmp --profile=Win7SP1x86_23418 pstree
Volatility Foundation Volatility Framework 2.6.1
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
 0x85093c90:explorer.exe                             1432   1320     28    756 2020-04-13 18:37:17 UTC+0000
[...]
. 0x83de43a8:update_v0.5.ex                          3388   1432      2     61 2020-04-13 18:38:00 UTC+0000
[...]
```

Je ne suis pas un expert Windows, mais de mémoire le processus de mise à jour, ce n’est pas du tout `update_v0.5.exe`. On peut essayer de l'extraire et regarder rapidement ce que c'est:

```bash
➜  python volatility/vol.py -f /opt/usr_land/memory.dmp --profile=Win7SP1x86_23418 procdump -p 3388 -D /opt/usr_land/Wu/
Volatility Foundation Volatility Framework 2.6.1
Process(V) ImageBase  Name                 Result
---------- ---------- -------------------- ------
0x83de43a8 0x00400000 update_v0.5.ex       OK: executable.3388.exe
```

Alors déjà, qu'est ce que c'est que ce fichier:

```bash
➜  file executable.3388.exe 
executable.3388.exe: PE32 executable (console) Intel 80386, for MS Windows
➜  strings executable.3388.exe
[...]
[info] entering the folder : %s
flag.txt
[info] file encryptable found : %s
	ENCRYPTOR v0.5
key.txt
[error] can't read the key-file :s
****Chiffrement termin
e ! Envoyez l'argent !
[...]
```

Déjà, pas de bol, ce n’est pas du .net. Ensuite, le binaire n’a pas l'air super friendly. Il parle de "flag.txt" et "key.txt".

## Récupération du flag chiffré et de la clé

On va voir si ces fichiers sont présents en mémoire:

```bash
➜  python volatility/vol.py -f /opt/usr_land/memory.dmp --profile=Win7SP1x86_23418 filescan > /opt/usr_land/Wu/filescan
Volatility Foundation Volatility Framework 2.6.1
➜  cat filescan| grep -E "flag\.txt|key\.txt"
0x000000003e6fa100      8      0 RW-rw- \Device\HarddiskVolume1\Users\IEUser\Desktop\key.txt
0x000000003ed13898      2      1 R--rw- \Device\HarddiskVolume1\Users\IEUser\Desktop\key.txt
0x000000003ed139f0      2      0 RW-rw- \Device\HarddiskVolume1\Users\IEUser\Desktop\flag.txt.enc
```

Il semblerait que ces fichiers soient présents. Par acquit de conscience, on peut regarder ce qu'il y a d'autre sur le bureau de l'utilisateur:

```bash
➜  cat filescan| fgrep '\IEUser\Desktop\'
0x000000003e6fa100      8      0 RW-rw- \Device\HarddiskVolume1\Users\IEUser\Desktop\key.txt
0x000000003e90b1e8      3      0 R--r-d \Device\HarddiskVolume1\Users\IEUser\Desktop\DumpIt.exe
0x000000003eb40430      8      0 R--rwd \Device\HarddiskVolume1\Users\IEUser\Desktop\desktop.ini
0x000000003ed13898      2      1 R--rw- \Device\HarddiskVolume1\Users\IEUser\Desktop\key.txt
0x000000003ed139f0      2      0 RW-rw- \Device\HarddiskVolume1\Users\IEUser\Desktop\flag.txt.enc
0x000000003ed66b60      6      0 R--r-d \Device\HarddiskVolume1\Users\IEUser\Desktop\update_v0.5.exe
0x000000003fdec360      1      1 RW-rw- \Device\HarddiskVolume1\Users\IEUser\Desktop\IEWIN7-20200413-183930.dmp
```

Bon, bah le malware est là aussi, ce n'était pas obligatoire de l'extraire des process. Il est temps de récupérer les deux fichiers qui nous intéressent.

```bash
➜  python volatility/vol.py -f /opt/usr_land/memory.dmp --profile=Win7SP1x86_23418 dumpfiles -Q 0x000000003ed13898 -D /opt/usr_land/Wu/
Volatility Foundation Volatility Framework 2.6.1
DataSectionObject 0x3ed13898   None   \Device\HarddiskVolume1\Users\IEUser\Desktop\key.txt
SharedCacheMap 0x3ed13898   None   \Device\HarddiskVolume1\Users\IEUser\Desktop\key.txt

➜  python volatility/vol.py -f /opt/usr_land/memory.dmp --profile=Win7SP1x86_23418 dumpfiles -Q 0x000000003ed139f0 -D /opt/usr_land/Wu/
Volatility Foundation Volatility Framework 2.6.1
DataSectionObject 0x3ed139f0   None   \Device\HarddiskVolume1\Users\IEUser\Desktop\flag.txt.enc
```

Une analyse rapide des fichiers:

```bash
➜  cat file.None.0x854fbc98.dat 
0ba883a22afb84506c8d8fd9e42a5ce4e8eb1cc87c315a28dd

➜  cat file.None.0x855651e0.dat
'{kp�U]
       SUU	]Y^\TQU^UWR[W\QTPQ
                                  ^UQUVYZWQRWZP%                                                                                                                   
                                  
➜  cat file.None.0x855651e0.dat | wc -c
4096
```

C'est étrange d'avoir une taille de 4096, mais pas beaucoup de bytes à l'affichage. Un rapide hexdump montre qu'il y a plein de null bytes (merci volatility, je suppose qu'une page mémoire c'est 4Kb, et qu'il pad avec du \x00).

```bash
➜  hexdump -C file.None.0x855651e0.dat 
00000000  27 7b 6b 70 1a 01 00 55  05 07 5d 0c 53 55 05 55  |'{kp...U..].SU.U|
00000010  09 5d 59 5e 06 5c 04 02  06 54 07 51 00 55 01 5e  |.]Y^.\...T.Q.U.^|
00000020  55 57 52 5b 57 5c 51 54  50 07 51 07 0b 5e 55 51  |UWR[W\QTP.Q..^UQ|
00000030  55 56 02 59 5a 07 05 02  57 51 52 01 0f 03 57 02  |UV.YZ...WQR...W.|
00000040  06 01 5a 50 0f 1b 6e 00  00 00 00 00 00 00 00 00  |..ZP..n.........|
00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00001000
```

On récupère les bytes qui nous intéressent et on en fait un fichier sans null bytes. Et juste pour le fun on sort un sexy one liner:

```bash
➜  hexdump -C file.None.0x855651e0.dat | cut -c11-60 | tr -d '\n ' | sed 's/0000//g' | sed 's/..$//g' | xxd -r -p | wc -c
71
```

On sait que ce n’est pas de l'AES, vu que ce n'est pas un multiple de 16. Par intuition, je ne pense pas que ce soit du RSA, EC ou autre chiffrement asymétrique. Donc on va se laisser penser que c'est un xor. Mais il est temps de reverse le malware. 

## Reverse du malware

Notre petit pote il ouvre bien le fichier "key.txt" et le stock dans un buffer:



![](https://i.imgur.com/3DuIzUO.png)



La fonction que j'ai renommée `encrypt_n_xor` prend le chemin du flag en paramètre:



![](https://i.imgur.com/wtMDg13.png)



En fin de compte, l'hypopthèse de départ était bonne, c'est bien un xor:



![](https://i.imgur.com/pq066hs.png)



C'est donc un xor où la clé est parcourue avec un décalage de +2 octets.

Si on récapitule ce qu'on cherchait au départ:

* Le malware -> Le binaire "update_v0.5.exe" ; 
* Le fichier chiffré -> Le fichier "flag.txt.enc" ;
* Le mécanisme de chiffrement -> Un xor un peu revisité avec le fichier "key.txt".

## Déchiffrer le flag

Il ne reste qu'à déchiffrer:

```python
from ENO import *
key = b"0ba883a22afb84506c8d8fd9e42a5ce4e8eb1cc87c315a28dd"
enc_dat = htos("277b6b701a01005505075d0c53550555095d595e065c0402065407510055015e5557525b575c5154500751070b5e5551555602595a070502575152010f03570206015a500f1b6e")
flag = ""

for i in range(len(enc_dat)):
    flag += chr(enc_dat[i] ^ key[(i+2)%len(key)])

print(flag)
```

## Flag

> FCSC{324cee8fe3619a8bea64522eadf05c84df7c6df9f15e4cab4d0e04c77b20bb47}

## Bonus

Il est possible d'utiliser le plugin "mft_parser" de volatility pour récupérer le contenu de "key.txt" et "flag.txt.enc". J'utilise toujours ce plugin au début d'un challenge de mem Windows, des fois que le flag soit dans les fichiers résidents :D 
