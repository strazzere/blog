---
title: Injecting code into DEX Files
tags:
  - android
  - dex code
  - g1
  - inject
  - inline
  - reverse engineering
  - secure code
url: injecting-code-into-dex-files.html
id: 92
categories:
  - android
  - coding
  - dex bytecode
  - reverse engineering
  - secure code
date: 2008-11-18 11:19:25
---

![Injecting code plausible and possible!](http://173.230.150.16/blog/wp-content/uploads/2008/11/04_datainjection-300x200.jpg "Inject code")
_Success!_ It seems completely possible, though quiet a pain to inject new code into existing dex files. This doesn't not appear like it would easily be done ON a device, though in the development setting it seems perfectly possible and completely do-able.

I'm working on a nice proof-of-concept example to show, though I don't think this is a "backdoor" to malware. Android has been set up well enough that to properly inject things it would require many things to be done, making it in my opinion extremely hard to do it on the fly on the device. I had to inject the code directly to the dex, resigned both the signature and hash makings for the file, then resign the whole package before reinstalling (after a complete uninstall since we don't have the same keys as the original package) onto the device. This is a _long_ way away from actually being able to do nasty things with it, which is clearly a good thing, since we don't want that to happen. This _does_ have practical uses of course, though it seems Google has done security rather well so that this process would most likely only be done by an actual developer for a user to _not_ notice an injected file... Otherwise they would have to allow unknown sources, packages would complain about key, so on and so on...

Hopefully more to come on this subject soon!