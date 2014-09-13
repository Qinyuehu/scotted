---
layout: post
title: "wpagui - The Dark Horse Wifi Manager"
date: 2014-09-13 08:33:00 -0700
comments: true
categories:
---

Lots of folks use NetworkManager or wicd without realizing wpasupplicant already
has a decent GUI (and CLI) for managing wifi networks. To use it effectively you have
to configure wpasupplicant for roaming, but not only is that pretty easy to do,
it's well documented (on Debian systems, in _/usr/share/doc/wpa_supplicant/README.Debian.gz_).

Why not just use NetworkManager or wicd? A few reasons:

* NetworkManager is huge, and requires large bits of either KDE or Gnome in order to
use the GUI. Since I'm using [i3wm](http://www.i3wm.org), I don't really want to fill my
disk up with a bunch of Gnome stuff simply to connect to wireless.
* wicd is crufty and janky.
* Both NetworkManager and wicd have bugs where they can't scan or connect in environments
that have large numbers of access points.

That last one is the killer for me. At work there are many dozens of visible access points
in most areas, and both NetworkManager and wicd can't cope.

Configuration
=============

Setting up wpasupplicant for roaming requires editing a couple of files to enable it, but
actual access point configuration is done via wpa_gui or wpa_cli as you prefer.

Modify **/etc/network/interfaces** so your wireless interface looks like:

```
auto wlan0
allow-hotplug wlan0
iface wlan0 inet manual
    wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
iface wlan0 inet6 auto

```

And create **/etc/wpa_supplicant/wpa_supplicant.conf**, making sure it's owned by root and
chmod 600:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
```

Don't forget to add yourself to the **netdev** group as needed.

Now, use **wpa_gui** (or **wpa_cli** if you wish) to scan and configure your wireless networks.

The wpa_supplicant.conf will have plaintext passwords - some people consider this an
issue, but since all my laptops use LUKS encrypted volumes, it's no risk for me in a
powered off system. And if someone did compromise a running system they could certainly
see the key if they had root access - but I can also steal that directly from wpasupplicant
on a running system with NetworkManager or wicd. So, in summary - there is no additional
risk in using wpasupplicant directly on a properly secured system.

wpasupplicant manages wifi, but not dhcp (where NetworkManager and wicd handle both).
Sometimes, this means I don't get an ipv4 DHCP address until I run:

```
$ sudo ifup wlan0
```

Eventually I'll modify wpasupplicant's postup action to do this automatically, but I don't
find this that big of a deal - certainly no worse than having to manually disconnect/connect
to access points that you end up having to do with NetworkManager anyway (and in other
operating systems - I see MacOS users having to do this constantly).
