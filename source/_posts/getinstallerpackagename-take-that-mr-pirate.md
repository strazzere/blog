---
title: 'getInstallerPackageName, take that Mr. Pirate!'
tags:
  - /data/system/packages.xml
  - android
  - anti-piracy
  - com.google.android.feedback
  - DRM
  - getInstallerPackageName
  - google
  - lvl drm
  - packagemanager
  - reverse engineering
url: getinstallerpackagename-take-that-mr-pirate.html
id: 374
categories:
  - android
date: 2010-09-13 23:59:44
---

I've been seeing posts lately in the wake of the LVL anti-piracy code being [broken](http://www.androidpolice.com/2010/08/23/exclusive-report-googles-android-market-license-verification-easily-circumvented-will-not-stop-pirates/), with different methods on how to keep your Android package safe. Anti-piracy measures have always been fascinating to me, so I love diving into them and taking a look at what developers think are good and why. One of the latest crazes is using the [PackageManager](http://developer.android.com/reference/android/content/pm/PackageManager.html)'s "[getInstallerPackageName](http://developer.android.com/reference/android/content/pm/PackageManager.html#getInstallerPackageName(java.lang.String))" to verify whether or not the Market actually installed the apk file or not.

There are a few issues with this, though they are only minor issues depending on how you look at them;

 * API call is a of the API Level 5, meaning it won't reach all android users (Android 2.0 and up) - and "may not always work"
 * It's only a viable method if protecting for the Android Market
 * It's simply just something else for a pirate to patch, nothing too hard
 * This prevents legitimate people from backing up the application, installing from without use of the Market, adb, etc.

It's pretty self explanatory for the API level issue. If you want to target people below Android 2.0, then your can't use this method to verify the installer source. It's not going to be available, which may or may not be an issue depending on what other API's you are targeting. On top of this though, it apparently doesn't always work;

> In a late edit, we removed a suggestion that you use a check that relies on GetInstallerPackageName when our of our senior engineers pointed out that this is undocumented, unsupported, and only happens to work by accident. –Tim ([Securing Android LVL Applications - Android-Developers/Blogspot](http://android-developers.blogspot.com/2010/09/securing-android-lvl-applications.html)

Another issue that arrises from this method, is that it's actually only set when being installed by the Android Market. If an application is not installed via the Market, attempting to retrieve this value will just return `null`. That's ok right? Well, if you'd like to lock out customers who want to use this on multiple devices, or keep older versions of your application - then no it's not. While this could be a small base of users, people may be purchasing your application and attempting to run it on a second Android device that doesn't have access to the Market. Or possibly just using Astro File Manager to keep separate versions of your application - these would all be broken if you use this method, as once installing from anything but the Market happens, it will not return the correct value.

Ok, well if you don't care about the rest of the cases so far and are willing to risk it, why bother? It may seem like your locking out a few legitimate users, but at this point you'll probably lock out more legitimate users than you will pirates. Using any of the numerous tools out there, or even the ones bundling within the Android SDK can find you the function call to getInstallerPackageName, and quickly circumvent the check to see if what was returned was `null` or `com.google.android.feedback`.

So how can legitimate rooted users get around this annoyance? Well, I wouldn't recommend it, but an easy way to get around this is simply editing your /data/system/packages.xml - inside this you will see the information for all the packages you have on your system. The actual important part you want to look at is the package header tag;
```
<package name="com.package.example.name" codePath="/data/app/com.package.example.name.apk" flags="0" ts="1283990162000" version="25" userId="10066" **installer="com.google.android.feedback"**>
```
The tag in bold is only apparent when it is installed via the Market. If you _don't_ have this in something that needs it... You could add it. If you want to change it to turn a different value, well here is where you can add it. Oh - and obviously this would require root. Edit: After looking a little bit deeper into this, I remember the trusty old "pm" command - which actually works around almost all these issues. By doing the following simple command via ADB you can set the installer to anything;
```
pm install -i installername com.example.package
```
This will perform an install setting the "installer" to whatever you've chosen it to be. No reboot required, or root privileges needed - just access via ADB.