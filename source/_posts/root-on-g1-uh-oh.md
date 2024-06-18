---
title: root on G1? uh oh...
tags:
  - android
  - reverse engineering
  - root
  - sudo
  - telnetd
url: root-on-g1-uh-oh.html
id: 57
categories:
  - android
  - reverse engineering
date: 2008-11-05 16:30:01
---

Well it didn't take long - but root has been gained on the G1 hardware! It's actually a suprisingly simple solution, though it will most likely be fixed on the next update release. For archiving purposes I'll post the method below;

```
First, a warning. Root is hidden from you for a reason. Do not attempt this if you a: are not a linux guru already or b: value your G1
Obtaining root gives you power to do stuff you probably shouldnt do without the safety of your phone telling you NO!!

That said, lets get started!
1: download pterminal http://android-dls.com/files/src.com.poidio.terminal.apk and run it
2: start the telnetd service (“cd /system/bin” then “telnetd”
3: telnet into the device (if you don’t know, do not try this)

Your job, to find out how to make this permanent (well, how to get it on the device itself, or how to get it from the adb shell) and find out how this helps us flash custom (edited, unsigned) updates.

Also keep in mind, your next OTA update might disable this, so if you are testing, might want to deny the update.

Original idea from (we think): SplasPood
Original thread: http://forum.xda-developers.com/showthread.php?t=441081

Digg this story!!: http://digg.com/gadgets/Tmo_G1_rooted
```

Source: [http://android-dls.com.nyud.net/forum/index.php?f=15&t=151&rb_v=viewtopic](http://android-dls.com.nyud.net/forum/index.php?f=15&t=151&rb_v=viewtopic)

And a little extra tid-bit on how to keep this ability through-out boots. Essentially making a 'sudo' command:

```
It’s so stupid easy to root Android (TC4-RC19 and TC4-RC-29), it’s almost depressing that it took anyone more than ten minutes to find this local root.

You’ll note that /system is read only if you read the output of mount:

/dev/block/mtdblock3 /system yaffs2 ro 0 0

You can launch telnetd on the phone (use a local terminal *on* the phone):

$ /system/bin/telnetd

This is curious as according to ‘ls’, telnetd isn’t setuid:

$ ls -l /system/bin/telnetd
-rwxr-xr-x root shell 9752 2008-09-13 01:13 telnetd

Two other programs in /system/bin have a setgid bit by default:

-rwxr-sr-x root inet 5616 2008-09-13 01:13 netcfg
-rwxr-sr-x root net_raw 26628 2008-09-13 01:13 ping

And yet, despite the lack of any setuid or setgid bit on telnetd… We can now telnet into the Android handset and setuid the shell (what? toolbox doesn’t drop privs if it sees this? ha!):

rorreoi@bridge:~/Desktop$ telnet 192.168.1.69
Trying 192.168.1.69…
Connected to 192.168.1.69.
Escape character is ‘^]’.
# id
uid=0(root) gid=0(root)
# chmod 4777 /system/bin/sh
Unable to chmod /system/bin/sh: Read-only file system
# mount -o remount,rw -t yaffs2 /dev/block/mtdblock3 /system
# chmod 4777 /system/bin/sh

Now when you execute ‘/system/bin/sh’, you should have root:

$ /system/bin/sh
#

This survives (as it should) a reboot. You should have local root until an OTA update removes the setuid bit from all of the programs you setuid.

Alternatively, you may want to setup a different program to give you root. It’s unlikely that you’d like your shell to always be running as uid/gid 0.

Fire up that setuid shell and then simply create a copy of your shell:

cd /system/bin/
cat sh > spawn-root-shell
chmod 4755 spawn-root-shell

You may also want to remove the setuid bit from /system/bin/sh if you care.
```

Source: [http://ioerror.livejournal.com/495953.html](http://ioerror.livejournal.com/495953.html)

Hmmm, this is great information. Now we should be able to access _any_ and _all_ files. I'm not on my developement machine right now, but you can bet yourself as soon as I get home I'll be checking access to the /data directory and seeing if we can dump any files from it!