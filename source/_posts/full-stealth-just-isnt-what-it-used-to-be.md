---
title: '"Full Stealth" just isn''t what it used to be!'
tags:
  - android
  - android spy
  - android stealth
  - com.retinax.android
  - mobile spy
  - mobile spy android
  - retinax
  - security
  - spy
  - stealth
url: full-stealth-just-isnt-what-it-used-to-be.html
id: 335
categories:
  - android
date: 2010-05-03 20:10:04
---

Taking a gander at one of my favorite android web sites today, I stumbled across an interesting application with an even more interesting claim. The article I'm referencing is located at [AndroidandMe.com](http://androidandme.com/2010/05/news/mobile-spy-delivers-an-app-for-those-that-like-to-watch/), the line that really caught my eye was as follows;

> The software is loaded onto the Android device via an .apk install and Retina-X assures subscribers that it is a “full stealth install” and that once installed it cannot be detected by the user.

I wonder how this would even be done? After a quick search of their site is doesn't look like there is a trial version available - though oddly, they do give you links to download the application... If you look close enough. Listed on their [user guide](http://www.mobile-spy.com/android-inst.html) they give you a run down on how exactly you install the application. I must say, for an application people must pay $99 a year for, it does not seem exceptionally user-friendly. Essentially they use a combination of _Download Crutch Lite_ and _apkInstaller_ to allow you to "easily" install their apk file. Once you've done this, you've now erased all tracks of this application, right? Well - not really, you just need to know where to look now.

Ok so we've installed the apk now, how is this thing hidden? Open up the app-draw, not there... Ok, well that would have been too easy - so I guess I'm glad it wasn't there. Now lets goto `Settings > Applications > Manage applications`. Hmmm, everything looks ok - oh, wait - no it doesn't. Looks like someone added an application called "SmartPhone", conveniently with a default icon too. This is pictured below.
![Where did this thing come from?](http://173.230.150.16/blog/wp-content/uploads/2010/05/combined-300x266.png "Hey, I didn't install that!")
Alright, well - the display name could always be changed here and what if that happens? How could we detect this application? Can we do it programmatically? Of course we could, in fact it's incredibly easy too. Since we know what that applications must retain the same package name to maintain itself with updates - can just programmatically check for this.
![Sadly this image is needed for a wordpress bug...](http://173.230.150.16/blog/wp-content/uploads/2010/05/code.png "Sadly this image is needed for a wordpress bug...")
Ta-da! We've successfully disproved another "stealth" application myth. I've also included the three lines of code needed to start the intent for uninstalling this stealth little gem of an application...
![I didn't install you, but I *will* uninstall you!](http://173.230.150.16/blog/wp-content/uploads/2010/05/remove-168x300.png "I didn't install you, but I *will* uninstall you!")