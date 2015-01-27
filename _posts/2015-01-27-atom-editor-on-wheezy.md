---
layout: post
title: "Atom editor on Wheezy"
date: 2015-01-27 10:40:42 -0700
comments: true
categories:
---

If you have systems running Debian Wheezy and want to check out the Atom editor,
[here's a great solution](https://github.com/atom/atom/issues/2020#issuecomment-48904370)
which doesn't require compromising your perfectly working system by installing
cross-release libraries from testing or unstable. It takes a bit to get it
setup, but it results in having a private version of libc just for Atom. The
same technique can be used for any binary that requires a newer version of libc.

How do you know you need to do this? Because you'll get the following when starting
Atom:

```
/usr/share/atom/atom: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.14' not found (required by /usr/share/atom/atom)
/usr/share/atom/atom: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.14' not found (required by /usr/share/atom/libchromiumcontent.so)
/usr/share/atom/atom: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.14' not found (required by /usr/share/atom/libgcrypt.so.11)
/usr/share/atom/atom: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.15' not found (required by /usr/share/atom/libgcrypt.so.11)
```
