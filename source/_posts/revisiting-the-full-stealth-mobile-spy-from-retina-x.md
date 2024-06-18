---
title: Revisiting the "full stealth" mobile spy from Retina-X
tags:
  - android
  - android spy
  - android stealth
  - com.retinax.android
  - hosts file
  - retinax
  - spy
  - spyaware
  - spyware
  - stealth
url: revisiting-the-full-stealth-mobile-spy-from-retina-x.html
id: 348
categories:
  - android
  - other
date: 2010-05-05 13:47:24
---

![Wait - who is john doe?!](http://173.230.150.16/blog/wp-content/uploads/2010/05/login-300x178.png "Wait - who is john doe?!")
I've gotten a few emails regarding my previous post, [“Full Stealth” just isn’t what it used to be!](http://strazzere.com/blog/?p=335), asking for a more depth on the subject. I've covered just about everything I found in the first posting - but I did go back and re-read the documentation provided on the web site. Looks sort of like a boo-boo on the architecture of the product.  

> 6. After the installation completes, power down the phone. Then, power the phone back up and bring up the Dialer. Enter the digits *12345# and then press the SEND button. The login screen should then appear. _Enter **your username/password** EXACTLY as you did when you created it. Then click LOGIN._

Wait, what?! I guess we're really going to rely on the fact notion that this application is very secure and stealthy. Sure hope someone whose being spied on doesn't have root and just snag the username and password. That could be embarrassing, spying on someone only to have them turn the tables on you since they now have your credentials. It honestly can't be that hard to implement a unique identifier for these phones to send that would link them to this account, could it? Oh well, just another reason to not purchase this product :)

For anyone who is rooted and might be worried about this application, you can go ahead and add the following line to your hosts file to block their server.  

> http://www.mobilespylogs.com/
  
On a side note - keep an eye out for _spyAware_ - it should be on the Android Market soon, a nifty little proof of concept tool I'll be using to show how to detect/prevent abuse of your phone.