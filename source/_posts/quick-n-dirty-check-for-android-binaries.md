---
title: 'Quick n dirty, check for Android binaries'
tags:
 - android
 - osint
 - crawling
categories:
 - android
date: 2020-05-09
---

Annoyingly, unless you're at a large AV shop or happen to work a place running one of the stores, maintaining a good repository of Android binaries can sometimes be tricky. Most options folks have are either outrageously priced (Virustotal), extremely limited to the number of downloads (apklab?) or you're likely just feeding other peoples intel programs (all of them?). So what is a simple reverser meant to do?

Beg, borrow and steal. Many of the aforementioned services do slowly trickle things out - usually limiting downloads or searches to ~10 a day. Some you can gain access to by sharing the right intel with the right people. Though, something I've found relatively interesting and useful lately was just simply using [APKMirror](https://www.apkmirror.com). Though, using them is obvious, right? Except it isn't exactly set up for a simplistic download or search. Except, the owners did seem to place in a nice little API call which can be very useful.

Before you burn a download from one of those coveted sources, hit APKMirror's upload API and see if the md5 is available;

```
curl 'https://www.apkmirror.com//wp-json/apkm/v1/apk_uploadable/098822a8624f7fd5e9ddb4c82c6f986c' \
  -H 'authority: www.apkmirror.com' \
  -H 'pragma: no-cache' \
  -H 'cache-control: no-cache' \
  -H 'dnt: 1' \
  -H 'upgrade-insecure-requests: 1' \
  -H 'user-agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.113 Safari/537.36' \
  -H 'accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' \
  -H 'sec-fetch-site: none' \
  -H 'sec-fetch-mode: navigate' \
  -H 'sec-fetch-dest: document' \
  -H 'accept-language: en-US,en;q=0.9' \
  --compressed
```

Since they're sitting behind Cloudflare, thus their WAF, you'll need to keep the bulk of those junk headers to avoid curl getting blocked. Simply drop this isn't whatever script or language you need. The return, is even better;

```
{
  "md5": "098822a8624f7fd5e9ddb4c82c6f986c",
  "status": "dupe",
  "link": "https://www.apkmirror.com/apk/snap-inc/snapchat/snapchat-10-80-0-0-release/snapchat-10-80-0-0-2-android-apk-download/",
  "title": "Snapchat 10.80.0.0 (arm64-v8a) (nodpi) (Android 4.4+)"
}
```

A nicely formed json blob which will give you what the file is and where to go look for the binary. Personally, I suggest not really abusing this, even though most of the bandwidth is probably just CF's getting soaked up do to cached binaries. As a nifty little note, it appears either OpenGAPPS was told about this, or figured it out themselves, as we can see this [pull request from their repo](https://github.com/opengapps/opengapps/pull/402/files/d77ec0770372efd4e84a3636df45668134a351c7), indicating that this was known since at least 2016;

```
for apk in $newapks; do
    if $(curl -s -S -A "OpenGAppsUploader" "https://www.apkmirror.com/wp-json/apkm/v1/apk_uploadable/$(md5sum "$apk" | cut -f 1 -d ' ')" | grep -q "uploadable"); then
        echo "Uploading $apk to APKmirror.com..."
        filename="$(basename "$apk")"
        curl -s -S -A "OpenGAppsUploader" -X POST -F "fullname=$name (OpenGApps.org)" -F "email=$email" -F "changes=" -F "file=@$apk;filename=$filename" "https://www.apkmirror.com/wp-content/plugins/UploadManager/inc/upload.php" > /dev/null
    else
        echo "Skipping $apk, already exists on APKmirror.com..."
    fi
done
```