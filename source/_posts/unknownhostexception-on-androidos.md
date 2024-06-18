---
title: '"UnknownHostException" on AndroidOS'
tags:
  - android
  - connect
  - g1
  - getOutputStream
  - Host is unresolved
  - System.err
  - UnknownHostException
url: 121.html
id: 121
categories:
  - android
  - coding
date: 2008-12-01 20:31:28
---

Alright so I'm doing some coding and kept running into this strange error while trying to connect to a site to perform a `POST` action.
```
12-01 18:27:52.175: WARN/System.err(764): java.net.UnknownHostException: Host is unresolved: www.strazzere.com:80
```
Obviously `www.strazzere.com` _is_ a valid and resolvable host since you can read this post. This occurred whenever `.connect()` or `.getOutputStream()` was called. So after banging my head against a wall for a bit (ok more like an hour or so), I decided to try the fundamentals of troubleshooting.

Does the emulator have internet? _Yeap - can connect to `www.google.com`_
Can the emulator get the host `www.strazzere.com`? _No... Strange - could have sworn it did though?_

Apparently something must have happened with emulator upon switching connections or after hibernation. The solution seems to be to reboot the emulator. After doing so, I checked connection with `www.strazzere.com` and it worked fine. Yet another thing we can sing all-hail-reboot for I guess. Maybe the next version of the emulator will handle these better, though it's not too bad as long as you know what is going on I guess.

Hopefully this article is found by some frustrated developer who hit the same wall I did!