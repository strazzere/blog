---
title: Electron code reuse, and other hackery
date: 2025-02-06 10:34:57
tags:
  - electron
  - typescript
url: electron-code-reuse-and-other-hackery.html
categories:
 - electron
 - typescript
---

# Electron and local environments

After a few years of dealing with a rather large [Electron](https://www.electronjs.org/) application. I've found a few different ways to approach things -- as with anyone dealing with large bohemoth codebases, it feels natural to carve it up and chunk out the code into more managable pieces. This leads us from having one large, insanely unplanned renderer, to having "nicer" (though not great) smaller packages which can be reused across the renderer, the main process and even outside in other non-electron focused services. This can lead to interesting problems though, such as making the code aware (without caring) where it is being executed. The `electron` package itself tends to abstract these things away whether the code is run on Windows/MacOS/Linux - however what about when you aren't utilizing the target code _within_ `Electron` anymore?

## isPackaged

`Electron` gives us a method, [app.isPackaged](https://www.electronjs.org/docs/latest/api/app#appispackaged-readonly) through it's main API, however this isn't useful to use unless we want to import `electron` into all of our code. Specifically it let's us determine

```
A boolean property that returns true if the app is packaged, false otherwise. For many apps, this property can be used to distinguish development and production environments.
```

## Are we in electron or not?

There are a few [process](https://www.electronjs.org/docs/latest/api/process) properties which are interesting in Electron. Specifically [process.resourcePath](https://www.electronjs.org/docs/latest/api/process) and [process.type](https://www.electronjs.org/docs/latest/api/process#processtype-readonly). These are both extremely interesting to me, as often in code I need to get an asset file - though the path for such a file will change depending on if it is packaged, running as development `electron` or as `node-js`.

So, what we want to do, is check to see if `process.resourcesPath` returns anything - which it will not in `node.js`. If it does return something, we want to check if it returns something _with_ `app.asar` inside it. This would mean (based on how we bundle things for distribution) that `electron` is bundled. This does mean, we will need to ignore expected `typescript` errors, as the type checking doesn't expect `resourcePath` to ever exist on the `process` object. This lands us to this type of statement;

```typescript
  // @@ts-expect-error
  const isPackaged = process?.resourcesPath
    ? fs.pathExistsSync(path.join(process.resourcesPath, "app.asar"))
    : false;
```

Now you can properly write functions to handle code differently based on whether the code is inside the `main` process, `renderer` or being run by some of your `node-js` code directly. Here is a prototype function I came up with which is semi-specific for my usecase, due to the local compile locations (`elect/` vs `elect/js` vs `src/cli/`);

```typescript
/**
 * Handle getting the resource correctly based on if the electron
 * application is packages, main or renderer in dev mode.
 *
 * @param {string} fileName
 * @returns {string}
 */
export function getResourcePath(fileName: string): string {
  // @@ts-expect-error
  const isPackaged = process?.resourcesPath
    ? fs.pathExistsSync(path.join(process.resourcesPath, "app.asar"))
    : false;

  if (isPackaged) {
    // @@ts-expect-error
    console.log(
      `isPackaged ${isPackaged} : ${path.join(process.resourcesPath, "app.asar")}`,
    );
    // @@ts-expect-error
    return path.join(`${process.resourcesPath}/app.asar/`, fileName);
  }

  // renderer (type renderer) is run out of elect/
  // main (type browser) is run out of elect/js/
  // node (type undefined) is run out of src/cli/
  // @@ts-expect-error
  const isRenderer =
    Object.prototype.hasOwnProperty.call(process, "type") &&
    process.type === "renderer";
  return path.join(
    isRenderer ? `${__dirname}/../` : `${__dirname}/../../`,
    fileName,
  );
}
```