---
layout: post
title: Human Stupidity...NOT A VM
tags:
- challenge
- hack
- human
- not a vm
- pwned
- real
- stupidity
---
Hey everyone so this is the story of how i hacked into a server. this is not a vm, it was an actual server and i wont be giving much infos cause i dont want to expose the target to attacks.
so lets say the name of the server i hacked into was www.stupid.com and the ip was 111.111.111.111. 
of course i started as always with nikto to get some vulnerabillities...they were interesting but something else was more important at a first glance!
there was a .zip file called stupid.com.zip so i figured this must be a backup file so i downloaded and it indeed was a backup file... ;)  */Human stupidity number 1 /*
so after i extracted it this is what we got

[![rh1](https://trickster0.files.wordpress.com/2015/10/rh1.png)](https://trickster0.files.wordpress.com/2015/10/rh1.png)
after checking a few files i checked the sftp-config.json file to check for possible passwords and this is what we actually got

[![rh2](https://trickster0.files.wordpress.com/2015/10/rh2.png)](https://trickster0.files.wordpress.com/2015/10/rh2.png)
i know that you cant see the password but trust me it was quite good so good job there :P 
anyway i found after a while an admin login page in the newcms folder path but unfortunately the pass and username didnt work there although by checking the admin php code i noticed that for the user's session to be created it needs to read some files in the var folder but there were no files like them in the backup file good job again at this point. 

[![rh3](https://trickster0.files.wordpress.com/2015/10/rh3.png)](https://trickster0.files.wordpress.com/2015/10/rh3.png)
so i had to find another way so
ofc after finding out that tftp on port 22 might be running i scanned with nmap but it was closed too bad :/ although the ftp was open so i thought lets try it out and use the same password and username    */Human stupidity number 2/*>root@kali:~# ftp 111.111.111.111
Connected to 111.111.111.111.
220---------- Welcome to Pure-FTPd [privsep] [TLS] ----------
220-You are user number 1 of 50 allowed.
220-Local time is now 01:43. Server port: 21.
220-This is a private system - No anonymous login
220-IPv6 connections are also welcome on this server.
220 You will be disconnected after 15 minutes of inactivity.
Name (111.111.111.111:root): stupid
331 User stupid OK. Password required
Password:
230 OK. Current restricted directory is /
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>


i wont be showing what i got in there but i went after the mysql file that i found out earlier! the var directory and the rest of the mysql db files were all there!!!
i checked the mysql.php file and check this out


>define('dbase','dbase');
define('host','host');
define('user','user');
define('password','password');
define('port','port');

$server[dbase]="stupid_site";
$server[host]="localhost";
$server[user]="stupid_site";
$server[password]="a8XXXXXXX";
$server[port]="3306";


more
i found the /etc/shadow and the /etc/passwd files too but lets not worry about that for now
/etc/shadow


>stupid1:$6$MTkxu6xGhH2lRj5D$ucH
stupid2:$6$tZPS9ALNEVkT1xMP$pLDkCjwJv622L0mH28liD
stupid3:$6$uc/QjiYI.uaw2xa1$uFoTlZxxVwrr2Msd82FtO
stupid4:$6$KJ0dPziRX8lVkTq4$CIi7j3Vm9jLOz2IXelpC3
stupid5:$6$Bk95YCxxhBuLKnSK$Jyq2RmFq12vJyu57m7FgO
stupid6:$6$ow82ApgjPeO5Xf14$ly242LU95a5xtCvh82zEh
stupid7:$6$3UdXUAEkctoPc17x$YlL4x1eIuCSFc/QbLaMev
stupid8:$6$K56LoyADGjYB3HKx$RVAJbzs1vF5Lk8JGZ4gIP
stupid9:$6$yyKMgLzqUXdgtbjJ$eky/XN4XaLHW.I8Y4iFOP


although i didnt try to crack the passwords cause it would take time i tried to figure out the login password for the admin panel!
i couldnt do much from the ftp so i went into mysql enumeration and the fun started!
so i scanned the mysql port to check if it is open...

[![rh6](https://trickster0.files.wordpress.com/2015/10/rh6.png)](https://trickster0.files.wordpress.com/2015/10/rh6.png)


>root@kali:~# mysql -h 111.111.111.111 -u stupid_site -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 259496
Server version: 5.5.45-cll MySQL Community Server (GPL)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| stupid_site      |
+--------------------+
2 rows in set (0.10 sec)

mysql>  use stupid_site;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql>  show tables;
+-------------------------+
| Tables_in_stupid_site |
+-------------------------+
| activity                |
| articles                |
| categories              |
| cmsmenu_new             |
| cmsrights               |
| deletedemails           |
| downloads               |
| emails                  |
| faq                     |
| guestbook               |
| languages               |
| links                   |
| mailqueue               |
| menu                    |
| menus                   |
| newsletter_articles     |
| newsletter_teaming      |
| newsletter_teams        |
| newsletters             |
| packaging               |
| pictures                |
| pictures_cropings       |
| pictures_types          |
| relations               |
| sitecategories          |
| subcategories           |
| users                   |
| variables               |
| youtube                 |
+-------------------------+
29 rows in set (0.09 sec)

mysql> mysql> select id,email,username,password,usertype from users;
+-------+-----------------------+----------+--------------------------+
| id | email              | username |password                        |
+-------+-----------------+----------+--------------------------------+
| 114| stupid@stupid.com  | stupid   |e10adc3949ba59abbe56e057f20f883e|
+-------+-----------------+----------+--------------------------------+
1 row in set (0.09 sec)

mysql>


the first thing i tried was looking for the hash online aaaaaaaaaand */Human stupidity number 3/*

[![rh4](https://trickster0.files.wordpress.com/2015/10/rh4.png)](https://trickster0.files.wordpress.com/2015/10/rh4.png)


**#FAIL**


**AND........BOOM**



[![rh5](https://trickster0.files.wordpress.com/2015/10/rh5.png)](https://trickster0.files.wordpress.com/2015/10/rh5.png)

so that is pretty much it i didnt get root access because mysql didnt run as root and didnt find another way although all of this was a real pentest and the whole thing pretty much happened was of human stupidity and ignorance! although it was a good lesson and practise for me. Of course i did no harm and i had no evil intentions!! so that is it until my next challenge ;)
ps: excuse my photoshop skills :P
