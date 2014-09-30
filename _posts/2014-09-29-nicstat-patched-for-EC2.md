---
layout: post
title: "nicstat Patched for EC2"
date: 2014-09-29 17:14:53 -0700
comments: true
categories:
---

Get the Source
==============
Get the patched [nicstat utility here](https://github.com/scotte/nicstat).
It has [several bugs](https://github.com/scotte/nicstat/blob/master/BUGS.md)
fixed, of interest to anyone running Linux, whether in EC2 or on physical hardware.

The Whole Story
===============

I recently learned about a useful performance monitoring utility that I wasn't
aware of - [nicstat](https://sourceforge.net/projects/nicstat/). Somewhat
coincidentally, I also discovered that my coworker
[Brendan Gregg](http://brendangregg.com) was coauthor of this utility for
Solaris back when he worked at Sun.

nicstat is modeled very much after iostat, vmstat, and similar tools. It works
great, but I found network utilization (%Util column) was not being calculated
correctly in EC2. This was no surprise, as a xen guest has no idea about
AWS imposed bandwidth throttles, but I thought it was strange that nicstat
reported an interface speed of 1.410Gbps when ethtool reports a 10Gbps interface
for c3.4xl - the instance type I was running this on.

Strange, but since nicstat has a -S option where the bandwidth can be overridden,
no problem. So I tried that - using measured bandwidth from iperf, but that didn't
work either and nicstat kept reporting a 1.410Gbps rate.

After perusing bugs, I [found the cause](http://sourceforge.net/p/nicstat/bugs/1/)
for the strange 1.410Gbps value - on 10Gbps interfaces there is an integer overflow
that happens resulting in a bogus value. Great - so this mystery was solved, but
why doesn't the -S option work?

It turns out that nicstat will ignore the -S option if it gets a valid result from
the SIOCETHTOOL call (getting it from the underlying hardware). Normally, you would
expect command line options to override values, but that wasn't the case. So, I
coded up a workaround for that. Great - except, the utilization values were still
wrong!

Back to the source again, where I found a bug in the implementation where utilization
is always computed as a half-duplex link on Linux. Easily fixed - and now I have a
working version of nicstat where I can provide an appropriate -S override for any
particular EC2 instance type (and in our use at Netflix, we have a wrapper script
that provides an appropriate value for each type).

As the current author of nicstat isn't actively maintaining it, I've
[forked the source to nicstat 1.95 on github](https://github.com/scotte/nicstat)
and [fixed several bugs](https://github.com/scotte/nicstat/blob/master/BUGS.md).
(3 of which I reported) that have not yet made their way into the official
version.

If you use nicstat, you may find this repository handy, rather than having to
apply several patches manually.

```
$ nicstat -S eth0:2000 -l
Int      Loopback   Mbit/s Duplex State
eth0           No     2000   full    up
lo            Yes        -   unkn    up

$ nicstat -n 1
    Time      Int   rKB/s   wKB/s   rPk/s   wPk/s    rAvs    wAvs %Util    Sat
23:19:29     eth0 34157.0 34140.3 39139.6 39136.8   893.6   893.3  14.0   0.00
23:19:30     eth0 37426.4 37462.9 42513.8 42546.7   901.5   901.6  15.3   0.00
23:19:31     eth0 35787.9 35787.2 41224.3 41224.3   889.0   888.9  14.7   0.00
23:19:32     eth0 25482.9 25482.2 33962.9 33961.9   768.3   768.3  10.4   0.00
23:19:33     eth0 29245.6 29244.4 36125.4 36125.4   829.0   829.0  12.0   0.00
^C
```
