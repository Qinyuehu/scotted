---
layout: post
title: "Nvidia kernel on backported kernel in Wheezy"
date: 2014-09-30 16:21:58 -0700
comments: true
categories:
---

If you like to use a backported kernel in Debian Wheezy, like I do, and have
systems with Nvidia graphics cards, you've likely discovered an incompatibility
between recent kernels (since 3.13) and the Nvidia driver.

Because of this problem, I've been using the Nouveau driver since 3.13,
but today I decided to give **nvidia-kernel-dkms** another try after getting
upgraded to the 3.16 kernel.

I was disappointed to find the same incompatibility still exists. Specifically,
the problem is that a *acpi_os_wait_events_complete* was removed in the Linux
kernel, but the Nvidia driver still references it. You'll you know you've hit
this problem when X doesn't start and **dmesg** or **/var/log/syslog** contains:

```
kernel: [   27.487723] nvidia: Unknown symbol acpi_os_wait_events_complete (err 0)
```

Instead of just doing my usual thing of switching back to the Nouveau driver, I was
curious how hard it would be to patch the driver. I know the patch for this exists,
and part of the whole point of DKMS is to make it easy to compile drivers that are not part
of the core kernel.

As it turns out, this problem is really, really easy to fix and you should be running
with the Nvidia driver in just a few minutes if you follow these instructions.

Instructions
============

* This assumes you are already running on a backported kernel. I'm using 3.16, but this
should work for 3.13 and 3.14 kernels as well (there was no backport of 3.15 in Wheezy,
as far as I am aware).
* First, follow the instructions
[here on the Debian wiki](https://wiki.debian.org/NvidiaGraphicsDrivers#wheezy-backports)
under the Wheezy backports section. Make sure you follow all of those 6 steps exactly.
* After rebooting, you'll have no X display and see the *Unknown symbol* error message
seen above.
* Switch to a VT (Alt-F1) and login to your system.
* As root, edit the file **/usr/src/nvidia-current-319.82/nv-acpi.c** (As of this
time, the *319-82* part may change if the backport is updated, and of course the
contents of the file might change as well - you may have to find the section in
question). Comment out lines
304-310 so the file looks like:

```
/*
    if (pNvAcpiObject->notify_handler_installed)
    {
        NV_ACPI_OS_WAIT_EVENTS_COMPLETE();

        // remove event notifier
        status = acpi_remove_notify_handler(device->handle, ACPI_DEVICE_NOTIFY, nv_acpi_event);
    }
*/
```
* Now run the following command to rebuild the Nvidia DKMS module:

```
$ sudo dpkg-reconfigure nvidia-kernel-dkms
```

* And finally, restart your display manager (I use lightdm, so yours might be different):

```
$ sudo /etc/init.d/lightdm stop
$ sudo /etc/init.d/lightdm start
```

* And that's it! There's no need to reboot, X will load the Nvidia driver automatically as
long as you set up **/etc/X11/xorg.conf.d/20-nvidia.conf** per the Debian wiki's instructions.

Of course, if you get a system update that changes the **nv-acpi.c** file, then you'll have to
reapply the patch. As long as that file isn't changed, things will otherwise be fine, even if
your kernel gets upgraded, since a new DKMS module will be built from the modified source.
