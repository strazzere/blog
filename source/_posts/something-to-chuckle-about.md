---
title: Something to chuckle about...
tags:
  - android
  - android-for-gods
  - code
  - comments
  - humor
  - slow-news
url: something-to-chuckle-about.html
id: 87
categories:
  - android
  - coding
  - random
date: 2008-11-13 15:35:31
---

Ok so I'm scouring the internet using my vast knowledge of googling and I come stumbling across a well... Quiet hilarious web site that has actual useful stuff related to the Android system. It's a double-edged sword too, because the code actually works _and_ it was funny - you know, in an immature way of course!
I guess I'll just have to throw it all out there and let you see it for yourself, the base of the project is located here: [http://code.google.com/p/android-for-gods/](http://code.google.com/p/android-for-gods/). Yes, that's right, Android.... FOR GODS! Yet, such a fitting title to let our mear human minds grasp what this project is about;

> android-for-gods Software that destroys mere mortals. Go suck a lemon, you [pleb](http://en.wikipedia.org/wiki/Plebeian#Modern_usage).

Yes, it did actually include a link to wikipedia for the meaning of "pleb" I did not add that myself. Even if you didn't know _what_ android was or even that this is open source, you _need_ to buy it for the tag line alone... Me? I'll take three already!

What is this project you ask? Let me use the creators own words to describe it:

> **Native Tongue** Take a bed full of rambunctious, naked wet sorority girls and add several pints of Guinness and you get "Android for Gods." It marks the most important event in humanity after the second coming of Christ and before the approach of a technological singularity. We bring a new, young, fresh, hip flavor to the otherwise old, saggy, wrinkled, smelly, disease-ridden mobile phone world ruled by BREW and Windows Mobile tyrants. We aim to design the most fun and attractive mobile software ever that brings girl's panties to their knees and saves puppies from euthanasia. Aimed at smart software reuse and intuitive user interface design, "Android for Gods" will leave a stain on your dress that you will not soon be able to wash out. Oh yeah, and it will run on Android OS too. Bitch.

You had me at panties... Wait, no the saving puppies... Wait, no - you had me at panties. But what does all this mean?! Can't we get some translation for this? Oh, sweet - they even provided that right below it...

> **Translation** "Android for Gods" is an experimental project started for two main reasons: (1) Learn the capabilities of the Android platform (2) Give something back to the world. The world is tired of unresponsive proprietary software that runs on expensive phones, we hope to make a difference.

Some of the true gold comes into play on a few of the examples that the authors have brilliantly created. It has useful information and it can argueably be the best commented code I've ever seen;
```
        public void onReceiveIntent(Context context, Intent intent) {
                Log.i("SMS IR", "We got a goddamn SMS of type:");
                Log.i("SMS IR", intent.getAction().toString());
                if (intent.getAction().equals(ACTION) ) {
                        Log.i("SMS IR", "YUP, it's a fucking SMS");
                        Log.i("SMS IR", "Let's screw around with it");
                        SmsMessage[] messages =
                                Telephony.Sms.Intents.getMessagesFromIntent(intent);
                        for(SmsMessage currentMessage : messages) {
                                Log.i("SMS IR", "This puppy is from:");
                                Log.i("SMS IR", currentMessage.getDisplayOriginatingAddress());
                                Log.i("SMS IR", "And it says:");
                                Log.i("SMS IR", currentMessage.getDisplayMessageBody());
                        }
                        Log.i("SMS IR",
                                        "We're done now, quit sending messages to yourself, loser.");
                }
        }
```

Or the equally creative;
```
        TextView tv = new TextView(this);
        // Here we can change the text of the TextView to
        // something that hits rooster's character spot on.
        tv.setText("jcmb loves men");
        // Now let's get get our toaster from out of the closet,
        // just like rooster's sexuality.
        Toast toast = new Toast(this);
        // Lets push some rye into that bad boy, just like
        // when rooster went to county and met Bubbha
        toast.setView(tv);
        // We don't want that toast getting burnt, set it for
        // an appropriate amount of time.
        toast.setDuration(Toast.LENGTH_LONG);
        // Pop it out before it burns!
        toast.show();
```

Lastly, I'd like to close this entry in tribute to the authors by using a quote from step 5, of the LogHowTo. Hopefully this brightened the day a little bit, and added a little humor to those android developers who might find this code via google.

> 5\. You're welcome, bitch.