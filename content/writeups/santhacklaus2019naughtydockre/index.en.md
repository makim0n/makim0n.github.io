---
weight: 4
title: "[Santhacklaus 2019] - Naughty Docker"
date: 2019-12-22T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["santhacklaus", "claranet", "forensic", "docker"]
categories: ["Writeups"]

lightgallery: true
---

> It looks like a naughty developer has been deploying a Docker image on a Santa production server a few days before Christmas. He was in a rush and was not able to properly pass all security checks on the built Docker image. Would be a shame if this image could give you an SSH access to the production server... http://46.30.204.47"

## Etat des lieux

En allant sur l'IP donnée dans l'énoncé, on trouve une note nous donnait accès au docker en question.

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_2f8aac3f4d10eadee50622e089a83af7.png)

Avec cette note, on sait que l'on cherche un accès SSH donc potentiellement:

* username ;
* password et / ou clé ;
* adresse IP de la machine de prod ;
* port ssh si non standard (22/TCP).

La première chose à faire est donc de récupérer ce docker:

```bash
> docker run --rm -p 3000:3000 -d santactf/app
Unable to find image 'santactf/app:latest' locally
latest: Pulling from santactf/app
844c33c7e6ea: Pull complete 
ada5d61ae65d: Pull complete 
f8427fdf4292: Pull complete 
f025bafc4ab8: Pull complete 
7a9577c07934: Pull complete 
add4f74c413b: Pull complete 
1ee7a33fb93f: Pull complete 
08ab1881dcea: Pull complete 
96f3027f0dbd: Pull complete 
cb67eac57f41: Pull complete 
bf44330d5df8: Pull complete 
4932e843cace: Pull complete 
f0b9c596601c: Pull complete 
Digest: sha256:621c884f7ddd0351fbb114e0b9c1d4d3b0e309cb5c5efc9ce872fd201af79cad
Status: Downloaded newer image for santactf/app:latest
fe704e6f41dc44e27d32d4b6de9d64cc7bf520cfcdc94c2c15b7e35acc3214b0

> docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
fe704e6f41dc        santactf/app        "docker-entrypoint.s…"   51 seconds ago      Up 49 seconds       0.0.0.0:3000->3000/tcp   cranky_khorana
```

Avant de commencer dans le vif du challenge, une petite recherche google a permis d'avoir des pistes à creuser:

> https://www.intrinsec.com/docker-leak/

Même si leur outil a l'air vraiment jolie, on va le faire à l'ancienne: avec du bash et du cervelet.

## Différence entre docker save et export

Avant de faire un docker save de notre image, il est important de s'intéresser à la différence entre un docker save et un docker export. Les deux ont plus ou moins la même finalité: sauvegarder un container et le redéployer.

La différence majeure est qu'un docker export ne va pas garder les meta données, ni les layer d'un container. Quant à un docker save, il va tout récupérer et faire un beau tar avec.

Le tool de intrinsec se base sur les layer du container pour permettre à l'utilisateur de chercher à l'intérieur et ainsi récupérer des secrets.

Cet article en parle bien : https://tuhrig.de/difference-between-save-and-export-in-docker/

## Sauver le soldat Santa

On peut voir sur l'image suivante, que l'archive tar générée par `docker save` contient l'architecture suivante:

* __hash.tar__: un hash par layer;
    * __json__: l'état du docker au moment du layer;
    * __layer.tar__: l'ensemble des fichiers du layer;
    * __VERSION__: la version du docker.
* __hash.json__: un "résumé" de tous les layers générés et les commandes associés;
* __manifest.json__: répertorie tous les hashs des différents layers;
* __repositories__ (format json): contient le hash de la dernière version du docker.

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_0904433c031ddbc64f15130b902b9119.png)

Les données que l'on chercher doivent être dans un des "layer.tar". 

## Localiser les fichiers utiles

La première chose à faire est de lister l'ensemble des fichiers dans les différents "layer.tar", ça sera plus simple pour grepper dedans après:

```bash
$ for i in $(ls | grep -E '^[a-f0-9]{64}$'); do tar -tf $i/layer.tar |tee "$i"_out; done

$ ls |grep out
0425a7cec03536e53ffcd89abeab827ee12987209a0d31085b43cf01b8c8b2cb_out
05f684efc5012328cdbde4e3814eed1240f41d06e4dcb44506705a7705cf199e_out
34c306ec137d785a0422942d8960b913fa7ff1ec66339b699bb3de02ce6770c6_out
3b139c8076d9c5ea6935edb9f644a4dd9caec1acb1217b9492fae4bbe3ff3725_out
5c1926c54c7194b9886f0007f2cfa0a9e166b8985842e55fc9d8e888254663de_out
986a7d85c875d79a1cd37dcd7e4e110cbf6ede072d062dfb0eaf10f006ebefd9_out
a67d35ec1f3fd055009a18e40d35ccbbb31da927e5d2c421ec43e98db0d08678_out
a934c4c71d618bab83432e24a12640ff9d474df3529fa790cb0f9d059d82c7dd_out
b4d27d4e4f5bb8bfe5bc92f8b3cbf3f0f5042fe120a40833fc0247deb728f961_out
b5116739ab4c01c43f0ba2feab4358c367ca86c085fad5223ce03d6bb7933fe3_out
be3d4ffa7682700bcbc51a8655568428c4979c5464169a286208e9e03f7673a5_out
c03933430e12aa2893ee247934be44c86f5c1878904d7a1bfe665ccd548bde5d_out
f0e0774e1b8e3c943a4b910e773664c81cfcadda8a388040c842fb37b8a0e467_out
```

Dans le fichier `ddde36e2209357c424cca26ac5a0b46c2f864be797c053bed700422177ba7261.json`, le résumé de tous les layers, on peut voir que l'utilisateur supprime le `.bash_history` et le `.bashrc`:

```bash
$ cat ddde36e2209357c424cca26ac5a0b46c2f864be797c053bed700422177ba7261.json | jq
```

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_b1c91ed63580e8e28ebaabccbd9cdb4f.png)

Maintenant, il faut localiser un "layer.tar" contenant un `.bash_history`:

```bash
$ grep -ari '/\.bash..' *_out
```

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_efabbd1d5b9e45487f16e8e1194e905e.png)

Il semblerait que notre archive soit le layer: `be3d4ffa7682700bcbc51a8655568428c4979c5464169a286208e9e03f7673a5`

## Extraction des premières informations

Il semblerait que la logique soit bonne:

```bash
$ cd be3d4ffa7682700bcbc51a8655568428c4979c5464169a286208e9e03f7673a5 && tar xvf layer.tar
```

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_afe991c5f2bf8c1e49a8f3aa39e8c3aa.png)

Les fameux fichiers supprimés, mais aussi des archives de backup, ça sent bon. Avec un simple `cat` sur ce fichier, on voit qu'il a fait beaucoup de commandes:

```bash
$ cat .bash_history |wc -l
153
```

Mais on sait qu'on cherche de quoi se connecter en ssh:

```bash
$ cat .bash_history | grep ssh
```

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_4e1b7bf3f0a60323447cce9109c089c3.png)

Dans la liste des informations à trouver, on vient de récupérer:

* username: rudolf-the-reindeer ;
* adresse IP de la machine de prod: 46.30.204.47 ;
* port ssh si non standard (22/TCP): 5700.

Il nous manque donc un mot de passe ou une clé. Normalement, c'est à ce moment qu'on se demande "Mais que contiennent les backups ?":

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_c8357e1edc0bcb36d507f7cb1c032116.png)

Ca tombe bien. Pour info, un `unzip -l *` ne fonctionne pas, je ne me suis pas amusé à faire un `for` pour le plaisir.

De toute façon, nous sommes face à une archive chiffrée:

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_d22311b9aad612d1b436487bd861204b.png)

## Déchiffrement du zip

S'il a créé un zip chiffré, il a dû faire la commande `zip` à un moment donné. Regardons dans le `.bash_history`:

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_9cfe412881b3b3f49d139477dd7de276.png)

Une variable d'environnement pour le password, ça nous fait une belle jambe. Il n'y a plus qu'à espérer que cette variable ait été initialisée dans ce même bash:

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_39a70f75cdab1e9e06b14c03bfdac395.png)

On a encore eu de la chance. Le mot de passe du zip est donc `25362`. Lors de l'extraction d'un zip, quelque chose de rigolo se passe:

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_7c428c87cc72aa9956bcd8f1dcf1186b.png)

Après, il y a 9 zip, c'est peut être la clé de l'un d'entre eux. Pour celà, bash for the win:

```bash
$ for i in $(ls); do echo $i && 7z t $i -p25362|grep Ok; done
```

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_aeb27beeded836e41cc8e22194b5ce73.png)

Il semblerait qu'une archive puisse être déchiffrée: 

> dev_141219_backup.zip

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_deab0aea70f4aa4fd66bca331172fc83.png)

Bonne nouvelle... Enfin presque:

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_ad0d545d64bb6b3538ebb23050977620.png)


Il semblerait que l'authentification SSH ait besoin d'une clé ET d'un mot de passe.

## El famoso passwordo

Je dois avouer que j'ai passé un peu de temps à chercher ce mot de passe dans les autres layers, bruteforce sans succès les autres zip pour voir si la clé changeait, etc... Mais au final, je me suis rappelé qu'à la création de ce layer, l'utilisateur avait supprimé 2 fichiers:

* `.bash_history`
* `.bashrc`

Que contient le `.bashrc`:

```bash
$ cat .bashrc|grep -v '#'
```

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_e21ffd52a175782c534915243a3e0d6f.png)

La fameux mot de passe: 

> HoHoHo2020!NorthPole

Il ne reste qu'à se connecter et collecter ce flag tant attendu:

![](/lib/images/writeups/2019_santhacklaus/naughtydocker/upload_af912408ac7271a7fb45d76988dd51ba.png)

## Flag

> SANTA{NeverTrustDockerImages7263}