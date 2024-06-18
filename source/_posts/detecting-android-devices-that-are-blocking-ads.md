---
title: Detecting Android devices that are blocking ads
tags:
  - /etc/hosts/
  - adblock
  - admob
  - ads
  - android
  - google
  - reverse engineering
url: detecting-android-devices-that-are-blocking-ads.html
id: 391
categories:
  - android
date: 2011-03-01 13:43:02
---

A friend of mine pointed me in the direction of an interesting app a few days ago. They had been using it for a while and just installed an ad-blocker, the type that just modifies your host file with a bunch of known ad providers. Then they're new favorite application stopped working, complaining that the ads where blocked. This may not have been the first attempt to detect ad removal, but it was the first one that I've looked at or even heard of. It's nothing fancy of course, but it's actually a nifty idea. The way it has been implemented there are plenty of ways to get around it, but it's definitely was a nifty piece of code. Quiet trivial to patch if someone wanted to, though I see no reason to not support the devs.

Anyway, I quickly just reversed the code, since most people don't care for reading the output of dexdump;
```
public static boolean isAdmobDisabled()
{
    if (admobDisabled == null)
      admobDisabled = Boolean.FALSE;
    try
    {
      FileReader localFileReader = new FileReader("/etc/hosts");
      BufferedReader localBufferedReader = new BufferedReader(localFileReader);
      String line;
      while (null != (line = localBufferedReader.readLine()))
      {
        if (!line.toLowerCase().contains("admob")) {
          admobDisabled = Boolean.TRUE;
          break;
        }
      }
    }
    catch (Exception e)
    {
      Log.d("Anti-adremoval", "Something bad happened!");
    }
  return admobDisabled;
}
```
 Maybe some other dev's will find this useful.
 
 UPDATE: It would appear the code I reversed was posted on [StackOverFlow](http://stackoverflow.com/questions/3452907/how-to-prevent-ad-blocker-from-blocking-ads-on-an-app/3453031#3453031). No sense in deleting this page though - next time I'll just have to Google a little better! :)