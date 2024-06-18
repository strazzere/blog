---
title: Beaten to the punch line...
tags:
  - android
  - beat to the punch
  - busybox
  - g1
  - root
  - SMS
  - sms backup
  - su
url: beaten-to-the-punch-line.html
id: 111
categories:
  - android
  - other
date: 2008-11-24 16:21:31
---

Well I was about half way done with my SMS application when I came upon this -- looks like a much easier solution (for me, with a rooted phone atleast) than to just make a whole new application to do it all. This one is from ultimind over at the [xda-developers forums](http://forum.xda-developers.com/showthread.php?t=448361);
```
> Playing around with the ls -R command, I found where the SMS database is kept, and it's somewhat readable in a text editor... _UPDATE (thanks staulkor): This database is viewable, and searchable using an SQLite database viewer._
> 
> > **Code:** /data/data/com.android.providers/telephony/databases/mmssms.db
> 
> Just run the following command to back it up to the SD Card:
> 
> > **Code:** busybox cp /data/data/com.android.providers.telephony/databases/mmssms.db /sdcard
> 
> Happy hacking
```

Obviously you need your phone rooted, and busybox installed - though for most that shouldn't be a huge problem. For now this suits my needs so the SMS project will most likely be put on hold until I have more time to finish it.