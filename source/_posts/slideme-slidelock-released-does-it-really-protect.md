---
title: SlideMe / SlideLock released - does it really protect?
tags:
  - android
  - AndroidOS
  - data harvesting
  - forward locking
  - g1
  - Protection Scheme
  - reverse engineering
  - SlideLOCK
  - SlideMe
  - web activation
url: slideme-slidelock-released-does-it-really-protect.html
id: 177
categories:
  - android
date: 2009-02-03 16:33:52
---

![SlideLOCK - protecting the apps?](http://173.230.150.16/blog/wp-content/uploads/2009/02/slide_lock_tm_wide.png "SlideLOCK - protecting the apps?")
```
SlideME is pleased to announce support for paid applications with our
release of SAM 2.3, the first billing solution for Android that includes a
mobile client. You can download SAM 2.3 at http://slideme.org/sam2.apk

Several weeks back, we ran into the issue of the G1 not protecting against
applications being removed from the device, so we are introducing SlideLock,
which you can embed into your application to prevent forwarding to another
device. This is independent from support for paid applications, and will
provide additional protection, if the developer chooses to use it.:
http://slideme.org/slidelock

Developers of paid applications will also need to sign in and approve the
new developer agreement and assign tokens, which authorize downloads.

This is a major release:

*Billing*
* No country restrictions on purchasing applications. Anyone can buy & sell.
* Support for global payments via SlideCollect including users paying via
their Amazon Payments account.
* High payout rates to content providers/developers ~98%:
http://SlideME.org/rate-schedule
* Sales tax is legitimately applied to sales, including all of the European
Union
* SlideME Prepaid MasterCard: developers from anywhere in the world can
receive a debit card and get paid without a need of a bank account.

* Multiple payout methods
* Fraud detection: SlideME has partnered with Retail Decisions, a world
leader in credit card fraud prevention, in order to combat fraud and further
reduce chargebacks. More information about ReD is available at
http://www.redplc.com
* The Shopper Guarantee – All customers receive the Shopper Guarantee. This
guarantees shoppers that they will either receive their digital downloads
via SAM, or they will receive their money back. This guarantee can increase
customer confidence in purchasing from SlideME website.

*Client*
* SAM 2.3 with a new user interface
* SAM 2.3 billing support
* SAM 2.3 supports secure method so shoppers never need to enter their
credit card or sensitive information from within their handset.

*DRM*
* SlideLOCK has been released to protect your applications and prevent
forwarding
*
Infrastructure*
* 24/7 support has been implemented to support you and users in the best
possible way.
* High Availability secured environment
* Distributed feeds of content on SlideME’s Marketplace to further promote
your applications
```
Alright everything seems all great and what not, but the whole SlideLOCK idea seems like it's trying a little too hard. Two big permissions must be added for the SlideLOCK to work correctly, both `READ_PHONE_STATE` and `INTERNET_ACCESS`.

![Keep your code secure!](http://173.230.150.16/blog/wp-content/uploads/2009/01/strictly_confidential-230x300.jpg "Keep your code secure!")
Why does slideLock need `READ_PHONE_STATE`? Well because it's grabbing your `NetworkOperator`, `NetworkCountryIso`, `DeviceSoftwareVersion`, `PhoneType`, `NetworkType`, `SimCountryIso`, `SimOperator`, `SimSerialNumber`, `SubscriberId` and `isNetworkRoaming` variables. It uses `INTERNET_ACCESS` to send these in an https request to the slideme website as follows;
```
sb.append("https://slideme.org/mobileapp/install-check?device_id=").append(mng.getDeviceId()).append("key=").append(key).
append("version=").append(version).append("versionCode=").append(versionCode)
.append("network_op=").append(mng.getNetworkOperator()).append("network_op_name=").append(mng.getNetworkOperatorName()).
append("network_iso=").append(mng.getNetworkCountryIso()).append("software_version=").append(mng.getDeviceSoftwareVersion())
.append("phone_type=").append(mng.getPhoneType()).append("network_type=").append(mng.getNetworkType())
.append("sim_iso=").append(mng.getSimCountryIso()).append("sim_op=").append(mng.getSimOperator())
.append("sim_serial=").append(mng.getSimSerialNumber()).append("subscriber_id=").append(mng.getSubscriberId())
.append("roaming=").append(mng.isNetworkRoaming()).append("lock_version=1.1");
```
So, in theory, what SlideLOCK does is ping it's site with _all_ your information (which, by the way, does it really need _all_ that information to make a unique id?!) and the application version and key. It checks this against a database which I'm guessing will show who has purchased this, then it sends back either a yes or a no. Pretty bloated up program for simple checking if a user bought a program, seems like that should be fairly easy for someone to throw together without using their own SDK. The whole "lock" idea seems to be a way of trying to shelter developers into thinking they are fully protected... My main concern is, while this idea may have worked on WindowCE/Mobile - what is going to stop a user from editing their hosts file? By default a application is set to run if the network connection appears down, so a simple edit to the hosts file should kill this... Though I'm not condoning this in anyway. Also a user would have to find someone to give them the files that are "locked" to their phone. I originally though they where modifying each .apk for users and having them hardcoded to a device id - guess not!

On a somewhat tangent... Isn't it strange that they never even ask for an `ANDROID_ID`? Honestly, it makes you think if they are just harvesting data from users while making a "match". I mean - the `SimSerialNumber` should provide something completely unique anyway...

I'd put my money on this be "circumvented" pretty fast and easily.