---
title: 'Android Zitmo Analysis: Now you see me, now you don''t'
tags:
  - android
  - DONT_KILL_APP
  - malware
  - reverse engineering
  - setComponentEnabledSetting
  - zeus
  - zitmo
url: javascript-malware-cross-contamination-in-android-apks.html
id: 642
categories:
  - android
  - reverse engineering
date: 2012-08-13 22:46:09
---

Early last week, Denis from Kaspersky had [blogged](https://www.securelist.com/en/blog/208193760/New_ZitMo_for_Android_and_Blackberry) about a new Zitmo (Zeus in the Mobile) variant which was affecting both Android and Blackberry. After some digging I was finally able to turn up a sample for analysis. Originally I figured it would be the same old sample as before, wouldn't do much and not be very sophisticated at all. Turns out I was half right, most of it was identical to previous samples - though I did learn new little trick from the malware this time around.

After I did a tear down I submitted a sample to Mila over at the [Contagio Mobile Dump](http://contagiominidump.blogspot.com), so if you'd like to follow along the sample I was dissecting you can find it [here](http://contagiominidump.blogspot.com/2012/08/new-zitmo-for-android-and-blackberry.html).
```
Filename: zitmo.apk
SHA256: 40286c6091c5a2d575702b1d88eaa94aa8eba524
MD5: e98791dffcc0a8579ae875149e3c8e5e
```

Firing up our trusty keytool, we can see what the certificate is - try to get an estimated date when this file was created;
```
champagne:zitmo-sample tstrazzere$ printcert contents/META-INF/*.RSA
Owner: CN=Android Debug, O=Android, C=US
Issuer: CN=Android Debug, O=Android, C=US
Serial number: 5007c89d
Valid from: Thu Jul 19 01:43:09 PDT 2012 until: Sat Jul 12 01:43:09 PDT 2042
Certificate fingerprints:
	 MD5:  A5:D9:08:26:EC:E0:74:4F:E8:89:35:53:F0:6E:28:55
	 SHA1: B3:23:BD:C7:A3:24:9F:C1:8E:85:44:A8:ED:09:2C:3A:41:3D:23:80
	 Signature algorithm name: SHA1withRSA
	 Version: 3
champagne:zitmo-sample tstrazzere$ ls -l contents/META-INF/*
-rw-r--r--@ 1 tstrazzere 776B Jul 24 07:12 contents/META-INF/CERT.RSA
-rw-r--r--@ 1 tstrazzere 1.1K Jul 24 07:12 contents/META-INF/CERT.SF
-rw-r--r--@ 1 tstrazzere 1.0K Jul 24 07:12 contents/META-INF/MANIFEST.MF
```


So, assuming that the malware author hasn't been messing around with timestamps, we can assume that the malware was created before or on July 24 of this year - and signed on that day as well. As a sanity check, the Android debug certificate validates this, since it was created before this date. This is in no way concrete that it was actually build/signed on that day - but it does give us a decent idea of the timeframe.

The next piece of interest lays within the AndroidManifest.xml - let's see how the author intends this application to be started;

```
<?xml version="1.0" encoding="utf-8"?>
<manifest
 xmlns:android="http://schemas.android.com/apk/res/android"
 android:versionCode="1"
 android:versionName="1.0"
 android:installLocation="1"
 package="com.security.service">
 <uses-permission
  android:name="android.permission.RECEIVE_SMS">
  </uses-permission>
 <uses-permission
  android:name="android.permission.SEND_SMS">
  </uses-permission>
 <uses-sdk
  android:minSdkVersion="4"
  android:targetSdkVersion="16">
  </uses-sdk>
  <application
   android:theme="@7F060000"
   android:label="@7F050000"
   android:icon="@7F020001"
   android:debuggable="true">
  <activity
   android:theme="@7F060001"
   android:name="com.security.service.MainActivity">
    <intent-filter>
     <category
      android:name="android.intent.category.LAUNCHER">
     </category>
     <action
      android:name="android.intent.action.MAIN">
     </action>
    </intent-filter>
   </activity>
   <receiver
    android:name="com.security.service.receiver.ActionReceiver"
    android:permission="android.permission.RECEIVE\_BOOT\_COMPLETED"
    android:enabled="true">
    <intent-filter>
     <category
      android:name="android.intent.category.HOME">
     </category>
     <action
      android:name="android.intent.action.BOOT_COMPLETED">
     </action>
     <action
      android:name="android.intent.action.USER_PRESENT">
     </action>
    </intent-filter>
   </receiver>
   <receiver
    android:name="com.security.service.receiver.SmsReceiver"
    android:enabled="true">
    <intent-filter
     android:priority="2147483647">
    <action
      android:name="android.provider.Telephony.SMS_RECEIVED">
     </action>
    </intent-filter>
   </receiver>
   <receiver
    android:name="com.security.service.receiver.RebootReceiver"
    android:permission="android.permission.REBOOT"
    android:enabled="true">
     <intent-filter>
     <action
      android:name="android.intent.action.REBOOT">
     </action>
     <action
       android:name="android.intent.action.ACTION_SHUTDOWN">
     </action>
    </intent-filter>
   </receiver>
  </application>
</manifest>
```

According the `AndroidManifest` above we can see the application has four main entry points;

 * `MainActivity` - launched as the main activity, which should be visible by the launcher/tray
 * `ActionReceiver` - launched when the `HOME`, `BOOT_COMPLETED` or `USER_PRESENT` intent is fired by the system
 * `SmsReceiver` - launched when the `SMS_RECEIVED` intent is fired, with a priority of `Integer.MAX_VALUE`
 * `RebootReceiver` - launched when the `REBOOT` or `ACTION_SHUTDOWN` intent is fired

This seemed like a bit to many receivers right off the bat, it seems odd that there is a reboot receiver - though maybe the author was trying to more than just intercept SMS messages? Let's first dig into the code and see how the malware is looking to get started. Loading up IDA Pro and a terminal with baksmali, there is something we can notice right off away, the authors where using the latest Android SDK - there is a ton of support code in the `android/support` directory used for backwards compatibility. It's actually a bit funny as there is more code in those files than the malware itself!

Looking at the `MainActivity` default create method, we can see the only code which a user might actually see when the malware is launched;
![MainActivity.onCreate](http://www.strazzere.com/blog/wp-content/uploads/2012/08/onCreate.png "MainActivity.onCreate")

Essentially the bulk of the of the code is just checking a `PersistenceManager` object, which is just a wrapper for the normal Android `SharedPreference` object. This checks to see if the application has been run or not before. If the application has not been run, it will mark the first run as having occurred - followed by running the `SmsReceiver.sendInintSms()` function. This function, pictured below, gets the default admin number (+46769436094) and sends it the message "INOK".
![SmsReceiver.sendInintSms()](http://www.strazzere.com/blog/wp-content/uploads/2012/08/sendInintSms1.png "SmsReceiver.sendInintSms()")

The next receiver to look at is the `ActionReceiver`. This is also very simple - essentially if the intent is either `BOOT_COMPLETED` or `USER_PRESENT` then it will attempt to kick of the `MainActivity`, which would cause the code above to be launched. This is also gated by the same `isFirstLaunch` function from the `PersistenceManager` class. The cleaned up code for this receiver is below, pretty self explanatory as well;
![ActionReceiver.onReceiver(...)](http://www.strazzere.com/blog/wp-content/uploads/2012/08/ActionReceiver.png "ActionReceiver.onReceiver(...)")

The main functionality of the malware is located where it has been for the last few Zitmo samples, inside the `SmsReceiver`. The `onReceive` method essentially houses a large switch statement which will allow the following commands to be parsed;

 * `on` - replies "ONOK" if sent from admin #, then sets the sms interceptor on
 * `off` - replies "OFOK" if sent from admin #, then sets the sms interceptor off
 * `set admin` - replies "SAOK" if sent from admin #, then sets a new admin # to user

All of the commands above will have their broadcast aborted, so no other application should receive the SMS message. If any other SMS is received other than the commands above the code follows a simple path. It will abort the broadcast aborted and the message sent to the admin number in the format of: "message ${SMS\_BODY}, F: ${SMS\_SENDER}". These basic commands, combined with the a PC infected with Zeus, is enough for the authors to programmatically intercept mTans for what would appear to be German bank users. So what, right? I said there was something interesting about this malware - yet everything described has been old stuff. Well the interesting part for me happened when I looked at the last receiver, which didn't seem like it would be necessary. The malware already has all the code it needs to function as normal, right? Well - it actually is going to attempt to hide itself from the user on a reboot. If we look back at the `AndroidManifest.xml` we can see the activity tag which lets the application appear in the launcher/tray of the device;

Previously, we've seen malware that avoids putting this into the manifest, trying to hide from the user. In this case, the malware does want to be found and hopefully executed by the user. The trick is, they only want this to be visible to the user for one boot. If we take a look at the `RebootReceiver.onReceive` function, we can see a simple code path that checks for the `REBOOT` or `ACTION_SHUTDOWN` intents. If either is caught by their receiver, it will call the `MainActivity.hideIcon()` function. The pseudo code for this function is as follows;

```
public static void hideIcon(Context context) {
    PackageManager packageManager = context.getPackageManager();
    packageManager.setComponentEnabledSetting(
      new ComponentName("com.security.service",
        "com.security.service.MainActivity"),
      COMPONENT_ENABLED_STATE_DISABLED,
      DONT_KILL_APP);
}
```

This code will dynamically remove the receiver component, even though it is still set within the `AndroidManifest.xml`. This means on reboot, the icon will no longer be present inside the launch/tray of the device, but the other receivers should still remain active. While this isn't rocket science - mainly just understand the Android APIs, it's something I've never seen any malware do. Performing a few Google searches, it doesn't seem to be something widely used or deployed in practice. I did perform a few searches though for applications that did this and found a few that used this to be "less annoying" to a user and hide when they where not wanted. This is also recommended for something like an option `BOOT_COMPLETE` intent, such as an option service that a user might not want from an application.

_TL;DR_ - You can dynamically disable you competent in your Android application, effectively "hiding" yourself from a users launcher/tray.