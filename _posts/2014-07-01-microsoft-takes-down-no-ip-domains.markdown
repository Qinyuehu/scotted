---
layout: post
title: "Microsoft Takes Down no-ip Domains"
date: 2014-07-01 07:43:59 -0700
comments: true
categories:
---

Today I discovered that I was unable to resolve DNS to my server at home, which
I access via a noip.com domain. Why? Because Microsoft was granted a federal
court order to seize 22 of noip.com's domains!

This was done without any attempt by Microsoft to resolve any perceived issues
with noip.com, and Microsoft claims this was done due to noip.com domains
hosting malware. This is an amusing statement when you consider the vast amount
of spam and malware that comes from hotmail.com and other Microsoft properties -
or as a result of Windows being insecure to begin with.

Here's the statement issued by noip.com: http://www.noip.com/blog/2014/06/30/ips-formal-statement-microsoft-takedown/ and the full text of this is below, as well.

Just another reminder to use open-source operating systems and stop supporting
companies such as Microsoft, Apple, and even Google.

```
We want to update all our loyal customers about the service outages that many of
you are experiencing today. It is not a technical issue. This morning, Microsoft
served a federal court order and seized 22 of our most commonly used domains
because they claimed that some of the subdomains have been abused by creators of
malware. We were very surprised by this. We have a long history of proactively
working with other companies when cases of alleged malicious activity have been
reported to us. Unfortunately, Microsoft never contacted us or asked us to block
any subdomains, even though we have an open line of communication with Microsoft
corporate executives.

We have been in contact with Microsoft today. They claim that their intent is to
only filter out the known bad hostnames in each seized domain, while continuing
to allow the good hostnames to resolve. However, this is not happening.
Apparently, the Microsoft infrastructure is not able to handle the billions of
queries from our customers. Millions of innocent users are experiencing outages
to their services because of Microsoft&#8217;s attempt to remediate hostnames
associated with a few bad actors.

Had Microsoft contacted us, we could and would have taken immediate action.
Microsoft now claims that it just wants to get us to clean up our act, but its
draconian actions have affected millions of innocent Internet users.

Vitalwerks and No­-IP have a very strict abuse policy. Our abuse team is
constantly working to keep the No-­IP system domains free of spam and malicious
activity. We use sophisticated filters and we scan our network daily for signs
of malicious activity. Even with such precautions, our free dynamic DNS service
does occasionally fall prey to cyber scammers, spammers, and malware
distributors. But this heavy-handed action by Microsoft benefits no one. We will
do our best to resolve this problem quickly.
```

Update
======

Microsoft has no settled with no-ip.com and admitted wrongdoing. The
details of the settlement were not made public, but I sure hope it was enough
money.

```
Microsoft has reviewed the evidence provided by Vitalwerks and enters into the
settlement confident that Vitalwerks was not knowingly involved with the
subdomains used to support malware.
```

More at http://www.noip.com/blog/2014/07/10/microsoft-takedown-details-updates/

Here's a good technical read on how Microsoft doesn't understand DNS and the
mistakes they made in blocking all 5 million domains instead of just the few
hundred they claim were involved in the malware:
http://www.unchartedbackwaters.co.uk/pyblosxom/microsoft_noip_dos

In case it's not obvious - you should think twice if your networking infrastructure
uses Microsoft DNS, DHCP, or other network services. Not only do they have a
clear history of not following the RFCs, it's pretty obvious they just honestly
don't understand how things work. It was suggested
[here](http://forums.smallnetbuilder.com/showthread.php?t=18093) to send
Microsoft a copy of the *DNS and Bind* book - not a bad idea!
