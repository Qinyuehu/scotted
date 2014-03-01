---
layout: post
title: "Port knocking with single package authorization"
date: 2014-03-01 09:48:45 -0800
comments: true
categories:
---

A few weeks ago I discovered [fwknop](http://www.cipherdyne.org/fwknop/) which
is a very clever mechanism to secure services. I'm using this so can ssh into a
Linux server on my home network without opening the sshd port up to the world.

Single packet authorization works by sending a single, encrypted UDP packet
to a remote system. The packet is never replied to in any way, but if if it's
validated by the remote system, then it uses iptables to temporarily open up
the service port for access. After a user-configurable amount of time (30
seconds by default), the port is closed, but stateful iptable rules keep
existing connections active.

This is really great because all ports to my home IP address appear, from the
internet, to be black holes - my router firewall drops all unknown packets, and
the specific ports open for fwknop are dropped via iptables on my Linux server.

Configuring this solution isn't too difficult if you are familiar with networking
and Linux network and system administration, but it can be a bit tricky to test.

The are three areas that need to be configured on the server-side:
* Fwknop needs to be configured with appropriate ports and security keys
* Service needs to listen on appropriate port
* Router firewall needs to forward fwknop and service ports to the server

I didn't use the default port as a bit of added measure, and additionally I'm
running sshd on a different port as well, as a small added bit of security.

On the client, I have a simple script that:
* Sends the authorization packet via the fwknop client
* ssh's into my server on the configured sshd port

The available documentation is decent, and I've found this solution works very
nicely for me, without exposing *any* detectible open ports on my network.
Unlike with simple port knocking, it is virtually impossible for someone to use
packet capture replay to access the system. Because all packets are dropped
unless the authorization packet opens up the service port, it is completely
undetectable via port scanning that fwknop is even in use.

Give it a try!
