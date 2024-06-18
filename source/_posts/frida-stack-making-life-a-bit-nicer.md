---
title: frida-stack, making life a bit nicer while reversing unknown issues
tags:
  - typescript
  - frida
  - reverse-engineering
  - android
url: frida-stack-life-easier-unknown-issues.html
categories:
 - typescript
 - frida
 - reverse-engineering
 - android
date: 2024-06-18
---

## A long time ago...
In a contract, long forgotten at this point, I had to do quite a bit of reverse engineering against a few "[hardened](https://github.com/strazzere/android-unpacker)" targets. The targets generally all followed the same types of protection schemes, though they would roll out new versions weekly as well as toggle different anti-debug/reverse/etc style protections. The client didn't actually run most of the code, but they wanted a stable environment which could handle bringing up these different targets without needing to worry about what was going on under the hood.

Instead of approaching this as a static project like I normally had done in the past, I began to (at the time) begrudgingly utilize [frida](https://frida.re/). Like everything in the reverse engineering world, tooling is contentious. Whether you're asking people if they use `IDA` (IlFaK WoN'T aCcEpT mY CrEdIt CaRd), `ghirda` (It'S nOt r2) - mentioning usage of `frida` will illicit lots of reactions. Over the years, I've become a true believer though, once I figured out how to bend the tooling to do exactly what I wanted it too. Enter, hopefully what becomes a bit of a series, some of my tooling from long expired contracts that I've found helpful when approaching hardened targets.

## frida-stack

Whenever I approached a hardened target I found myself wanting to just get a lay of the land while I file up the bulk of my tooling. Get `IDA` to start processing any native code that looks interesting, disassemble everything with `baksmali` and fire up the application and attach to it with `frida` for a nice `repl` session.

Often when something is hardened even slightly, something will get detected as you begin poking around. Just having a `repl` session open often won't trigger these types of anti-debug actions, as it only has injected `frida` into the process space. Sometimes it will occur once you've begun hooking functionality.

With this in mind, I tend to just inject and hook my generic anti* utilities into the process. These often rely on hooking `libc` functionality at a low level, just to see when things get entered/exited, etc. One such example function I've found useful is to hook `_exit` from `libc` (often alongside with `abort`, `exit`, `kill`)

```typescript
function hook_exit() {
  const _exitPtr = Module.findExportByName('libc.so', '_exit');

  if (_exitPtr) {
    const _exit = new NativeFunction(_exitPtr, 'int', ['int']);

    Interceptor.replace(
      _exitPtr,
      new NativeCallback(
        function (status) {
          console.log(`[+] _exit : ${status} from ${this.context.pc)}`);
          return _exit(status);
        },
        'int',
        ['int'],
      ),
    );
  }
}
```

This can be thrown into a `repl` or script to inject pretty easily. Sometimes just injecting something like this will trigger different packers/protections to freak out. Granted, if they detect the injection, and they are _not_ utilizing `_exit` we won't see much other than the process die, which is why it can be sometimes nice to just hook a large variety of things wholesale and store those scripts for repeated usage. Though if they _do_ utilize `_exit` we would get some output like;

```sh
[Pixel 4::com.example.package ]-> [+] _exit : 0 from 0x7713d25000
```

Awesome! This is fun and useful because we now know where the `_exit` function was called from. Obviously, due to our code before, the process still was allowed to exit, which means we can't skim the `/proc/${pid}/maps` to see what that memory specifically was allocated too.

Normally in `frida` the usage pattern is to do something like the following to that address;

```typescript
const f = Module.getExportByName('libcommonCrypto.dylib',
    'CCCryptorCreate');
Interceptor.attach(f, {
  onEnter(args) {
    console.log('CCCryptorCreate called from:\n' +
        Thread.backtrace(this.context, Backtracer.ACCURATE)
        .map(DebugSymbol.fromAddress).join('\n') + '\n');
  }
});
```
(From the `Thread.backtrace` example on [frida.re/docs/javascript-api](https://frida.re/docs/javascript-api/))

This can work well, and when it does we can get a nice layout of which module the context came from. Though I've found, especially when digging into dynamically allocated code, this tends to fail. This brings me to the bulk of my code snippet;

```typescript
    /**
     * Return a decorated string, similar to DebugSymbol.fromAddress
     * and Process.getModuleFromAddress, however if those fail we
     * will forcefully look up the address association via the mappings.
     *
     * For some reason, DebugSymbol.fromAddress doesn't always work,
     * nor does Process.getModuleFromAddress, so utilize enumerating the
     * addresses manually to figure out what the module is and the local
     * offset inside it.
     *
     * @param address Address to look up details for.
     * @returns string of relevant data `0x7713d25000 libsharedlib.so:0x1aae8`
     */
    static getModuleInfo(address: NativePointer) {
      const debugSymbol = DebugSymbol.fromAddress(address);

      if (debugSymbol.moduleName) {
        // Add local offset?
        return debugSymbol.toString();
      }

      // When hooking we might get something interesting like the following;
      //  [
      //    {
      //      "base": "0x76fa7000",    <==== [anon:dalvik-free list large object space]
      //      "protection": "rw-",           we don't actually care about this
      //      "size": 536870912
      //    },
      //    {
      //      "base": "0x771e939000", <==== this isn't the actual base, we need to refind that
      //      "file": {
      //        "offset": 663552,
      //         "path": "/apex/com.android.runtime/lib64/bionic/libc.so",
      //         "size": 0
      //      },
      //     "protection": "rwx",
      //     "size": 4096
      //   }
      // ]

      const builtSymbol = {
        base: ptr(0x0),
        moduleName: '',
        path: '',
        size: 0,
      };

      let ranges = Process.enumerateRanges('').filter(
        (range) => range.base <= address && range.base.add(range.size) >= address,
      );

      ranges.forEach((range) => {
        if (range.file) {
          builtSymbol.path = range.file.path;
          const moduleNameChunks = range.file.path.split('/');
          builtSymbol.moduleName = moduleNameChunks[moduleNameChunks.length - 1];

          builtSymbol.base = range.base.sub(range.file.offset);
        }
      });

      ranges = Process.enumerateRanges('').filter(
        (range) => range.base <= builtSymbol.base && range.base.add(range.size) >= builtSymbol.base,
      );

      ranges.forEach((range) => {
        if (builtSymbol.base === ptr(0x0) || builtSymbol.base < range.base) {
          builtSymbol.base = range.base;
        }
        builtSymbol.size += range.size;
      });

      return `${builtSymbol.base} ${builtSymbol.moduleName}:${address.sub(builtSymbol.base)}`;
    }
```
(Source [https://github.com/rednaga/frida-stack/blob/main/lib/index.ts#L68-L128](https://github.com/rednaga/frida-stack/blob/main/lib/index.ts#L68-L128))

If the `DebugSymbol.fromAddress` works, awesome, let's use it. Otherwise, let's seek the `Process` ranges and look for what is _currently_ sitting there and decorate a string with exactly that.

Utilizing the above code we can now transform the original example into something like this;

```typescripts
[Pixel 4::com.example.package ]-> [+] _exit : 0 from 0x7713d25000 libexamplesharedlib.so:0x1aae8
```

This means we can see the original address, which often is pretty useless, but more importantly any process/modules names as well as the offset from the base. This ends up being much more functional than just the random address in a now dead process - and is something actionable we can toss into our disassembler of choice and begin to start recursing backwards to find who else may call these functions.

### Extended usage

Obviously it's fair game to just take the snippet above, however you can also install this snippet as a module for usage within your own scripts.

Simply install it by utilizing npm;

```sh
# npm install frida-stack
```

Then import it and you will get the function statically accessible, or you can utilize the two helper functions to generate stacks whenever you'd like. One is a method of just getting your place/stack inside the `Java` context via `Stack.java()`. The other you will need to pass a `NativePointer` to `Stack.native(this.context.pc)`.

So to tune the script above we used as an example;

```typescript
import { Stack } from 'frida-stack'

function hook_exit() {
  const _exitPtr = Module.findExportByName('libc.so', '_exit');

  if (_exitPtr) {
    const _exit = new NativeFunction(_exitPtr, 'int', ['int']);

    Interceptor.replace(
      _exitPtr,
      new NativeCallback(
        function (status) {
          console.log(`[+] _exit : ${status} from ${Stack.getModuleInfo(this.context.pc)}`);
	        console.log(Stack.native(this.context)
          return _exit(status);
        },
        'int',
        ['int'],
      ),
    );
  }
}
```

Hopefully this saves others some time while reversing things.