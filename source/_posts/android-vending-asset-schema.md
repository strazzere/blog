---
title: Android Vending Asset Schema
tags:
  - android
  - apk
  - DRM
  - g1
  - google
  - is_forward_locked
  - market place
  - refund_timeout
  - signature
  - Vending
url: android-vending-asset-schema.html
id: 211
categories:
  - android
  - other
  - reverse engineering
date: 2009-03-06 16:21:45
---

**Vending Assets Database;**
`/data/data/com.android.vending/databases/assets.db`

```
CREATE TABLE assets(
_id INTEGER PRIMARY KEY AUTOINCREMENT,
server_id INTEGER UNIQUE,
content_uri TEXT,
state TEXT,
download_pending_time INTEGER,
download_start_time INTEGER,
install_time INTEGER,
uninstall_time INTEGER,
size INTEGER,
type TEXT,
package_name TEXT,
is_forward_locked TEXT,
signature TEXT,
refund_timeout INTEGER,
version_code INTEGER,
server_string_id TEXT);
```

Here is an example entry:
```
_id: 173
server_id: -8619153599380214487
content_uri: content://downloads/download/240
state: UNINSTALLED
download_pending_time: 1233920808560
download_start_time: 1233920811792
install_time: 1233920833173
uninstall_time: 1233936584707
size: 498702
type: 1 (1 = app, 4 = game?)
package_name: com.netdragon
is_forward_locked: false
signature: 7LARjEsYODOYdkX2eWj8yP0Ye-M#498702
refund_timeout: NULL
version_code: NULL
server_string_id: -8619153599380214487
```

Didn't have much time as of lately to post, so I figured I'd drop this schema and an example entry on the blog. It shows how the Vending application stores "assets". I'm still doing more research, but it looks like the previous "DRM" that we've mentioned isn't the only security measure that will be in place. As you can see `is_forward_locked` is a field, though all applications I've run across have this disabled (both protected and unprotected). The `version_code` and `refund_timeout` also seem to not be used.