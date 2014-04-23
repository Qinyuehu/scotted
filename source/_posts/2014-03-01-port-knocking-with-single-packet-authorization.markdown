---
layout: post
title: "Port knocking with single packet authorization"
date: 2014-03-01 09:48:45 -0800
comments: true
categories:
 - linux
 - software
---

A few weeks ago I discovered [fwknop](http://www.cipherdyne.org/fwknop/) which
is a very clever mechanism to secure services. I'm using this so I can ssh into a
Linux server on my home network without opening the sshd port up to the world.

<img src="http://www.cipherdyne.org/images/fwknop_tutorial_network_diagram.png"/>

Single packet authorization works by sending a single, encrypted UDP packet to a
remote system. The packet is never ACKd or replied to, but if it's validated by
the remote system, then it uses iptables to temporarily open up the service port
for access (the filter is limited to the client's IP address). If the packet
isn't valid, it is simply ignored. In either case, to an external observer the
packet appears to go into a black hole. After a user-configurable amount of time
(30 seconds by default), the service port is closed, but stateful iptable rules
keep existing connections active.

This is really great because all ports to my home IP address appear, from the
internet, to be black holes - my router firewall drops all incoming packets, and
the specific ports open for fwknop are dropped via iptables on my Linux server.

Configuring this solution isn't too difficult if you are familiar with networking
and Linux network and system administration, but it can be a bit tricky to test.

Server Configuration
--------------------

There are four areas that need to be configured on the server-side:

* Fwknop needs to be configured with appropriate ports and security keys
* iptables policy needs to be created for each service port
* Services need to listen on appropriate ports
* Router firewall needs to forward fwknop and service ports to the server

My per-service iptables policies are done in */etc/rc.local* and look like:

```
/sbin/iptables -I INPUT 2 -i eth0 -p tcp --dport 54321 -j DROP
/sbin/iptables -I INPUT 2 -i eth0 -p tcp --dport 54321 -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

It's subtle, but note the rule order of "2" is used, instead of the default (which is "1").
This is to make sure that these rules are in the table *after* the FWKNOP_INPUT rule, which
fwknop will create when it starts. Likewise, the order of the rules above is important, and
are in the order they are to make sure the ESTABLISHED,RELATED rule ends up before the DROP
rule. When loaded, the second rule will displace the first as they have the same priority.

Before you start fwknop, and open up ports on your firewall, don't forget to make sure these
rules are in place. In that case, if you create these rules manually, use rule order "1"
instead of "2", as you are creating the rules before fwknop has added it's rule.

*/etc/fwknop/fwknopd.conf* excerpt from the server:

```
PCAP_FILTER                 udp port 12345;
```

On my debian testing/Jessie server I also had to add this line to *fwknopd.conf*:

```
PCAP_DISPATCH_COUNT            1;
```

*/etc/fwknop/access.conf* excerpt from the server:

```
SOURCE                    ANY
REQUIRE_SOURCE_ADDRESS    Y
KEY_BASE64                SOME_BASE64_ENCODED_KEY
HMAC_KEY_BASE64           SOME_BASE64_ENCODED_HMAC_KEY
```

I didn't use the default port as a bit of added measure, and additionally I'm
running sshd on a different port as well, as a small added bit of security.

Adding an additional port in sshd is really simple, just add an additional Port
line and restart sshd:

```
Port 22
Port 54321
```

Only port 54321 is port forwarded on the router, but I can still use port 22
while on my home network.

Client Configuration
--------------------

On the client, I have a simple script that:

* Sends the authorization packet via the fwknop client
* ssh's into my server on the configured sshd port

The script looks something like:

```
fwknop my.host.fqdn
ssh -p 54321 my.host.fqdn
```

Excerpt from *.fwknoprc* on the client:

```
[my.host.fqdn]
ACCESS                      tcp/54321
SPA_SERVER                  my.host.fqdn
SPA_SERVER_PORT             12345
KEY_BASE64                  SOME_BASE64_ENCODED_KEY
HMAC_KEY_BASE64             SOME_BASE64_ENCODED_HMAC_KEY
USE_HMAC                    Y
ALLOW_IP                    resolve
```

From this config, you can see that the fwknop port is 12345, and sshd is
listening on 54321 (though these aren't the real ports or FQDN in use). The
KEY_BASE64 and HMAC_KEY_BASE64 values need to match between client and server. I
chose to use
[symmetric keys](http://www.cipherdyne.org/fwknop/docs/fwknop-tutorial.html#fwknop-rijndael)
but you can use asymmetric keys via GPG if you prefer.

See the fwknop documentation for more information on configuring everything.
There are a lot of options, so you'll have to figure out what to do based on
your individual needs.

I'm using a free dynamic DNS service so that I don't have to remember the
dynamic IP address assigned by my ISP.

Further Reading
---------------

The [documentation](http://www.cipherdyne.org/fwknop/docs/fwknop-tutorial.html)
is decent, and I've found this solution works very nicely for me, without
exposing *any* detectible open ports on my network. Unlike with simple port
knocking, it is virtually impossible for someone to use packet capture replay to
access the system. Because all packets are dropped unless the authorization
packet opens up the service port, it is completely undetectable via port
scanning that fwknop is even in use.

Give it a try!
