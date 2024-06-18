---
title: InsomniDroid - great presentation on Android Reversing Tools
tags:
  - android
  - android malware
  - crackme
  - cryptax
  - fortiguard
  - insomnidroid
  - insomnihack
  - reverse engineering
  - tools
url: insomnidroid-great-presentation-on-android-reversing-tools.html
id: 470
categories:
  - android
  - reverse engineering
date: 2012-03-06 13:55:11
---

Recently, Axelle from Fortinet aka Crypto Girl, presented a well done paper at [InsomniHack 2012](http://www.scrt.ch/en/insomnihack/2012/presentation) - which she just posted on her [twitter](http://www.twitter.com/cryptax) account. It does a _very_ good job at outlining the basics of how to get into Android reversing and the Android reverse engineer tools. Since she works in the AV industry, it's based on that view - a defensive analyst looking at malware. She does a great job describing the contents of an APK, how to read specific files (AndroidManifest.xml, etc) - giving multiple ways too. It's nice to see how different people tackle these types of issues.

It's also a nice thing to see someone outline all the different applicable tools. I haven't seen anyone outside of a few Google people mention dexdump in a long time - definitely an underrated tool! It was interesting to see the list of tools she listed for decompiling the apk, specifically _ded_. I mention _ded_ specifically since I've never actually seen anyone use it or mention it outside of the academic work done around it. _Dex2jar_ was obviously mentioned, it's quiet possibly the most (over)used tool out there. Personally I'm not that big of a fan of the project, though it definitely has some great use cases and people seem to love it.

Axelle did a great job providing a tutorial with a clean step by step of what to look for when tackling malware. Great steps provided about which tool to use, how to identify interesting points and then a nice little "ops! the tool broke, lets switch to the next one!" which can often happen due to the early stages of development for most of these tools. She also provides a slide on how synthetic access is created when inner classes access private members of their enclosing classes. It's an interesting concept that seems to be lost on many beginner reversers in the Android world.

There are even a few slides on anti-emulator code, which has always been an interesting thing to me. She mentions how the Honeynet challenge implemented a few tricks and how one could either (1) modify the application or (2) modify the emulator. I've seen a few other people mention "hiding the emulator", which has sent me into an interesting little tangent of research. Hopefully I'll have time to add that into an expect Blackhat/Defcon presentation I'm working on. Personally I'm not a fan of the simplistic approach of just changing the application or the emulator imei - there have to be a ton of easier ways for an attacker to find the emulator and the defender to hide it.

Another awesome aspect of this presentation is how Axelle really did try to provide information about _every_ tool. I've seen many people mention the use of Androguard, but not what features they use. I've personally messed around with it a bit, but haven't found the urge to use it more often in my day to day work. I keep coming back to thinking, awesome features and implementation, though it's just so much information I haven't found it useful. One of Axelle's slides I think sums it up pretty well;
![Uhmm -- maybe?](http://www.strazzere.com/blog/wp-content/uploads/2012/03/androguard.png "Worth the trouble?")
Enough about the presentation - you should read it yourself. The PDF is available from [Fortinets's site](http://www.fortiguard.com/sites/default/files/insomnidroid.pdf) ([local mirror](http://www.strazzere.com/papers/insomnidroid.pdf))

As a last side note -- it's extremely refreshing to see an Android presentation that does _not_ include this "presentation filler" known as the "Android Architecture" picture :) ![](http://www.strazzere.com/blog/wp-content/uploads/2012/03/android-architecture.jpg "I HATE THIS PICTURE")

**Edit:** I forgot to link the crackme challenge which Axelle also provided at InsomniHack - which is also called InsomniDroid. It's published over at root-me.org if anyway wants to take the challenge. It was a fun quick little challenge, available [here](http://www.root-me.org/en/Challenges-148/Realist/Insomni-Droid.html?lang=en).