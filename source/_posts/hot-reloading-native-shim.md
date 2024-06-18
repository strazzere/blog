---
title: Hot reloading to fix LD_LIBRARY_PATH issues
tags:
  - android
  - native-shim
  - reverse engineering
url: hot-reloading-native-shim.html
categories:
 - android
 - reverse engineering
 - native-shim
 - shim
date: 2022-11-24
---

## Where is my libart at?

Since approximately Android 10, `libart` ([Android Runtime](https://source.android.com/docs/core/runtime)) has been moved to the `apex` ([Android Pony EXpress](https://source.android.com/docs/core/ota/apex)) directory. This was a design decision by Google which allowed them to perform updates to the DVM/ART subsystem without requiring an OEM to push a full update -- and leveraging Google Play to push down smaller packages. This is a win for users, the ecosystem, etc - for both speed, security. Though this sometimes presents an issue for non-standard usecases. One of which is some long standing code I use often for debugging and attacking different applications, the [native-shim](https://github.com/rednaga/native-shim). While it isn't anything super special, mostly just some boilerplate code and some `dlopen` calls, it's an approach I've utilized for almost 10 years.

### What broke due to APEX?

Nothing really "broke", just we no longer know where to look for `libart`. If we dig into the [linker code](https://cs.android.com/android/platform/superproject/+/master:bionic/linker/linker.cpp) we will see that the default paths generally look like we would expect;

```
#if defined(__LP64__)
static const char* const kSystemLibDir        = "/system/lib64";
static const char* const kOdmLibDir           = "/odm/lib64";
static const char* const kVendorLibDir        = "/vendor/lib64";
static const char* const kAsanSystemLibDir    = "/data/asan/system/lib64";
static const char* const kAsanOdmLibDir       = "/data/asan/odm/lib64";
static const char* const kAsanVendorLibDir    = "/data/asan/vendor/lib64";
#else
static const char* const kSystemLibDir        = "/system/lib";
static const char* const kOdmLibDir           = "/odm/lib";
static const char* const kVendorLibDir        = "/vendor/lib";
static const char* const kAsanSystemLibDir    = "/data/asan/system/lib";
static const char* const kAsanOdmLibDir       = "/data/asan/odm/lib";
static const char* const kAsanVendorLibDir    = "/data/asan/vendor/lib";
#endif

...

static const char* const kDefaultLdPaths[] = {
  kSystemLibDir,
  kOdmLibDir,
  kVendorLibDir,
  nullptr
};
```

This is generally speaking, where we would expect to find all the libraries needed, so utilizing `dlopen("libart")` seemingly would continue to work. Except now we can see that this is no longer where it is held;

```
beyond1q:/ # find / -name libart.so
/system/apex/com.android.runtime.release/lib/libart.so
/system/apex/com.android.runtime.release/lib64/libart.so
```

### Fixing our shim

The dead simple fix would be to just ensure the correct path, depending on if the binary is 64 bit or not, is added to our `LD_LIBRARY_PATH`.

```
beyond1q:/data/local/tmp # LD_LIBRARY_PATH=/system/apex/com.android.runtime.release/lib64/ ./shim ./libHelper.so
```

While the above would suffice, it's not exactly great. I'm pretty lazy and will easily forget this the next time around. So let's make this programmatic which brings us to another minor issue. Prior to exploring this problem, I wasn't positive you could change the `LD_LIBRARY_PATH` during execution. Technically, you can in a few different hacky ways, but we will actually opt for not modifying it on the fly, but just setting the variable if needed and re-executing ourselves correctly.

```
#if __aarch64__
#define APEX_LIBRARY_PATH "/apex/com.android.runtime/lib64/"
#else
#define APEX_LIBRARY_PATH "/apex/com.android.runtime/lib/"
#endif

...

  char* path = getenv("LD_LIBRARY_PATH");
  if(path == NULL || strstr(path, "apex") == NULL) {
    if(setenv("LD_LIBRARY_PATH", APEX_LIBRARY_PATH, 1) != 0) {
      printf(" [!] Error setting LD_LIBRARY_PATH via setenv!");
      return -1;
    } else {
      printf(" [+] Success setting env - reloading shim\n");
      int rc = execv( "/proc/self/exe", argv);
    }
  }
```

The above snippet allows us to properly detect which path would be required to find `libart`, then looks to see if our path contains this already. If not, we set the environment we are currently running in and then re-execute ourselves via `execv` and retain the original arugments. This will transparently allow all `dlopen`, `dlsym`, etc, calls to work as originally intended and have the extra paths we need. This style of code isn't anything ground shaking, however it took me a while to both track down the issue and figure out a programmatic way of tackling it.

The full commit can be found [here](https://github.com/rednaga/native-shim/commit/c6e66d65f75f0febdb0b56f23fd29ea1aa83d6e8) for building locally.

Now viola! We can continue running the `native-shim` as we always have. Have fun doing more unpacking on newer systems.