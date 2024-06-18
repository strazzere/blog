---
title: ADC2 and Downloading Apps
tags:
  - ADC2
  - android
  - android os
  - apk
  - assetId
  - download
  - GET apk
  - saving adc2 apps
  - without adc2
  - without vending
url: adc2-and-downloading-apps.html
id: 295
categories:
  - android
  - other
  - reverse engineering
  - updating
date: 2009-09-29 20:08:12
---

![Finding the assetId...](http://173.230.150.16/blog/wp-content/uploads/2009/09/printscreen-300x252.png "Finding the assetId..")
So recently I've been having quiet an experience with downloading apk files from the android market and through the ADC2 review application. The ADC2 application actually just uses the same downloader that the market does -- so that's a quick reason as to _why_ so many other people are also having this trouble.

Anyway the fix that I have been using is the [downloading snippet](http://strazzere.com/blog/?p=293) I posted previously in conjunction with the [authtoken snippet](http://strazzere.com/blog/?p=291).

The one thing I needed to find was the `assetId`, which conviently was located in an xml file of the currently being reviewed application. This makes it so you don't have to fire up wireshark or tcpdump to grab the `assetId`.

Simple load up adb or the terminal on your phone and navigate to, `/data/data/com.google.android.challenge/shared_prefs` and view the file `com.google.android.challenge.xml`, the contents will look something like this:
```
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
<int name="adc.status" value="1" />
<boolean name="accepted.tos" value="true" />
<string name="current.developer.name">United Swe</string>
<string name="current.size">430k</string>
<string name="current.description">Send money to a friend with just their phone number. Get money too. Or create an IOU. Same as cash but with phone numbers and phone messages. Easy to track. Convenient. And rewarding too! Test mode allows working with &quot;play money&quot; to try out all the features. Even works with legacy text messaging phones. Feedback?</string>
<string name="install.target.packagename">com.fonepays</string>
<string name="current.package.name">com.fonepays</string>
<string name="current.asset.id">-7878887740798669019</string>
<long name="last.nag.time" value="1254261597127" />
<string name="current.permissions">android.permission.ACCESS_WIFI_STATE|android.permission.ACCESS_NETWORK_STATE|android.permission.CHANGE_NETWORK_STATE|android.permission.INTERNET|android.permission.STATUS_BAR|android.permission.VIBRATE|android.permission.READ_CONTACTS|android.permission.READ_PHONE_STATE|android.permission.RECEIVE_SMS|android.permission.SEND_SMS|android.permission.TELEPHONY_SERVICE</string>
<string name="current.label">FonaPays - The Easy Way to Pay</string>
</map>
```
We can easily see above that the string value of `current.asset.id` is the `assetId` we need. This will let you easily download the apk files to your computer and install them to your phone in no time :) Hopefully this will help some people who are having the same trouble I was!

Also, just a little note - this will let you obviously save the apk files opposed to not retaining them after submitting a review. This could also be accomplished by doing the old method of a pull from `/data/app` used for backing up normal applications - since the ADC2 apps are saved to the same location. Enjoy!