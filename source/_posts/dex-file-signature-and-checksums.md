---
title: DEX File signature and checksums
tags:
  - android
  - bytecode
  - dex
  - java
  - reverse engineering
url: dex-file-signature-and-checksums.html
id: 416
categories:
  - android
  - dex bytecode
  - reverse engineering
date: 2008-10-30 02:49:03
---

DEX files, which are the compiled bytecode files that run on the Android OS. Essentially they are like the java bytecode, except they use a modified VM which is called "Dalvik" that was developed for the Android OS.

So long story short - I want to know the structure of these files, and how to edit or patch them. Why? I enjoy reverse engineering things! Anyway there is a wealth of information that I could find on the following sites;

Shane Isbell's Weblog : [http://www.jroller.com/random7/entry/android_s_dex_class_structure](http://www.jroller.com/random7/entry/android_s_dex_class_structure) RetroCode, Dex File Formate : [http://www.retrodev.com/android/dexformat.html](http://www.retrodev.com/android/dexformat.html )

These site both have great information, thoughmore so on the second link. What intrigued me was that they must have a 'checksum' and a 'SHA-1' signature. The best information I could find though only eluded to this (Retrocode);
>**Notes:** All non-string fields are stored in little-endian format. It would appear that the checksum and signature fields are assumed to be zero when calculating the checksum and signature.

Well that doesn't really help me if I'd like to patch a DEX file and then redo the signature and checksum. Especially since we don't know what exactly is being used to calculate either or what checksum is being used to well, calculate the checksum! Good thing we can extract .jar files and decompile class files, even more so thank goodness google hasn't used obstufication on any of the classes. An excerpt from `dx\com\android\dx\dex\file\DexFile.class` after being decompiled into DexFile.jar.
```
private static void calcSignature(byte bytes[])
{
  MessageDigest md;
  try
  {
    md = MessageDigest.getInstance("SHA-1");
  }
  catch(NoSuchAlgorithmException ex)
  {
    throw new RuntimeException(ex);
  }
  md.update(bytes, 32, bytes.length - 32);
  try
  {
    int amt = md.digest(bytes, 12, 20);
    if(amt != 20)
      throw new RuntimeException((new StringBuilder()).append("unexpected digest write:").append(amt).append("bytes").toString());
  }
  catch(DigestException ex)
  {
    throw new RuntimeException(ex);
  }
}

private static void calcChecksum(byte bytes[])
{
  Adler32 a32 = new Adler32();
  a32.update(bytes, 12, bytes.length - 12);
  int sum = (int)a32.getValue();
  bytes[8] = (byte)sum;
  bytes[9] = (byte)(sum >> 8);
  bytes[10] = (byte)(sum >> 16);
  bytes[11] = (byte)(sum >> 24);
}
```
Ah hah! Now we know how to calculate the signature and the checksum. Essentially is reads in all the bytes of the program, and it disreguards the first 32 bytes and calculates the signature using SHA-1. Then it calculates the checksum disreguarding the first 12 bytes (so it includes the signature). Excellent, lets drop this code into our own program so we can recalculate those values;
```
/*
*         ReDEX.class
*         Coded: Timothy Strazzere
*
*         10/29/2008
*
*         Pass a .dex file as an argument and it will output the current
*         checksum and signature that is in the file, then it will output
*         the checksum and signature as it calculates them.
*
*/

import java.security.*;
import java.util.zip.Adler32;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

public class ReDEX {

public static void main(String[] args) {
  if (args.length == 1) {
    try {
      File file = new File(args[0]);

      byte[] barr = null;
      barr = getBytesFromFile(file);

      System.out.print(:Original Checksum: ");
      for(int i = 8; i<12; i+=4)
      System.out.printf("0x%02X%02X%02X%02X ", barr[i+3], barr[i+2], barr[i+1], barr[i]);
      System.out.print("\nOriginal Signature: 0x");
      for(int i = 12; i<32; i+=4)
      System.out.printf"%02X%02X%02X%02X ", barr[i], barr[i+1], barr[i+2], barr[i+3]);

      calcSignature(barr);
      calcChecksum(barr);
      System.out.print("\n\nNew Checksum: ");
      for(int i = 8; i<12; i+=4)
        System.out.printf("0x%02X%02X%02X%02X ", barr[i+3], barr[i+2], barr[i+1], barr[i]);
      System.out.print("\nNew Signature: 0x");
      for(int i = 12; i<32; i+=4)
      System.out.printf("%02X%02X%02X%02X ", barr[i], barr[i+1], barr[i+2], barr[i+3]);
    }
    catch (Exception e) {
      System.err.println("File input error");
    }
  }
  else
    System.out.println("Invalid parameters");
}

public static byte[] getBytesFromFile(File file) throws IOException {
  InputStream is = new FileInputStream(file);

  // Get the size of the file
  long length = file.length();

  if (length > Integer.MAX_VALUE) {
    // File is too large
  }

  // Create the byte array to hold the data
  byte[] bytes = new byte[(int)length];

  // Read in the bytes
  int offset = 0;
  int numRead = 0;
  while (offset < bytes.length && (numRead=is.read(bytes, offset, bytes.length-offset)) >= 0) {
    offset += numRead;
  }

  // Ensure all the bytes have been read in
  if (offset < bytes.length) {
    throw new IOException("Could not completely read file "+file.getName());
  }

  // Close the input stream and return bytes
  is.close();
  return bytes;
}
private static void calcSignature(byte bytes[])
{
  MessageDigest md;
  try
  {
    md = MessageDigest.getInstance("SHA-1");
  }
  catch(NoSuchAlgorithmException ex)
  {
    throw new RuntimeException(ex);
  }
  md.update(bytes, 32, bytes.length - 32);
  try
  {
    int amt = md.digest(bytes, 12, 20);
    if(amt != 20)
      throw new RuntimeException((new StringBuilder()).append("unexpected digest write:").append(amt).append("bytes").toString());
  }
  catch(DigestException ex)
  {
    throw new RuntimeException(ex);
  }
}

private static void calcChecksum(byte bytes[])
{
  Adler32 a32 = new Adler32();
  a32.update(bytes, 12, bytes.length - 12);
  int sum = (int)a32.getValue();
  bytes[8] = (byte)sum;
  bytes[9] = (byte)(sum >> 8);
  bytes[10] = (byte)(sum >> 16);
  bytes[11] = (byte)(sum >> 24);
}
```
Excellent, now I can easily recalculate the signature and the checksum of the dex files. Note that the checksum is actually in little-endian so you need to reverse it when entering it in a hex editor.

So, now the Android OS will allow you to install this dex file, after signing it in the correct package (apk). Though as of right now the program still crashes upon launching the file... Hhhmmmmm we'll have to look into this more I guess.