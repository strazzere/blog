---
title: Android Turrent Defense "Badge System" under the hood
tags:
  - android os
  - android turrent defense
  - atd
  - badge
  - badge system
  - bytecode
  - dex
  - dex bytecode
  - g1
  - reverse engineering
url: android-turrent-defense-badge-system-under-the-hood.html
id: 227
categories:
  - android
  - coding
  - reverse engineering
date: 2009-04-03 14:48:54
---

![Pew pew! Take that green circles!](http://173.230.150.16/blog/wp-content/uploads/2009/04/mad_v101_20090328_runmode-200x300.png "Pew pew! Take that green circles!")
A recent addition to the android market has been ATD, Android Turret Defense. This is a Plox-like game, though it has the "maze" strategy element combined in it. Strangely -- it reminds me of a few old maps I used to play with friend for starcraft... Anyway I finally got around to beating it which isn't too difficult once you get the hang of placing turrets and a get a decent strategy. At the end it awards you with a "badge code" -- not sure exactly what the author intends to use this for, but I decided to take a look at how these are created. I was interested in how they where generated, and to see if people could easily replicate them, or if there would be any deterrents to keep people from just sharing them. Again, this is possibly completely useless information, since we have no idea _what_ these codes will be used for. The _could_ be used for tournaments, downloads, prizes - or maybe to just "give" you an image of a badge... As of right now we just don't know.
Below is a dump of the function we will be analyzing with my comments in it (highlighted green), they should be pretty easy to follow:
```
.method private createBadgeCode()Ljava/lang/String;
// Date now = New Date();
new-instance v2,java/util/Date
invoke-direct {v2},java/util/Date/ ; ()V

// SimpleDateFormat dateFormat = new SimpleDateFormat(“yyMMddhhmm”);
new-instance v5,java/text/SimpleDateFormat
const-string v7,”yyMMddhhmm”
invoke-direct {v5,v7},java/text/SimpleDateFormat/ ; (Ljava/lang/String;)V

// StringBuilder raw = new StringBuilder();
new-instance v7,java/lang/StringBuilder
invoke-direct {v7},java/lang/StringBuilder/ ; ()V

// raw.append(dateFormat.format(now));
invoke-virtual {v5,v2},java/text/SimpleDateFormat/format ; format(Ljava/util/Date;)Ljava/lang/String;
move-result-object v8
invoke-virtual {v7,v8},java/lang/StringBuilder/append ; append(Ljava/lang/String;)Ljava/lang/StringBuilder;
move-result-object v7

// raw.append(difficulty);
iget v8,v12,tx/games/atd_world.difficulty I
invoke-virtual {v7,v8},java/lang/StringBuilder/append ; append(I)Ljava/lang/StringBuilder;
move-result-object v7

// raw.append(“tensaix2j”);
const-string v8,”tensaix2j”
invoke-virtual {v7,v8},java/lang/StringBuilder/append ; append(Ljava/lang/String;)Ljava/lang/StringBuilder;
move-result-object v7

// Bytes[] rawbytes = raw.toString.getBytes;
invoke-virtual {v7},java/lang/StringBuilder/toString ; toString()Ljava/lang/String;
move-result-object v4
invoke-virtual {v4},java/lang/String/getBytes ; getBytes()[B
move-result-object v0

/* Below code refined;
int sum = 0;

for(int i = 0; i < rawbytes.length(); i++) sum += rawbytes[i]; */
const/4 v6,0
const/4 v3,0
l3c1e:
// length = rawbytes.length();
array-length v7

// if( v3 > v7 ) goto: l3c30
if-ge v3,v7,l3c30

// v7 = rawbytes(v0);
aget-byte v7,v0,v3

// v6 += v7;
add-int/2addr v6,v7

// v3 ++;
add-int/lit8 v3,v3,1
goto l3c1e

l3c30:

// StringBuilder badge = new StringBuilder();
new-instance v7,java/lang/StringBuilder
invoke-direct {v7},java/lang/StringBuilder/ ; ()V

// v8 = Math.random();
invoke-static {},java/lang/Math/random ; random()D
nop
move-result-wide v8

// v10 = 4652007308841189376;
const-wide v10,4652007308841189376 ; 0x408f400000000000

// v8 = Math.round(v8*v10);
mul-double/2addr v8,v10

// I thought it only took one variable??
invoke-static {v8,v9},java/lang/Math/round ; round(D)J
move-result-wide v8

// v10 = 1000
const-wide/16 v10,1000

// v8 += v10;
add-long/2addr v8,v10

// badge.append(v8);
invoke-virtual {v7,v8,v9},java/lang/StringBuilder/append ; append(J)Ljava/lang/StringBuilder;
move-result-object v7

// badge.append(dateFormat.format(now));
invoke-virtual {v5,v2},java/text/SimpleDateFormat/format ; format(Ljava/util/Date;)Ljava/lang/String;
move-result-object v8
invoke-virtual {v7,v8},java/lang/StringBuilder/append ; append(Ljava/lang/String;)Ljava/lang/StringBuilder;
move-result-object v7

// badge.append(difficulty);
iget v8,v12,tx/games/atd_world.difficulty I
invoke-virtual {v7,v8},java/lang/StringBuilder/append ; append(I)Ljava/lang/StringBuilder;
move-result-object v7

// badge.append(sum);
invoke-virtual {v7,v6},java/lang/StringBuilder/append ; append(I)Ljava/lang/StringBuilder;
move-result-object v7

// return badge.toString();
invoke-virtual {v7},java/lang/StringBuilder/toString ; toString()Ljava/lang/String;
move-result-object v1
return-object v1
.end method
```
An example of the output of this function is;
_1310090403121501473_

Broken down the output looks like this;
**1310**090403121501473, (round(random * const)+1000
1310**0904031215**01473, Date in yyMMddhhmm format.
13100904031215**0**1473, "0" Difficulty, Noob = 0, Normal = 1, Pro = 3
131009040312150**1473**, sum of bytes (date + difficulty + "tensaix2")

I'll post more later if the "badge system" is every finished and released. Hopefully this serves as a decent example on how to reverse simple android programs... Enjoy!