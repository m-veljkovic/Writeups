## The tangled web - Web challenge (200 points)

This was a second web challenge, with an URL: http://fun.ritsec.club:8007.

The main thing here was to crawl complete webpage and inspect responses. I used Burp Suite for that task.

At first, URL that was different was http://fun.ritsec.club:8007/Fl4gggg1337.html, unfortunately, that was a troll.
Second thing that is worth paying attention was http://fun.ritsec.club:8007/Stars.html. 
It contains a paragraph that is encoded in base64. 
**UklUU0VDe0FSM19ZMFVfRjMzNzFOR18xVF9OMFdfTVJfS1I0QjU/IX0=**, we can decode this using *Decoder* tab in Burp Suite or running this command in terminal: *_echo "UklUU0VDe0FSM19ZMFVfRjMzNzFOR18xVF9OMFdfTVJfS1I0QjU/IX0=" | base64 -d_*.

Given output is our flag: **RITSEC{AR3_Y0U_F3371NG_1T_N0W_MR_KR4B5?!}**
