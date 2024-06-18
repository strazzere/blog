---
title: Conquering the mysterious userId (Fixing Android Market downloading)
tags:
  - android market
  - android market api
  - apk
  - downloading apks
  - reverse engineering
  - userId
  - vending.apk
url: conquering-the-mysterious-userid-fixing-android-market-downloading.html
id: 411
categories:
  - android
  - reverse engineering
date: 2012-02-04 17:07:41
---

Originally I found the easiest way to download applications was just [spoofing the final request](http://strazzere.com/blog/?p=293). This worked - and worked well for the entire time I had been using it. Though it bugged me that I couldn't find out _where_ the actual `userId` was coming from. So I keep digging and digging and finally came up with the solution which is posted below. I never really shared this though, since as I said - other people just used the hack and didn't seem to care much. The main concern people had was "how do I get it faster" - which to that I always replied "learn whats going on". Plenty of people figured it out, though now, according to the the emails I've been getting recently and the ones on the Android-Market-Api Google Group, this stopped old hack has working. So, I guess it's about time to spill the rest of the mysery.

The requests originally looked like this when being sniffed over the wire;

`http://android.clients.google.com/market/download/Download?userId=###&deviceId=###&assetId=###`

Though recently they started looking like this;

`http://android.clients.google.com/market/download/Download?packageName=com.package.name&versionCode=##&token=###`

Well - this didn't matter much to me since I don't use that project, but I've helped enough people through side channels that I figured the information is as good as public. So I've decided to write up a little "what is actually going on".

Just a little while ago I committed an updated _[market.proto](https://code.google.com/p/android-market-api/source/diff?spec=svn47&r=47&format=side&path=/trunk/AndroidMarketApi/proto/market.proto)_ with the necessary fields to request downloads from the Android Market. Essentially I've added the `GetAssetRequest` and the `GetAssetRequest` which was previously missing. I also fixed the `unknown` field which is known as the `isSecure` field. This boolean field lets the market know what type of request is coming in and which token will be used.

In the order versions of the vending apk, not everything was send over HTTPS. When most normal requests (get application info, queries, etc) where over unsecured HTTP, the GetAssetsRequest was considered "secure" and only went over HTTPS. This meant the `isSecure` field had to be set and a new authtoken would be used. Instead of the `android` service authtoken being sent, the `androidsecure` token would be sent. This is probably done so that if someone was sniffing requests, they couldn't just reuse your _android_ token and get away with it, though it may have been done for other reasons.

So what was the mystery of the `userId`? Well - it was never found on the device, because it was never actually stored on the device. It was only stored sometimes inside of URLS which had been cached or saved to databases. If properly using the protobuf, this relieves the person from needing to find these values, since the market themselves provide them! It was a bit annoying to figure this out finally, but also a huge relief since I wasn't going crazy not finding the `userId` anymore. So let's sum this all up;

1. Get the `android secure` token for your account, use this as the `authtoken` for your next request
2. Build proper request protobuf; Use `authtoken` from above, set `isSecure` to true and add the `assetId` to a `GetAssetRequest`
3. Receive the `GetAssetResponse` and build an HTTP request from it using the; `blobUrl` as the URL and set a cookie using the `downloadAuthCookieName`/`downloadAuthCookieValue` pair.

Lastly, I'd like the apologize for the delay in this information. I feel a bit bad since I've known this for a while, but most people who ask me questions where just using the hack, which -- until very recently worked fine. Also, it has always been possible to just reverse the apk yourself and figure out what was going on ;)