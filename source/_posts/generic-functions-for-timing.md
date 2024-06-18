---
title: Generic functions with generic arguments for timing
tags:
  - typescript
url: generic-functions-generic-arguments-timing.html
categories:
 - typescript
date: 2024-05-09
---

## Adding timing for debuggability
In a rather large and hard (annoying?) to debug electron application I've been working on. Between things running in the main process and the renderer process, having to `Promise.race` different types of extended calls to usb and serial devices - it can be complicated to know exactly what is going on and what is failing. Utilizing the `Chrome Inspector` has yielded good results, though sometimes it is just important to have good timing with logs during a development build. Specificially I've found this almost required when testing across platforms as Windows handles `libusb` interactions substantially differently from Linux/MacOS.

### First few passes

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any
type AnyFunction = (...args: any[]) => any

function timingFunctionWrap(f: AnyFunction, ...args: Parameters<typeof f>): ReturnType<typeof f> {
  console.time(f.name)
  let result = f(...args)

  console.timeEnd(f.name)

  return result
}
```

This worked well for a few functions I had, though when getting heavier into device communications I needed to also handle `async` functions. Wanting to generalize this even more, it lead to the code looking like so;

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any
type AnyFunction = (...args: any[]) => any

async function timingFunctionWrap(f: AnyFunction, ...args: Parameters<typeof f>): Promise<ReturnType<typeof f>> {
  console.time(f.name)
  let result = f(...args)

  if (result instanceof Promise) {
    result = await Promise.resolve(result)
  }

  console.timeEnd(f.name)

  return result
}
```

This also worked well, and is pretty close to what I ended up. The only final part of the functionality that I wanted to add was signal handling which was not as straight forward as I had originally expected.

### Signal handling

Some of my device communications end up passing around an `AbortSignal` as a means of not waiting forever incase disconnects or timeouts occur. This is often an optional parameter for the functions, so it isn't something I can just build into the `AnyFunction` type like I had hoped;
```typescript
type AnyFunctionWithAbortSignal = (...args: [...any[], signal: AbortSignal]) => any;
```

This above bit is not viable since the function prototypes often contain signal as `signal?: AbortSignal`. In all honesty, we probably could just change many places in the code to no longer have the signal be optional, as it is essentially always used. Though now this feels like a code golf and I wanted to find out a solution without refactoring everything. 

This lead to the following code;

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any
type AnyFunction = (...args: any[]) => any

async function timingFunctionWrap(f: AnyFunction, ...args: Parameters<typeof f>): Promise<ReturnType<typeof f>> {
  console.time(f.name)
  let result = f(...args)

  if (result instanceof Promise) {
    result = await Promise.resolve(result)
  }

  console.timeEnd(f.name)

  const signal = args.pop() as AbortSignal
  if (signal && signal.aborted) {
    console.log(`${f.name} was aborted`)
  }

  return result
}
```

The above works, _for me_, however it does have an issue because we are making an assertion without checking. This is just assuming that whatever the last argument is inherently always an `AbortSignal`. Thinking about it for a bit, I needed to have a guard, as I can't just check the using the `instanceof` and we can make this an assertion using `is` like this;

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any
function isAbortSignal(signal: any): signal is AbortSignal {
  return signal instanceof AbortSignal
}
```

Pulling this into it's own function allows us to utilize the `is` and let the linter know everything below this the type of the object.

## Finalizing the code

```typescript
// eslint-disable-next-line @typescript-eslint/no-explicit-any
type AnyFunction = (...args: any[]) => any

// eslint-disable-next-line @typescript-eslint/no-explicit-any
function isAbortSignal(signal: any): signal is AbortSignal {
  return signal instanceof AbortSignal
}

async function timingFunctionWrap(f: AnyFunction, ...args: Parameters<typeof f>): Promise<ReturnType<typeof f>> {
  console.time(f.name)
  let result = f(...args)

  if (result instanceof Promise) {
    result = await Promise.resolve(result)
  }

  console.timeEnd(f.name)

  const signal = args.pop()
  if (isAbortSignal(signal) && signal.aborted) {
    console.log(`${f.name} was aborted`)
  }

  return result
}
```

So now we have a neat little function which can wrap any function, assert the time it took to finish and notify us if it was aborted.
