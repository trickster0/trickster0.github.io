---
layout: post
title: Solving the BFS Ekoparty 2019 Exploitation Challenge
tags: []
---


This is a quick write up about how one of our team members, Thanasis, solved the challenge for EkoParty 2019. This was a fun challenge and thanks to Lukas and Nico from Blue Frost Security for making it happen(and for supporting our community).  

More information about the challenge can be found at:  

[https://labs.bluefrostsecurity.de/blog/2019/09/07/bfs-ekoparty-2019-exploitation-challenge/](https://labs.bluefrostsecurity.de/blog/2019/09/07/bfs-ekoparty-2019-exploitation-challenge/)  

The application as the requirements provided, need to run in windows 10 x64 (RS6) version and the goal is to bypass ASLR and execute a calc.exe process.  

By opening the application we can see via netstat that it binds on port 54321 on 0.0.0.0 (all the machine’s interfaces).   

By opening Ghidra and going to the main function it is obvious that some checks need to bypassed in order to correctly send a payload to the application.  
In Ghidra, if we check the function that is called after the new connection is accepted, we see this: 

[![](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/1.png)](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/1.png)


Upon first check, it checks for the first 0x10 bytes(16 chars) as a header.  
The second and third checks: If the header starts with 0x393130326f6b45(Ekoparty2019) then we are allowed to send a user_message as long as it is smaller than 0x201 bytes(513 chars).  
The last fourth check is quite important, we can send all this packet structure but it needs to be aligned correctly for 8 bytes. Meaning we could send 16,24,32 and so on.  
After we succeed in sending a big buffer, it appears that the application crashes after 529 bytes or so.  
By sending 528 bytes structured correctly with the cookie included in the beginning, we notice that before the calling function sub_140001170, we actually control the RAX, which is the 513 bytes.  

Before this, there is this instruction

`lea     rcx, unk_7FF6A8A9E520`   

 unk_7FF6A8A9E520, holds an array with this structure

[![](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/2.png)](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/2.png)

By sending the 513 characters, for example as A or \x41 we can make it so the function will return our byte + the rest of the pattern. In this case c3c3c3c3 + ourbyte+488b01.  

The function sub_140001170 before it returns this value turns it to little endian, making it ourbyte+488b01c3c3c3c3. So we get 41488b01c3c3c3c3.  

This value will be used in WriteProcessMemory as lpBuffer, basically copying these bytes to the function sub_7FF6A8A91000 as instructions allowing to control what we can execute when we reach it.  

Although this is quite good, it provides a limitation of instructions, meaning we can only use instructions byte+488b01c3c3c3c3.
I made a quick script in python producing all the values in a file

```
byte=0x00
endbyte=0xff  
start ="848b01c3c3c3"  
for i in xrange(byte,endbyte+1):  
            print format(i,'X')+ start
 ```


With a one-liner bash I got all the values:

`for i in $(cat list_instructions);do echo -e "\n$i" && rasm2 -b 64 -D $i ; done  > instructions`  

One good thing in this case is that we can actually control the RCX from our input buffer with the characters provided from 513 till 528.  

The first thing I had to do was, get the process address from PEB.  

By sending in our payload these are the last bytes:  

"\x65\x65\x65\x65\x65\x65\x65\x65\x60\x00\x00\x00\x00\x00\x00\x00"

We could achieve and acquire the PEB. \x65 is meant for the combination from the previous instructions.  

65488b01c3 

```
0:  65 48 8b 01             mov    rax,QWORD PTR gs:[rcx]  
4:  c3                      ret
```


It is well known that in x64 bit windows, GS register is a special register which points to PEB by providing the accurate offset. In this case since we could control RCX, we pointer GS directly to the PEB which is at offset 0x60 hence the highlighting.  

Since the application will always sends us back the data leaked we can get this address and use it.  

The next step would be to get the Image Base Address of the application.  

Image Base Address is located from the PEB + 0x10 offset. In this case we had to set the address + 0x10 as a pointer to RCX to be able to leak the address.  

In this case, according to our possible instructions we chose:  

```
0:  47 8b 01                mov    rax,QWORD PTR [rcx]  
3:  c3                      ret
```


The first byte 47 and these as before are the last bytes of our payload:  

"\x47\x65\x65\x65\x65\x65\x65\x65 + address+0x10"

As an end goal we need to create a ROP chain to execute calc.exe.  

Since we would like to bypass ASLR, leakage is already useful but in case we would need to execute something, we would have to bypass DEP as well.  

In this case it is good that we have, in the beginning of the application, a winexec call.  

[![](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/3.png)](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/3.png)

Therefore, in the end we will call calc.exe through winexec but, winexec requires that the application will be executed to be pointed at, hence a pointer that points to the string calc.exe and a null terminator.  

Somehow I had to be able to find that place in memory with my string. The best way was to get the StackBase Limit and get towards the stack base to find where it is.  

First, I had to leak StackBase Limit.  

StackBaseLimit is in the TEB at 0x10 offset through the GS register.  

The initial request I used :

```
0:  65 48 8b 01             mov    rax,QWORD PTR gs:[rcx]  
4:  c3                      ret
```

I controlled the RCX by setting it to 0x10.  

After actually getting the leaked address of the Stack Base Limit, it is time for a loop towards the Stack Base to find the correct string which would be calc.exe.  

By doing a loop, I started leaking the memory cells of the stack up to a point where it detected my string.  

The moment the string was found, I saved into a counter and multiplied by 0x08 to get how many cells down the stack I had to go.  

So now I had the address of the string.  

In the above scenario I used: 

```
0:  47 8b 01                mov    rax,QWORD PTR [rcx]  
3:  c3                      ret
```

With RCX as the Stack Base Limit and constantly adding 0x08 to it.  

The next step would be to get the winexec’s address on the stack. By checking the .rdata of the application I could see the offset of it.
[![](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/4.png)](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/4.png)

In this case, I need to leak the address from Image Base Address + 0x9010 offset.  

By using exactly the same instructions as before:

```
0:  47 8b 01                mov    rax,QWORD PTR [rcx]  
3:  c3                      ret
```

Then adding RCX as the Image Base Address+0x9010 , I get the leaked address for Winexec on the stack.  

For the final request to the application I used 

```0:  51                      push   rcx
1:  48 8b 01                mov    rax,QWORD PTR [rcx] 
4:  c3                      ret
```

I set the RCX to a pivot gadget “add rsp,78h ; ret”, so I can stack pivot.  

I used Ropper and rp++ to get gadgets out of the application.  

Thankfully, the ret instruction gets us to a point in our buffer.  

According to MSDN Wincalc requires 2 arguments, the name of the application and a number which will set the mode of the window.  

In windows 10 x64 , the calling convention is rcx,rdx,r8,r9 and top of the stack.  

The structure of the packet is this. The whole packet is the cookie + 528 characters.  

Structure:
| 16 junk bytes | padding  |
|pop_rax_gadget| Pop Image Base Address for having a valid address on RAX because the only pop rdx and pop rdx gadgets set bad values to it. |
|Image Base Address – 0x08| valid address |
|pop_rdx_gadget| pop rdx gadget to put 0x01 for the Wincalc second argument.|
|0x01| Winexec UINT   uCmdShow |
|pop_rax_gadget| again for the same reason that the pop rcx gadget will set bad value to rax |
|pop_rcx_gadget| set the pointer address that points to calc.exe\x00 |
|address_pointing_calc| address that points to calc.exe\x00 |
|72 junk bytes| padding |
|ret_gadget| just a return gadget to fix the stack alignment to 16-byte format, because CreateProcessA is called inside the Winexec function which includes movabs instruction. Movabs instructions check if the stack is aligned and if not it will raise an exception. |
|winexec_leaked_address| winexec address on the stack. |
|add_rsp_0x78| adds to current RSP + 0x78 bytes to reach the next stack pivot. |
|120 junk bytes| padding. |
|add_rsp_0x78| adds to current RSP + 0x78 bytes to reach the next stack pivot. |
|120 junk bytes| padding. |
|add_rsp_0x28| adds to current RSP + 0x28 bytes to reach the next stack pivot. |
|40 junk bytes| padding. |
|add_rsp_0x58| adds to current RSP + 0x58 bytes to reach the original return pointer address and continue the execution of the application instead of crashing it.|
|8 junk bytes| padding. |
|calc.exe\x00| string to set in memory. |
|15 junk bytes| padding.|


Gadgets Used:
| Gadget Address | Gadget |
| ---  | --- |
|0x14000158b| add rsp, 0x78 ; ret  ; |
|x0000000140004525|pop rdx; add byte ptr [rax], al; cmp word ptr [rax], cx; je 0x4530; xor eax, eax; ret; |
|0x140001167 | pop rax ; ret  ; |
|0x00000001400089ab | pop rcx; or byte ptr [rax], al; add byte ptr [rax - 0x77], cl; add eax, 0x4b12; add rsp, 0x48; ret;|
|0x0000000140001164 | add rsp, 0x58; ret; |
|0x14000158f | ret  ;  |
|0x00000001400011d5 | add rsp, 0x28; ret;|



Full Working exploit: [https://github.com/trickster0/BFS-Ekoparty-2019-challenge](https://github.com/trickster0/BFS-Ekoparty-2019-challenge)  

Proof of Concept:

[![](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/5.png)](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/5.png)


Results:  

[![](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/unknown.png)](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/unknown.png)
