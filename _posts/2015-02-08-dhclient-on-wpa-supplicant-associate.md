---
layout: post
title: "dhclient On wpa_supplicant Associate"
date: 2015-02-08 11:41:43 -0700
comments: true
categories:
---

As mentioned [here](/2014/09/wpagui-the-dark-horse-wifi-manager), I'm not a
fan of NetworkManager or wicd, and prefer using wpa_supplicant directly in
/etc/network/interfaces via wpa-roam. A downside to this approach is dhclient
isn't smart enough to keep trying to acquire an IP address when wifi connections
are lost and re-established, or when connecting to a different access point.

Fixing this in a reliable way isn't obvious, but it's not too hard either.
**wpa_cli** has a **-a** option where it will run as a daemon and execute an
arbitrary script each time the system connects or disconnects from a wifi
network. We can hook into this, killing off dhclient on disconnect, and
starting it up again on connect.

Here's my hook script, which I keep in **/etc/wpa_supplicant/dhcpaction.sh**:

```
#!/bin/sh

IFNAME=$1
CMD=$2

if [ "$CMD" = "CONNECTED" ]; then
    /sbin/dhclient -v -pf /run/dhclient.${IFNAME}.pid -lf /var/lib/dhcp/dhclient.${IFNAME}.leases ${IFNAME}
fi

if [ "$CMD" = "DISCONNECTED" ]; then
    /bin/kill $(/bin/cat /run/dhclient.${IFNAME}.pid)
fi
```

In my **/etc/rc.local** file I have the following, which starts wpa_cli in
it's special action script mode on boot (note /var/log/local must exist and
have appropriate permissions):

```
/sbin/wpa_cli -a /etc/wpa_supplicant/dhcpaction.sh >/var/log/local/wpa_action.log 2>&1 &
```

That's all there is to having wpa_supplicant reliably keep a wifi interface up!
