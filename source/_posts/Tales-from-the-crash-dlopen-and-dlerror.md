---
title: 'Tales from the crash; dlopen() and dlerror() patterns'
tags:
 - android
 - frida
 - reverse-engineering
categories:
 - android
 - reverse engineering
 - coding
date: 2020-05-17
---

## The setup

Recently, while attacking a unspecified application, I had the need to do something I consider pretty normal. Hook `dlopen()` and `dlsym()` and inject some results for instrumenting what the application believes should be going on. Annoyingly, the way the application worked was to load an annoying amount of shared libraries. The majority of these libraries had been protected with some unknown obfuscators and a packer -- so replicating the environment in isolation would be very tedious. My normal pattern in a simplistic case would be just to use the [native-shim](https://github.com/rednaga/native-shim) -- it's easy, works and gives you a clean "JVM" to work with. Though due to all the re-injections and checks inside protected binaries, the JVM would need to be prepopulated with too much dalvik non-sense. So the obvious solution? Use [Frida](frida.re) (or some other generic hooking framework I guess?)!

Very simplistically, I needed to hook `dlopen()` to watch for a specific shared library, which after some static analysis, appeared to only introduce anti-debugging tricks. After the `dlopen()` is called, the main loader would then `dlsym()` some functions and check to see if they exist, call then and check the return values. So basically, the underlying deobfuscated code likely looked like this;

```
bool beingDebuged() {
    void *handle;
    bool (* antiFunction1)(void);
    bool (* antiFunction2)(void);
    bool (* antiFunction3)(void);
    bool (* antiFunction4)(void);

    handle = dlopen ("libantidebugstuff.so", RTLD_LAZY);
    if (!handle) {
        fputs ("Error initializing antidebug!", stderr);

        // library seemed to assume if antidebugging fails to load,
        // that it's probably being debugged
        return true;
    }

    antiFunction1 = dlsym(handle, "antiFunction1");
    if (antiFunction1 == NULL)  {
        fputs("Error getting antiFunction1!", stderr);

        // Same as above
        return true;
    }

    // .. get other functions, etc

    return ((*antiFunction1)() && 
        (*antiFunction2)() && 
        (*antiFunction3)() && 
        (*antiFunction4)())
}
```

The above is not the actual code -- it was more complex and interweaved throughout the application making it a bit more annoying to solve holistically. If it where the above code, we could just replace the whole bit with a simple NOP function or something. We also had a secondary goal of figuring out where/when/why these calls had been getting triggered.

In the simplified above example though, can you see what might go wrong with this code? There is an edge case here which is sort of an odd one, that I can only really imagine happening when reverse engineering. Though, dependant on how this code interacts, could potentially reproduce if this code essential for the application to run and fails silently.

## There was an attempt...

So without much thought I simply just sketched out some a simplistic frida function hook per usual and injected it. A gadget was injected into the APK and a handful of files had been stripped out, different ABIs and the antidebug library since we didn't want it mucking around with any of the memory. If they wanted to open the targeted antidebug library, we would just return a fake handle so it won't error out, then catch that fake handle and return my [fake functions](https://strazzere.com/2020/05/making-up-frida-functions/) for the application to call. That could let us trap the functions or get stacktraces to see where/when it was being utilized;

```
var cookie = 0xd1ffd1ff;

Interceptor.attach(Module.findExportByName(null, 'dlopen'), {
    onEnter: function(args) {
        this.path = Memory.readUtf8String(args[0]);
        this.mode = args[1]);
     },
    onLeave: function(retval) {
        console.log("dlopen(" + "path=\"" + Memory.readUtf8String(path) + "\"" + ", mode=" + mode + ")");
        if (name !== null && name.endsWith('libantidebugstuff.so')) {
            retval.replace(cookie)
        }
        return retval;
    }
});

Interceptor.attach(Module.findExportByName(null, 'dlsym'), {
    onEnter: function(args) {
        this.handle = args[0];
        this.name = Memory.readUtf8String(args[1]);
     },
    onLeave: function(retval) {
        if(this.handle == cookie) {
            console.log("dlsym(" + "handle=\"" + this.handle + "\"" + ", name=" + this.name + ") ret : ", retval);
            switch(this.name) {
                case 'antiFunction1':
                    retval.replace(antiFunction1);
                    break;
                case 'antiFunction2':
                    retval.replace(antiFunction2);
                    break;
                case 'antiFunction3':
                    retval.replace(antiFunction3);
                    break;
                case 'antiFunction4':
                    retval.replace(antiFunction4);
                    break;
                default:
                    console.log("Don't know how to handle requested function : ", this.name);
            }
        }
    }
});
```

Fire it up - worked fine. Tweaked a few things, it crashes. Wait, what? NPE in some unrelated library for the system? This didn't make sense. Fire it up again with changes removed, breaks again. Except the NPE is somewhere else. What. The. Crap.

Figured it out yet? It took me a while to understand two things. The original code I was injecting against has a bad a pattern in it (which seems surprisingly common?) and I also chose a poor way to interact with this function.

## The bad pattern

So, per `dlopens())`, the return value will be `NULL` if it fails;
```
       On  success,  dlopen() and dlmopen() return a non-NULL handle for the loaded library.  On error (file could not be found, was not readable,
       had the wrong format, or caused errors during loading), these functions return NULL.

       On success, dlclose() returns 0; on error, it returns a nonzero value.

       Errors from these functions can be diagnosed using dlerror(3).
```

This makes sense, and it looks like both the original author and I are covering for this case.

```
    handle = dlopen ("libantidebugstuff.so", RTLD_LAZY);
    if (!handle) {
```

The above code will error if `NULL`, perfect.

```
Interceptor.attach(Module.findExportByName(null, 'dlopen'), {
    onEnter: function(args) {
        this.path = Memory.readUtf8String(args[0]);
        this.mode = args[1]);
     },
    onLeave: function(retval) {
        console.log("dlopen(" + "path=\"" + Memory.readUtf8String(path) + "\"" + ", mode=" + mode + ")");
        if (name !== null && name.endsWith('libantidebugstuff.so')) {
            retval.replace(cookie)
        }
        return retval;
    }
});
```

In the interceptor code, we will catch the `retval`, which will be `NULL` as we've removed the file and it won't be found. We then change the `retval` to not be `NULL` and the program will be happen and continue it's execution. Everyone is great, right? Nope, we made a decently sized mistake in some assuming here...

It would appear, that we spaced out about `dlerror()`, as did the original author (maybe?). The man page of `dlopen` even told us about it above;

`Errors from these functions can be diagnosed using dlerror(3).`

Oh yea, if we let the `dlopen` function actually get called, a `dlerror()` will be set, and remain until someone asks for it. This didn't cross my mind originally as the original author wasn't using it, so big deal, right? Sadly, no - this was the whole source of the error. By utilizing `Interceptor.attach()` we're merely hooking _before_ and _after_ the function call. As `libantidebugstuff.so` no longer exists, this means the `dlerror()` call will return a file not found error. Since there was no gating used done by the original author or us based on a `dlerror()` this will just persist until someone asks for it, wrongly consuming this error as their own. Thus downstream issues which libraries never expected.

## Fixing the issue

So for our intercepting of the functions to be fixed, we need to replace our `attach()` call with a `replace()` call, then we can choose if we want to call the true underlying function. Alternatively you could consume the `dlerror()` instead, but this feels "wrong" to me.

```
var cookie = 0xd1ffd1ff;
var dlopen = new NativeFunction(Module.findExportByName(null, 'dlopen'), 'pointer', ['pointer', 'int']);
Interceptor.replace(dlopen, new NativeCallback(function(path, mode) {
    console.log("dlopen(" + "path=\"" + Memory.readUtf8String(path) + "\"" + ", mode=" + mode + ")");
    var name = Memory.readUtf8String(path);
    if (name !== null && name.endsWith('libantidebugstuff.so')) {
        console.log("[*] Found the call to libantidebugstuff, replacing with our cookie and not calling dlopen");
        return new NativePointer(cookie);
    }
    return dlopen(path, mode);
}, 'pointer', ['pointer', 'int']));
```

After replacing the original `dlopen()` hook with the above code, it all worked fine and the random NPEs which had been racing each other, all disappeared.

This sort of had me thinking about `dlerror()` though and the "proper usage" of it. Generally speaking, I guess most usecases would be sort of fine to ignore this function and just rely on the `NULL` being returned. Though this seems like it could cause poor results in a complex binary _especially if_ a failed `dlopen()` in your application is considered not fatal. For this targeting application, it just considered any type of failure to be that the application is being debugged. However, this wouldn't cause the application to exit by design, though it would crash as other parts of the system pick up the `dlerror()`. So in this case, maybe no one would care?

Regardless, it would seem the "safest" form of usage for a `dlopen()` or `dlsym()` call that is allowed to fail would be something like;

```
    char *error;

    handle = dlopen ("libantidebugstuff.so", RTLD_LAZY);
    if ((error = dlerror()) != NULL) {
        fputs (error, stderr);

        // library seemed to assume if antidebugging fails to load,
        // that it's probably being debugged
        return true;
    }

    antiFunction1 = dlsym(handle, "antiFunction1");
    if ((error = dlerror()) != NULL)  {
        fputs(error, stderr);

        // Same as above
        return true;
    }
```

The above snippet will properly gate in the a similar manner as before, but also prevent an issue where `dlerror()` could return a value to another function.

## TLDR

Don't let functions you know will fail, fail, when reversing. Seems obvious, though sometime when you're reversing in smaller target it doesn't matter. Hopefully this is mildly interesting to folks as it annoyed me for a day or so trying to figure out what was going on.