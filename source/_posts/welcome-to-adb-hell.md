---
title: Welcome to ADB HELL!
tags:
  - adb
  - adb hell
  - android
  - coding hell
  - easter egg
  - google
  - google humor
  - humor
url: welcome-to-adb-hell.html
id: 395
categories:
  - android
date: 2011-03-08 15:01:13
---

A coworker just pointed out something that really made my day. Apparently if you type "adb hell" opposed to "adb shell", it will give you an adb shell with hellish colors:
![Welcome to hell... ADB HELL!](http://173.230.150.16/blog/wp-content/uploads/2011/03/adbhell.png "We want your eyes to BLEED!")
Thanks Google, I always enjoy your humor :) For those interested, the code that does this can be found here:
[http://codesearch.google.com/codesearch/p?hl=en#2wSbThBwwIw/adb/commandline.c&l=848](http://codesearch.google.com/codesearch/p?hl=en#2wSbThBwwIw/adb/commandline.c&l=848)
```
char h = (argv[0][0] == 'h');

if (h) {
    printf("\x1b[41;33m");
    fflush(stdout);
}
```
**Edit:** Some people have been mentioning it didn't work for them, so I found the commit so you know if you'll have it or not. Thanks Daniel Sandler and Mike Lockwood for the Easter Egg!
[http://android.git.kernel.org/?p=platform/system/core.git;a=commitdiff;h=9c73d17e870e448ea1f036bda70736ae0ae6bf2e](http://android.git.kernel.org/?p=platform/system/core.git;a=commitdiff;h=9c73d17e870e448ea1f036bda70736ae0ae6bf2e)