---
title: Compiling cupcake with some changes for the G1 (HTC Dream)
tags:
  - android
  - android os
  - compile
  - cupcake
  - donut
  - extract-files.sh
  - g1
  - google
  - make
  - master
  - source.android.com
  - tmobile
url: compiling-cupcake-with-some-changes-for-the-g1-htc-dream.html
id: 220
categories:
  - android
  - coding
  - updating
date: 2009-03-23 23:34:32
---

![Easier than baking a cake!](http://173.230.150.16/blog/wp-content/uploads/2009/03/android-cupcake-200x300.jpg "Compiling cupcake...")
Currently working on some random things on the [Android OS source](http://source.android.com) - which of course if I can get my little, "additions", to work properly I'll be releasing them. Though I was having a few problems getting the code to compile - so I figured I'd throw together this post just in case anyone else is having similar trouble.

Essentially in the post I was originally reading [here](http://forum.xda-developers.com/showthread.php?t=462512), stated the following;
```
At the time of writing, the Android Dream build was broken. I needed to do the following to make it work:
* Several (relatively minor) changes in the Dream audio driver code to fix compilation issues.
* Copied libOmxCore.so to mydroid/out/target/product/dream/system/lib (this was a missing step in the Building for Dream documentation, and something that should be in the HTC provided script)
```
Well that wasn't the problem I was getting though -- the error I had been getting was actually:
```
In file included from frameworks/base/libs/audioflinger/AudioResamplerCubic.cpp:20:
system/core/include/cutils/log.h:68:1: warning: this is the location of the previous definition
target thumb C++: libaudioflinger <= frameworks/base/libs/audioflinger/AudioFlinger.cpp make: *** No rule to make target out/target/product/dream/obj/lib/libaudio.so’, needed by out/target/product/dream/obj/SHARED_LIBRARIES/libaudioflinger_intermediates/LINKED/libaudioflinger.so’. Stop.
…
make: *** No rule to make target /tmp/dream/target/product/dream/obj/lib/libcamera.so', needed by /tmp/dream/target/product/dream/obj/SHARED_LIBRARIES/libcameraservice_intermediates/LINKED/libcameraservice.so’. Stop.
…
make: *** No rule to make target /tmp/dream/target/product/dream/obj/lib/libOmxCore.so', needed by /tmp/dream/target/product/dream/obj/SHARED_LIBRARIES/libopencorecommon_intermediates/LINKED/libopencorecommon.so’. Stop.
…
/media/other-storage/mydroid/prebuilt/linux-x86/toolchain/arm-eabi-4.2.1/bin/../lib/gcc/arm-eabi/4.2.1/../../../../arm-eabi/bin/ld: warning: librpc.so, needed by /tmp/dream/target/product/dream/obj/lib/libaudio.so, not found (try using -rpath or -rpath-link)
…
make: *** No rule to make target /tmp/dream/target/product/dream/system/lib/libaudio.so', needed by /tmp/dream/target/product/dream/system/lib/libaudioflinger.so’. Stop.
…
make: *** No rule to make target /tmp/dream/target/product/dream/system/lib/libcamera.so', needed by /tmp/dream/target/product/dream/system/lib/libcameraservice.so’. Stop.
…
make: *** No rule to make target /tmp/dream/target/product/dream/system/lib/libOmxCore.so', needed by /tmp/dream/target/product/dream/system/lib/libopencorecommon.so’. Stop.
```
(This list was compiled over multiple re-runs of make)

The simple fix was essentially what was stated in the original post, though it just took me a few minutes to figure this out. After running the locate command like a mad man and not finding anything on my developer machine, I finally reread *everything* and checked the device for the file – and there it was!

I just ended up throwing this into my `extract-files.sh`;
```
adb pull /system/lib/libaudio.so proprietary/libaudio.so
adb pull /system/lib/libcamera.so proprietary/libcamera.so
adb pull /system/lib/libOmxCore.so proprietary/libOmxCore.so
adb pull /system/lib/librpc.so proprietary/librpc.so
```
So my resulting file looked like this;
```
#!/bin/sh

mkdir -p proprietary
adb pull /system/etc/AudioFilter.csv proprietary/AudioFilter.csv
adb pull /system/etc/AudioPara4.csv proprietary/AudioPara4.csv
adb pull /system/etc/gps.conf proprietary/gps.conf
adb pull /system/bin/akmd proprietary/akmd
adb pull /system/lib/libhtc_ril.so proprietary/libhtc_ril.so
adb pull /system/lib/libaudioeq.so proprietary/libaudioeq.so
adb pull /system/lib/libqcamera.so proprietary/libqcamera.so
adb pull /system/lib/libaudio.so proprietary/libaudio.so
adb pull /system/lib/libcameraservice.so proprietary/libcameraservice.so
adb pull /system/lib/libcamera.so proprietary/libcamera.so
adb pull /system/lib/libOmxCore.so proprietary/libOmxCore.so
adb pull /system/lib/librpc.so proprietary/librpc.so
chmod 755 proprietary/akmd

adb pull /system/etc/wifi/Fw1251r1c.bin proprietary/Fw1251r1c.bin
adb pull /system/etc/wifi/tiwlan.ini proprietary/tiwlan.ini
```

`libaudio.so`, `libcameraservice.so`, `libcamera.so`, `libOmxCore.so`, `librpc.so` then needed to be copied over to `TARGET/dream/target/product/dream/obj/lib/` and also to the locked folder `TARGET/dream/target/product/dream/system/lib/`

Ta-da! After approximately 10 runs, it finally compiled for me... Hopefully this helps anyone who ran into the same problems as me.