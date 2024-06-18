---
title: Uncertified Devices and Bad Advice
tags:
 - android
 - emulator
 - reverse-engineering
categories:
 - android
 - reverse engineering
 - emulator
date: 2020-06-07
---

## Emulators with official apps

Nothing new about folks wanting to have emulators which look like real devices. This has been a want that has been around for a while, and a wildly helpful project is the [OpenGapps Project](https://opengapps.org/) which will build these apps for you and assist in getting the installed and set up. Though upon doing this, you'll often see some crashes and the following screens;

![Annoying](/images/action-required.png)

Annoying, right? Of course this isn't certified, but whatever - let's just bypass this by registering our device at the link provided [https://www.google.com/android/uncertified/](https://www.google.com/android/uncertified/) which looks like the following;
![Seems reasonable](/images/directions.png)

## Adding a Uncertified Device to an exception list

The wordage on this page recently changed, it appears about a month ago, to now have those explicit instructions. Looking at them, it's unclear to me if these could ever work on a device;

```
$ adb root
$ adb shell 'sqlite3 /data/data/com.google.android.gsf/databases/gservices.db \ 
    "select * from main where name = "android_id";"'
```

Most custom roms aren't just plain emulators with root enabled, this won't work at all, as you don't inherently have `adb` with `root` access. Whatever, likely just a silly assumption, and since we are using an emulator with root we don't really care right now. The latter part is more interesting and confusing. If we reduce it to one line can you see what's likely going to (not) happen?

`adb shell 'sqlite3 /data/data/com.google.android.gsf/databases/gservices.db "select * from main where name = "android_id";"'`

Ok, so, your shell is going to run `adb shell 'pipe whatever this command is to the devices shell'`, right?

Then the `sqlite3 /data/data/com.google.android.gsf/databases/gservices.db "select * from main where name = "android_id";"` part is going to run the `sqlite3` command pass in an argument for the file, `/data/data/com.google.android.gsf/databases/gservices.db` and then a query, quoted so that it won't be parsed as multiple arguments `"select * from main where name = "android_id";"`.

Except that last bit, seems horribly wrong? The output from running this on a (relatively) stock emulator will net the following;

```
adb shell 'sqlite3 /data/data/com.google.android.gsf/databases/gservices.db "select * from main where name = "android_id";"'
Error: no such column: android_id
```

Note the error, `no such column: android_id`, even though that SQL is not attempting to use `android_id` as a column, this is meant to be the key, `name` is the column we're meant to be pivoting on.

## Simple fix

So the fix should be obvious, escape the key name and keep the SQL from getting broken up

```
adb shell 'sqlite3 /data/data/com.google.android.gsf/databases/gservices.db "select * from main where name = \"android_id\";"'
android_id|437766DEADBEEF7953
```

Nothing overly complicated, obviously, though it can be annoying when good instructions/directions are changed to things that won't work out of the box. Maybe more proof reading next time, though not like I do that, this post is likely riddled with errors or maybe this only affected my own set ups? Oh well I guess?