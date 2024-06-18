---
title: Getting the Android Market Token (for use in getting live data)
tags:
  - android
  - android market
  - androidsecure
  - apps
  - com.google.android.googleapps
  - data
  - games
  - live
  - live android market
  - market
  - MyMarket
  - service
  - sierra
  - vending.apk
url: getting-the-android-market-token-for-use-in-getting-live-data.html
id: 291
categories:
  - android
  - other
  - reverse engineering
date: 2009-09-15 13:52:15
---

![Mmmmm... Market Data...](http://173.230.150.16/blog/wp-content/uploads/2009/01/android_market_combo_8282008_wide_600x297-300x148.png "Mmmmm... Market Data...")
I've been sitting on this code for a while now. I had been tooling around with it mostly about a year ago when I was collecting live market data - it actually took a long time to figure out what the actual token was. Not sure if this method will be changed since there is apparently a big update for the market coming up, I'm assuming it won't change though - otherwise older versions of the market would break. For people who don't know - the http requests the vending apk is sending holds a `token` which there was no link to how it was obtained or formed. It turns out to just be a simple token requested like any other google service. The "service" name just turned out to be `android`. Other "services" that the market uses are `androidsecure` and `sierra` - the latter being the codename for google-checkout.

The main reason this was a pain to figure out was because it's being handled by the `com.google.android.googleapps` package, not the _vending.apk_ package.

Hopefully this snippet will be useful for some people, like the MyMarket creators or anyone else trying to do market data analysis. I'll try to post later how to actually send data and receive it using this token. Enjoy for now!
```
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileNotFoundException;
import java.io.FileWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.UnsupportedEncodingException;
import java.net.InetAddress;
import java.net.MalformedURLException;
import java.net.Socket;
import java.net.URL;
import java.net.URLConnection;
import java.net.URLEncoder;


public class auth {

	// declare variables for use in gettig auth-token
	private final static String account = "yourAccount@gmail.com";
	private final static String password = "yourPassword";

	// service must be 'android' for market-data
	private final static String service = "android";

	public static void main(String[] args) throws UnsupportedEncodingException, MalformedURLException, IOException {

		// Prepare data for being posted
		String rdata = URLEncoder.encode("accountType", "UTF-8") + "=" + URLEncoder.encode("HOSTED_OR_GOOGLE", "UTF-8");
        rdata += "&" + URLEncoder.encode("Email", "UTF-8") + "=" + URLEncoder.encode(account, "UTF-8");
        rdata += "&" + URLEncoder.encode("Passwd", "UTF-8") + "=" + URLEncoder.encode(password, "UTF-8");
        rdata += "&" + URLEncoder.encode("service", "UTF-8") + "=" + URLEncoder.encode(service, "UTF-8");

      // Send data
      URL url = new URL("https://www.google.com/accounts/ClientLogin");
      URLConnection conn = url.openConnection();
      conn.setDoOutput(true);

      // Write post
      OutputStreamWriter wr = new OutputStreamWriter(conn.getOutputStream());
      wr.write(rdata);
      wr.flush();

      // Get the response
      BufferedReader rd;
      String line;
      StringBuffer resp = new StringBuffer();
      try {
    	  rd = new BufferedReader(new InputStreamReader(conn.getInputStream()));
    	  while ((line = rd.readLine()) != null) {
    		  resp.append(line);
    		  System.out.println(line);
    	  }
    	  wr.close();
    	  rd.close();
      } catch (FileNotFoundException e1) {
    	  // Catch bad url
    	  System.out.println("Error: Bad url address!");
      } catch (IOException e1) {
    	  // Catch 403 (usually bad username or password
    	  if(e1.toString().contains("HTTP response code: 403"))
    		  System.out.println("Error: Forbidden response! Check username/password or service name.");
      }

      String token = resp.toString().substring(resp.toString().indexOf("Auth=")+5);

      try{
           //Create file
          FileWriter fstream = new FileWriter("auth.token");
              BufferedWriter out = new BufferedWriter(fstream);
          out.write(token);
          //Close the output stream
          out.close();
          }catch (Exception e){//Catch exception if any
           System.err.println("Error: " + e.getMessage());
          }
	}
}
```
