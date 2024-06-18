---
title: '"Insecure" spyware and the ransomware counterparts'
tags:
  - android
  - backdoor
  - dlp mobile
  - malware
  - reveal anti sms spy
  - sms replicator
  - spyware
url: insecure-spyware-and-the-ransomware-counterparts.html
id: 384
categories:
  - android
  - reverse engineering
  - secure code
date: 2010-11-17 14:12:06
---

A few weeks back a company named DLP Mobile got some press about a ["new spy app"](http://bits.blogs.nytimes.com/2010/10/27/android-app-forwards-private-text-messages/?src=twt&twt=nytimesbits#), which well - it just very similar to all the other ones we've seen. It "hides" itself and forwards its text messages to other numbers. Very sneaky and genius, no? Wait, let me rephrase that, no - not very sneaky or genius :) Anyway, that's not really what I want to blog about. Shortly after the application was removed from the Android Market by Google \[link\], another interesting one appeared - ["Reveal: Anti SMS Spy"](http://www.androlib.com/android.application.com-dlp-smssecretreplicatorfinder-qnjnx.aspx).

Not only has the company published a spyware application, but they decided to sell a program to remove it too. Interesting move by them, I guess? Seems like an attempt a FUD and trying to make some quick cash. While I have no idea the true intentions of the creators, I do find it interesting that the price of the application keep increasing that the price of the application kept increasing which each press grab. A nice little article actually brought this up again early this week at the [Register](http://www.theregister.co.uk/2010/11/16/android_spyware/).

On top of all this nice news - there where quiet a few tear downs of the application. A good one, written by [Axelle Apvrille of Fortinet](http://blog.fortinet.com/hidden-feature-in-android-spyware/), highlights the "backdoor" inside it. It's also interesting to note that this backdoor can be found in all variants/rebrands of the application, SMSNanny and SMSReplicator (non-secret). It reminds me of the old "Remote Administration Tools" like SubSeven which often contained "master passwords" for people to take over with.

I'd like to end this short post by quoting Axelle, as I feel she's completely correct in this statement:

> Seriously, it is very lucky SMS Replicator Secret is not remotely configurable, otherwise attackers could have randomly scanned the networks for infected phones and spy their incoming messages…

Wise words crypto girl, though sadly I fear that people making these applications for a quick buck will never want to take the time write secure programs.