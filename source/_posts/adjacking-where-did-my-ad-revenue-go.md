---
title: Adjacking... Where did my ad revenue go?
tags:
  - adjacking
  - admob
  - adsense
  - android
  - android money
  - mobile
  - monetizing
  - money
  - reverse engineering
url: adjacking-where-did-my-ad-revenue-go.html
id: 323
categories:
  - other
  - secure code
date: 2010-03-17 21:28:53
---

![Make money, money...](http://173.230.150.16/blog/wp-content/uploads/2009/08/android-money.jpg "Easy cash with no work!")
It's been a while since I've posted any article, sadly between work, contracts after work, spam, having a life and volleyball I don't have much time to spend on my blog. Research is still going strong - but very little has trickled out from me over the past few months.

Something I'd like to finally post has come to my attention. Approximately a year ago when I first started looking into the mobile ad networks, I thought of something sinister. While I never intend to do anything evil, since I'm always looking for ways to protect myself, my first thoughts always seem to be, "how do I abuse this?" It all started when people started trying to think of ways to monetize their application. Do we charge up front, or do we try to make a few bucks off a huge user-base using ads?

My first question is, how are the ads secured? Much like other applications that are tracked by application, most use a "application id" or "publisher id". This is a super-secret code that is used for identifying traffic from you, right? Alright, well unlike a website advertisement, which has a referrer - mobile ads have no actual way to differentiate traffic other than this "unique" id.

So what? Whats the issue with that? Well, there is a big issue with this. There is a coined term, "adjacking", that essentially means "falsified" clicks. Originally this term meant you hijacked the javascript of google adsense, and made a click anywhere on your website appear to be a click on your adsense ad. Though, I'm "word-jacking" this term, because I feel my definition is a little more appropriate. Essentially, with the ability to easily decompile/modify an apk file - someone can quiet easily steal your ad traffic, this hi-jacking your ads… Adjacking.

Is this something new? No - but beware of it. I've had this article lying around for a bit, more uninterested in publishing for the idea that people would actually attempt to do this if I brought it up. Upon first writing this, I quickly made a program that attempted to make a database of signatures of programs. This program downloaded legit (free) applications and grabbed the signature from the META-INF folder of the apk. Then it attempted to find versions available for download on the internet…. For the most part, the version where always the same - with a rare instance of someone resigning it with little modification to the file, often to help localize it. Though now, I've seen and heard of an increase of people downloading their application, replacing the ID in the apk, and replacing it with their own.
![Keep your code secure!](http://173.230.150.16/blog/wp-content/uploads/2009/01/strictly_confidential-230x300.jpg "Keep your code secure!")
What to do about this? Well, hopefully the ad networks figure something out, though I'm not sure they honestly care much. I've sent emails to a few of the big providers with no responses and a few "we'll look into it" replies. I don't see a big downside for them - maybe if more people complain they'll get the hint. I'm sure right now they'll just get the traffic, for traffic's sake. Most applications that have been modified probably don't drive in much or take away much from other people. Though if they do, they could "act" upon these and actually shut people down… Will the correct developer ever see this money? Probably not… Though if your try hard enough you might see something.

The sad part is, most of the people modifying the applications are now no better than a scripting kiddie. There are enough tools available now to make this an easy job. Maybe if people start looking into this, these people will be rooted out - since they must fill in "legit" information to open an account. 

Anyways, I've been looking at some protection schemes for this, hopefully I'll have time to post some soon. I'll post a little tutorial on obfuscating (manually) your adsense/admob/blah code to protect yourself :)