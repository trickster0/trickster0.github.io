---
layout: post
title: Custom ReadMemory API
tags: []
---


After the great job and inspiration by [x86matthew](https://twitter.com/x86matthew) and his [blogpost](https://www.x86matthew.com/view_post?id=read_write_proc_memory) I decided to play with it as well for x64 bit.  
The NTAPI function in this method is RtlFirstEntrySList from ntdll.dll. Its definition like Matthew mentioned in his blog is similar like this:  
```
DWORD RtlFirstEntrySList(DWORD* Address)
```  

In his blog, only the x86 version is referenced and used, so I was curious and took a look myself.  
Unfortunately, in x64 bit the actual function looks like this:  

[![](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/pre-mod.png)](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/pre-mod.png)  

We can see that there are 2 minor issues in the x64 bit version of this function, first one is that the argument of RtlFirstEntrySList will be +8,  
`mov rax, qword ptr(rcx+8)`  
which is easy to solve by just adding -8 in the passed address argument. FYI, for my POC, I am just reading the address of RtlFirstEntrySList but using reference of its address.  
You can alter it be removing the reference(&) to just read the contents of that address.  
First issue bypassed! Second one is not possible to evade that easy since it will perform `and al, 0F0h` in the byte we want to fetch, hence losing the accuracy of the byte and obtaing a completely wrong address.  
The only way around this would be to patch the 2 bytes that perform the logical AND instruction like I do so, in the POC with WriteProcessMemory.  
After patching the 2 bytes our function will look like this and it will be executed via NTDLL as normal.  

[![](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/after-mod.png)](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/after-mod.png)  

Obviously the result will come back as little endian, but you can always allocate the bytes in a buffer and print or use them in a proper manner.  

[![](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/output-mod.png)](https://github.com/trickster0/trickster0.github.io/raw/master/assets/img/favicons/output-mod.png)  

Make sure you read the blog posts and research of Matthew, there are some great stuff on his blog.  

