---
layout: post
title: 'HEVD: Kernel Stack Buffer Overflow in Rust!'
tags: []
---
Hello,
so this is a real quick explanation of the kernel buffer overflow showcasing rust programming language.
Hacksys driver has a buffer overflow because it doesn't check the size of the copied input into the stack.
By using the appropriate IOCTL code, we can contact the Symbolic Link, hence the device and send our own input,
trying to control the EIP register and then jumping back to user space into our shellcode that will check the PID 
of our process and the system's, which is always 4 and then replace our token with the system's.
Below is the source code in rust and the output of getting System privileges.
Obviously this is windows 7 and there is no need to reason to deal with SMEP since it was not implemented for windows 7.

Main Code:

[![](https://trickster0.files.wordpress.com/2019/04/rust1.png)](https://trickster0.files.wordpress.com/2019/04/rust1.png)

Cargo.Toml:

[![](https://trickster0.files.wordpress.com/2019/04/rust2.png)](https://trickster0.files.wordpress.com/2019/04/rust2.png)

Shell:

[![](https://trickster0.files.wordpress.com/2019/04/rust3.png)](https://trickster0.files.wordpress.com/2019/04/rust3.png)
