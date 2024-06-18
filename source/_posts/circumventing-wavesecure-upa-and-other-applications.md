---
title: Circumventing WaveSecure UPA and other applications
tags:
  - android
  - android os
  - locked
  - locking phone
  - pm disable
  - safe mode
  - Uninstall Protection Add-On
  - unlocking phone
  - WaveSecure
  - WaveSecure UPA
url: 351.html
id: 351
categories:
  - android
date: 2010-05-07 13:24:52
---

Continuing with the tread of "stealth" and "locking" devices I've decided to look at [WaveSecure](https://www.wavesecure.com/). I've been getting many emails regardin the _Uninstall Protection Add-on_. The concept behind WaveSecure and the UPA is that they monitor each other - locking the phone if either one is installed. Seems like a pretty slick way to protect the application - though like everything else in the world, there are loop holes. I've gotten a few emails from people asking how secure it really is, and even how to circumvent it if they forget the password or locked the phone. While I'm never sure what these peoples intentions really are, assuming they're not lying to me there is a good use case for knowing how to remove program that "lock down" your phone.
![Ops? How'd that happen!?](http://173.230.150.16/blog/wp-content/uploads/2010/05/device-locked-180x300.png "Ops? How'd that happen!?")
When removing either WaveSecure or the UPA program, you'll be prompted with a locked phone - as shown by the picture above. How do we get around that? How can we make this better? Well, there is a quick way to get around this that can work for or against you. If you've enabled debug mode on your phone which allows your to connect via ADB then your in luck - this is really easy. Simply using adb we can quickly disable the locking program.

If the Uninstall Protection Add-on was removed from your phone, then WaveSecure is the application locking your phone. The inverse is also true, this is important to know since we need to know which package to disable. UPA's package name is "com.wsandroid.uninstall_listener" while the main application (WaveSecure) package name is "com.wsandroid". Following the instructions below you can quickly disable the nessicary application to get into your phone;
```
adb shell pm disable
```
This will disable the package, as long as adb has root access - otherwise it will attempt to kill the process which should also gain you access. This method should also work for nearly any package you wish to disable.

The second method for getting around this lock is rebooting the phone into _Safe Mode_ - this will prevent any applications that are not system based from starting up. This includes any malware, spyware or locking applications. The good (bad?) thing about this is that you do not need to be rooted or have adb enabled to get into Safe Mode. Safe Mode can be booted into by holding "Menu" during boot up, though googling for specific directions for your phone might yield different results.