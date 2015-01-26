---
layout: post
title: "CLOSE_WAIT socket leak in java.net.HttpURLConnection"
date: 2015-01-25 11:28:01 -0700
comments: true
categories:
---

I came across an interesting socket leak a few weeks ago which
was maddeningly difficult to track down and reproduce so it could be
fixed.

This all started when I noticed a large number of CLOSE_WAIT sockets to
the AWS metadata service, 169.254.169.254 on every AWS instance - see
http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-metadata.html :

```
tcp6       1      0 10.0.0.1:43396      169.254.169.254:80      CLOSE_WAIT
tcp6       1      0 10.0.0.1:43438      169.254.169.254:80      CLOSE_WAIT
tcp6       1      0 10.0.0.1:43045      169.254.169.254:80      CLOSE_WAIT
tcp6       1      0 10.0.0.1:43317      169.254.169.254:80      CLOSE_WAIT
[...]
tcp6       1      0 10.0.0.1:43388      169.254.169.254:80      CLOSE_WAIT
tcp6       1      0 10.0.0.1:43279      169.254.169.254:80      CLOSE_WAIT
tcp6       1      0 10.0.0.1:43357      169.254.169.254:80      CLOSE_WAIT
tcp6       1      0 10.0.0.1:43134      169.254.169.254:80      CLOSE_WAIT
```

The number of these would increase, by about 2 a minute, until the kernel
does it's housekeeping for these sort of sockets - at max, there'd be
around 120-130 of them.

CLOSE_WAIT means the socket has been closed by the TCP stack but is
waiting on the application (Java in this case) to acknowledge the closure
and clean things up from it's side. So it seems to clearly be a bug
in our code, but I wasn't sure where. I used tcpdump ("sudo tcpdump -A
host 169.254.169.254") to see which requests were being made and made an
educated guess on which service might be responsible, eventually tracking
it down. This code was a couple of years old, and didn't change except
that some new metadata requests were added for VPC support, but the
problem didn't occur with the older version of the client library.

It seemed strange that new requests would cause this, but after some more
tcpdumps I discovered that one of the new requests always results in a
HTTP 404 from the metadata service, and these requests always resulted
in a CLOSE_WAIT socket (matching up the ports in tcpdump with those from
netstat). This request was only useful in VPC, but running it in EC2
Classic isn't really a problem, it just means no metadata for that case.

It seemed pretty obvious that somehow the code must not be closing the
socket after a 404, but as another engineer and I looked at the
code, it didn't actually do anything different whether it was an HTTP 200
or a 404, aside from parsing the request or not. We tried making sure
all data was read from the InputStream, closing the InputStream more
aggressively, calling disconnect() explicitly, and a few other things,
but nothing helped. The code uses Java's HttpURLConnection:
http://docs.oracle.com/javase/7/docs/api/java/net/HttpURLConnection.html

After trying all these different things, the other engineer noticed
a method on HttpURLConnection called getErrorStream() - documented as
"Returns the error stream if the connection failed but the server sent
useful data nonetheless".

Turns out that whoever wrote HttpURLConnection (who I refer to as
"bonehead") decided to implement it sort of emulating stdout/stderr -
if a request is a successful HTTP result code, then the output goes
into one stream, but if it wiss an error then the output goes into a
different stream. This doesn't really help with coding because either
way you still have to check the HTTP response code to see if it failed
or not, but with this implementation you need another whole code block
to process the ErrorStream if there was an error.

The docs for HttpURLConnection only say the following, you'll note they
don't say anything about closing the ErrorStream: "Calling the close()
methods on the InputStream or OutputStream of an HttpURLConnection after a
request may free network resources associated with this instance but has
no effect on any shared persistent connection. Calling the disconnect()
method may close the underlying socket if a persistent connection is
otherwise idle at that time".

The fix was simple - a block of code where if the HTTP response code is
not a 200, instead of just returning failure, we get the ErrorStream,
read all the data off of it (discarding it), then close it.
