---
title: More spoofing of the android id...
tags:
  - Android_ID
  - androidid
  - com.google.android.googleapps
  - g1
  - imsi
  - reverse engineering
  - spoof
  - spoofing android_id
  - tmobile
url: more-spoofing-of-the-android-id.html
id: 235
categories:
  - android
date: 2009-04-09 10:30:16
---

![Finding the Android ID](http://173.230.150.16/blog/wp-content/uploads/2009/04/android_id.bmp "Finding the Android ID")
[In a previous article](http://strazzere.com/blog/?p=217) I had posted some information about how it is possible to spoof the Android ID that is returned when calling the Settings.System.Android_ID function. Though I had noted at the end of my post the following;
>_Note that this will NOT change the android id used by google products since they use one that is linked to your gmail account that the phone is associated with..._
This bugged me a little bit because I wanted to know how the google applications where using and getting the android information - did they pull it directly from the hardware? Where they just using private API that was more secure? So after a little research, I found exactly what was going on, and again how it would be possible to spoof the id.
Essentially the google applications use googleapps to store the android id, this is the program on the phone named "com.google.android.googleapps". This is a very interesting program that sadly developers do not have access to as of yet, though hopefully this will change shortly.
Anyway, this program is also susceptible to being force-fed spoofed values. The method is essentially the same as the previous one, though just performed on a different database. From within adb or the shell, do the following;
```
$sqlite3 /data/data/com.google.android.googleapps/databases/accounts.db update system set value=’deadbeef0000badf00d’ where name=’androidId’;
```
This program also store in that table the imsi number linked to the phone, for simcard tracking purposes.