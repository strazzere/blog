---
title: Back to the basics
tags:
  - /proc/pid/mem
  - android market
  - GameCIH
  - Open Source Annoyed
  - reverse engineering
  - vending.apk
url: back-to-the-basics.html
id: 382
categories:
  - android
  - life
  - other
date: 2010-10-05 02:24:22
---

It's been a fun and interesting ride on my Android reversing adventure. Nothing explicit to post now, but I just did want to touch on a few things I've seen over the past few days. I've been doing some research in the anti-piracy field and seen a few different interesting things. One interesting thing I came across is direct memory access, while it's not possible to inject dalvik opcodes - we could in theory do this with root access. I know, not much help - huh? Still some pretty interesting things we might be able to do with the _/proc/pid/mem_ address spaces if we did have root. An interesting application that uses this is [GameCIH](http://www.appbrain.com/app/com.cih.gamecih), interesting use-case and even more interesting UI for it! Either way I'll hopefully post a few proof of concepts projects that I have been dabbling with off and on for a while.

Another thing I've come to realize as I delve more into Android protection schemes, is that your binaries are never truly protected. While I know it's never safe to assume that your binaries are a safe asset, as they almost never are on any operating system - for some reason I was under the assumption that they would be more protected with user based permissions on Android. Though this just seems to not be true - as even "protected" applications have their _/lib_ directory world-readable, and even loadable by any other application. This isn't really a security break by any means, but something I found interesting as I look deeper into things.

On a side rant, I've become increasingly frustrated with the amount of people a demanding support for open source code. It's quiet annoying to see people ranting on and on about issues they have with code, when they honestly don't take the time to debug any of their issues. I guess it boils down to the same issues you find day after day on a forum, people don't want to search for an answer - they just want to be given the answer. It tends to get even more maddening when you know these people are just trying to turn a buck off the open source project and never even thank people for their help! Shuck -- rant mode off for now.

A better note to end on though - I've gotten back into my Android Market (Vending) reversing roots. It's been a little bit since I last looked at some of the dalvik code - but it appears Google has done some interesting things with it. The protobuf has evolved for sure, as well as the Market itself. There are suggestion feedback while searching, many more application shelfs for carriers and tons of extra fields I never really traversed. It's an awesome thing to have seen evolve and exceptionally interesting to have worked on since day one. I'm sure I'll be blogging more about it at time goes on. So stay tuned for it if your interested, as I'm sure I'll be working on it all week. I've also just gotten my hands on a G2, so it's the first time I've been using a non-rooted phone - another thing to distract me from doing work :) - hopefully I'll be posting more on Market data though. It's always been interesting stuff, I guess I'll have to finally finish some of the projects of mine!