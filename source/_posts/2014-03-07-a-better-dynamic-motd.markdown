---
layout: post
title: "A better dynamic MOTD"
date: 2014-03-07 19:51:08 -0800
comments: true
categories:
---

![A better dynamic MOTD](/images/dynamic-motd.png)

Debian based systems (and derivatives such as Ubuntu) have a facility built into
PAM that can display a dynamically generated MOTD on login. Debian doesn't use
this by default, but Ubuntu does. I wanted to add this to my Debian
testing/Jessie boxes, but the Ubuntu version performs horribly - if you've ever
wondered why Ubuntu hangs for a second or so upon login while displaying the
MOTD, this is why.

Taking a closer look at the Ubuntu */etc/update-motd.d/* files, it was clear to
me why the default Ubuntu implementation is so slow - Two reasons, in fact.
First, text that doesn't change frequently is generated every time, such as the
hostname banner and the script to display the number of available updates. The
latter is horribly slow and something that doesn't need to be checked at *every*
login anyway. Second, the script for truly dynamic content forks way more
processes than necessary and can be easily tuned and improved.

With my revisions to these scripts and process, my logins are instantaneous and
I've even added running the MOTD display on each invocation of
[urxvt](http://software.schmorp.de/pkg/rxvt-unicode), the terminal program I
use.

Here's how to implement a much better dynamic MOTD. The source for this is
available in my [linux-configs github project](https://github.com/scotte/linux-configs).

Make static content static
--------------------------

What I did was separate the scripts into two configuration directories - one
containing dynamic content, generated on each execution (just like the default),
and a second for static content, which is only generated occasionally via cron
(every 30 minutes).

The cron job uses *run-parts* to run the static content scripts, which write to
*/var/run*. These files are then read directly via the dynamic content scripts.

Here's a brief overview of the layout, but more detail about the scripts is
provided below as well.

[/etc/update-motd_local.d](https://github.com/scotte/linux-configs/tree/master/etc/update-motd_local.d)
contains the static content scripts, run by a simple cron job.

[/etc/update-motd.d/](https://github.com/scotte/linux-configs/tree/master/etc/update-motd.d)
contains the dynamic content scripts. These scripts are also responsible for
displaying the static files from /var/run. There must at least be one script
in this directory for each static script, but there can also be additional
dynamic content scripts with no corresponding static content. Note that scripts
that simply cat the statically generated files are simply symbolic links to
*00-header*.

*/var/run/motd_local-* will contain the static content files.

And [here's the crontab](https://github.com/scotte/linux-configs/blob/master/configs/crontab-root).

Make dynamic content faster
---------------------------

As mentioned above, the default */etc/update-motd.d/10-sysinfo* file from Ubuntu
does considerable more forking than is necessary - even doing things such as

    cat foo | awk   # Don't do this!

instead of:

    awk <foo

The cat and pipe are entirely unnecessary.

Additionally, some of the awk scripts piped to other awk scripts, or were pipes
from grep to awk, which can all be handled via a single awk script. Also,
commands like "ps" or "free" were being run when the information is already
available in "/proc" My resulting script runs about 3 times faster than the
original, entirely excluding the static content improvements, and is
significantly nicer on system resources!

Where ssh would hang for a second or so on each login, it's now instantaneous.

You'll need figlet and update-notifier-common packages, if you don't already
have them.

    $ sudo aptitude install figlet update-notifier-common

A closer look at the scripts
----------------------------

### The dynamic scripts

Here's the updated */etc/update-motd.d/10-sysinfo*:

```
#!/bin/bash
#
#    10-sysinfo - generate the system information
#    Copyright (c) 2013 Nick Charlton
#
#    Authors: Nick Charlton <hello@nickcharlton.net>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

#
# The upstream version of this script was very inefficient - forking processes
# when not needed. This version significantly reducses the number of processes
# required to get the same info, and as a result is much, much faster.
#
# Additionally, static-ish stuff like the hostname and packages to install
# is only generated once every 30 minutes (or as configured in cron).
#
# As a result, this shaves off the amount of time required to login to the system
# by about 1 second or so, and when running as part of urxvt is nearly instant.

load=`awk '{print $1}' /proc/loadavg`
root_usage=`df -h / | awk '/\// {print $(NF-1)}'`
memory_usage=`awk '/^MemTotal:/ {total=$2} /^MemFree:/ {free=$2} /^Buffers:/ {buffers=$2} /^Cached:/ {cached=$2} END { printf("%3.1f%%", (total-(free+buffers+cached))/total*100)}' /proc/meminfo`
swap_usage=`awk '/^SwapTotal:/ { total=$2 } /^SwapFree:/ { free=$2} END { printf("%3.1f%%", (total-free)/total*100 )}' /proc/meminfo`
users=`users | wc -w`
time=`awk '{uptime=$1} END {days = int(uptime/86400); hours = int((uptime-(days*86400))/3600); printf("%d days, %d hours", days, hours)}' /proc/uptime`
processes=`/bin/ls -d /proc/[0-9]* | wc -l`
ip=`/sbin/ifconfig eth0 | awk -F"[: ]+" '/inet addr:/{print $4}'`

printf "System load:\t%s\t\tIP Address:\t%s\n" $load $ip
printf "Memory usage:\t%s\t\tSystem uptime:\t%s\n" $memory_usage "$time"
printf "Usage on /:\t%s\t\tSwap usage:\t%s\n" $root_usage $swap_usage
printf "Local Users:\t%s\t\tProcesses:\t%s\n" $users $processes
echo
```

The scripts that cat the static content look like the below, and actually,
there's just one of these, the rest are simply symbolic links to the first, as
we use the script filename to determine which static file to show:

```
#!/bin/sh
#
# symlink this to additiona files as needed, matching scripts in
# /etc/update-motd_local.d

cat /var/run/motd_local-$(basename $0)
```

In my case, I have *00-header*, *20-sysinfo*, and *90-footer*, matching the same
dynamic scripts.

### The static scripts

The static scripts are *00-header*, *20-sysinfo*, and *90-footer*, as listed
just above. There is *not* a *10-sysinfo* script in the static scripts, since
that is dynamic only. Make sure you understand *run-parts*, as it is key to how
these scripts are executed.

Let's take a look at *00-header* next. This isn't much different than the
original, except we dup the stdout file descriptor to write to our file in
/var/run (a partial filename is passed in as the first argument).

I also chose a [figlet](http://www.figlet.org/) font that I like better as
well, which doesn't take up quite as much space and, well, it looks spiffy.

```
#!/bin/sh
#
#    00-header - create the header of the MOTD
#    Copyright (c) 2013 Nick Charlton
#    Copyright (c) 2009-2010 Canonical Ltd.
#
#    Authors: Nick Charlton <hello@nickcharlton.net>
#             Dustin Kirkland <kirkland@canonical.com>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

OUT=${1}$(basename $0)
exec >${OUT}

[ -r /etc/lsb-release ] && . /etc/lsb-release

if [ -z "$DISTRIB_DESCRIPTION" ] && [ -x /usr/bin/lsb_release ]; then
        # Fall back to using the very slow lsb_release utility
        DISTRIB_DESCRIPTION=$(lsb_release -s -d)
fi

figlet -f smslant $(hostname)

printf "Welcome to %s (%s).\n" "$DISTRIB_DESCRIPTION" "$(uname -r)"
printf "\n"
```

Here's *20-sysinfo* - the most expensive part of the original script, which
determines how many packages are out of date and whether a system reboot is
needed:

```
#!/bin/bash
#
#    20-sysinfo - generate the system information
#    Copyright (c) 2013 Nick Charlton
#
#    Authors: Nick Charlton <hello@nickcharlton.net>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

OUT=${1}$(basename $0)
exec >${OUT}

/usr/lib/update-notifier/apt-check --human-readable
/usr/lib/update-notifier/update-motd-reboot-required
echo
```

And finally, the diminutive *90-footer* which just appends the content of
*/etc/motd.tail* if it exists:

```
#!/bin/sh
#
#    99-footer - write the admin's footer to the MOTD
#    Copyright (c) 2013 Nick Charlton
#    Copyright (c) 2009-2010 Canonical Ltd.
#
#    Authors: Nick Charlton <hello@nickcharlton.net>
#             Dustin Kirkland <kirkland@canonical.com>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

OUT=${1}$(basename $0)
exec >${OUT}

[ -f /etc/motd.tail ] && cat /etc/motd.tail || true
```

Those scripts could probably use some fencing, such as checking that the
directory argument is valid, and that the directory exists.

### The crontab entry

The crontab to run the static scripts every 30 minutes is quite trivial:

```
*/30 * * * * /bin/run-parts --arg=/var/run/motd_local- /etc/update-motd_local.d
```

**Important: Don't forget to run that same command in */etc/rc.local* or the static
content files won't be populated in /var/run until the cron job runs!**

### My urxvt launcher script

Here's my *~/bin/urxvt* that I use to launch urxvt:

```
exec /usr/bin/urxvt -e sh -c "run-parts /etc/update-motd.d; exec $SHELL"
```

And finally, my *i3* keybinding line:

```
bindsym $mod+Return exec bin/urxvt
```

The only other non-obvious detail is that */etd/motd* needs to be a symbolic
link to */var/run/motd*, which you can set up like so:

```
$ sudo rm /etc/motd
$ sudo ln -s /var/run/motd /etc/motd
```

If you don't already have them, you'll need a couple of packages:

```
$ sudo aptitude install figlet update-notifier-common
```

And that about covers it! With this, I have a nice dynamic MOTD which doesn't
slow me down.

Note that the scripts above may become out of date over time. Check my [linux-configs github repo](https://github.com/scotte/linux-configs) for the latest, up-to-date version.
