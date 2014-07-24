---
layout: post
title: "VOIP Solution Round FOUR"
date: 2014-07-23 19:54:05 -0700
comments: true
categories:
 - communication
 - telephony
---

Oh no, if you've been reading this blog for a long time I already know what you
are thinking - here we go,
[switching VOIP providers yet again](http://www.sail2software.com/2012/11/voip-solution-round-three.html).
Don't worry, [Flowroute](http://www.flowroute.com) continues to be my SIP
provider of choice (they are, quite simple, awesome).

No, this round was prompted by a hardware failure when our 5 year old Linksys
PAP2TNA bit the dust. A couple of days ago my wife went to use the phone - so
she hit the Talk button to take it off-hook, but then realized she needed some
additional information so hung up. The ATA was making a dial-tone, because I
could hear it from across the room. Two minutes later she hit the Talk button
again, and got... silence. No dial-tone, just nothing.

Hmmm, that's weird - so I go and take a look at the Linksys ATA and find it
boot-looping. The power LED would flash green, the LAN LED would flash green,
then the power LED would go red and it would reboot itself. I cycled the power,
even left it unplugged for the day, but it appears that it decided to randomly
fry itself.

Well, time for a new ATA. I had heard about Obihai's affordable ATAs and found
that the local Fry's had an [OBi100](http://www.obihai.com/obi100pr) in stock
for around $40. The OBi100 is a very basic model, supporting just a single POTS
line, but that's all we need.

OBi devices have two modes of configuration - you can browse directly to the
device on your local network to configure services, or auto-provision through
the [ObiTalk](http://www.obitalk.com) website. Although I'm familiar enough with
SIP configuration that I don't mind direct device config, I decided to provision
it via ObiTalk.

Flowroute isn't one of Obihai's "certified" providers, but it's just SIP and I
found a
[config guide](https://support.flowroute.com/entries/26064109-Obihai-OBi202-Configuration-Guide)
at Flowroute's website. The config guide was for direct device config, but the
fields are the same, so it was easy enough to provision.

And it worked! Well, actually it only sort of worked. I could make outbound
calls via Flowroute, but inbound calls to my DID would just result in a busy
signal. At this point, I wasn't sure if it was something in the SIP negotiation
preventing Flowroute from routing my DID, or something in the OBi device, or
just something I screwed up in the config.

So, I gave up on ObiTalk - I deleted my device from ObiTalk, reset the OBi
device to factory defaults, and then started manual SIP configuration. Guess
what? It worked - fully! Both inbound and outbound calls were working fine.

At this point, I still wasn't sure where the problem was - all the possibilities
were still, well, possibilities. So this morning I opened a support case with
Flowroute. Realize that I'm about as small potatoes of a customer that Flowroute
can have - I give them just a couple of bucks a month for
DID, E911, and per-minute usage. Yet, each time I've opened a support case they
have been responsive, thorough, and just plain fantastic.

Today was no exception,
with the support person providing the key information that they have seen issues
on OBi devices where *X_InboundCallRoute* is misconfigured. The result of this
is that everything is great on the Flowroute end, but the OBi device doesn't
know what to do with the call and gives a busy signal. She suggested making
sure the value for this was "ph". That seemed to ring a bell - I remember
noticing that my manual configuration had this value, but the provisioned value
from ObiTalk did not.

I thanked Flowroute support, but they didn't stop there - they gave me instructions
on doing a tcpdump so that if I wasn't able to get it to work from ObiTalk, they
could look at the SIP conversation to debug it. I was stunned! I've worked with
dozens and dozens of vendors and providers over the years, and given
several of them tcpdumps which they had no clue what to do with. In contrast,
Flowroute was willing to dig this deep, into a problem for which I already had
a workaround. That's pretty cool. I can think of a few companies who could learn
from this.

I am happy to report that after re-provisioning my Obi100 from ObiTalk, and then
going into the advanced settings to reset *X_InboundCallRoute*, I have a fully
working, auto-provisioned device! It's pretty annoying that ObiTalk
corrupts, for some unknown reason, the correct value and overwrites it with a
wrong one. I don't know why they do this, but it nearly resulted in
me returning my OBi100 to the store for a refund.

Flowroute - thanks again for your help and support in getting this fully working!

And for anyone reading this - if you are paying your phone company, cable company,
or VOIP provider more than $5 a month for phone service you are getting seriously
ripped off. Cancel that, pick up an OBi100 for $40, and go set up an account at
[Flowroute](http://www.flowroute.com). They will provision a DID instantly in
the area code of your choosing, and you'll be up and running pretty darn quick.
I don't know if they can port existing numbers,
but we switched to bouncing through a Google Voice number several years ago,
so our friends+family know to use that number, not the ANI (CallerID) number
they see when we call them.

One final note - Obihai used to work directly with Google Voice, but Google
disabled that service. Obihai is doing an incredibly *horrible* job of getting
the word out, as all of their web pages still say that this is a feature of
the device - certainly they sell more devices with this false claim in their
advertising, but I wonder how many get returned solely because of it. At any
rate, it's yet another case of Google killing off a useful service, but I can
see how they weren't making any money off it.
