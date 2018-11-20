## Special Force - Web challenge ( 100 points)

This is a web challenge and we are provided with an URL:http://fun.ritsec.club:8005/.
Upon opening we can see that we are presented with **Ship leaderboard** If we enter one of the presented ships, we can see its records.
If we type **'**, the text changed to *Something went wrong with your record query! What are you trying to do???* which clearly indicates that this is a SQL injection.

This challenge is an easy one, we can just query all records just by running this **' or 'x'='x** . 

The flag is **RITSEC{hey_there_h4v3_s0me_point$_3ny2Lx}**  
