---
title: How to manually access "restricted" market areas
tags:
  - android
  - g1
  - gsm.operator.alpha
  - gsm.operator.iso-country
  - gsm.operator.numeric
  - gsm.sim.operator.alpha
  - gsm.sim.operator.iso-country
  - gsm.sim.operator.numeric
  - purchasing apps
  - restricted buying
  - reverse engineering
  - t-mobile
url: how-to-manually-access-restricted-market-areas.html
id: 260
categories:
  - android
date: 2009-05-05 11:24:50
---

![Mmmmm... Market Data...](http://173.230.150.16/blog/wp-content/uploads/2009/01/android_market_combo_8282008_wide_600x297-300x148.png "Mmmmm... Market Data...")
I've noticed some people have been having trouble with Andrea's program, the [Market-Enabler](http://strazzere.com/blog/?p=255) - so prior to him launching his GPL'ed code for the program. I thought that I would explain what it is actually doing.
While I'm not 100% sure, I believe that the problems people are having are with the super-user whitelist application. So running these commands yourself via adb or the terminal on the phone should resolve any of the issues. Also, an obvious thing should be that, you DO need root! This method has been testing on 1.43 JF, 1.5 JF and theDUDE and has worked fine on all of them.

**What to do**
Simply fire up adb or the terminal on your phone, and type in `getprop`. This will return a large list of variables - you should either make a back up of these or write them down. Simply typing `getprop > prop.backup` will make a back up of these values and pipe them to the file named `prop.backup`. This will be needed incase you mess anything up. To enable seeing of paid applications that are available to the US, the following commands must be run:
```
su
setprop	gsm.sim.operator.numeric	31026
setprop	gsm.operator.numeric	31026
setprop	gsm.sim.operator.iso-country	us
setprop	gsm.operator.iso-country	us
setprop	gsm.operator.alpha	T-Mobile
setprop	gsm.sim.operator.alpha	T-Mobile
kill $(ps | grep vending | tr -s ‘ ‘ | cut -d ‘ ‘ -f2)
rm -rf /data/data/com.android.vending/cache/*
```
Essentially you are setting the US standard values (for the T-Mobile Operator) on your phone, these values are the `gsm.sim.operator.numeric`, `gsm.operator.numeric`, `gsm.sim.operator.iso-country`, `gsm.operator.alpha` and `gsm.sim.operator.alpha`.

The next line simply looks for the vending application, and kills the process. The next time will remove the cache files you have stored from the vending application. These two lines essentially make sure your phone will regrab the values you have set, so you can properly view the market data.

**What about other markets?**
I've gotten quiet a few comments and questions regarding this. The same method used to set your phone to the US T-Mobile market could be used to set your phone to any other market. All you need to know is the values from that area. So get your friend to give you the values for their area and carrier and simply replace the values above with it.
**Again... Yet another note**
I did not make the program Market-Enabler, I only helped Andrea and have also translated his posted. Please when using this information quote your sources, both my site and Andrea's Androidiani.com - it's only fair to do so!

Enjoy :)