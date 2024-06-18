---
title: Android Market DRM busted < 12 hrs!
tags:
  - /data/app
  - /data/app-private
  - android
  - android DRM
  - app-private
  - DRM
  - drm broken
  - g1
  - google
  - market
  - market DRM
  - protected applications
  - reverse engineering
  - root
  - SlideLOCK
url: android-market-drm-busted-12-hrs.html
id: 185
categories:
  - android
date: 2009-02-20 14:50:29
---

Well paid applications have officially rolled out, or rather - reached my phone through the android market. The first thing I wanted to find out, is what is the protection scheme used?! My first assumption is that it might be something tied to your account, or android id - something sort of like SlideLOCK which I previously mentioned. So after looking around the market I downloaded a few applications and I also decided to download some "protected" free applications. Mainly I wanted free applications to toy with, so I didn't feel bad possibly violating paid ones :)

The programs I've started with where _Inaugural Address 2009_ and _Phandroid News_. Both are "protected" applications. What does this mean exactly? After a little digging it was clear exactly what "protected" actually meant;
![Location, location, location!](http://173.230.150.16/blog/wp-content/uploads/2009/02/location-300x229.jpg "Location, location, location!")
So first I checked as you can tell, the SD card. Originally I was checking posts at xda-devs and someone posted that they where located on the SD card and where .zip files. This is _false_, the packages are .apk and located in the `/data/app-private` folder as you can see from the screen shot above. Well, what does this mean exactly? This means a non protected app, which is stored in `/data/app` can be pulled using `adb`, while a protected app is located in `/data/app-private` is not allowed to be pulled by `adb`. Though... with a rooted phone - this isn't exactly a problem since it can pull any file!  

Alright, so what? Isn't there some more protection involved in this?!  

Sadly, that's what I (wrongly) assumed too. What I assumed was that there would be some more protection other than just things being "protected" in a folder. Possibly, it being linked to the android id? Maybe it not being able to be installed on another device or through a "non-market" source. Or maybe the market would track it, since it does for updates on market installed applications... Well, sadly, no - no other protection seems to be present.  
All "protected" applications can be dexdumped like a normal one, can be pulled (as long as you have a rooted phone), then reinstalled. The even worse part, is that protectd applications don't stay in the protected folder when installed via a web download or `adb` install. They simply get pushed into the normal `/data/app` folder... Some protection huh? Oh, and my first assumption that the market might look for these also appears to be false. The market does not check any applications installed from an "unknown source" - so that idea for protection is a complete bust.  

```
C:\Documents and Settings\tstrazze\Desktop\eclipse\android-sdk-windows-1.0_r1\t
ols>adb install phandroidnews.apk
374 KB/s (0 bytes in 1286982.003s)
pkg: /data/local/tmp/phandroidnews.apk
Success

C:\Documents and Settings\tstrazze\Desktop\eclipse\android-sdk-windows-1.0_r1\t
ols>adb shell
# cd data
cd data
# cd app-private
cd app-private
# ls
ls
# cd ..
cd ..
# cd app
cd app
# ls
ls
GWiFi.apk
com.a0soft.gphone.aClockPro.apk
com.aevumobscurum.androidlite.apk
com.android.ussdbal.apk
com.angryredplanet.android.rings_extended.apk
com.ap.SnapPhoto.apk
com.appdroid.anycut.apk
com.avaw.luklukpro.apk
com.biggu.shopsavvy.apk
com.blau.android.phandroidnews.apk
com.bonfiremedia.android_ebay.apk
com.brilaps.android.txtract.apk
com.google.android.apps.uploader.apk
com.google.code.p.mariosimulator.apk
com.infonow.bofa.apk
com.larvalabs.retrodefencelite.apk
com.lindaandny.lindamanager.apk
com.memoryup.apk
com.mlogiciels.a.pipe.apk
com.nextmobileweb.wikipedia.apk
com.nubinews.fullreader.apk
com.phonalyzr.apk
com.r2.MemCuad.apk
com.roundhill.androidWP.apk
com.shazam.android.apk
com.snctln.game.WordWrench.apk
com.taskManager.rootTaskManager.apk
net.sunflat.android.papijump.apk
net.sunflat.android.papipole.apk
net.sunflat.android.papiriver.apk
org.connectbot.apk
org.openintents.pocketplay.apk
org.ouroborus.android.downloadcrutchlite.apk
ru.orangesoftware.areminder.apk
src.com.poidio.terminal.apk
#
```

  
What sort of makes this whole ironic deal even odder, is the refund policy. Within twenty-four hours of a download, you are allowed to uninstall and get a full refund of the product you purchased. So you could theoretically just rip all the applications you want, not that I am suggesting this at all.

Want to try this process? Well I've uploaded [inaugural address](http://www.strazzere.com/android/address.usa2009.apk) to my site for testing. This is the protected application that is _free_ on the market. As you notice it will install to the `/data/app` directory, _not_ the `/data/app-private` directory. I've tested this process with multiple application on my rooted G1, and a friend also verified that installed "protected" applications through an unknown source also works on thier stock G1 (non-rooted). (If the author wants me to remove this, I will gladly do so - I just wanted an example and figured a free application/free service would not mind the attention)  

So what are developers to do? Well, recently I bashed on the SlideLOCK in a previous post... Though - the more I looked at it, this seems like the best option for developers to protect their applications. Though, hopefully google will remedy this solution and fast. Until then though, strike-one google market.  

_Edit: The author of phandroid news asked me to remove his application, so I have. In it's place I've uploaded the inaugural address 2009 application unless that author would like me to remove it._

_**Edit2:**_ To clarify, I've tested this with a free PROTECTED application. This is NOT the same as a free non-protected application, at phandroid he shows what the step looks like to have a "protected" application opposed to a "non-protected" one. Also the protected applications appear to be identical to the paid applications, which are also protected.