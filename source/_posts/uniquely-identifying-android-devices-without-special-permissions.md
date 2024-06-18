---
title: Uniquely Identifying Android Devices without special permissions.
tags:
  - algorithms
  - android
  - Android_ID
  - android.provider.system.settings
  - coding
  - g1
  - java
  - unique id
url: uniquely-identifying-android-devices-without-special-permissions.html
id: 113
categories:
  - android
date: 2008-11-24 17:29:27
---

Something that you always come across in writing algorithms for devices that you want to lock down is getting a grasp of the actual device it is intended to be on. Sometimes programmers want to lock a registration code not only to a name for registration, but also to a device. This can cut down on "sharing" of serial numbers etc. I was doing some research and looking for device specific information when I stumbled upon a few things. They are right out there in the open, but here they are just in case you have not seen them yet.

In [Android.Provider.Settings.System](http://code.google.com/android/reference/android/provider/Settings.System.html) we have some interesting values that could be of use, one specifically is "Android_ID". From the documentation it is the following;
> String ANDROID_ID The Android ID (a unique 64-bit value) as a hex string. “android_id”
Though while this is considered a "unique key", please keep in mind that if a program has write access to the Settings, which is possible, this could be changed easily. Though it could be a safe assuming that it should not be changed, and upon normal program usage it wouldn't be changed. Anyway, to retrieve this ID you just use the following snip-it of code;
```
import Android.Provider.Settings.System;
...
String Android_ID = System.getString(this.getContentResolver(), System.ANDROID_ID);
```
Also, note that in an emulator this will return "null", though a real device will return an actual value. The nice thing about this tid-bit of code is that you are not required any special permissions to call it, since it's essentially a passive call to get information. No write access is (obviously) required.