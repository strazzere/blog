---
title: Disable SSL Certificate Checking for Android Market/Vending.apk
tags:
  - android
  - AndroidOS
  - certification check
  - disable ssl
  - disable_ssl_cert_check
  - g1
  - market place
  - mytouch
  - reverseengineering
  - ro.secure
  - sapphire
  - ssl cert
  - ssl certificate
  - tmobile
  - vending.apk
  - vending.disable_ssl_cert_check
url: disable-ssl-certificate-checking-for-android-marketvendingapk.html
id: 298
categories:
  - android
date: 2009-10-05 16:26:09
---

![Changing the properties..](http://173.230.150.16/blog/wp-content/uploads/2009/10/setgetprop-300x135.png "Changing the properties..")
_**Note:** This might be well, a little low level for some people to understand - though I'm sure this will end up finding the right people. If you understand this post - then you'll understand the significance of what the purposes of doing this would be for_ :)

Just randomly looking to do specific things with the Market, I finally figured out a way to force the Market into accepting any certificate. It turns out there is actually a little bit of code (leftover from testing?) that if special parameters are set, will allow you to disable ssl certificate checking. This essentially allows us to well, do lots of things we otherwise wouldn't be able to do!

Obviously your going to need root to enable this - and I'm going to provide you the way I've enabled it; using adb. Again like I've stated countless times, anything you do via ADB you should be able to do via a console on the device, I'm not sure why people always repaste everything and say just do it via a console thinking it's some big news... The two variables we're going to have to set are `ro.secure` and `vending.disable_ssl_cert_check`. Personally, my devices both have `ro.secure` already set (properly) to 0. `vending.disable_ssl_cert_check` is a new variable that we will be creating, setting it to "TRUE". Simply fire up adb and run the following commands (as root);
```
setprop ro.secure 0
setprop vending.disable_ssl_cert_check TRUE
```
You can quickly run a `getprop` to verify you typed everything correctly; `ro.secure` should be located as the first variable and the vending one is the last (since it is new). You can verify that these settings are correct and working by watching your console log for the Vending.apk via DDMS. The following string should appear while loading the Market: "Turning off SSL certification check." This can be seen below:
!["Turning off SSL Check"](http://173.230.150.16/blog/wp-content/uploads/2009/10/turnoffsslcheck1.png "Turning off SSL Check")
Also as a final reminder, every reboot these variables must be reset, since there is no program actually setting them already. You must reinitialize these variables (in my case, I only have to initialize `vending.disable_ssl_cert_check`) if you would like to use this mode.

Hopefully this will be of use to someone other than myself!