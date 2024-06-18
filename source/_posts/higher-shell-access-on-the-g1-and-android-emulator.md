---
title: Higher shell access on the G1 and Android Emulator?
tags:
  - android
  - g1
  - reverse engineering
  - root
  - shell
url: higher-shell-access-on-the-g1-and-android-emulator.html
id: 52
categories:
  - android
date: 2008-11-04 01:38:21
---

Well maybe we've hit some good news, and frankly I've been swamped digging into code looking for ways to access the folders and files that we've been locked out of. The patching example will have to take the back seat for now, but it will be posted! I promise... Mostly tomorrow if I have the chance to finish throwing it together.
Anyway after some digging in the files located on both the emulator and the G1 hardware device I've found some interesting things. One that will take the forefront right now is the "permissions.xml" file, located in "/system/etc"
The file has some comments in it, comments that make it very helful to us!
```
<!-- This file is used to define the mappings between lower-level system
     user and group IDs and the higher-level permission names managed
     by the platform.

     Be VERY careful when editing this file!  Mistakes made here can open
     big security holes.
-->

    <!-- The following tags are assigning high-level permissions to specific
         user IDs.  These are used to allow specific core system users to
         perform the given operations with the higher-level framework.  For
         example, we give a wide variety of permissions to the shell user
         since that is the user the adb shell runs under and developers and
         others should have a fairly open environment in which to
         interact with the system. -->

    <!-- RW permissions to any system resources owned by group 'diag'.
         This is for carrier and manufacture diagnostics tools that must be
         installable from the framework. Be careful. -->
```
Hmmm.. This could be good news for us, saddly the file is read-only! We can pull it, using `./adb -e` (or `-d` depending on the device) `/system/etc/permissions.xml` but we can't push it back. After scouring the internet for some relevant information, it seems that someone has been able to snag the upcoming update for the android os, and found out how to install it. This information I found at the [xda-developers forum](http://forum.xda-developers.com/forumdisplay.php?f=448) and as in mid-writing of this post their forum seems to be bogged down another good link is located at [Android Community](http://androidcommunity.com/forums/f28/learn-how-to-update-using-your-microsd-card-5784/).

Though as of right now I'm going to get cracking on trying to find some type of hole we can poke and give ourselves extra permissions and possibly resign the update package and get it to run. Hopefully I'll be back soon with good news!