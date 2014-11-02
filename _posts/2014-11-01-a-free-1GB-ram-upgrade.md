---
layout: post
title: "A Free 1GB RAM Upgrade"
date: 2014-11-01 17:13:47 -0700
comments: true
categories:
---

Last spring a built a cheap little Linux box for a server, consolidating
NAS storage and a few other functions (fetchmail receiver, IMAP server,
IRC bouncer, btsync master, Plex server, network backups, etc). Although
I wasn't on any particular budget, I was going for cheap but decent, with
noise and power consumption in mind, and ended up paying around $250 for a
case, mobo, CPU, and RAM. I already had a 1TB SATA drive from a Buffalo
Cloudstor NAS (what a slow piece of garbage - except for the drive).

I went with an AMD A6-6400K (dual core 3.9GHz). It has an onboard Radeon GPU,
which I don't care about as this is a headless server. What I didn't know
at the time, is that this integrated GPU uses system RAM, and the MSI
mainboard I got, an FM2-A75IA-E53, is somewhat limited in what options it
exposes for managing this.

So I built the system, and it ran great, but I noticed only 3GB of RAM
was available to Linux, even though the BIOS shows 4GB, and lshw showed
a 4GB module installed. Suspecting the BIOS was allocating RAM for the GPU,
I double checked the settings, but there was simply no way to choose the
amount of RAM consumed by the GPU.

I found a BIOS update, but then discovered there was no way to install
the update - you cannot update the BIOS from within the BIOS unless you
have Windows installed (what the heck, who came up with this idea?) - and
in fact the BIOS updates are distributed as a .exe file. Useless to me!

At this point, I gave up and just ignored the issue. 3GB really is enough
RAM for this server, and I just lost interest in trying to solve it for
a few months. I could have returned the motherboard, but in the end
decided dealing with Fry's return department wasn't worth the effort and
3GB was enough versus the hassle. So, I forgot all about it for months.

A couple of weeks ago, I remembered this whole thing and decided to try
and figure it out again. I started with the BIOS update, and found that
it could be done by creating an MS-DOS bootable USB stick, and found
the instructions to do that. And it worked! Great, now that I'm on the
latest BIOS I bet they fixed this.

My hopes were quickly dashed as I discovered there were still no options
in the MSI BIOS to control RAM for the GPU. I started to think that
maybe it wasn't this simple - maybe there was something else at fault here.
Maybe the BIOS wasn't exposing things correctly to Linux, or maybe some
esoteric IOMMU driver was needed, or the Linux kernel needed to
coordinate with the GPU, so I installed the radeon driver hoping that
maybe Linux had to ask for the RAM back once aware of the GPU. All of this,
whether crazy or not, led nowhere. And yes, of course I am running
a 64bit kernel.

I did a ton of Google searches and research on this particular problem,
back after building the system, then more recently as I worked through this
problem. I found vague references to others with similar issues, but nobody
had a solution that jived with the problem I was having. Searches of MSI's
own support site also turned up nothing, and I was about to check with their
tech support site (perhaps I should have done this from the beginning), when
I thought I'd try one last thing - was there a way to disable the GPU? I
didn't really want to go down this road, as while the server is headless, I
do occasionally plug it in to a monitor. I was also a bit concerned that if
I disabled the GPU then the system wouldn't POST at all and I'd have to
force the CMOS to be cleared on the BIOS and figure out how to get things back.
That all seemed like more effort than just ignoring the missing 1GB of RAM
to me.

So I [duckduckgo'd](https://duckduckgo.com) on how to disable the GPU in
the BIOS, wading through article after article and several posts on MSI's
tech support site when I came across
[this post](https://forum-en.msi.com/index.php?topic=166204.msg1218352).
Oddly, I had never come across this post in all my searches about how
to reduce the GPU memory, but this seemed to indicate there was a very
sneaky trick - to reduce GPU RAM, you have to select "Dual Graphics", then
(and only then) will the MSI BIOS expose the option to change RAM
allocation.

This is so very frustrating of MSI to bury this option inside some other
option that no sane person would think to try. It seems so simple now
that I know the answer, but just like a video game cheat code, you have
to know the exact right set of options in order to expose this one.

So now, I was able to reduce the 1GB allocation to a minimum of 32MB. That's
still a small chunk of wasted RAM, but 32MB doesn't make me lose any sleep.

So I rebooted, and... YES! Linux now has 4(ish)GB RAM available.
