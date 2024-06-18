---
title: Dex/Apk file PatchTest example
tags:
  - android
  - apk
  - challange
  - crackme
  - dex
  - patchme
  - reverse engineering
url: dexapk-file-patchtest-example.html
id: 43
categories:
  - android
  - reverse engineering
date: 2008-10-31 19:18:21
---

Happy Halloween everyone! As promised I've written up a small example program to show how to patch the android dex file, repackage as an apk and get it to work on your android device or emulator. This is actually _extremely_ easy once you understand the steps, or have the tools to do it for you!

For an example program I've attached "PatchTest.rar" and "PatchTest.apk". These are both the original \*unpatched\* programs we are going to use. The rar file contains the source and project files so you can load them up in Eclipse and play around with them yourself. Both of these files have been tested in the emulator and on the G1 hardware device and work fine. Here is the bulk of the code in which we are looking at though:
```
/*
 * File:	Patchtest.java
 * Coded:	Timothy Strazzere
 * Date:	10/31/2008 (Happy Halloween!)
 *
 * 			This is an example of a brutally simple file for
 * 			the android OS that will simple start up, call
 * 			the function registered() and either keep the text
 * 			to "not registered" if registered() returns false.
 * 			Otherwise if registered() returns true it will
 * 			change the text to "registered".
 *
 * 			The way it is set up it should NEVER return registered,
 * 			so it is perfect fora simple example of how to
 * 			patch a file :)
 */

package com.Android.Patchtest;

import android.app.Activity;
import android.graphics.Color;
import android.os.Bundle;
import android.widget.TextView;

public class Patchtest extends Activity {
	TextView status;
	TextView webaddress;

    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        status = (TextView)this.findViewById(R.id.status);

        // Call Registered Function, if true - change the text!
        if(registered()){
        	this.status.setText("Registered!");
        	this.status.setTextColor(Color.GREEN);
        }
    }

    // Stupid 'registered' function that should NEVER return true
    protected boolean registered() {
    	int x = 12345;
    	int y = 67890;

    	x %= y;

    	if(x == 0)
    		return (true);

    	return(false);
    }
}
```

So essentially this program starts, calls the "registered" function, if it returns true then it will change our TextView to "Registered!" and that is what we want. If we look at the registered function, it simple initializeds two values, performs a modulus operation on the two and judges the result. This means the program should _never_ return the result true, so we should never have the program show the text "Registered!"
![Not Registered Screen](http://173.230.150.16/blog/wp-content/uploads/2008/10/badboy-200x300.png "Not Registered Screen")
Files:
[Patchtest.apk - MD5: AF65D0D7399D3F1D4768AFB523981666](http://strazzere.com/blog/wp-content/uploads/2008/10/patchtest.apk)
[Patchtest.rar - MD5: B60FF4BE1B680365F483EB288278F8E5](http://173.230.150.16/blog/wp-content/uploads/2008/10/patchtest.rar)

Next up I'll post the patched .apk file and an explanation on how to do it. Until then, try it yourself and use the tools I provided in previous posts to see if you can do it yourself! It has also been submitted to crackmes.de by a friend, maybe someone there will also have fun with it.