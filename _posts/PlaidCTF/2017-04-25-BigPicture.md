---
title: "PlaidCTF -- BigPicture Writeup"
categories:
  - PlaidCTF
tags:
  - CTF
  - pwn
---

> Big picture

<div class="notice--info">
<strong>Name</strong>: BigPicture<br>
<strong>Category</strong>: pwn<br>
<strong>Points</strong>: 200<br>
<strong>Solves</strong>: 67<br>
<strong>Given</strong>: <code>bigpicture</code>, <code>bigpicture.c</code>, <code>libc-2.23.so</code><br> <!-- Link the files here to my repository with it -->
</div>


# Solution sumary (solved after the CTF was finished):
Based my exploit on this writeup: [bigpicture write up](https://amritabi0s.wordpress.com/2017/04/24/plaid-ctf-2017-bigpicture-write-up)<br><br>
Sumary:
1. Find the `libc offset` in relation to our buffer
2. Find the `libc base` address by leaking some value in the libc data section which contains a pointer to semewhere else in the libc (I used `__memalign_hook`)
3. Overwrite `__free_hook` with a pointer to `system`
4. Write `"/bin/sh\x00"` in the buffer, since the free is going to be called on that buffer
5. Execute the `quit` command and the `free(buf)` call should instead call `system("/bin/sh")` and spawn a shell
6. **The write up explains everything in more detail, go read it!**

## What I learned for the next CTF:
1. If we allocate a large enought chunk, that chunk will not be on the heap section, but instead the malloc will call `mmap` and (at least for this case) the address will be **relative to the LIBC_BASE address**, allowing to find where the libc is in relation to the chunk allocated.
2. I also learned that you can find the libc base address by looking at some pointers in the libc data section, (I assumed it was possible, but had never tried it before)
3. As I had previouly learned the libc hooks (malloc_hook, free_hook..) are a great way to redirect code execution if you find a way to overwrite them
