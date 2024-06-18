---
title: PDFViewer (working) on JF 1.5 and other builds
tags:
  - android
  - byte code
  - dex
  - FilePicker.apk
  - g1
  - htc
  - libpdfreader.so
  - patch
  - pdf android
  - pdf viewer
  - PDFViewer.apk
  - reverse engineering
  - sapphire pdf
  - t-mobile
url: pdfviewer-working-on-jf-15-and-other-builds.html
id: 266
categories:
  - android
date: 2009-05-14 13:46:51
---

So a few days ago I got an email concerning the HTC PDF viewer which apparently comes bundled with the HTC Sapphire. Saddly, there has not yet been a release of it for the HTC Dream. The original thread on xda-developers can be found [here](http://forum.xda-developers.com/showthread.php?t=506595) which essentially was what the person was directing me too. The problem with this apk seemed to be that it was "locked" to HTC only devices... But - the _HTC_ Dream is an HTC device, right? Not according to this program...
![What? HTC Dream IS HTC?!](http://173.230.150.16/blog/wp-content/uploads/2009/05/protected-200x300.png "What? HTC Dream IS HTC?!")
Anyway - long story short, success! I've successfully patched the file so that it should be able to be loaded on any HTC Android device. Have a blast reading your pdfs now!
![FTW](http://173.230.150.16/blog/wp-content/uploads/2009/05/ftw-200x300.png)
Required files for this to work;
```
[libpdfreader.so](http://www.strazzere.com/android/libpdfreader.so)
[FilePicker.apk](http://www.strazzere.com/android/FilePicker.apk)
libpdfreader.so must be pushed using adb (or shell) to /system/lib
FilePicker.apk must be pushed using adb (or shell) to /system/app

Note: To push the files to /system, you will need to remount it as rw with the following command:
mount -o rw,remount -t yaffs2 /dev/block/mtdblock3 /system
```

Finally -- download and install (either through adb or your favorite package installer) the patched apk! You can download that here, [PDFViewer.apk](http://www.strazzere.com/android/PDFViewer.apk). This was tested on JF 1.5 and 1.45 and seems to work perfect. Please post your programs if any should arise.

Enjoy! :)