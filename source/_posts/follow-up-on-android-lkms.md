---
title: Follow up on Android LKMs
url: 735.html
id: 735
categories:
  - android
  - coding
date: 2014-07-24 21:30:35
tags:
---

As promised, I’ve posted a few Android LKMs over on github just now. Hopefully as time allows I’ll be able to commit more of my LKMs, however for the time being only two are ready to see the light of day.

One of them is a simple anti-ptracer LKM which is mainly grafted from old code I remember seeing in a zine somewhere, at some point in time. It’s very old at this point but was useful in dealing with something that using ptrace in an odd fashion. The bulk of this is dropped into this directory [https://github.com/strazzere/android-lkms/tree/master/antiptrace](https://github.com/strazzere/android-lkms/tree/master/antiptrace) \- hopefully with just a little modification to the Makefile, you can have it compiling in no time.

The second was an attempt to do `LD_PRELOAD` like hooking just to test the results. To be blunt, the results where not very good and it was a bad idea to do this many things in the kernel as it slowed the emulator down quiet a bit. Though, I figured it might be useful for someone to view and get some ideas. I was targeting a specific process and while the end result did not end up being that great - it did help me realize what files where being touched and when in the set up process. The bulk of this code is found at [https://github.com/strazzere/android-lkms/tree/master/open-read-write](https://github.com/strazzere/android-lkms/tree/master/open-read-write).

None of this is ground breaking or relatively new. However, in trying to get these example to work a while back, I found most of the articles online about LKMs to be great for the theory but contains unusable code and instructions. Hopefully these examples can give people a more clear indication of specifically how to get LKMs working for Android emulators.
On a bit of a specific note, lots of LKM chatter is about rootkiting devices and how to get the `SYS_CALL_TABLE`. I’ve avoid that as I’m working under the assumption that I’ve compiled the kernel myself and knowingly have root access to enable these LKMs. For this purpose I use a simple sed script to grab the address and inject it into the header files. The other articles online have plenty of suggestions on how to do this dynamically, most still work - I just didn’t care to have that bloat in my code as it was unnecessary for my setup.

Originally posted as a comment to this post, pre-migration away from WordPress, Collin Mulliner mentioned, "this is what I use, not too much bloat 🙂"; 

```
static unsigned long **get_sys_call_table(void) {
#ifdef CONFIG_MIPS
  return 0x8006a770;
#else
  unsigned long int offset = PAGE_OFFSET;
  unsigned long **sct;

  while (offset < ULLONG_MAX) {
    sct = (unsigned long **)offset;
    if (sct[__NR_close] == (unsigned long *) sys_close)
      return sct;
    offset += sizeof(void *);
  }
  return NULL;
#endif
}
```