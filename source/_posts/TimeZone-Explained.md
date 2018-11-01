---
title: TimeZone Explained
date: 2018-10-28 23:12:24
tags:
---
  
http://www.linuxsa.org.au/tips/time.html  

Linux Tips Linux, Clocks, and Time
==================================
* * *

Introduction
------------

This document explains how to set your computer's clock from Linux, how to set your timezone, and other stuff related to Linux and how it does its time-keeping.

Your computer has two timepieces; a battery-backed one that is always running (the \`\`hardware'', \`\`BIOS'', or \`\`CMOS'' clock), and another that is maintained by the operating system currently running on your computer (the \`\`system'' clock). The hardware clock is generally only used to set the system clock when your operating system boots, and then from that point until you reboot or turn off your system, the system clock is the one used to keep track of time.

On Linux systems, you have a choice of keeping the hardware clock in UTC/GMT time or local time. The preferred option is to keep it in UTC because then daylight savings can be automatically accounted for. The only disadvantage with keeping the hardware clock in UTC is that if you dual boot with an operating system (such as DOS) that expects the hardware clock to be set to local time, the time will always be wrong in that operating system.

Setting your timezone
---------------------

The timezone under Linux is set by a symbolic link from `/etc/localtime`[\[1\]](http://www.linuxsa.org.au/tips/time.html#note1) to a file in the `/usr/share/zoneinfo`[\[2\]](http://www.linuxsa.org.au/tips/time.html#note2) directory that corresponds with what timezone you are in. For example, since I'm in South Australia, `/etc/localtime` is a symlink to `/usr/share/zoneinfo/Australia/South`. To set this link, type:

`ln -sf ../usr/share/zoneinfo/_your/zone_ /etc/localtime`

Replace `_your/zone_` with something like `Australia/NSW` or `Australia/Perth`. Have a look in the directories under `/usr/share/zoneinfo` to see what timezones are available.

\[1\] This assumes that `/usr/share/zoneinfo` is linked to `/etc/localtime` as it is under Red Hat Linux.

\[2\] On older systems, you'll find that `/usr/lib/zoneinfo` is used instead of `/usr/share/zoneinfo`. See also the later section ``[The time in some applications is wrong](http://www.linuxsa.org.au/tips/time.html#wrongtime)''.

Setting UTC or local time
-------------------------

When Linux boots, one of the initialisation scripts will run the `/sbin/hwclock` program to copy the current hardware clock time to the system clock. `hwclock` will assume the hardware clock is set to local time unless it is run with the `--utc` switch. Rather than editing the startup script, under Red Hat Linux you should edit the `/etc/sysconfig/clock` file and change the ```UTC`'' line to either \`\`UTC=true'' or \`\`UTC=false'' as appropriate.

Setting the system clock
------------------------

To set the system clock under Linux, use the `date` command. As an example, to set the current time and date to July 31, 11:16pm, type ```date 07312316`'' (note that the time is given in 24 hour notation). If you wanted to change the year as well, you could type ```date 073123161998`''. To set the seconds as well, type ```date 07312316.30`'' or ```date 073123161998.30`''. To see what Linux thinks the current local time is, run `date` with no arguments.

Setting the hardware clock
--------------------------

To set the hardware clock, my favourite way is to set the system clock first, and then set the hardware clock to the current system clock by typing ```/sbin/hwclock --systohc`'' (or ```/sbin/hwclock --systohc --utc`'' if you are keeping the hardware clock in UTC). To see what the hardware clock is currently set to, run `hwclock` with no arguments. If the hardware clock is in UTC and you want to see the local equivalent, type ```/sbin/hwclock --utc`''

The time in some applications is wrong
--------------------------------------

If some applications (such as `date`) display the correct time, but others don't, and you are running Red Hat Linux 5.0 or 5.1, you most likely have run into a bug caused by a move of the timezone information from `/usr/lib/zoneinfo` to `/usr/share/zoneinfo`. The fix is to create a symbolic link from `/usr/lib/zoneinfo` to `/usr/share/zoneinfo`: ```ln -s ../share/zoneinfo /usr/lib/zoneinfo`''.

Summary
-------

*   `/etc/sysconfig/clock` sets whether the hardware clock is stored as UTC or local time.
    
*   Symlink `/etc/localtime` to `/usr/share/zoneinfo/...` to set your timezone.
    
*   Run ```date MMDDhhmm`'' to set the current system date/time.
    
*   Type ```/sbin/hwclock --systohc [--utc]`'' to set the hardware clock.
    

Other interesting notes
-----------------------

The Linux kernel always stores and calculates time as the number of seconds since midnight of the 1st of January 1970 UTC regardless of whether your hardware clock is stored as UTC or not. Conversions to your local time are done at run-time. One neat thing about this is that if someone is using your computer from a different timezone, they can set the TZ environment variable and all dates and times will appear correct for their timezone.

If the number of seconds since the 1st of January 1970 UTC is stored as an signed 32-bit integer (as it is on your Linux/Intel system), your clock will stop working sometime on the year 2038. Linux has no inherent Y2K problem, but it does have a year 2038 problem. Hopefully we'll all be running Linux on 64-bit systems by then. 64-bit integers will keep our clocks running quite well until aproximately the year 292271-million.

Other programs worth looking at
-------------------------------

*   `rdate` \- get the current time from a remote machine; can be used to set the system time.
    
*   `xntpd` \- like `rdate`, but it's extremely accurate and you need a permanent 'net connection. `xntpd` runs continuously and accounts for things like network delay and clock drift, but there's also a program (`ntpdate`) included that just sets the current time like rdate does.
    

Further information
-------------------

*   `date(1)`
    
*   `hwclock(8)`
    
*   `/usr/doc/HOWTO/mini/Clock`
