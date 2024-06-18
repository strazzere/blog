---
title: 'Detecting an emulator, and getting around detection'
tags:
  - android
  - Android_ID
  - android.provider.system.settings
  - emulator
  - emulator detection
  - reverse engineering
  - secure.android_id
  - spoof id
  - spoofing android_id
url: detecting-an-emulator-and-getting-around-them-it.html
id: 379
categories:
  - android
  - reverse engineering
  - secure code
date: 2010-09-23 15:10:40
---

So recently I got an email asking about spoofing your Android ID, normally I just redirect people to my other articles, but I guess this one was different. The person specifically was looking to test applications on their emulator, and needed to avoid "emulator detection". It didn't seem like anything tricky - but I googled quick with not many results returned. The things that where returned where pretty... Well - overcomplicated. My first thought was that this should just be located in one of the many sqlite3 db's - and it turns out it was. Simple little work around yet again, have to love those!

First off, lets see how most applications detect emulators. It's done using a simple call to _Secure.getString()_ using _Secure.ANDROID_ID_ (previously this was done using _System.getString()_ and _System.ANDROID_ID_, though these methods are now deprecated). The "detection" often relies on this returning _null_ when it is read inside an emulator. With this said, below is a normal emulator detection snippet;
```
String android_id = Secure.getString(getContentResolver(), Secure.ANDROID_ID);
if (android_id == null) {
  // This was run inside an emulator, don't let the user continue!
} else {
  // Oh ok, everythings normal, lets keep going
}
```
Very simple, doesn't do anything to complicated, just checks for _null_ being returned. So how can we get this to not return `null` in an emulator? Easy -- just a simple sqlite3 insert command!

Open a shell to your emulator, and navigate to `/data/data/com.android.providers.settings/databases/`, open up the db in sqlite3 and insert a value using `androidid` as the key, like the following;
```
# cd /data/data/com.android.providers.settings/databases
# sqlite3 settings.db
SQLite version 3.6.22
Enter ".help" for instructions
Enter SQL statements terminated with a ";"
sqlite> insert into secure ('name', 'value' ) values ('android_id','deadbeef4badcafe');
sqlite> .exit
```
That's all you need to do. Now all programs that use that method for detecting if it is an emulator will be returned the value you've entered as the `android_id`.