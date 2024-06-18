---
title: Android USB Accessory Sniffing
tags:
  - android
  - usb accessories
  - frida
  - typescript
  - reverse engineering
url: android-usb-accessories-and-frida.html
categories:
 - android
 - reverse engineering
 - frida
date: 2022-09-27
---

## Using a Typescript Frida-gadget for usb debugging

Recently on a project I was looking to do some "low level" analysis of between an application and a usb device plugged directly into the phone. This wasn't anything I had ever actually done before, so it took a little bit of digging to figure out what was actually needed. On top of that, it had been a little bit since I had written a `frida` gadget. For the heck of it, I decided to just try and write everything in Typescript. Mostly this was a challenge to myself, but also an attempt to see if this would speed up any gadget writing in the future since it would allow me to rely on strong typing --- or so I hoped.

### Boilerplate gadget code

[oleavr](https://github.com/oleavr), as always, was kind enough to have a good example of what a gadget should look like for `frida` which my code is based off of at [frida-agent-example](https://github.com/oleavr/frida-agent-example). The main benefit I've found to utilizing this approach is being able to have `VSCode` open on your project, the `watch` command running, and the interact repl for your agent also running. Between the typescript bindings and the repl, you can quickly debug and test while just saving a file and having everything recompile at once.

### Finding the Usb Device

While splunking in the Android docs, we can find that [android.hardware.usb.UsbAccessory.openAccessory()](https://developer.android.com/reference/android/hardware/usb/UsbManager#openAccessory(android.hardware.usb.UsbAccessory)) is the likely method we will want to hook. Specifically from the docs;

```
public ParcelFileDescriptor openAccessory (UsbAccessory accessory)

Opens a file descriptor for reading and writing data to the USB accessory.
```

So we will want to hook this, intercept the return value of type `ParcelFileDescriptor` then utilize the [getFd()](https://developer.android.com/reference/android/os/ParcelFileDescriptor#getFd()) method. This means, based on the docs, from a hooking/app perspective all the USB interaction is occuring transparently over a file descriptor. Great - so much like any type of file descriptor hooking, we only really care now about a few things;

1. What is the file descriptor I care about
2. Hook the `read()` and `write()` methods and filter by the file descriptor.

To perform this hooking in the gadget, we will use essentially the code below;

```
  Java.perform(() => {
    const usbManager = Java.use('android.hardware.usb.UsbManager')
    usbManager.openAccessory.overload('android.hardware.usb.UsbAccessory').implementation = (usbAccessory: NativePointerValue) => {
      if (debug) {
        log(`usbManager.openAccessory(android.hardware.usb.UsbAccessory) called`)
      }
      const ret = usbManager.openAccessory.call(this, usbAccessory)
      const parcelFileDescriptor = Java.cast(ret, Java.use('android.os.ParcelFileDescriptor'))
      
      log(`***** usb accessory FD : ${parcelFileDescriptor} ${parcelFileDescriptor.getFd()}`)
      accessoryFD = parcelFileDescriptor.getFd()
      return parcelFileDescriptor
    }
  })
```

Above the `accessoryFD` variable would just be something that is scoped to be accessible from other hooks we create later for the `read` and `write` functionality.

### Stitching it all together

For a `read` and `write` hook, all we need to do is a simple native `Interceptor.attach()` to `libc.so`, something like the following;

```
const libcWrite = Module.findExportByName('libc.so', 'write')
if (libcWrite) {
  Interceptor.attach(
    libcWrite, {
      onEnter: function(args) {
        if (accessoryFD && args[0].toInt32() === accessoryFD) {
          const size = args[2].toInt32()
          log(`* usb accessory write`)
          log(hexdump(args[1], { length: size }))
          log(Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n') + '\n')
        }
      }
    }
  )
}

const libcRead = Module.findExportByName('libc.so', 'read')
if (libcRead) {
  Interceptor.attach(
    libcRead, {
      onEnter: function(args) {
        this.fd = args[0].toInt32()
        this.size = args[2].toInt32()
      },
      onLeave: function(ret) {
        if (accessoryFD && this.fd.toInt32() === accessoryFD) {
          log(`* usb accessory read`)
          log(hexdump(ret, { length: this.size }))
          log(Thread.backtrace(this.context, Backtracer.ACCURATE).map(DebugSymbol.fromAddress).join('\n') + '\n')
        }
      }
    }
  )
}
```

Now from here on out we will be able to simply dump any of the "traffic" which is being sent to or received from the usb device. Blammo!

### Unsolved

In my example code which is linked below, I was able to define and track down all the proper types, except for one. While utilizing a `java.lang.Thread` to generate stacktraces - which is something I often to do to highlight "where" in the code some function is called, it is annoyingly hard to figure out how to type this object.

```
// eslint-disable-next-line @typescript-eslint/ban-types
let threadObj: Java.Wrapper<{}>
...

      const ThreadDef = Java.use('java.lang.Thread')
      threadObj = ThreadDef.$new()
```

While the typescript compiler itself doesn't seem to have any issues with this (which might be a misnomer since it is `frida-compile` doing some of the lifting?) the linter hates this throwing the following error;

```

/home/diff/repo/rednaga/android/usb-accessory-gadget/agent/index.ts
  4:29  error  Don't use `{}` as a type. `{}` actually means "any non-nullish value".
- If you want a type meaning "any object", you probably want `Record<string, unknown>` instead.
- If you want a type meaning "any value", you probably want `unknown` instead.
- If you want a type meaning "empty object", you probably want `Record<string, never>` instead  @typescript-eslint/ban-types
```

Which, to be honest, I'm not sure I fully understand. Obviously it wants a proper type, though none of those seem correct.

The autogenerated documents seem to indicate that this is the actual return value, so I'm pretty unclear on what is happening here.
`function Java.use<{}>(className: string): Java.Wrapper<{}>`

If anyone has ever solved for this, I'd be curious what the solution ends up being.

### TLDR - Give me the code
A fully working example to dump `read` and `writes` between an Android APK and a USB device can be found in this repository, [usb-accessory-gadget](https://github.com/strazzere/usb-accessory-gadget).