---
title: PhotoVault disappoints for protection
tags:
  - /data/data/com.android.PhotoVault
  - android
  - apps
  - buyer beware
  - dream
  - g1
  - LokPix
  - market
  - Pacific Software Solutions
  - password
  - PhotoVault
  - PhotoVaultPassword.xml
  - t-mobile
url: photovault-disappointments-for-protection.html
id: 239
categories:
  - android
date: 2009-04-21 14:56:37
---

![Enter Pin screen](http://173.230.150.16/blog/wp-content/uploads/2009/04/enter-pin-200x300.png "enter-pin")
A new-ish application was released a few days ago by [Pacific Software Solutions](http://pacificsoftwaresolutions.net/), (at the time of writing this, there is no website, just a coming soon banner) called PhotoVault. The application seems to perform rather well, and seems to also meet a need many users have been asking for - hiding photos from people who just pick up your phone. According to the comments provided on the application, some people seem to be having it force close many times, seems like an "Application Not Responding" force close, so that should be easily fixed. Personally I wasn't experiencing any of these errors.
![Public Media for all to see](http://173.230.150.16/blog/wp-content/uploads/2009/04/media-200x300.png "media")
Though sadly the protection method is uses is pretty weak. I figured it would be something atleast encrypted with a static password within the program itself. Worse than what I had first assumed, it's simple stored as a plain text value in a file, more specifically, `/data/data/com.android.PhotoVault/shared_prefs/PhotoVaultPassword.xml`. Not even that secret with the file name, where they?
![Ops? I found the password...](http://173.230.150.16/blog/wp-content/uploads/2009/04/password.png "password")
So this program isn't exactly government grade-security, but it should stop nosy family members or other people from snooping around at your pictures. Granted, if you really don't want people to see those pictures, maybe you shouldn't have taken them in the first place? Just figured I'd share this with everyone else, hopefully no one gets burned by any false assumption that this program is _secure_ and will _protect_ your files from someone who really wants to get at them. If you want more protection, there was an application I quickly looked at before named "LokPix" which seemed much better than this one.