# [ECSC] - Petites notes


## Description

> Connaissez-vous bien le format PCAPng ?

## Etat des lieux

On commence donc ce challenge avec une capture réseau. Premier réflexe de CTF avec un format de flag : `strings`.

```bash
➜  petitenote hexdump -C petites_notes.pcapng | grep -C 10 'ECSC'
001b3f80  00 0a 00 00 2f 03 6e 73  31 05 6b 61 73 65 63 02  |..../.ns1.kasec.|
001b3f90  61 74 00 0a 68 6f 73 74  6d 61 73 74 65 72 c0 10  |at..hostmaster..|
001b3fa0  5c c0 3a 9f 00 00 40 00  00 00 08 00 00 10 00 00  |\.:...@.........|
001b3fb0  00 00 0a 00 bc 00 00 00  06 00 00 00 80 00 00 00  |................|
001b3fc0  00 00 00 00 5e c2 99 15  a8 8b 11 2f 4a 00 00 00  |....^....../J...|
001b3fd0  4a 00 00 00 a4 3e 51 0c  ed 79 bc ee 7b 2e 15 90  |J....>Q..y..{...|
001b3fe0  08 00 45 00 00 3c 7d 96  40 00 40 06 6e 95 c0 a8  |..E..<}.@.@.n...|
001b3ff0  01 19 34 a6 58 29 83 5e  01 bb 05 ef b9 51 00 00  |..4.X).^.....Q..|
001b4000  00 00 a0 02 72 10 7e 3a  00 00 02 04 05 b4 04 02  |....r.~:........|
001b4010  08 0a 00 03 c4 c7 00 00  00 00 01 03 03 07 00 00  |................|
001b4020  01 00 09 00 45 43 53 43  7b 63 53 68 6c 00 00 00  |....ECSC{cShl...|
001b4030  00 00 00 00 80 00 00 00  06 00 00 00 6c 00 00 00  |............l...|
001b4040  00 00 00 00 5e c2 99 15  79 e0 14 2f 4a 00 00 00  |....^...y../J...|
001b4050  4a 00 00 00 a4 3e 51 0c  ed 79 bc ee 7b 2e 15 90  |J....>Q..y..{...|
001b4060  08 00 45 00 00 3c fa 0e  40 00 40 06 f2 1c c0 a8  |..E..<..@.@.....|
001b4070  01 19 34 a6 58 29 83 60  01 bb 8f fc 3e c9 00 00  |..4.X).`....>...|
001b4080  00 00 a0 02 72 10 6e b3  00 00 02 04 05 b4 04 02  |....r.n.........|
001b4090  08 0a 00 03 c4 c7 00 00  00 00 01 03 03 07 00 00  |................|
001b40a0  6c 00 00 00 06 00 00 00  6c 00 00 00 00 00 00 00  |l.......l.......|
001b40b0  5e c2 99 15 c6 66 17 2f  4a 00 00 00 4a 00 00 00  |^....f./J...J...|
001b40c0  a4 3e 51 0c ed 79 bc ee  7b 2e 15 90 08 00 45 00  |.>Q..y..{.....E.|
```

On peut voir le début du flag, mais il n'apparaît pas directement dans les communications:



![](https://i.imgur.com/9oZe5TJ.png)

_Fig 1_: Recherche de la chaine "ECSC"



Je me suis donc mis à la recherche d'une chaîne à coté, comme `e5dH`:



![](https://i.imgur.com/tdXiBDV.jpg)

_Fig 2_: Début et fin de paquets



Donc le début de flag, et probablement la suite, se trouve "entre" des paquets. Maintenant, il va falloir RTFM les fonctionnalités de Wireshark.

> https://packetu.com/2013/06/18/using-the-wireshark-commenting-feature/

Un peu de `tshark` et voilà notre flag :

![](https://i.imgur.com/jE9JWHS.png)

## Flag

> ECSC{cShle5dOKYBfjLNzT}
