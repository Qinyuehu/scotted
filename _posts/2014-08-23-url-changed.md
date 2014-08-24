---
layout: post
title: "URL Changed"
date: 2014-08-23 12:35:36 -0700
comments: true
categories:
---

I decided to change this blog's URL from the previous sail2software.com domain to my
default github user page of scotte.github.io. I did some trickery so old permalinks
to the old domain get redirected to the new domain and new permalink URLs - so as long
as I renew sail2software.com, those old permalinks that are embeded in random places
around the internet will still work.

Why did I make this change? A couple of reasons:

* Simplicity, for one. Github pages makes it so darn easy to publish static blogs
with Jekyll, and while it's easy enough to use a custom domain, it's one less thing
I ultimately have to maintain now.

* Although I initially thought the name sail2software was somewhat catchy, over time I
grew less fond of it. Ultimately, this is a space of various personal thoughts and
reference, it's not a company or organization - so might as well just associate it
directly with myself. It doesn't matter how tired I am with my name, it's my name. :-)

* Having a CNAME on a user page forces *all* project github pages for a particular
github user to use that CNAME (they are relative URLs off it). I found this a bit
annoying, and while I could have just used a project page with a CNAME (which is what
I was doing before), I just sort of decided to use my user project page, so I went with it.

* The internet is polluted with a lot of vanity domain names. In the end, I don't think
a custom domain ends up resulting in any more traffic - I guess I'll find out!

Why is scotte.github.io OK, where sail2software.blogspot.com (the default blogger
domain back when this was a blogger site)? I don't really have a good answer for that.
It is a bit shorter, I guess, but there's more to it in there somewhere.

One consequence of this change is that when I re-imported all the disqus comments they
lost some of the info - all the comments (what few there are) still exist, but user
info is now a bit more generic. Oh well...

Now that I've had a couple of days on straight Jekyll, I am super super happy I made
the switch off Octopress. I no longer have to run rake commands to do anything, or
manage the separate deployment branch that it uses. I just check into **master** and
the rest is magic, I don't have to do anything else. Love that!

```
"The more that things change the more they stay the same" - RUSH Circumstances
```
