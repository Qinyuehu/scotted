---
layout: post
title: "Stop using telnet to test network connectivity"
date: 2015-03-12 11:43:53 -0700
comments: true
categories:
---

It's common to use Telnet for testing network connectivity to arbitrary
systems and ports. People use this method because it's simple and it works,
but this is 2015 and it's time to stop!

Using Telnet for this is like renting a U-Haul to give a note to your
neighbor. Instead, try **netcat** (to replace Telnet anyway, not to deliver
notes to your neighbor :-) ).

Netcat (nc) has been around since 2007 and is a much better tool than Telnet -
in fact it was written exactly for this sort of thing, and with the right
options will immediately exit, so unlike telnet you don't have to
type *^]^d* to exit Telnet every time you run it.

Examples:

```
$ nc -vzw 5 google.com 443
DNS fwd/rev mismatch: google.com != lax02s22-in-f14.1e100.net
google.com [216.58.216.46] 443 (https) open

$
```

Woah! That's awesome, no? Not only did it confirm that we can get to google
(the port is **open**), but it provided some useful DNS information as well as
the protocol name from /etc/services. And, note that it exited straight back to
the command line.

Let's look at the options passed above:

* **-v**   : Verbose (without this, nc doesn't output much).
* **-z**   : Zero-I/O mode (exit as soon as the connection is established).
* **-w 5** : Wait 5 seconds for a connection (default waits much longer).

Now let's try an example where we cannot establish a connection, and see what
that looks like:

```
$ nc -vzw 5 duckduckgo.com 123
DNS fwd/rev mismatch: duckduckgo.com != ec2-50-18-192-251.us-west-1.compute.amazonaws.com
DNS fwd/rev mismatch: duckduckgo.com != ec2-54-215-176-19.us-west-1.compute.amazonaws.com
DNS fwd/rev mismatch: duckduckgo.com != ec2-50-18-192-250.us-west-1.compute.amazonaws.com
duckduckgo.com [50.18.192.251] 123 (ntp) : Connection timed out

$
```

That's nice, right? We again get some useful DNS information and then verification
that we couldn't connect to that port.

You can use netcat for interactive testing too, by eliminating the -z option.
Here's the rough equivalent of doing a curl request. I typed in the HTTP
request "GET / HTTP/1.0" and the rest is the response from the server:

```
$ nc google.com 80
GET / HTTP/1.0
HTTP/1.0 200 OK
Date: Thu, 12 Mar 2015 18:32:13 GMT
Expires: -1
Cache-Control: private, max-age=0
Content-Type: text/html; charset=ISO-8859-1
[...]
```

The [Wikipedia entry for netcat](https://en.wikipedia.org/wiki/Netcat) provides
some great additional examples, and quite a bit more information than the manpage.

So the next time your muscle memory starts to type "telnet", quickly remind
yourself to use "nc" instead!
