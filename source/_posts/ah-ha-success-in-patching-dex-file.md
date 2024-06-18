---
title: Ah-ha! Success in patching DEX file!
tags:
  - android
  - dex bytcode
  - patching
  - reverse engineering
  - success
url: ah-ha-success-in-patching-dex-file.html
id: 30
categories:
  - android
  - dex bytecode
  - reverse engineering
date: 2008-10-30 21:50:48
---

![Successfully patching an android application](http://173.230.150.16/blog/wp-content/uploads/2008/10/device-200x300.png "Success!")
It had been bugging me tons since the application kept crashing. I knew the signature and checksum where correct since it wasn't barfing on installation of the .apk file. So I kept thinking and thinking, finally I decided to do something useful... Look at the traces log! Here we can clearly see that an exception is being thrown... But why?
```
----- pid 210 at 2008-10-30 21:15:29 -----
Cmd line: com.android.KeyGenMe

DALVIK THREADS:
"main" prio=5 tid=3 NATIVE
  | group="main" sCount=1 dsCount=0 s=0 obj=0x400113a8
  | sysTid=210 nice=0 sched=0/0 handle=-1095390052
  at android.os.BinderProxy.transact(Native Method)
  at android.app.ActivityManagerProxy.handleApplicationError(ActivityManagerNative.java:2023)
  at com.android.internal.os.RuntimeInit.crash(RuntimeInit.java:302)
  at com.android.internal.os.RuntimeInit$UncaughtHandler.uncaughtException(RuntimeInit.java:75)
  at java.lang.ThreadGroup.uncaughtException(ThreadGroup.java:853)
  at java.lang.ThreadGroup.uncaughtException(ThreadGroup.java:850)
  at dalvik.system.NativeStart.main(Native Method)

"Binder Thread #2" prio=5 tid=13 NATIVE
  | group="main" sCount=1 dsCount=0 s=0 obj=0x43360018
  | sysTid=215 nice=0 sched=0/0 handle=972800
  at dalvik.system.NativeStart.run(Native Method)

"Binder Thread #1" prio=5 tid=11 NATIVE
  | group="main" sCount=1 dsCount=0 s=0 obj=0x4335efb8
  | sysTid=214 nice=0 sched=0/0 handle=972616
  at dalvik.system.NativeStart.run(Native Method)

"JDWP" daemon prio=5 tid=9 VMWAIT
  | group="system" sCount=1 dsCount=0 s=0 obj=0x4335e2a0
  | sysTid=213 nice=0 sched=0/0 handle=799384
  at dalvik.system.NativeStart.run(Native Method)

"Signal Catcher" daemon prio=5 tid=7 RUNNABLE
  | group="system" sCount=0 dsCount=0 s=0 obj=0x4335e1e8
  | sysTid=212 nice=0 sched=0/0 handle=796600
  at dalvik.system.NativeStart.run(Native Method)

"HeapWorker" daemon prio=5 tid=5 VMWAIT
  | group="system" sCount=1 dsCount=0 s=0 obj=0x42533028
  | sysTid=211 nice=0 sched=0/0 handle=793976
  at dalvik.system.NativeStart.run(Native Method)

----- end 210 -----
```
I decided to do yet another smart thing, that I should've done - and redumped the dex file and see if it was making any sense... Of course! I edited the wrong opcode. Apparently in my overwhelming dumbness I tried changing the registers and a exception thrown for the statement. This is something the Dalvik-VM did not agree with, thus the barfing.

I'm going to recreate a nice little example with source code of a simple patch performed on a dex file, and I'll outline the process used to do so. Hopefully I'll have this posted sometime tomorrow!