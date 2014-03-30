---
layout: post
title: "systemd is in the trees"
date: 2014-03-29 16:02:40 -0700
comments: true
categories:
 - linux
 - software
---

You read that right - systemd "is in the trees... It's coming!"*\**

If you pay attention to things happening in the world of Linux, you probably
know that [systemd](http://freedesktop.org/wiki/Software/systemd/) will be
replacing sysvinit for Debian systems.

In the last couple of days I've switched 5 Debian testing/Jessie systems
from sysvinit to systemd. Overall, it's been a smooth transition, with just a
couple of problems I ran into which were not ultimately systemd issues.

It's easy to test out, because you can selectively boot with systemd as long
as you don't install **systemd-sysv**, which enables systemd by default. I
think this is a good way to test, and if the system boots OK and everything
works as expected, you can then go all-in and enable systemd by default.

I also discovered the debian systemd implementation doesn't run **/etc/rc.local**
by default, but that's easy to solve as well.

The debian wiki has
[excellent documentation on systemd](https://wiki.debian.org/systemd) and
includes links for other common distributions as well.

As far as the two (non systemd related) issues I ran into:

- On one server system, any USB related operations would hang with a kernel call
stack dumped to dmesg. It turned out that this was some issue related to
upgrading to the 3.13 kernel and had nothing to do with systemd. I verified this
because I ran into the same issue when booting with sysvinit under the kernel. I
was able to workaround this problem by unplugging a certain USB device (an
external hard drive) until after the system booted. This system is working fine
with systemd, and the external drive, I just have to unplug it if I need to
reboot. Had I simply rebooted before trying out systemd I would have found this
issue and not been debugging the wrong thing.

- On my Acer C720 chromebook (yes, it runs Debian), the kernel would panic
when I attempted to boot with **init=/bin/systemd**. I'm a bit embarrassed to
admit what the cause was... Well, I forgot one little important step in this
process: I forgot to install systemd!

Installing systemd
------------------

Make sure your system is fully up to date and running the latest kernel. I.e.

    $ sudo aptitude full-upgrade

At this point you should reboot if any major updates, such as a new kernel,
were installed. You only want to be debugging one problem, not two as I
described in my issues above!

**Now is a really good time to make a full backup of your system, as well.**

To start with, we'll install systemd alongside sysvinit:

    $ sudo aptitude install systemd

This will likely bring along a few dependencies, such as *libpam-systemd*,
*libsystemd-daemon0*, *libsystemd-journal0*, and *libsystemd-login0*.

Testing systemd
---------------

Now you'll need to reboot and manually edit the boot line in grub to enable
systemd. To do this, hit '**e**' at the grub boot menu for your kernel,
and append **init=/bin/systemd** to the line that starts with **linux**. It
might look something like the following, depending on the options for your
specific kernel:

    linux   /vmlinuz-3.13-1-amd64 root=/dev/mapper/root-root ro quiet init=/bin/systemd

Press **F10** or **ctrl+x** to boot. If all goes well, and it should, your
system will boot and run like normal. You can tell if systemd is in use if
*PID* 1 is **systemd* and not **init**, for example:

    $ ps -p1
    PID TTY          TIME CMD
      1 ?        00:00:01 systemd

Watch out! some **ps** options will show it as */sbin/init*, so be careful
not to confuse yourself.

At this point, we can also use systemctl:

    $ systemctl status
    [long output deleted]

The nice thing about testing this way is that the systemd switch is not
persistent. If you reboot again, you are back to sysvinit.

If you are happy at this point, and your system is stable, you can install
*systemd-sysv*, which will remove sysvinit. You can also run in this testing
configuration for a week or so if you prefer.

Enabling systemd by default
---------------------------

Once you do this, there's no going back! If your system started ok with
systemd, it should be safe to do this - just be aware that you absorb some
risk...

    $ sudo aptitude install systemd-sysv

This will remove sysvinit and make systemd the default. Next, reboot:

    $ sudo shutdown -r now

Your system should reboot cleanly with systemd. Once again, you can use
**systemctl status** to verify that everything looks good.

Longer term testing
-------------------

If you aren't ready to commit to systemd, you can always modify your grub
configuration so that it boots with systemd by default, but if you run into
trouble you can always disable it again. Note that if you have a non-booting
system due to systemd you'll have to boot a rescue livecd/liveusb and modify
**/boot/grub/grub.cfg** manually.

To configure grub to boot with systemd by default, add **init=/bin/systemd**
to the **GRUB_CMDLINE_LINUX_DEFAULT** line of **/etc/default/grub** such that
it looks something like:

    GRUB_CMDLINE_LINUX_DEFAULT="quiet init=/bin/systemd"

Note that yours may look different.

Configuring rc.local under systemd
----------------------------------

Finally, let's get **rc.local** working under systemd.

Create the following file as **/etc/systemd/system/rc-local.service**:

```
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU General Public License as published by
#  the Free Software Foundation; either version 2 of the License, or
#  (at your option) any later version.

[Unit]
Description=/etc/rc.local Compatibility
ConditionPathExists=/etc/rc.local

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```

This file needs to be executable, or systemd won't run it (I discovered this
the hard way), and we need to tell systemd to enable our new service:

    $ sudo chmod +x /etc/systemd/system/rc-local.service
    $ sudo systemctl enable rc-local

After rebooting, you can use **systemctl** to check that everything worked
as expected:

```
$ systemctl status rc.local
rc-local.service - /etc/rc.local Compatibility
   Loaded: loaded (/etc/systemd/system/rc-local.service; enabled)
   Active: active (exited) since Sat 2014-03-29 15:49:01 PDT; 1h 15min ago
  Process: 1097 ExecStart=/etc/rc.local start (code=exited, status=0/SUCCESS)
```

And that's all there is to it! So far, systemd is a rather nice upgrade
over the aging sysvinit. It will take some time for all your various services
and programs to get updated with systemd init scripts, but until then the
default systemd compatibility mode will still start all those scripts for you.

*\** Yes, this is a reference to the Kate Bush lyric from
[*Hounds of Love*](https://en.wikipedia.org/wiki/Hounds_of_Love),
originating in the movie
[*Curse of the Demon*](http://www.imdb.com/title/tt0050766/) aka
[*Night of the Demon*](https://en.wikipedia.org/wiki/Night_of_the_demon).
