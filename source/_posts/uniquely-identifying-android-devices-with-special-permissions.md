---
title: Uniquely Identifying Android Devices with special permissions
tags:
  - android
  - code
  - g1
  - hardward id
  - identifiers
  - java
  - phone number
  - READ_PHONE_STATE
  - secure code
  - unique
  - unique id
url: uniquely-identifying-android-devices-with-special-permissions.html
id: 116
categories:
  - android
  - coding
date: 2008-11-26 11:29:43
---

Previously I wrote about how to [uniquely identify Android devices without special permissions](http://strazzere.com/blog/?p=113). However, maybe you want to get into the nitty-gritty and get an even more unique identifier for the device. This can be done, but you need special permissions. Essentially what I mean by "special permissions" is that the user will be prompted when installing that "this application tries to do this". "This" referring to (in this specific case) _Reading Phone State_. This doesn't mean it's doing anything bad, however users might be turned off if your calculator wants to read the phones state etc. This is just how Google has set up the installation of applications, so that a user is properly notified of what an application is given permission to do.

What are the kind of identifiers we can get with this special permission? We can grade the "Device ID" which is the IMEI number, the phone number, Software Version (not sure if it's currently being used?), Sim Serial Number and Subscriber ID.

That should be enough unique identifiers for _anyone_ to come up with some hash! Heck, phone number alone should be enough since it would be readily known by a customer and easy to use.

To get these values add the following somewhere in your program;
```
import android.telephony.*;

...

		TelephonyManager mTelephonyMgr =
        	(TelephonyManager)getSystemService(TELEPHONY_SERVICE);


		String imei = mTelephonyMgr.getDeviceId(); // Requires READ_PHONE_STATE
        String phoneNumber=mTelephonyMgr.getLine1Number(); // Requires READ_PHONE_STATE
        String softwareVer = mTelephonyMgr.getDeviceSoftwareVersion(); // Requires READ_PHONE_STATE
        String simSerial = mTelephonyMgr.getSimSerialNumber(); // Requires READ_PHONE_STATE
        String subscriberId = mTelephonyMgr.getSubscriberId(); // Requires READ_PHONE_STATE
```

Note that you MUST add permission access to `android.permission.READ_PHONE_STATE` otherwise your program will force close. This is added in the `AndroidManifest.xml` like the following;
```
<uses-permission android:name="android.permission.READ_PHONE_STATE"></uses-permission>
```

On the emulator it will output things like the following;
```
DeviceId(IMEI) = 000000000000000
DeviceSoftwareVersion = null
Line1Number = 15555218135
SimSerialNumber = 89014103211118510720
SubscriberId(IMSI) = 310995000000000
```
One should also note that a real device currently returns "00" for Device Software Version, so it's possible that this is something reserved for future use. These could all easily be used in some type of registration algorithm that you want to tie to a certain device. Also if you choose your identifiers properly you could allow your registration code to be complaint across all versions of your product. Using a phone number for example could allow a user to use your application on any phone they put their SIM card into. If you want to prevent this you could tie it to both the device ID and the phone number.