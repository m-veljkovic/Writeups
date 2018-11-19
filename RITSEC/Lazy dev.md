## Lazy dev - Web challenge (400 points)

The final web challenge with a link: http://fun.ritsec.club:8007. Hmm it takes us back to *The tangled web* challenge.
At first, I was thinking that it was a mistake, but after reviewing all pages I crawled, I saw a comment which lead us further.
Comment was found in http://fun.ritsec.club:8007/Stars.html and it goes like this: 

```
<!-- REMOVE THIS NOTE LATER -->
<!-- Getting remote access is so much work. Just do fancy things on devsrule.php --> 
```
So I went to http://fun.ritsec.club:8007/devsrule.php and I am welcomed with:
```
Not what you input eh?
This param is 'magic' man.
```
Well, it is said that the parameter is *magic*. 

I started poking around and after a while I finally figured out that it is LFI to RCE ! 
We have to use **php://input** wrapper !

I intercepted the GET request with Burp Suite and changed to *_POST /devsrule.php?magic=php://input HTTP/1.1_* .
Next thing was to add POST data so I ran ```<?php system('id'); ?>``` and I got this response: 
**Not what you input eh?<br>This param is 'magic' man.<br><br>uid=33(www-data) gid=33(www-data) groups=33(www-data)**
Next step was to find the flag. With simple poking around we found our flag in *_/home/joker/flag.txt_*

To read flag we execute ```<?php system('cat /home/joker/flag.txt'); ?>``` and we got flag in response!

Flag is: **RITSEC{WOW_THAT_WAS_A_PAIN_IN_THE_INPUT}**

