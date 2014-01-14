---
layout: post
ctf: "Break In 2014"
title: "Reverse track (1 to 7)"
author: "Mathieu Binette"
---

##Reverse 1 (Simple) - 100pts

The code isn't very long and we can quickly see that the function `callMeMaybe` will never be called in a normal execution

```

[...]
.text:0000000000400918 mov     [rbp+var_4], 1
.text:000000000040091F cmp     [rbp+var_4], 0
.text:0000000000400923 jnz     short loc_40092F
.text:0000000000400925 mov     eax, 0
.text:000000000040092A call    callMeMaybe
.text:000000000040092F mov     edi, 1
.text:0000000000400934 mov     eax, 0
.text:0000000000400939 call    _sleep
[...]

```

If we look more closely at the `callMeMaybe` function, we notice there are a bunch of encrypted data (or so it looks like), and then some code that could be roughly translated in Python as:

```python

encrypted_bytes = [ ... ]

current_time = time.time()
x = pow( 2, current_time % 16 )

decoded_string = ""
for i in xrange(0, 13): #Loop 12 times
    a = encrypted_bytes[ i_50 - x*4 ]
    b = encrypted_bytes[ i_90 - x*4 ]
    if i % 2 == 0:
        decoded_string += chr((a ^ b) % 128)
    else:
        decoded_string += chr((a + b) % 128)
    x += 1

print decoded_string

```

For this challenge, you could simply edit the value of `x` to 0 before the decryption loop starts to get the full decrypted key: `break_in@iiit`.

* * *

##Reverse 2 (Welcome) - 100pts

I found that a static analysis approach to this problem worked very well.

The first thing you notice when checking out the `main` was that the program accepted two (and only two) arguments. It then proceeded to check if the two arguments formed a valid pair (as per a certain "br0 code" algorithm). 

The reversed algorithm for building a pair was:

```python

original = "..."

final = ""
for c in original:
    for i in xrange(0,3):
        final += chr(ord(c)+i)

print final

```

The key was the br0 code for the event, `felicity`, which is `fghefglmnijkcdeijktuvyz{`.

* * *

##Reverse 3 (Let's Float) - 200pts

When I first ran the executable, the program output wasn't too helpful.

> The author of this question is: NEHAL JW

> He is also known as 31415504713243874993144365269619144804738951469063106995958138362195207906814846878> 639559523605055359252709769216.000000

> Tell me what FELICITY is known as?

> Answer in the format md5(str(x)[0:10])

However, once you decompile it...

```

mov     rax, 574A204C4148454Eh
[...]
mov     edi, offset format ; "The author of this question is: "
[...]
mov     edi, offset aHeIsAlsoKnownA ; "He is also known as %lf\n"

```

Where:

```

W  J     L  A  H  E  N
57 4A 20 4C 41 48 45 4E

```

It is then easy to see that FELICITY would be the `%f` representation of `0x59544943494C4546`, which is `209535973878143263198517842832595276808993002060900303432659985500058499002587050139517918698749870058982388832275731054592.000000`.

* * *

##Reverse 4 (Delme) - 300pts

Delme was an interesting program... I think ?

After some investigation, I noticed a function really similar to the decrypting algorithm of `Reverse 1`. 

I was able to reuse the same algorithm described in `Reverse 1` to get the key for this problem: `blowme_is_not_blimey`

* * *

##Reverse 5 (Whereami) - 200pts

After my luck on `Reverse 4`, I decided to look for a decrypting funtion again. I found one identical to `Reverse 1` and `Reverse 4`, gave my script a spin again , and got the following key: `gdb_is_false`.

I don't know why gdb is false, but yay!

* * *

##Reverse 6 (Bomb) - 400pts

Never change your winning formula. I went looking for that same function once again.

However, the organizers wisened up, and modified their algorithm a little. Here is my new updated script:

```python

encrypted_bytes = [ ... ]

current_time = time.time()
x = pow( 2, current_time % 16 )

decoded_string = ""
for i in xrange(0, 13): #Loop 12 times
    a = encrypted_bytes[ i_40 - x*4 ]
    b = encrypted_bytes[ i_80 - x*4 ]
    if i % 3 == 0:
        decoded_string += chr((a * b) % 128)
    else:
        if i % 2 == 0:
            decoded_string += chr((a ^ b) % 128)
        else:
            decoded_string += chr((a + b) % 128)
    x += 1

print decoded_string


```

And you get the key: `t!ck_t!ck_b00m`

* * *

##Reverse 7 (b00m2) - 400pts

It seems like the organizers know what we're up to:

> If you want to have more fun, stop looking at assembly, and try to figure out what exactly the program wants.

Well - that is a weird request for a *reversing* challenge, isn't it?

After looking at the code, I noticed the organizers added a couple of decoy decrypt functions (about 5). Since I was too lazy to try them all, I decided to give it a spin through gdb.

The program was waiting for input on a socket, and seemed to loop a couple of times. I just skipped over all the socket stuff (sorry guys!) and made it to the defusing part of the bomb (the real defusing).

Once I got there, I noticed there were two of the multiple decrypt functions called. I decided to start with the last one since the first one clearly outputs "This is not the key" (but that could have been a trap..). I reused `Reverse 6`'s script and got the following key: `phoos_phoos_bumb`.
