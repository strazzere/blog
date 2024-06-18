---
title: Promising result for injecting code...
tags:
  - android
  - dex
  - dex code
  - file format
  - inject code
  - injection
  - offset
  - reverse engineering
url: promising-result-for-injecting-code.html
id: 102
categories:
  - android
  - dex bytecode
  - reverse engineering
date: 2008-11-20 17:08:24
---

It's coming along, but it doesn't seem to be as easy as I'd have hoped. Sort of have a working example but I don't want to release it until I can definitely identify what needs to be patched and why and other things like exactly by how much etc for things to be injected. Just a little output of some of my notes from the tests I've been running. Nothing to mind blowing but some notes incase someone is interested, slash incase I lose the piece of paper;

```
Things you must patch to successively inject code:

Length of file in bytes (0x20)
Absolute offet of string table (0x34)
type of checksum? (0x38)
number of fields in field table (0x44)
Absolute offet to field table (0x48)
number of methods in method table (0x4C)
absolute offset of method table (0x50)
another checksum? (0x54)
absolute offset of class definition? (0x58)
```