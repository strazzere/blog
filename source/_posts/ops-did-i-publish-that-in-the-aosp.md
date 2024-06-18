---
title: 'Ops, did I publish that in the AOSP?'
tags:
  - android
  - AOSP
  - droiddreamcleaner
  - experimental
  - security
url: 524.html
id: 524
categories:
  - android
  - coding
date: 2012-03-22 14:55:27
---

A while back when grep'ing through the AOSP for package manager references, I noticed something weird;
```
tstrazzere@spinach:~/repo/android$ grep -ir "/system/bin/pm" *
...
packages/experimental/droiddreamclean/droiddreamclean.c:  fp = popen("/system/bin/pm list packages", "r");
packages/experimental/droiddreamclean/droiddreamclean.c:    printf("failed to run /system/bin/pm list packages. not removing apps.\n");
packages/experimental/droiddreamclean/droiddreamclean.c:    system("/system/bin/pm uninstall com.android.providers.downloadsmanager");
packages/experimental/droiddreamclean/droiddreamclean.c:    system("/system/bin/pm uninstall com.android.providers.ammanage");
Binary file packages/experimental/AndroidVendorSecurityTool/assets/droiddreamcleanall matches
...
```
What's this directory? What is contained in it? Let's look at the readme;
```
tstrazzere@spinach:~/repo/android/packages/experimental$ ls -lhGg
total 60K
-rw-rw-r-- 1  143 2011-12-13 15:23 Android.mk
drwxrwxr-x 5 4.0K 2011-12-13 15:23 AndroidVendorSecurityTool
drwxrwxr-x 4 4.0K 2011-12-13 15:23 BugReportSender
drwxrwxr-x 4 4.0K 2011-12-13 15:23 CameraPreviewTest
-rw-rw-r-- 1 2.2K 2011-12-13 15:23 CleanSpec.mk
drwxrwxr-x 5 4.0K 2011-12-13 15:23 DreamTheater
drwxrwxr-x 2 4.0K 2011-12-13 15:23 droiddreamclean
drwxrwxr-x 4 4.0K 2011-12-13 15:23 ExampleImsFramework
drwxrwxr-x 4 4.0K 2011-12-13 15:23 LoaderApp
drwxrwxr-x 2 4.0K 2011-12-13 15:23 procstatlog
-rw-rw-r-- 1  844 2011-12-13 15:23 README
drwxrwxr-x 4 4.0K 2011-12-13 15:23 RpcPerformance
drwxrwxr-x 4 4.0K 2011-12-13 15:23 StrictModeTest
drwxrwxr-x 4 4.0K 2011-12-13 15:23 UiAutomation
drwxrwxr-x 3 4.0K 2011-12-13 15:23 UiAutomationDemo
 
tstrazzere@spinach:~/repo/android/packages/experimental$ cat README
The packages/experimental/ directory is for NON-SHIPPING code that is
not included with any flavor of the device, the SDK, or any other kind
of public release, but which might be useful for someone as a test
harness, development tool, or general fun times.
 
>> Every package under this directory must have a README file <<
 
Official SDK development samples should NOT go here, they should go in
development/samples/ instead.
 
Unlike the rest of the tree, code in experimental/ is NOT built by
default, and may be arbitrarily broken.  Caveat user!  Individual
packages must be built directly with "mmm" or equivalent:
 
   mmm packages/experimental/BugReportSender
 
Like a communal fridge, this directory will be cleaned periodically.
Every major release, we intend to remove and archive any package that
does not have an active owner and users.
```

Ops! Did someone mean to publish this repo? It's all a bunch of experimental and interesting code. Granted it's a bit old now, but still very interesting to look at. Specifically there are what looks to be the precursor to fragments and ui testing automation, done and committed before the final work was committed to AOSP. There is also the DroidDreamCleaner utility which is interesting. While it was an fix for an old issue, it's just interesting to see how Google coders handled the issue without having to reverse it. We can even see a "DreamThreater" application which looks like some work done to make an Android screen saver. Sadly not all this code can compile since it relies on some code which isn't accessible to us and the released branches of AOSP. It seems this code may have been mistakenly committed to the public branch after the kernel.org mishaps, as it appears to have been made public after AOSP became available on Google's own servers.

If you don't have all of AOSP pulled, you can get it by just cloning the following repository;
```
git clone https://android.googlesource.com/platform/packages/experimental
```
Again, nothing ground breaking - but definitely an interest repository of code to take a look at, if not to see how Google coders work on "internal" code which isn't released but to see their comments and documentation that is sometimes stripped from AOSP :)