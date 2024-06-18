---
title: Compiling an Android Emulator Kernel for Loadable Kernel Modules
tags:
  - android
  - compile
  - emulator
  - kernel
  - LKM
  - qemu
url: compiling-an-emulator-kernel-for-loadable-modules.html
id: 727
categories:
  - android
  - coding
  - reverse engineering
date: 2014-07-20 20:47:39
---

So you want to rootkit the emulator? These are rough notes I took while attempt to get this working on my own machine (OSX 10.8.5) - results may vary. According to random posts I've seen, OSX is a bit finicky and no one really gets it to work right - Ubuntu everything is apparently just peachy. You've been warned though, YMMV.
_If on OSX, you must install libelf - this was the only dependency I was missing. If you don't have this the build will randomly fail and not be exceptionally helpful about why._
**Building the Kernel**
Clone emulator kernel directory then get the branch of the kernel we want
```
git clone https://android.googlesource.com/kernel/goldfish
...
cd goldfish
git checkout -t origin/android-goldfish-2.6.29 -b goldfish
```
Before we build the kernel we must configure it, though we don't want the default configuration (this doesn't actually let the emulator boot) and we want to ensure LKM support is present. Note that the feature to load is seperate from the unload feature, you must enable both. Let's first copy the configuration for `goldfish_armv7_defconfig`. Then manually change the LKM state (`goldfish_armv7_defconfig` will default to having modules enabled and loadable, but not unloadable) to any features we need.
```
make ARCH=arm goldfish_armv7_defconfig ...
make ARCH=arm menuconfig
```
If you are compiling on OSX, you will want to manually edit the `.config` file to not include `CONFIG_NETFILTER`, simply comment this line out before proceeding. You will be prompted to confirm this change prior to compiling as well. If you do not make this change you'll see an issue similar to this appear;

```
make[2]: *** No rule to make target \`net/netfilter/xt_CONNMARK.o', needed by \`net/netfilter/built-in.o'. Stop.
make[1]: *** [net/netfilter] Error 2
make: *** [net] Error 2
```
Compile the kernel, modify the `CROSS_COMPILE` switch as necessary for your builders setup;
```
make ARCH=arm SUB_ARCH=arm CROSS_COMPILE=$ANDROID_NDK_ROOT/toolchains/arm-linux-androideabi-4.4.3/prebuilt/darwin-x86/bin/arm-linux-androideabi-
```
If you are unsure where to point _CROSS_COMPILE_, you can try below to help narrow it down, assuming you have
```
$ANDROID_NDK_ROOT set;
find $ANDROID_NDK_ROOT | grep '\\-gcc$'
```
**Making the Emulator use the Newly Compiled Kernel**
The easiest way to use the kernel immediately is to just point an already existing AVD image at it via the emulator command;
```
emulator -kernel arch/arm/boot/zImage -avd -verbose
```
You don't need -verbose, obviously, though it would be helpful the first time to watch for the potential segfault or if the kernel image was bad. If the "android" doesn't appear in the emulator box and nothing is streaming to the console, you likely borked one of the above steps. You should try to enable the "Use Host GPU" setting for the avd as well, since this appears to drastically improve performance of the emulator for most MacBook Pros. Another potential way to use the kernel is to copy the kernel from `arch/arm/boot/zImage` into one of your platforms, this will cause all AVDs using that platform to use the customer kernel. The path for that is something like `$ANDROID_SDK_ROOT/system-images/android-17/armeabi-v7a/kernel-qem`. If you run into something like below (verbose output from emulator command, happen almost immediately);
```
emulator: Kernel parameters: qemu.gles=0 qemu=1 console=ttyS0 android.qemud=ttyS1 android.checkjni=1 ndns=3 
emulator: Trace file name is not set

emulator: autoconfig: -scale 0.821094
Segmentation fault: 11
```
This is likely from some weirdness in the avd from not shutting down properly (or just using it, the emulator is horrid). The easiest way to recover is just kill it (`rm -rf ~/.android/avd/*`) and recreate the avd.

**Making a LKM**
Shortly I'll post an example to github, but for now here is a very simple LKM that should compile fine. Makefile, should be fully complete, you may need to change the path of both `KERNEL_DIR` and `CCPATH`;
```
obj-m := android_module.o
KERNEL_DIR := ~/repo/android-kern/goldfish/
CCPATH := $(ANDROID_NDK_ROOT)/toolchains/arm-linux-androideabi-4.4.3/prebuilt/darwin-x86/bin/
CCPATH_EXT := $(CCPATH)arm-linux-androideabi-
EXTRA_CFLAGS=-fno-pic
ARCH=arm
SUBARCH=arm

all:
        make ARCH=arm CROSS_COMPILE=$(CCPATH_EXT) -C $(KERNEL_DIR) M=$(PWD) modules
clean:
        make -C $(KERNEL_DIR) M=$(PWD) clean
        rm -rf *.c~
        rm -rf *.o
        rm -f modules.order
```

The helloworld code for android_module;

```
#include <linux/module.h>
#include <linux/kernel.h>

int init_module(void)
{
  printk(KERN_ALERT "Hello android kernel...\\n");
  return 0;
}
void cleanup_module(void)
{
  printk(KERN_INFO "Goodbye android kernel...\\n");
}

MODULE_AUTHOR("Tim 'diff' Strazzere");
MODULE_LICENSE("GPL"); // This prevents a "module license 'unspecified' taints kernel." msg
MODULE_VERSION("1");
MODULE_DESCRIPTION("Ptracers gunna ptrace");
```

Dump both of these into any directory and run make after making the appropriate changes. You should then have an android_module.ko file. From here just push it to the emulator via adb, then use insmod/lsmod/rmmod as needed and enjoy. Depending on the time I have for the rest of the week, I'll try to dump some kernel modules I've written and used for malware analysis on my github.