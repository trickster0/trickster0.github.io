---
layout: post
title: 'Tr0ll2: The Revenge Of The Tr0ll!!'
tags:
- hacking
- lmao
- tr0ll2
- vm
- vulnhub
- zip
---
Hello everyone this is tr0ll 2 as i promised. Time to get some root access on the server, cause i didnt do much these days, so i will stop blabbing and start to explain what is going on and how everything happened... ;)

Of course as always i started an nmap scan to our dear tr0ll server>root@kali:~# nmap 192.168.124.131 -sV

Starting Nmap 6.49BETA4 ( https://nmap.org ) at 2015-10-12 18:13 EDT
Nmap scan report for 192.168.124.131 (192.168.124.131)
Host is up (0.00016s latency).
Not shown: 997 closed ports
PORT STATE SERVICE VERSION
21/tcp open ftp vsftpd 2.0.8 or later
22/tcp open ssh OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
80/tcp open http Apache httpd 2.2.22 ((Ubuntu))
MAC Address: 00:0C:29:7C:A4:A9 (VMware)
Service Info: Host: Tr0ll; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.51 seconds

so this is where i started the ftp enumeration. I tried a couple of passwords and guessing. i actually got in and checked for the files and downloaded the only one i found there.


[![troll21](https://trickster0.files.wordpress.com/2015/10/troll21.png)](https://trickster0.files.wordpress.com/2015/10/troll21.png)

the file was password protected to i left hanging around for later... :/
lets go to the http service enumeration. this is the index page

[![troll22](https://trickster0.files.wordpress.com/2015/10/troll22.png)](https://trickster0.files.wordpress.com/2015/10/troll22.png)

the source didn't contain anything interesting so lets move ahead. i thought to check the robots.txt and ok this is not a damn robots.txt file this is a huge list!!!i mean come on!!!

[![troll23](https://trickster0.files.wordpress.com/2015/10/troll23.png)](https://trickster0.files.wordpress.com/2015/10/troll23.png)
so because of this big list after i tried a few paths, they were fake.... -_- so i made a list and checked it with dirb

more

>root@kali:~/Desktop# cat locf
noob
nope
try_harder
keep_trying
isnt_this_annoying
nothing_here
404
LOL_at_the_last_one
trolling_is_fun
zomg_is_this_it
you_found_me
I_know_this_sucks
You_could_give_up
dont_bother
will_it_ever_end
I_hope_you_scripted_this
ok_this_is_it
stop_whining
why_are_you_still_looking
just_quit
seriously_stop


>root@kali:~/Desktop# dirb http://192.168.124.131 locf

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Mon Oct 12 18:20:40 2015
URL_BASE: http://192.168.124.131/
WORDLIST_FILES: locf

-----------------

GENERATED WORDS: 21

---- Scanning URL: http://192.168.124.131/ ----
==> DIRECTORY: http://192.168.124.131/noob/
==> DIRECTORY: http://192.168.124.131/keep_trying/
==> DIRECTORY: http://192.168.124.131/dont_bother/
==> DIRECTORY: http://192.168.124.131/ok_this_is_it/

---- Entering directory: http://192.168.124.131/noob/ ----

---- Entering directory: http://192.168.124.131/keep_trying/ ----

---- Entering directory: http://192.168.124.131/dont_bother/ ----

---- Entering directory: http://192.168.124.131/ok_this_is_it/ ----

-----------------
END_TIME: Mon Oct 12 18:20:40 2015
DOWNLOADED: 105 - FOUND: 0

after checking the directories...

[![troll24](https://trickster0.files.wordpress.com/2015/10/troll24.png)](https://trickster0.files.wordpress.com/2015/10/troll24.png)

[![troll25](https://trickster0.files.wordpress.com/2015/10/troll25.png)](https://trickster0.files.wordpress.com/2015/10/troll25.png)
i was in a dead end so i checked the size of the pictures of the paths what do you know...there is a fatter troll cat!!! it seems it has eaten a bit more data that it should... ^_^

>root@kali:~/Desktop# wget http://192.168.124.131/dont_bother/cat_the_troll.jpg
--2015-10-12 18:23:23-- http://192.168.124.131/dont_bother/cat_the_troll.jpg
Connecting to 192.168.124.131:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15873 (16K) [image/jpeg]
Saving to: ‘cat_the_troll.jpg’

cat_the_troll.jpg 100%[=====================>] 15.50K --.-KB/s in 0s

2015-10-12 18:23:23 (338 MB/s) - ‘cat_the_troll.jpg’ saved [15873/15873]

root@kali:~/Desktop# file cat_the_troll.jpg
cat_the_troll.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, baseline, precision 8, 500x302, frames 3
root@kali:~/Desktop# strings cat_the_troll.jpg
JFIF
#3-652-108?QE8<M=01F`GMTV[\[7DcjcXjQY[W
)W:1:WWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWWW
"aq2
\vRH
sdwTi
aDP
aDP
\z!$
`aDc
(Q@0S
}}HQ\)
...snippet...
8Jh;
gYCJ
pV}A
7U 4
]=%em;
lj\p
*/ p?E$
Look Deep within y0ur_self for the answer

the "y0ur_self" word seems like a password or path... who knows... lets check it as a directory

[![troll26](https://trickster0.files.wordpress.com/2015/10/troll26.png)](https://trickster0.files.wordpress.com/2015/10/troll26.png)
AN ANSWER.TXT FILE!!! lets see if it is another troll or a real thing

>root@kali:~/Desktop# wget http://192.168.124.131/y0ur_self/answer.txt
--2015-10-12 18:26:09-- http://192.168.124.131/y0ur_self/answer.txt
Connecting to 192.168.124.131:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1412653 (1.3M) [text/plain]
Saving to: ‘answer.txt'

answer.txt 100%[=====================>] 1.35M --.-KB/s in 0.02s

2015-10-12 18:26:09 (74.0 MB/s) - ‘answer.txt’ saved [1412653/1412653]

root@kali:~/Desktop# cat answer.txt
QQo=
QQo=
QUEK
QUIK
QUJNCg==
QUMK
QUNUSAo=
QUkK
QUlEUwo=
QU0K
QU9MCg==
....snippet.....

This is totally encrypted in base64 so..

>root@kali:~/Desktop# base64 -d answer.txt >clanswer.txt
A
A
AA
AB
ABM
AC
ACTH
AI
AIDS
AM
AOL
AOL
ASCII
ASL
ATM
.......snippet.........

it seems like this is a wordlist and as we remember we had a password protected zip file ;) we are gonna attempt to crack it

>root@kali:~/Desktop# fcrackzip -v -D -u -p clanswer.txt lmao.zip
found file 'noob', (size cp/uc 1300/ 1679, flags 9, chk 1005)

PASSWORD FOUND!!!!: pw == ItCantReallyBeThisEasyRightLOL
root@kali:~/Desktop# unzip lmao.zip
Archive: lmao.zip
[lmao.zip] noob password:
inflating: noob

root@kali:~/Desktop# file noob
noob: PEM RSA private key,

and apparently this is an rsa ssh file called noob ;) so all good here

>root@kali:~/Desktop# cat noob
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAsIthv5CzMo5v663EMpilasuBIFMiftzsr+w+UFe9yFhAoLqq
yDSPjrmPsyFePcpHmwWEdeR5AWIv/RmGZh0Q+Qh6vSPswix7//SnX/QHvh0CGhf1
/9zwtJSMely5oCGOujMLjDZjryu1PKxET1CcUpiylr2kgD/fy11Th33KwmcsgnPo
q+pMbCh86IzNBEXrBdkYCn222djBaq+mEjvfqIXWQYBlZ3HNZ4LVtG+5in9bvkU5
z+13lsTpA9px6YIbyrPMMFzcOrxNdpTY86ozw02+MmFaYfMxyj2GbLej0+qniwKy
e5SsF+eNBRKdqvSYtsVE11SwQmF4imdJO0buvQIDAQABAoIBAA8ltlpQWP+yduna
u+W3cSHrmgWi/Ge0Ht6tP193V8IzyD/CJFsPH24Yf7rX1xUoIOKtI4NV+gfjW8i0
gvKJ9eXYE2fdCDhUxsLcQ+wYrP1j0cVZXvL4CvMDd9Yb1JVnq65QKOJ73CuwbVlq
UmYXvYHcth324YFbeaEiPcN3SIlLWms0pdA71Lc8kYKfgUK8UQ9Q3u58Ehlxv079
La35u5VH7GSKeey72655A+t6d1ZrrnjaRXmaec/j3Kvse2GrXJFhZ2IEDAfa0GXR
xgl4PyN8O0L+TgBNI/5nnTSQqbjUiu+aOoRCs0856EEpfnGte41AppO99hdPTAKP
aq/r7+UCgYEA17OaQ69KGRdvNRNvRo4abtiKVFSSqCKMasiL6aZ8NIqNfIVTMtTW
K+WPmz657n1oapaPfkiMRhXBCLjR7HHLeP5RaDQtOrNBfPSi7AlTPrRxDPQUxyxx
n48iIflln6u85KYEjQbHHkA3MdJBX2yYFp/w6pYtKfp15BDA8s4v9HMCgYEA0YcB
TEJvcW1XUT93ZsN+lOo/xlXDsf+9Njrci+G8l7jJEAFWptb/9ELc8phiZUHa2dIh
WBpYEanp2r+fKEQwLtoihstceSamdrLsskPhA4xF3zc3c1ubJOUfsJBfbwhX1tQv
ibsKq9kucenZOnT/WU8L51Ni5lTJa4HTQwQe9A8CgYEAidHV1T1g6NtSUOVUCg6t
0PlGmU9YTVmVwnzU+LtJTQDiGhfN6wKWvYF12kmf30P9vWzpzlRoXDd2GS6N4rdq
vKoyNZRw+bqjM0XT+2CR8dS1DwO9au14w+xecLq7NeQzUxzId5tHCosZORoQbvoh
ywLymdDOlq3TOZ+CySD4/wUCgYEAr/ybRHhQro7OVnneSjxNp7qRUn9a3bkWLeSG
th8mjrEwf/b/1yai2YEHn+QKUU5dCbOLOjr2We/Dcm6cue98IP4rHdjVlRS3oN9s
G9cTui0pyvDP7F63Eug4E89PuSziyphyTVcDAZBriFaIlKcMivDv6J6LZTc17sye
q51celUCgYAKE153nmgLIZjw6+FQcGYUl5FGfStUY05sOh8kxwBBGHW4/fC77+NO
vW6CYeE+bA2AQmiIGj5CqlNyecZ08j4Ot/W3IiRlkobhO07p3nj601d+OgTjjgKG
zp8XZNG8Xwnd5K59AVXZeiLe2LGeYbUKGbHyKE3wEVTTEmgaxF4D1g==
-----END RSA PRIVATE KEY-----

i tried logging in with this as user noob it actually worked but the connection closed, so i thought time to check this exploit 
[exploit](http://resources.infosecinstitute.com/bash-bug-cve-2014-6271-critical-vulnerability-scaring-internet/). it is bash shellshock vulnerabllity

[![troll27](https://trickster0.files.wordpress.com/2015/10/troll27.png)](https://trickster0.files.wordpress.com/2015/10/troll27.png)
as you can see it gave us access but not a normal shell so i used the pty.spawn of python to get one ;)
apparently netcat aint allowed. after i ran the /bin/bash for some reason anything i wrote was doubled, i think it is cause of the buggy kali sana cause i tried it again later and worked like a charm althought i forgot to take some new pictures.
i checked a couple of places for interesting files and in the filesystem i found a folder called nothing_to_see_here... so i went inside and there was another folder choose wisely so i thought here we are there are 3 doors!! door1,door2,door3
all of them had a file called r00t that we would have to BOF it since it get user input and get root since it gives us the 0 id we so much want.

[![troll28](https://trickster0.files.wordpress.com/2015/10/troll28.png)](https://trickster0.files.wordpress.com/2015/10/troll28.png)

so this is door number 1 and thx a lot btw for this -_-. if you guys keep seing some things double it is fine! and you are file too kali sana ain't though :/ but since it worked who cares

[![troll210](https://trickster0.files.wordpress.com/2015/10/troll210.png)](https://trickster0.files.wordpress.com/2015/10/troll210.png)
so door2 seemed pretty normal and probably the correct one to devote it some time but i checked the third one just to be sure..and w8 what???executing shell?for a moment i was like.. this cant be so easy and obviously i was right :/ it trolled me.
so door number 2 it is!aaaaand segfault error

[![troll211](https://trickster0.files.wordpress.com/2015/10/troll211.png)](https://trickster0.files.wordpress.com/2015/10/troll211.png)
Of course i checked for protections and ASLR is off so yay. NX is off too :D actually there is no protection!

[![troll212](https://trickster0.files.wordpress.com/2015/10/troll212.png)](https://trickster0.files.wordpress.com/2015/10/troll212.png)

[![troll213](https://trickster0.files.wordpress.com/2015/10/troll213.png)](https://trickster0.files.wordpress.com/2015/10/troll213.png)

after executing gdb with 500 digits i noticed that it overwrote EIP at 268 bytes so i executed this to check the ESP

[![troll214](https://trickster0.files.wordpress.com/2015/10/troll214.png)](https://trickster0.files.wordpress.com/2015/10/troll214.png)
after checking my friend google for an exec /bin/bash shellcode we got a good one and executed it the r00t file...of course we added a NOP sled just for better results :D and made the EIP jump to our esp address where all the magic happens

[![troll215](https://trickster0.files.wordpress.com/2015/10/troll215.png)](https://trickster0.files.wordpress.com/2015/10/troll215.png)
aaaand LETS GET THAT FLAG BITCH!

[![troll216](https://trickster0.files.wordpress.com/2015/10/troll216.png)](https://trickster0.files.wordpress.com/2015/10/troll216.png)
that is all folks thx for the tr0ll2 and i hope we will get a third one so we can complete this trilogy!
