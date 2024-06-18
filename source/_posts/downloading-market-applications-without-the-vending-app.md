---
title: Downloading market applications without the vending app
tags:
  - android
  - AndroidDownloadManager
  - assetId
  - authToken
  - deviceId
  - downloading apk
  - market
  - market alternative
  - reverse engineering
  - userId
  - Vending
  - vending.apk
url: downloading-market-applications-without-the-vending-app.html
id: 293
categories:
  - android
  - coding
  - other
  - reverse engineering
date: 2009-09-23 19:08:35
---

![Mmmmm... Market Data...](http://173.230.150.16/blog/wp-content/uploads/2009/01/android_market_combo_8282008_wide_600x297-300x148.png "Mmmmm... Market Data...")
It turns out downloading a free application is actually pretty easy to reproduce. The things required by the google android servers are just four variables. The server needs to know your `userId`, `authToken`, `deviceId` and the applications `assetId`.

The `userId` is a unique number that is associated only with your gmail account, the one that is currently linked to the phone. I'm working on getting a generic way to grab this number, though I believe the request is buried in an ssl request to the google servers. So for now, you can obtain your own userId by doing a tcpdump of your market traffic, just do a download of an application and look for a `GET` request in wireshark. There does not appear to be a "hard" maximum character on this, I've seen userIds as low as 8 in length and as high as 13. A bad userId will return a 403 forbidden response.

The `authToken` is sent in cookie form to the server, to authenticate that the user is using a valid and non-expired token and well, is who they say they are! This is linked to the userId and must match the account that the userId is taken from. Expired tokens will return a 403 forbidden response.

The `deviceId` is simply your `android_id`, and is linked in anyway to the authtoken or user-id, so feel free to spoof this.

The `assetId` is a number (negative or positive) that identifies the current stream of the application you wish to download. More on this later when I cover how to get live market data. Note that this number is not always the same - it (appears) to change when something from the application is changed. Originally I referred to this in my research as a `cacheAppID` for just that purpose.
```
// Downloading apk's without vending/market
// Coded by Tim Strazzere
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.UnsupportedEncodingException;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLEncoder;
import java.net.HttpURLConnection;

public class main {
	public static void main(String[] args) {
		// current assetId for the yahoo search apk
		String assetId = "7884814897504696499";
		// input your userId
		String userId = "12345678901";
		// spoof your deviceId (ANDROID_ID) here
		String deviceId = "2302DEAD532BEEF5367";

		// input your authToken here
		String authToken = "DQAAA...BLAHBLAHBLAHYOURTOKENHERE";

		String cookie = "ANDROID=" + authToken;

		try {
			// prepare data for being 'get'ed
			String rdata = "?" + URLEncoder.encode("assetId", "UTF-8") + "=" + URLEncoder.encode(assetId, "UTF-8");
			rdata += "&" + URLEncoder.encode("userId", "UTF-8") + "=" + URLEncoder.encode(userId, "UTF-8");
			rdata += "&" + URLEncoder.encode("deviceId", "UTF-8") + "=" + URLEncoder.encode(deviceId, "UTF-8");

			// Send data
			URL url = new URL("http://android.clients.google.com/market/download/Download" +rdata);
			HttpURLConnection conn = (HttpURLConnection)url.openConnection();

			// For GET only
			conn.setRequestMethod("GET");

			// Spoof values
			conn.setRequestProperty("User-agent", "AndroidDownloadManager");
			conn.setRequestProperty("Cookie", cookie);

			// Read response and save file...
			InputStream inputstream =  conn.getInputStream();
			BufferedOutputStream buffer = new BufferedOutputStream(new FileOutputStream("out.put"));
			byte byt[] = new byte[1024];
			int i;
			for(long l = 0L; (i = inputstream.read(byt)) != -1; l += i )
				buffer.write(byt, 0, i);

			inputstream.close();
			buffer.close();

			System.out.println("File saved...");
		}
		catch (FileNotFoundException e) {
			System.err.println("Bad url address!");
		}
		catch (UnsupportedEncodingException e) {
			System.out.println(e);
		}
		catch (MalformedURLException e) {
			System.out.println(e);
		}
		catch (IOException e) {
			if(e.toString().contains("HTTP response code: 403"))
				System.err.println("Forbidden response received!");
			System.out.println(e);
		}
	}
}
```
Hopefully someone will find this stuff useful ;) Better than me just sitting on it forever!