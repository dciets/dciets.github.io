---
layout: post
ctf: "CSAW CTF Quals 2016"
title: "Web 150 - mfw"
---

The about page mentioned that technologies used to build the website. That fact that Git was listed indicated that the .git folder might be publicly available.

![About screenshot](/img/writeups/csaw-quals-2016-mfw-about.png)

This was confirmed by reading the file /.git/config which showed the configuration of the local repository.

![Git config](/img/writeups/csaw-quals-2016-mfw-git.png)

Using [Dumper](https://github.com/internetwache/GitTools/tree/master/Dumper) we retrived the Git repository and we began code reviewing until stumbling upon this piece of code.

![Source code](/img/writeups/csaw-quals-2016-mfw-code.png)

The parameter $file is referenced into a double-quoted string meaning its value will be inserted into the string. In PHP, the assert function is basically the eval function in disguise.
This means that we escape the single quoted string inside the assert statement and have PHP code execution.

The final payload: [http://web.chal.csaw.io:8000/?page='.passthru('cat templates/flag.php').'](http://web.chal.csaw.io:8000/?page=%27.passthru%28%27cat%20templates/flag.php%27%29.%27)

Written by @mixbo and @becojo.
