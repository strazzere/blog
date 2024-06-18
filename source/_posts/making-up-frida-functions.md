---
title: '"Making up" functions in Frida'
tags:
 - android
 - frida
 - reverse engineering
categories:
 - reverse engineering
 - android
date: 2020-05-08
---

Recently I had the need to generically hook some functions which I didn't actually want a target binary to load. You can think of this as, a shared library which was expecting another shared library to also exist, then utilize some functions from it. The amount of functions it was going to use, approximately ~1k, would have been really annoying for me to manually stub out.

Anyway, regardless of the purpose, I ran into a need to "create" a "new" function so that I could hook it or modify it. Originally I expected that I might just be able to use an allocated blob of memory, which would return the type of `NativePointer`, however this is not a "hookable" address in memory it would appear.

This simple solution to this was to allocate memory, then used `MemoryPatcher`;

```
// Allocate memory inside the 
var stubFunc = Memory.alloc(Process.pageSize);

// Patch the "function" with some basic instructions
Memory.patchCode(stubFunc, Process.pageSize, function (code) {
    if(Process.arch.includes("arm64")) {
        var cw = new Arm64Writer(code, { pc: stubFunc });
        cw.putLabel('done');
        cw.putRet();
        cw.flush();
    } else {
        var cw = new ArmWriter(code, { pc: stubFunc });
        cw.putLabel('done');
        cw.putRet();
        cw.flush();
    }
});
```

By doing the above, we're giving both Frida and the targetted application a "function" which can now be called and does nothing. So, in theory you could modify the above to return a value, do actual work or whatever in arm/arm64 code, or just pass the pointer back to the application and hook it in Frida like the following;

```
Interceptor.attach(stubFunc, {
    onEnter: function(args) {
        console.log(" >> stubFunc onEnter");
        // You can access the args being sent here from the targetted application
     },
    onLeave: function(retval) {
        // Or do work here and return whatever value
        retval.replace(0xd1ff)
        console.log(" >> stubFunc onLeave : ", retval);
    }
});
```

Using this pattern, we can now mock out different non-existant or blocked libraries. This can be useful for removing obfuscated, protected or just annoying code.