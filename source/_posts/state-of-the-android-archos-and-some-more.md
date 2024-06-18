---
title: State of the Android Archos... And Some More
tags:
  - a cheesy alternative
  - android
  - android market
  - appslib
  - appslibrary
  - archos
  - archos 5 it
  - archos 5 it root
  - archos android
  - root
  - State of the Archos Android
url: state-of-the-android-archos-and-some-more.html
id: 304
categories:
  - archos
  - coding
  - reverse engineering
  - secure code
date: 2009-10-23 16:24:08
---

Lately I've been receiving a bunch of emails regarding Android Market data and the Archos 5 IT. So I figured maybe a blog post would be the most appropriate place to attempt to address all the repeat questions, and heck - maybe answer a few before they are emailed to me! Recently I've been working on numerous projects, my focus has mainly been on the Archos 5 IT as it's my new toy! If you've been following me on twitter, you've seen my little picture showing I've gotten root (yay!).
[![Archos 5 IT Rooted](http://173.230.150.16/blog/wp-content/uploads/2009/10/rooted-225x300.jpg "Archos 5 IT Rooted")](http://173.230.150.16/blog/wp-content/uploads/2009/10/rooted.jpg)
Regarding root on the Archos 5 IT, I'm currently running the firmware 1.1.01, the root method appears to work fine on the newest 1.2.03(?) though I have not updated my device yet. No, not out of fear of loosing the root method, more from the advice of other people saying more things are broken in this new upgrade - so I'd like to keep my device running smoothly for what I do until I can fully root it. What do I mean by that? Well essentially we have root on the device, but on reboot we loose root. I'm currently working with [einstein_](http://archos.g3nius.org/) to modify the bootloader to accept any android img. This will allow us to modify the android image, and keep root after a reboot. It's posing trickier than we'd hoped so, people will just have to wait. Why are we waiting? Without a changable android image, no current android programs requiring root will function properly (there is no 'su' command to run) - so there is no reason to release it unless we want to see people brick their devices. 
![AppsLibrary - a cheesy alternative](http://173.230.150.16/blog/wp-content/uploads/2009/10/cheesy-300x225.jpg "AppsLibrary - a cheesy alternative")]
One of the other projects I've been working on is a web based version of AppsLib for the Archos 5 IT. This is essential a "cyrket"/"androlib" for the Archos library. The reason I'm doing this is because the current AppsLib application is garbage, there appear to be updates just about each week, yet each release appear to only cause more crashes... Maybe just for me, but I doubt that. Anyway I've posted some screen shots for what it's going to look like on some forums and I'm relatively close to releasing it. It's almost at the process of just being migrated from my developement machine to this server. Also note that it's never probably going to be the most functional thing in the world, but it works - more than I can say for the application version right now. The features on release will most likely be, list ten applications in the date of release, give the information available for the item and a link to download it. I'll add searching and category sorting later on hopefully.

A word on the Android Market data. I've not yet had time to write up all my posts on how to collect, spoof and do what not with the data. This _will_ come, though maybe not in the most timely fashion. I know many people are emailing me saying they want to make an open source market client that downloads stuff, well I highly doubt that will happen. Yes we can download applications, yes we can get all the data. The problem lies with some SSL chatter that we cannot and probably will not decrypt. 

Lastly, I want to remind people that I do have a paying job, a loving girlfriend and other activities I love doing outside of the computer/android realm. Please hang in there while I take care of my own things first and then work on these as I see fit. People have been telling me that certain ones are more important than the others, but it comes down to this is a hobby and not my real job. I do it in spare time and I've been making time for it enough lately. So try not to be too hard on me when I don't release information you want immediately, when you want it. So, thats my appology on that -- and that was my little State of the Archos (Android)