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
19:31:17     eth0    2.31    1.36    5.20    4.74   455.4   294.6  0.00   0.00
19:31:19     eth0    0.46    0.51    4.99    2.99   94.00   175.3  0.00   0.00
19:31:19     eth0    0.26    0.21    2.00    1.00   130.5   214.0  0.00   0.00
19:31:20     eth0    0.14    0.69    2.00    4.00   74.00   177.5  0.00   0.00
19:31:21     eth0    3.96    1.02   11.00   10.00   368.7   104.7  0.00   0.00
^C
```
