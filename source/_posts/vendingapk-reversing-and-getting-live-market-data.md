---
title: Vending.apk reversing and getting live market data...
tags:
  - android
  - apk
  - buildPostParameters
  - bytecode
  - Cain &amp; Abel
  - data
  - g1
  - java
  - market
  - reverse engineering
  - vending.apk
  - Wireshark
url: vendingapk-reversing-and-getting-live-market-data.html
id: 158
categories:
  - android
date: 2009-01-03 00:59:03
---

![Mmmmm... Market Data...](http://173.230.150.16/blog/wp-content/uploads/2009/01/android_market_combo_8282008_wide_600x297-300x148.png "Mmmmm... Market Data...")
So I'm not sure why I didn't think to try to get live market data? For some reason I just **figured** it would be done with SSL or something so I just didn't try it. Long story short, after building a market-cache parser - I realized, DOH! You can just get the market data live using certain requests! Luckily the parser I made worked fine with the data that I was getting sent back.

It took a little reversing of Vending.apk to see exactly what type of encoding was being used - I sort of guessed right just by looking at what the phone was sending via a Cain&Abel and Wireshark dump. Though google just _happened_ to have some data in there that would through errors if you tried decoding the data directly or certain ways.

I'll post come up some of the reversing I did on Vending.apk - more specifically the `.buildPostParameters` routine which is where everything was created for post. It's actually pretty interesting stuff, and it helped find some routines online that google has publicly available through apache... Though I didn't find them in the Android libraries :) (nice of them, no? haha)

Anyway, tournament tomorrow followed by a snowboarding trip tomorrow - so I'll probably post that data on monday!