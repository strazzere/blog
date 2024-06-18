---
title: Dexdump alternatives...
tags:
  - alternatives
  - bytecode
  - Dedexer
  - dex
  - dex bytecode
  - dexdump
  - Gabor
  - jasmin
  - java
  - reverse engineering
url: dexdump-alternatives.html
id: 165
categories:
  - android
date: 2009-01-12 12:50:10
---

Thanks to my friend Gabor, over at [http://mylifewithandroid.blogspot.com/](http://mylifewithandroid.blogsport.com/) has created a really well done dex file dissembler. The direct link for the post is [here](http://mylifewithandroid.blogspot.com/2009/01/disassembling-dex-files.html) and the source code is all free and located at [dedexer.sourceforge.net](http://dedexer.sourceforge.net/).

It's nice as it outputs the format in jasmin like the following;
```
.method public calc1(I)I
    packed-switch    v2,0
        ps418_422    ; case 0
        ps418_426    ; case 1
        ps418_42a    ; case 2
        default: ps418_default
ps418_default:
    const/4    v0,15
l420:
    return    v0
ps418_422:
    const/4    v0,2
    goto    l420
ps418_426:
    const/4    v0,5
    goto    l420
ps418_42a:
    const/4    v0,6
    goto    l420
    nop
.end method
```

Opposed to the normal;
```
000418: 2b02 0c00 0000                         |0000: packed-switch v2, 0000000c // +0000000c
00041e: 12f0                                   |0003: const/4 v0, #int -1 // #ff
000420: 0f00                                   |0004: return v0
000422: 1220                                   |0005: const/4 v0, #int 2 // #2
000424: 28fe                                   |0006: goto 0004 // -0002
000426: 1250                                   |0007: const/4 v0, #int 5 // #5
000428: 28fc                                   |0008: goto 0004 // -0004
00042a: 1260                                   |0009: const/4 v0, #int 6 // #6
00042c: 28fa                                   |000a: goto 0004 // -0006
00042e: 0000                                   |000b: nop // spacer
000430: 0001 0300 faff ffff 0500 0000 0700 ... |000c: packed-switch-data (10 units)
```

Great work Gabor, and keep up the good work!