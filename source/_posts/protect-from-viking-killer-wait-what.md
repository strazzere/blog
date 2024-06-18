---
title: 'Protect from viking killer! Wait, what?'
tags:
  - android
  - AOSP
  - comment humor
  - google
  - humor
  - javadoc
  - viking
  - viking killer
  - 'XXX: PROTECT FROM VIKING KILLER'
url: protect-from-viking-killer-wait-what.html
id: 357
categories:
  - android
  - other
date: 2010-05-10 13:05:11
---

Much similar to a previous post I had, [“Brutal” Google coding humor…](http://strazzere.com/blog/?p=325), I was perusing over some code in the AOSP and found an interesting comment:
```
// XXX: PROTECT FROM VIKING KILLER
```
Below is the full snippet from the file, `logwrapper.c`.
```
void child(int argc, char* argv[]) {
    // create null terminated argv_child array
    char* argv_child[argc + 1];
    memcpy(argv_child, argv, argc * sizeof(char *));
    argv_child[argc] = NULL;

    // XXX: PROTECT FROM VIKING KILLER
    if (execvp(argv_child[0], argv_child)) {
        LOG(LOG_ERROR, "logwrapper",
            "executing %s failed: %s", argv_child[0], strerror(errno));
        exit(-1);
    }
}
```
I'm not sure if "Viking Killer" is an inside joke, an actual good comment or what. Though read allowed - it sounds like a badly named european adult film...