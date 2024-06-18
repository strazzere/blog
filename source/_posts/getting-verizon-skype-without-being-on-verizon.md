---
title: Getting "Verizon" Skype... without being on verizon
tags:
  - android
  - market enabler
  - market spoof
  - phone spoof
  - skype
  - tmobile
  - verizon
  - verizon spoof
url: getting-verizon-skype-without-being-on-verizon.html
id: 328
categories:
  - android
  - other
date: 2010-03-25 10:30:11
---

![Hello there, Mr. hidden skype apk](http://173.230.150.16/blog/wp-content/uploads/2010/03/foundyou-180x300.png "Ah ha - found you!")
So I've been reading all the hype about the new Skype application released on Verizon. Sadly I neither have a droid or any friends with a rooted droid. As one would assume, the apk is located in `/data/app-private` so I needed access to a rooted droid to get this app... Or do I?

Silly me, I forgot all the research I had done for Market Enabler! After a friend sent me their `getprop` I was off spoofing my values, no more than two minutes went by and ta-da! Got myself the new Skype application.

The following `setprop` commands must be run to gain access to the Verizon only part of the market.
```
setprop gsm.sim.operator.numeric 310004
setprop gsm.operator.numeric 31000 
setprop gsm.operator.alpha "Verizon Wireless"
setprop gsm.sim.operator.alpha Verizon
setprop gsm.sim.operator.iso-country us
setprop gsm.operator.iso-country us
```
Note that you will need to restart your Vending (Market) application to have these values take affect, you can do that by running the following commands via a terminal:
```
# ps | grep vending
app_5     2699  75    181176 26384 ffffffff afe0dca4 S com.android.vending
# kill 2699
```
Your process id may not be 2699, so fill in whatever it actually is to kill the right process.

Now, running the Skype application is going to take more work than a few simple `getprop` and `setprop` commands... Well at least thats what I think so far, I haven't actually looked at the apk file. Until that's figured out, your phone is just going to return the following error screen:

![Pffhh, yea right...](http://173.230.150.16/blog/wp-content/uploads/2010/03/verizon-only-phone-180x300.png "Verizon Wireless phones only?")