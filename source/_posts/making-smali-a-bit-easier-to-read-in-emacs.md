---
title: Making smali a bit easier to read in Emacs
tags:
  - android
  - baksmali
  - color scheme
  - emacs
  - reverse engineering
  - smali
  - smali-mode
url: making-smali-a-bit-easier-to-read-in-emacs.html
id: 419
categories:
  - android
  - reverse engineering
date: 2012-02-05 16:06:22
---

![Oh that? Just some pretty colors while reversing...](http://173.230.150.16/blog/wp-content/uploads/2012/02/smali_coloring.png "Oh that? Just some pretty colors for smali...")
Spending a large amount of time using baksmali and reading through small files can be rather dull without the right tools. A while back I noticed a few people making color scheme files for vim, Notepad+ and other tools I didn't use. So after reading up on a quick few tutorials I created a smali-mode for Emacs.

The code is up on my [github](https://github.com/strazzere/Emacs-Smali), I haven't actually touched it in a while until last night. I noticed a few other people actually pulled clones of it and made a few minor fixes! Some good fixes too, making the loading much faster and fixing some things I wasn't too sure about myself for making an Emacs mode.

Anyway - hopefully other people find this useful and maybe more people will contribute to the project. For people looking for the other color schemes here are some of those resources;

Jon Larimer's vim scheme;
 [http://codetastrophe.com/smali.vim](http://codetastrophe.com/smali.vim)
Lohan+'s Notepad+ color scheme;
 [https://sites.google.com/site/lohanplus/files/smali_npp.xml](https://sites.google.com/site/lohanplus/files/smali_npp.xml) ([Directions here](http://androidcracking.blogspot.com/2011/02/smali-syntax-highlighting-for-notepad.html))