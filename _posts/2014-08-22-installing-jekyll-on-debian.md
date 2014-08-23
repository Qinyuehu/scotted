---
layout: post
title: "Installing Jekyll on Debian"
date: 2014-08-22 09:47:06 -0700
comments: true
categories:
  - jekyll
  - linux
  - debian
---

Here's the minimal steps required to install Jekyll on a Debian system so you can test
things out before pushing up to github pages (or wherever).

```
$ sudo aptitude install ruby ruby-dev rubygems nodejs
$ sudo gem install jekyll
```

As far as I know there aren't any other required dependencies that won't get pulled in automatically.

I know there are a ton of instructions on installing Jekyll out there, but nearly all of them miss
**ruby-dev** and **nodejs** which are required. Without you'll get some subtle and vague errors,
especially if **ruby-dev** is missing which will cause the **gem** command to fail.
