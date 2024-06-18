---
title: Spoofing your Android_ID
tags:
  - /data/data/com.android.providers.settings/databases/settings.db
  - android
  - Android_ID
  - dead00beef
  - g1
  - google
  - spoof
  - spoof android_id
  - spoof id
  - sqlite
  - t-mobile
url: spoofing-your-android-id.html
id: 217
categories:
  - android
date: 2009-03-20 12:57:50
---

With the (hopefully) eminent release of the cupcake update for the android os – this is supposed to be fixed with the addition of `Settings.Secure`; though I guess we won’t know until it gets released?

Anyway — basically you can modify system values in a specific database, so any program that calls Settings.`System.Android_ID` can be given any id of your choosing.

Just load up adb and navigate to the sqlite database: `/data/data/com.android.providers.settings/databases/settings.db`

Run the query;
```
select * from system;
```
You’ll see plenty of information which can then be changed. Running the following;
```
update system set value=’dead00beef’ where name=’android_id’;
```
Would give you an ultra-slick `android_id` of `dead00beef` 🙂

Note that this will NOT change the android id used by google products since they use one that is linked to your gmail account that the phone is associated with...