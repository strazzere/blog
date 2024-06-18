---
title: Purchase applications outside of the "authorized" zones
tags:
  - andrea
  - android
  - androidiani
  - g1
  - italy
  - market
  - market enabler
  - paid apps
  - purchasing apps
  - restrict
  - reverse engineering
  - spoof
  - t-mobile
  - Vending
  - vodafone
url: purchase-applications-outside-of-the-authorized-zones.html
id: 255
categories:
  - android
date: 2009-05-03 11:03:00
---

![Paid apps on an italian G1](http://173.230.150.16/blog/wp-content/uploads/2009/05/android-market-paid-apps-italy-200x300.png "Paid apps on an italian G1")
First let me just make a little disclaimer that this is my own translation from Italian to English and I used google-translate a little bit also. So try to hang in there with me since my Italian isn't very good, and I've tried to rephrase things to make a little more sense in English. For the original article (Italian) go to [androidiani.com](http://www.androidiani.com/applicazioni/applicazioni-a-pagamento-anche-in-italia-con-marketenabler-2177) and for the translated (via google-translate) go here [translated androidiani.com](http://translate.google.com/translate?prev=hp&hl=en&js=n&u=http%3A%2F%2Fwww.androidiani.com%2Fapplicazioni%2Fapplicazioni-a-pagamento-anche-in-italia-con-marketenabler-2177&sl=auto&tl=en).

As announced previously, today will be the launch day of the program we have called "Android Enabler". This is a program that should help people across the world (and in the US) who have been restricted thus far with the inability to purchase applications on the Android Market.

If you have previously read the blog (and if you do not, perhaps it is time you subscribe ;)) you would know that [yesterday](http://strazzere.com/blog/?p=249) we launched a preview of the application and what will be going on. On to the news now though...

**The Builders**
We start by presenting Tim Strazzere, an American friend (in fact I recently discovered he has ancestors from Sicily), who write an interesting articles on reversing applications and control systems from the Android OS. [(Link to blog)](http://strazzere.com/blog/)
He has written a few "bomb shell" articles that have been about reversing the market. After contacting him I discovered that he was working on a [scraper](http://en.wikipedia.org/wiki/Scraper_site) of the market.
Then we came to meet on gchat where we talked about the possibilities and how we could accomplish such a task of purchasing outside of the US. I provided the out-of-the-country phone and knowledge, and provided the reversing knowledge. ;)
The other coder of this application is myself, Andrea Baccega, that was absolutely bored to ask his friend over the ocean to purchase applications, reimburse his fee, and then get the application to be able to review it for my site. _ (Tim: I was getting bored of that myself!)_

**The method**
Tim and I quickly set to work to find ways to enable the market for pay applications outside of the US (which at the time was the only viable market, with UK soon to follow). First I will explain the methods we tried and the solutions we used, this will hopefully let you understand the process we used for which the final solution came about.
To be able to play around with the data the market would send to the phone, we needed to understand how the data was exchanged with google servers. It was not enough to simple snag the packages by a simple [mitm (man in the middle) attack](http://en.wikipedia.org/wiki/Man-in-the-middle_attack). As one might expect, the data being exchanged was not always clear...
At this point, if we wanted to understand what we where sending to google, we had to reverse the market and understand exactly what function is employed to encode and decode the data being sent and received. More importantly though, we had to understand what was going on within these two methods.
Surprisingly, we found that the method of encoding and decoding that are used are extensively documents and were Baser64.decodeWebSafe and Base64encodeWebSafe. Both part of the google data API. Though the values that the android where producing where often a _little_ nerfed and not out-of-the-box uses of these functions.
Once we decoded and sniffed a good number of packages from both the US and Italy we knew what information apparently was going to be required to change:

* **Name** of the operator (which in Italy would often be IT and Vodafone, in the US would often be T-Mobile)
* **Operator number** (which in Italy was 22210 though it needed to be 31026) _This is a number made up of the MCC + MNC_

According to our theory, simply by changing every occurrence of these values, we should be able to access the paid applications on the market from anywhere.
After a quick brainstorming session we figured the following routes would be the best solutions:

1. Intercept the outgoing packet and change on the fly (too costly, too time consuming and would probably require writing a driver or massive patch)
2. Change the core android functions that manages requests and posts to the internet.
3. Change the core android functions that return the operator name and operator number.

**Framework.jar**
At fire glance, I immediately worked to rebuild the entire Android OS and create a patch that would manage and restore the values from Italian to the US values.
Although this process had turned out to be a long and painful process, which I'll spare you the details of. Essentially I was recompiling the [ServiceState](http://developer.android.com/reference/android/telephony/ServiceState.html) class. (it should be easy to guess where)
Once recompiled with the changed we crossed our fingers and with some effort, managed to get it to run on my Italian G1 running "thedude" patched firmware.
To our surprise, however, the enormous amount of work done was not enough to make it fully functional. The market seemed to be pulling data from other locations that just the ServiceState class. The data it was pulling turned to be half Italian and Half American, so we started to work on a different solution.

**Setprop**
In the amended version of "thedude" I wanted to make one more final test. For those who do not know android has a list of global variables that are used as real properties.
Some of the properties are written, while others are read-only values. Through a few bash commands, we discovered that the previous patch I had created was not changing all the properties that we needed to change, so these had to be manually changed through the shell.
After several attempts we succeeded ... _We finally saw paid applications!_ ... Now Tim says:

> "Why don't we just see if we can do this through all setprop commands, opposed to even using the patched ServiceState class?"

Done... This method also worked, so it was much easier to do. Next I downloaded the latest version of the android firmware are just released by JesusFreke.
Retried all the setprop commands we had figured out, crossed out fingers - and then reopened the market. Success, it works!
The result of this discovery is that all the idea we had, we basically in vain. They all would have worked - though this was much more simple than we originally thought it would be.
Moreover, the final solution we came to, was not firmware dependent - there is no reason to patch framework.jar so you could use this on any firmware, as long as you have root.
![Market Enabler](http://173.230.150.16/blog/wp-content/uploads/2009/05/market-enabler-200x300.png "Market Enabler")

**Application**
I will just quickly go over the disclaimer for this applications and a quick how to;

_Disclaimer_

* As usual, I am not responsible for the use of this application. This application is to be used as a Proof of Concept only, for enabling paid applications on your phone.
* If using this program to enable paid applications, I am not responsible for anything that happens to your phone or applications that you've purchase should this be fixed at a later date.
* The application will be issued under the GPL shortly, once I can gather up the code and clean it up a little bit.

_**How to Use the program**_
The applications only has two buttons ;) a right and a left one.

The left button enables paid applications on the market The right button restores the normal values (in case some applications have problems after changing the values)

That's it! Doesn't get much easier than that. ;)

_**Note:**_
Each reboot you must restart this application to enable the paid applications on the market.

**Download Market Enabler**
The _Market Enabler_ can be downloaded from [here](http://www.androidiani.com/Firmwares/download.php?id=6).

To install the application you can simply run the command:
```
adb install MarketEnabler.apk
```
Or open the Android Browser to this link: http://tinyurl.com/marketenabler

For problems with the application, refer to our [forum! (Italian)](http://www.androidiani.com/forum/) Or post here for help in English.

**Note:**
To use the market to pay for applications you have to register an account on google checkout. Google Checkout is a secure payment platform similar to paypal. To do this, use your pc to navigate to [this web site](https://checkout.google.com/?saveUserPref=true&hl=it). The instructions will guide you to sign up step by step ;)

**Donations:**
Donations are always welcome! Also I will publish a list of all donors, who have supported our work.
To donate (even $1), click [here](http://www.androidiani.com/supporta-androidianicom). (Click the "Donate" button that says paypal)

**Copyright**
You can copy of take ideas from this program/post and redistribute the post/application (which will soon be released under the GPL) as long as the source of the article and the download link is not changed.