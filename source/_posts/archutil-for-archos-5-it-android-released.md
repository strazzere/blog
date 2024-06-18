---
title: Archutil for Archos 5 IT Android Released
tags:
  - a5it
  - aes key
  - android
  - aos
  - aos decrypt
  - aos file
  - archos
  - archos 5 android
  - archos 5 it
  - archutil
  - archutil-a5it
  - bootloader
  - devmpk
  - gamesmpk
  - hddmpk
  - plugmpk
  - relmpk
  - root
url: archutil-for-archos-5-it-android-released.html
id: 311
categories:
  - android
date: 2009-10-28 10:22:56
---

![Look ma, keys!](http://173.230.150.16/blog/wp-content/uploads/2009/10/keys1-278x300.png "Oh, there you are keys!")
A little while back on a forum CheBuzz was kind enough to post ArchUtil publicly. Below is a copy of the post:

>archutil download: [http://download.openpma.org/archutil/archutil.tbz](http://download.openpma.org/archutil/archutil.tbz) Well, since everything has been active on the Archos hacking front, I think I will take this opportunity to release archutil. I have been working on this utility for a while and I consider it to be mostly bug free. Please leave me a note if you find anything that doesn't work as advertised.
>
>This will allow you to decrypt nearly all AOS2 files. There is no documentation besides the code and the built-in help. If you have any questions, I'd be happy to answer them. It also verifies different signatures and will tell you what key was used to verify it. Code is also included to sign files with a built-in private key, or with a private key passed on the command-line. And finally, functionality is included to sign a firmware update file with your own private key.
>
>Most of this has been initially tested. None of it has been thoroughly tested. Again, feel free to let me know of any bugs that you find.
>
>_And let me just get the first question out of the way: it will not work for A5IT files. It probably would if somebody could find its keys.
>
>Now let me get the second question out of the way: this is not useful for hacking your Archos unless you know their private key. And let's not waste time talking about brute-forcing the key, shall we? It's just not feasible at this time._


Thankfully he released this, because it made it so much more simple to use the keys we found shortly after in the Archos 5 IT Android. So after some work by [EiNSTeiN](http://archos.g3nius.org/) and myself we where able to extract the keys from the flash of the device and plunk them into CheBuzz's utility. 

Anyway, in case this of any use to anyone, you can download the sources for the util (packaged now as archutil-a5it) you can download them here: [http://www.strazzere.com/android/archos/archutil-a5it.rar](http://www.strazzere.com/android/archos/archutil-a5it.rar)

This will help you unpack aos files. You can pack aos files too -- though they will not work on the Archos 5 IT Android since we do not have the private key for it. The keys that have been included as the AES key, Bootloader, RelMPK, DevMPK, PlugMPK, HDDMPK and GamesMPK.

Enjoy!