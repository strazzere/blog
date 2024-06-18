---
title: Quickly setting up an Android emulator for fuzzing/hacking
tags:
  - android
  - AOSP
  - emulator
  - fuzzing
  - hacking
  - reverse engineering
url: quickly-setting-up-an-android-emulator-for-fuzzinghacking.html
id: 409
categories:
  - android
  - coding
  - reverse engineering
date: 2012-01-14 22:15:43
---

Normally I like spending time fuzzing/causing segfaults with real devices, not sure why - maybe it's just because it just feels more gratifying. Though recently for a small project at work, I need to be able to do multiple devices for longer periods of time and swap out different modules relatively fast. This would be a huge problem - but I wanted to automate it and do the least amount of work possible. A challenge that comes up sometimes with the somewhat stock phones I use for testing, is that some compiled modules/elf binaries never worked that well. Sometimes due to carrier modifications, sometimes due to Cyanogen modifications other times just due to me being stupid and compiling things horribly wrong. Since I just got my desktop machine all set up, it seemed like the perfect time to try and automate this on a box I could just SSH into and let run all the time!

After a quick bit of Googling, I didn't see many instructions on how to modify emulator images fast. Lots of people said, generate an emulator, then change the `system.img` by mounting it, then restart the emulator and make sure it doesn't overwrite changes. That's a pain though, I don't want to mount anything! Though this did remind me of building the AOSP which can product the `system.img` -- perfect!

After making all my changes, some Dalvik changes and some Webkit changes, kicked off a full build via the normal commands; 

```
tstrazzere@spinach:~/repo/android$ . build/envsetup.sh
including device/samsung/maguro/vendorsetup.sh
including device/samsung/tuna/vendorsetup.sh
including device/ti/panda/vendorsetup.sh
including sdk/bash_completion/adb.bash
tstrazzere@spinach:~/repo/android$ lunch full-eng

============================================
PLATFORM_VERSION_CODENAME=AOSP
PLATFORM_VERSION=4.0.1.2.3.4.5.6.7.8.9
TARGET_PRODUCT=full
TARGET_BUILD_VARIANT=eng
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv7-a
HOST_ARCH=x86
HOST_OS=linux
HOST_BUILD_TYPE=release
BUILD_ID=OPENMASTER
OUT_DIR=out
============================================

tstrazzere@spinach:~/repo/android$ make -j4
============================================
PLATFORM_VERSION_CODENAME=AOSP
PLATFORM_VERSION=4.0.1.2.3.4.5.6.7.8.9
TARGET_PRODUCT=full
TARGET_BUILD_VARIANT=eng
TARGET_BUILD_TYPE=release
TARGET_BUILD_APPS=
TARGET_ARCH=arm
TARGET_ARCH_VARIANT=armv7-a
HOST_ARCH=x86
HOST_OS=linux
HOST_BUILD_TYPE=release
BUILD_ID=OPENMASTER
OUT_DIR=out
============================================
...
target Symbolic: libwebcore (out/target/product/generic/symbols/system/lib/libwebcore.so)
target Strip: libwebcore (out/target/product/generic/obj/lib/libwebcore.so)
Install: out/target/product/generic/system/lib/libwebcore.so
target Executable: webcore_test (out/target/product/generic/obj/EXECUTABLES/webcore_test_intermediates/LINKED/webcore_test)
Finding NOTICE files: out/target/product/generic/obj/NOTICE_FILES/hash-timestamp
target Symbolic: webcore_test (out/target/product/generic/symbols/system/bin/webcore_test)
target Strip: webcore_test (out/target/product/generic/obj/EXECUTABLES/webcore_test_intermediates/webcore_test)
Combining NOTICE files: out/target/product/generic/obj/NOTICE.html
Installed file list: out/target/product/generic/installed-files.txt
Target system fs image: out/target/product/generic/obj/PACKAGING/systemimage_intermediates/system.img
Install system fs image: out/target/product/generic/system.img
```

While waiting for the compilation to finish - I prepped a "target platform" to build an emulator image for. To quickly do this I just copied the `android-15` directory under the `platforms` directory of the sdk. This will give us the proper structure and files required while keeping the normal platform for android-15 intact. 

```
tstrazzere@spinach:~/android/android-sdk-linux/platforms$ cp -r android-15 custom
tstrazzere@spinach:~/android/android-sdk-linux/platforms$ cd custom/
tstrazzere@spinach:~/android/android-sdk-linux/platforms/custom$ ls -l
tstrazzere@spinach:~/android/android-sdk-linux/platforms/custom$ ls -l
total 13124
-rw-rw-r-- 1 tstrazzere tstrazzere 13402051 2012-01-14 18:46 android.jar
-rw-rw-r-- 1 tstrazzere tstrazzere 1371 2012-01-14 18:46 build.prop
drwxrwxr-x 4 tstrazzere tstrazzere 4096 2012-01-14 18:46 data
-rw-rw-r-- 1 tstrazzere tstrazzere 1796 2012-01-14 18:46 framework.aidl
drwxrwxr-x 2 tstrazzere tstrazzere 4096 2012-01-14 18:46 images
drwxrwxr-x 4 tstrazzere tstrazzere 4096 2012-01-14 18:46 renderscript
-rw-rw-r-- 1 tstrazzere tstrazzere 206 2012-01-14 18:46 sdk.properties
drwxrwxr-x 3 tstrazzere tstrazzere 4096 2012-01-14 18:46 skins
-rw-rw-r-- 1 tstrazzere tstrazzere 398 2012-01-14 18:46 source.properties
drwxrwxr-x 2 tstrazzere tstrazzere 4096 2012-01-14 18:46 templates
tstrazzere@spinach:~/android/android-sdk-linux/platforms/custom$ cd images/
tstrazzere@spinach:~/android/android-sdk-linux/platforms/custom/images$ ls -l
total 121400
-rwxrwxr-x 1 tstrazzere tstrazzere 1445004 2012-01-14 18:46 kernel-qemu
-rw-rw-r-- 1 tstrazzere tstrazzere 359448 2012-01-14 18:46 NOTICE.txt
-rw-rw-r-- 1 tstrazzere tstrazzere 149875 2012-01-14 18:46 ramdisk.img
-rw-rw-r-- 1 tstrazzere tstrazzere 119406144 2012-01-14 18:46 system.img
-rw-rw-r-- 1 tstrazzere tstrazzere 2948352 2012-01-14 18:46 userdata.img
tstrazzere@spinach:~/android/android-sdk-linux/platforms/custom/images$ rm system.img
rm: remove regular file `system.img'? y
tstrazzere@spinach:~/android/android-sdk-linux/platforms/custom/images$ cp ~/repo/android/out/target/product/generic/system.img $PWD
tstrazzere@spinach:~/android/android-sdk-linux/platforms/custom/images$ ls -l
total 170976
-rwxrwxr-x 1 tstrazzere tstrazzere 1445004 2012-01-14 18:46 kernel-qemu
-rw-rw-r-- 1 tstrazzere tstrazzere 359448 2012-01-14 18:46 NOTICE.txt
-rw-rw-r-- 1 tstrazzere tstrazzere 149875 2012-01-14 18:46 ramdisk.img
-rw------- 1 tstrazzere tstrazzere 170172288 2012-01-14 18:48 system.img
-rw-rw-r-- 1 tstrazzere tstrazzere 2948352 2012-01-14 18:46 userdata.img
```
Awesome, we copied over the `system.img` too the nessicary directory, lets just edit a few files so I don’t forget what this when I build my emulators. Open up the `source.properties` file and edit the version number to something you can remember, I just changed mine to say `4.0.3-hacked`;
```
tstrazzere@spinach:~/android/android-sdk-linux/platforms/custom$ android list targets
Available Android targets:
...
----------
id: 22 or "android-15"
Name: Android 4.0.3-hacked
Type: Platform
API level: 15
Revision: 1
Skins: WXGA (default)
ABIs : armeabi
----------
...
```

Now I can just create an emulator like normal and fire it up;
```
tstrazzere@spinach:~/android/android-sdk-linux/platforms/custom$ android create avd -t 27 -n dalvik_webcore_fuzz
Auto-selecting single ABI armeabi-v7a
Android 4.0.3-hacked is a basic Android platform.
Do you wish to create a custom hardware profile [no]
Created AVD 'dalvik_webcore_fuzz' based on Android 4.0.3-hacked, ARM (armeabi-v7a) processor,
with the following hardware config:
hw.lcd.density=240
vm.heapSize=48
hw.ramSize=512
tstrazzere@spinach:~/android/android-sdk-linux/platforms/custom$ emulator -avd dalvik_webcore_fuzz
```

Now it just time to attach your fuzzing and other automated tools to the emulator.