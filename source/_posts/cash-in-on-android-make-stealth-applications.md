---
title: Cash in on Android - make "stealth" applications!
tags:
  - android
  - android money
  - android stealth
  - android.permission
  - g1
  - mobistealth
  - mytouch
  - privacy concerns
  - sarcasm
  - stealth
  - vaporware
url: cash-in-on-android-make-stealth-applications.html
id: 283
categories:
  - android
  - other
  - random
  - secure code
date: 2009-08-27 16:16:11
---

![Make money, money...](http://173.230.150.16/blog/wp-content/uploads/2009/08/android-money.jpg "Easy cash with no work!")
<sarcasm> Ah, so you want to make money fast and do little work, while charging a boat load of money? Well, welcome to the bandwagon! First, you need to throw together a hastily made ~~scam~~ product, something to slurp up all your phones information and let it be viewable from a website... Something that just uses all the android permissions you can wrap your mind around;
```
android.permission.Access_Fine_location
android.permission.Access_Network_State
android.permission.Battery_Stats
android.permission.Camera
android.permission.Read_Calendar
android.permission.Read_Contacts
android.permission.Read_owner_Data
android.permission.Read_Phone_State
android.permission.Read_SMS
android.permission.Receive_MMS
android.permission.Receive_SMS
```
This is just a small list of "useful" things people seem to well, deem "useful" in knowing. Next set up a simple method to dump all this data onto the device and prepare it for transfer. <sarcasm>One would assume you'd encrypt this information and send it securely, though - that might take development time so why bother wasting your resources? </sarcasm> Hardcode values into your product for "securely" connecting to your server and have it dump information off.

Next to make your claim of application being "stealth" be correct, change your manifest from:
```
<category android:name="android.intent.category.LAUNCHER" />
```
To
```
<category android:name="android.intent.category.INFO" />
```
This makes the application not appear on the launcher, also known as the tray. People tend to associate this with "stealth". <sarcasm> Most people know stealth equates to, no icon! Just because it still registers as an application under application management doesn't mean people will find it! </sarcasm>

For your web page and server, simply chose a small host - like the one I use for my blog. Dirt cheap, plenty of space and plenty of bandwidth - it's probably against the TOS to do such a thing, but who cares? Bluehost is only $6.95 a month - if you get **one customer** you could **cover your server costs!**

Next set up a simple web interface that displays this data being dumped onto the servers. That will let you cull the data for your users - what they're going to be paying for of course. Next thing is to spiff up your web site and make it look flashy. Put things like "ONLY $99.99 PER YEAR", because by adding "only" it somehow makes it seem like a deal. Then throw some banners saying "guaranteed" and "uptime certified" without references to what this actually means - it just makes it seem more legit. Obviously you should add some things stating to "protect children" or catch your "cheating spouse" because well, those sound like valid uses to such an application. Try to stay away from words like "over-protective", "spying" or "snooping" as it may make a potential user realize the reasons they might _really_ use this product. Another great thing to add to the website is pictures of phones which potentially will exist or haven't come out yet. Just assume that all Android Software will be the same and all devices will work prior to testing on them, simple say they are supported. By supporting more phones, you look more important and appear to be trustworthy since you've claimed you phone works on Hero models. Most average people don't have a Hero phone, if you have one, well -- you must not be average! Oh, don't forget to write up a quick and easy EULA, saying essentially:

> We're not evil, we don't sell your information, we just use it for you! If you have an issue with the functionality of our program, we'll work to fix it. If we can't fix it, we'll give you a refund. Don't do this if it's illegal. If you do something illegal - then it's your fault, not ours.

While this obviously isn't much of a EULA, you can't say you didn't say so! Besides, this type of "guarantee" is perfect and bulletproof. If there is a bug - then you fix it, if it's simply "I don't like this product", well - sorry? That's not a _problem_ with the _software_, that's a problem with your outlook of our software... Silly customer!

There you go, that's a pretty straight forward tutorial on how to make tons of cash with an everyday program that does little to no work. Simply market this tool to people of ages 16 to 30, and you'll get plenty of people who won't read your "fine print" (all two sentences of it) and you'll cash in! Last but not least, once you grab the money - you haven't guarenteed functionality beyond seven days of people purchase, so take your money, close your server and go to your next ~~scam~~ application</sarcasm>

_Note: I hope people could detect my sarcasm tags..._