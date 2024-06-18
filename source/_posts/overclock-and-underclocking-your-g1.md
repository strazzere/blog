---
title: Overclock and Underclocking your G1
tags:
  - android os
  - clock speed
  - cpu
  - g1
  - overclock
  - system settings
  - tmobile
  - underclock
url: overclock-and-underclocking-your-g1.html
id: 232
categories:
  - android
date: 2009-04-09 09:22:02
---

Just yesterday I noticed there was an application published to overclock your G1 -- it reminded me of a few files that I had seen when browsing system files on my device. At first I thought it must be a scam, or using some feature just hidden into the system, much like the garbage collection method used in memory free pro - but it seems t actually change the value. Whether this really speeds up the G1 without harming it is left unknown right now. Anyway, the file I had previous seen and was watching was `acpu_clk`. It can be found in the `/system/debugfs/clk_rate/` folder. So, essentially to get your current CPU do the following in the terminal or through adb;
```
$sh #cat /system/debugfs/clk_rate/acpu_clk
```
To modify this file to change your clock speed you need to remount your /system to be writeable and then modify the file.
```
>mount -o rw,remount -t yaffs2 /dev/block/mtdblock3 /system
mkdir /system/debugfs
mount /system/debugfs /system/debugfs -t debugfs
```
This will remount debugfs, the folder to be visible and allow you to edit it. To change the value simple pipe and echo to the file;
```
echo '245760000' > /system/debugfs/clk_rate/acpu_clk
```
_**Afternote:** Just noticed that someone at xda-developers posted nicer shell code than mine, so I've just pasted in theirs. Enjoy!_