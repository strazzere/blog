---
title: A Lesson in Safe Dex
tags:
  - android
  - apkfuscator
  - blackhat
  - blackhat 2012
  - bytecode obfuscation
  - conference
  - dalvik
  - defcon
  - dex
  - dex bytecode
  - presentation
  - reverse engineering
url: a-lesson-in-safe-dex.html
id: 605
categories:
  - android
  - dex bytecode
  - reverse engineering
date: 2012-08-02 15:29:02
---

![Presenting at Blackhat 2012](http://www.strazzere.com/blog/wp-content/uploads/2012/08/presentation-cropped.png "Presenting at Blackhat 2012")
It's been almost a full week since my talk, [Dex Education: Practicing Safe Dex](https://www.blackhat.com/html/bh-us-12/bh-us-12-briefings.html#Strazzere), though I think I'm only now beginning to recover. The past few months have truly been a whirlwind of both working on dissecting malware at Lookout and working on putting together a solid presentation for BlackHat. So far I've been unable to draw a crowd like Charlie, though maybe someday I'll have people sitting in the aisles fighting for a seat during a presentation. Until then the people who went will just have to deal with the extra legroom. Over all the presentation seemed to go over pretty well, some interesting chats afterwards with some smart people. A few people where interested in the slides and proof of concept code, so I told them I would [tweet it](https://twitter.com/timstrazz/status/228617373840728064) and also make a blog post about it.

My slides are available [here](http://www.strazzere.com/papers/DexEducation-PracticingSafeDex.pdf) with the proof of concept code being hosted on my github page [here](https://github.com/strazzere/APKfuscator).

The proof of concept crackme code on the same github page as well shortly. I've got some extra content that I wasn't able to fit into the slide-deck, heck it was 96 slides as is after trimming some things out. While I didn't intend to try and cover everything possible to break most analysis tools, I wanted to attempt to cover as much as possible. Over the course of a few days or weeks, I'll try to roll out details in my blog about how certain things worked, mainly for people who where unable to attend the presentation, hear my explanations or ask me things at the conference. Feel free to reach out to me if there is anything I've missed or you would live a better explanation about.

A few people asked me about Blackhat and Defcon - wondering if it's worth attending. So to step on a soap box just for a minute, I'll give the mini speech that I normally tell people. Conferences are only worth what you put into them, go to talks that seem interesting and are outside of your direct field of work. Why attend talks outside the direct field of work? I've found it's a great way to try and find different perspectives, which often can be related back into your own work and field. It is also quiet hard to appreciate a talk on something that you deal with daily, definitely very important to try and keep this in mind if you do see those types of talks. As a presenter myself, I found it exceptionally hard to not go _too_ low level while still feeling like I can add value to everyone in the audience. After attending the talks you chose, _meet_ the presenters and pick their brains, this is honestly where you can learn the most. As I have said, it's really hard to make a presentation accessible for a whole audience, talking directly with these people will give you so much more information than the slides often do. The people you meet at the bars (for Blackhat @ Caesars goto the Galleria bar) are often people you talk to online already. Make friends, go outside that comfort zone and buy some people drinks. Most everyone is friendly, if they aren't - don't drink with them. Almost all conferences are worth going to, Blackhat and Defcon included, mainly due to the talent it attacks that you can find hanging out at the bars.

Probably the greatest thing about Blackhat for me was to meet some really great people I've only had the pleasure of talking to online. Talking with Mila, the mind behind Contagio Dump, was really great - able to pay her back a little for all the hard work she does with a beer or two. Got to talk with some of the original DroidSecurity (now AVG) guys, Elad and Oren, it's _never_ a dull moment talking to an Israeli reverse engineer - just look at [Zuk](http://www.twitter.com/ihackbanme). Another interesting person who I got to hang out with was along side me in the malware talk track, [@snare](http://www.twitter.com/snare). He did some crazy things with [EFI rootkits for OSX](http://ho.ax/posts/2012/07/black-hat-usa-2012/), pretty scary and interesting stuff all in the same talk.

People often say it isn't what you know, but who you know. I'd argue the security space is a ying and yang of both; to be a valuable (reverser) engineer you need to know your stuff and the people to help you succeed.

Enough on this soapbox, hopefully you enjoy the slides and code. If you ever run into me at a conference - let's have a beer or two and chat.