---
title: More Identifiers without special permissions
tags:
  - android
  - android.os.Build
  - board
  - brand
  - fingerprint
  - g1
  - host
  - model
  - test-keys
  - unique id
url: more-identifiers-without-special-permissions.html
id: 118
comments: false
categories:
  - android
date: 2008-12-01 11:33:06
---

So it was a long, long weekend - hope everyone enjoyed it and had a good turkey day. Sorry for the lack of updates, but snowboarding took precedence over blogging. I've stumbled across some more identifiers that can be used for the Android OS, even better since these are obtained without special permissions.

This one comes from [android.os.Build](http://code.google.com/android/reference/android/os/Build.html).

Now the output from the emulator will look something like this;

```
Model: generic
Device: generic
Fingerprint: generic/generic/generic/:1.0/110632/110632:sdk/test-keys
Board: unknown
Brand: generic
ID: 110632
Host: undroid12.corp.google.com
Product: generic
Tags: test-keys
User: android-build
Type: sdk
Time: 1222115712000
```

When I get on my developer machine I will provide a dump of what these values look like on a G1 device. Though by looking at this output quickly, I wonder if the "fingerprint" would be able to provide a way of seeing if a phone is rooted or not? I guess we'll see -- I'll be interested to see what the differences in all these results are between my phone and my bothers phone; mine being rooted and his not.

Anyway, with these values, combined with past values we've found both with [special permissions](http://strazzere.com/blog/?p=116) and [no special permissions](http://strazzere.com/blog/?p=113), you should be able to hash something together for a unique identifier.

```
import android.os.Build;
...
        StringBuilder Buffer = new StringBuilder();
        String NEW_LINE = System.getProperty("line.separator");

        Buffer.append("Model: "+ (Build.MODEL).toString() + NEW_LINE);
        Buffer.append("Device: "+ (Build.DEVICE).toString() + NEW_LINE);
        Buffer.append("Fingerprint: "+ (Build.FINGERPRINT).toString() + NEW_LINE);
        Buffer.append("Board: "+ (Build.BOARD).toString() + NEW_LINE);
        Buffer.append("Brand: "+ (Build.BRAND).toString() + NEW_LINE);
        Buffer.append("ID: "+ (Build.ID).toString() + NEW_LINE);
        Buffer.append("Host: "+ (Build.HOST).toString() + NEW_LINE);
        Buffer.append("Product: "+ (Build.PRODUCT).toString() + NEW_LINE);
        Buffer.append("Tags: "+ (Build.TAGS).toString() + NEW_LINE);
        Buffer.append("User: "+ (Build.USER).toString() + NEW_LINE);
        Buffer.append("Type: "+ (Build.TYPE).toString() + NEW_LINE);
        Buffer.append("Time: "+ (Build.TIME) + NEW_LINE);
```


**updated on December 1, 2008 at 9:00pm**
From a rooted G1 device:

```
Model: T-Mobile G1
Device: dream
Fingerprint: tmobile/kila/dream/trout:1.0/TC4-RC30/116143:user/xda-dev
Board: trout
Brand: tmobile
ID: TC4-RC30
Host: undroid11.corp.google.com
Product: kila
Tags: ota-rel-keys,release-keys
User: android-build
Type: user
Time: 12255048530
```

Interesting results... :)