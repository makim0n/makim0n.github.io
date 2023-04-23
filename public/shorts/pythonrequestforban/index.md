# Python3 Requests for fun and... Ban


## The goal

I always wanted to learn how to use `requests` library in Python3. It's very helpful in daily life or in CTF...

And actually, there is `EuropeanCyberWeek` CTF, so let's try to do something useful: __Auto flag submitter__

## State of the art

First of all, I need the requests structure, to get this information, just open `BurpSuite` (if you're a bit stupid like me), or you're `Firefox web console` and request the chosen challenge (`Drone Wars 3` for example).

```
POST /chal/19 HTTP/1.1
Host: challenge-ecw.fr
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:62.0) Gecko/20100101 Firefox/62.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://challenge-ecw.fr/challenges
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 143
Cookie: session=<JSON Web Token>
Connection: close

key=test&nonce=a36e189c9edf96b7ff561b1dc65bf6e0715af51eb5cb46f36a63d45c57665a24859242501669a17cac47eec04e1dc4e3f1646abd028d32a4e629929f1d0e816d
```

We have few interesting things:

* Request: challenge-ecw.fr/chal/19
* URL: `https://challenge-ecw.fr/challenges#DroneWars%20(3/3)`
* Cookie: JSON Web Token
* key: The submitted flag (here is `test`)
* nonce: Random number to avoid CSRF vulnerability

## Python requests

Ok, now we know how to do our script. Start to install the desired library:

```bash
$ pip install --user Requests
```

Initialize a `Session()` object, this object will get all headers and after you can do `GET` or `POST` requests normally.

I had to do a little trick to get the `nonce` random:

```python
>>> import requests
>>> import re
>>> s = requests.Session()

>>> cookie = {'session':'.eJwNz0FrgzAUwPGv[...]52miEJF5lBEfaqEd1dHs'}
>>> url = 'https://challenge-ecw.fr/challenges#DroneWars%20(3/3)'
>>> html = s.get(url, cookies=cookie).text.split('\n')

>>> for i in html:
>>>     if re.search('nonce', i):
>>>         nonce=i.split('=')[4][:-1]

>>> print(nonce)

36e189c9edf96b7ff561b1dc65bf6e0715af51eb5cb46f36a63d45c57665a24859242501669a17cac47eec04e1dc4e3f1646abd028d32a4e629929f1d0e816d
```

It works! Now we just have to do a little `POST` requests with our `key` and `nonce`.

```python
>>> data = {'key':'ECW{Bite de poulet}', 'nonce':nonce}
>>> print(s.post('https://challenge-ecw.fr/chal/19', cookies=cookie, data=data).text)
{
  "status": "2",
  "message": "Incorrect."
}
```

I flagged a challenge to see if the message was different, and it was. Instead of "Incorrect", you got "Correct" (`CTFd` guys are very smart :D).

## Full code

```python
#!/usr/bin/python3

import requests
import re
import time

s = requests.Session()

cookie = {'session':'.eJwNz0FrgzAUwP[...]JF5lBEfaqEd1dHs'}
url = 'https://challenge-ecw.fr/challenges#DroneWars%20(3/3)'
html = s.get(url, cookies=cookie).text.split('\n')

for i in html:
    if re.search('nonce', i):
        nonce=i.split('=')[4][:-1]

data = {'key':'ECW{Bite de poulet}', 'nonce':nonce}

g = open('counter.txt', 'r')
j = int(g.read().split('\n')[0])
g.close()

while(1):
    coucou = s.post('https://challenge-ecw.fr/chal/19', cookies=cookie, data=data)
    nope = coucou.text
    if nope.split('"')[7] == "Incorrect":
        j = j+1
        f = open('counter.txt', 'w')
        f.write(str(j)+'\n')
        f.write(str(nope))
        f.close()
    else:
        f = open('counter.txt', 'a')
        f.write(str(nope))
        f.close()
        exit()
    time.sleep(600)
```

I know, mine has more features... But I got a new `status` :O

```bash
$ ./coucou.py
{
  "status": "3",
  "message": "You are banned."
}
```

Maybe the flag was `ECW{MD5(Bite de poulet)}`...

## Conclusion

To conclude this little blog post, from a personal point of view, I found `ECW CTF` boring. CTF are used to learn things, useful or not, nevermind. Here, we have guessing and bruteforce challenge...

Well, I'm actually banned, have fun. But don't overuse my little spammer, try to be eligible for the final it's really interesting, I did it last year (2017).

Thanks reading this, take care of yourself <3
