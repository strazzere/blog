---
title: Dex File Header Dump
tags:
  - bytecode
  - code
  - dalvik vm
  - dex bytecode
  - dex file
  - dex header
  - java
  - reverse engineering
url: 109.html
id: 109
categories:
  - android
  - coding
  - dex bytecode
  - reverse engineering
date: 2008-11-22 21:27:40
---

I've been working on the header a little more - so I figured I'd post some code I just finished throwing together quickly. It's not all the code, since most of it is experimental and I'm not finished doing it, but this will provide people with the information on how to dump the dex file header information.
```
/* File: DexNfo.java
 *
 * Coded: Timothy Strazzere
 * Date: 11/22/08
 *
 * Dump header information from a dex file, only supports '035' dex files, though will
 * attempt to dump rest of the information, but will just warn you otherwise.
 *
 * Some code has been removed as it isn't sure if it full works properly yet.
 *
 */


import java.security.*;
import java.util.zip.Adler32;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;

/* To do...
 *
 * lots...
 *
 */
public class DexNfo{

	public static void main(String[] args) {
        if (args.length == 1) {
        	try {
        		File file = new File(args[0]);

        		byte[] barr, newbytes = null;
        		newbytes = barr = getBytesFromFile(file);

        		// add switch for this?
        		System.out.printf("Original information: " + args[0]);

        		int magic = 0;

        		for(int i = 0; i<8; i++)
        			magic+=barr[i];

        		if(magic!=483) // technically anything higher should be a new dex file... 'dex 036' etc..
        			System.out.printf("\n** Warning: Magic; bad dex file or unsupported version loaded!");

        		// Don't output char 4 since it's a newline char
        		System.out.printf("\nMagic: %c%c%c %c%c%c", barr[0], barr[1], barr[2], /*barr[3],*/ barr[4], barr[5], barr[6], barr[7]);

        		System.out.print("\nChecksum: ");
        		for(int i = 8; i<12; i+=4)
        			System.out.printf("0x%02X%02X%02X%02X ", barr[i+3], barr[i+2], barr[i+1], barr[i]);

        		System.out.print("\nSignature: 0x");
        		for(int i = 12; i<32; i+=4)
        			System.out.printf("%02X%02X%02X%02X ", barr[i], barr[i+1], barr[i+2], barr[i+3]);

        		System.out.printf("\nLength: 0x%02X%02X", barr[33], barr[32]);

        		if(barr[36]!= 112) // currently is always 0x70==(int)112...
        			System.out.printf("\n** Warning: Header Length; bad dex file or unsupported version loaded!");
        		System.out.printf("\nHeader Length: 0x%02X", barr[36]);

        		// endian tag
        		int endian = 0;

        		for(int i = 0; i < 4; i++)
        			endian += barr[40+i];

        		if(endian != 276) // Currently always should be 0x78563412, which when added = int 114
        			System.out.printf("\n** Warning: Endian Tag; bad dex file or unsupported version loaded!");
        		System.out.printf("\nEndian Tag: 0x%02X%02X%02X%02X", barr[40],
        				barr[41], barr[42], barr[43]);

        		// map offset
        		System.out.printf("\nMap Offset: 0x%02X%02X", barr[53], barr[52]);

        		// string table size
        		System.out.printf("\nString Table Size: 0x%02X%02X", barr[57], barr[56]);

        		// string table offset
        		System.out.printf("\nString Table Offset: 0x%02X%02X", barr[61], barr[60]);

        		// type table size
        		System.out.printf("\nType Table Size: 0x%02X%02X", barr[65], barr[64]);

        		// type table offset
        		System.out.printf("\nType Table Offset: 0x%02X%02X", barr[69], barr[68]);

        		// Prototype table size
        		System.out.printf("\nPrototype Table Size: 0x%02X%02X", barr[73], barr[72]);

        		// Prototype table offset
        		System.out.printf("\nPrototype Table Offset: 0x%02X%02X", barr[77], barr[76]);

        		// Field table size
        		System.out.printf("\nField Table Size: 0x%02X%02X", barr[81], barr[80]);

        		// Field table offset
        		System.out.printf("\nField Table Offset: 0x%02X%02X", barr[85], barr[84]);

        		// Method table size
        		System.out.printf("\nMethod Table Size: 0x%02X%02X", barr[89], barr[88]);

        		// Method table offset
        		System.out.printf("\nMethod Table Offset: 0x%02X%02X", barr[93], barr[92]);

        		// Class table size
        		System.out.printf("\nClass Table Size: 0x%02X%02X", barr[97], barr[96]);

        		// Class table offset
        		System.out.printf("\nClass Table Offset: 0x%02X%02X", barr[101], barr[100]);

        		System.out.println();

        		// add switch for this too?

        		calcSignature(newbytes);
        		calcChecksum(newbytes);
        		System.out.print("\n\nNew Checksum: ");
        		for(int i = 8; i<12; i+=4)
        			System.out.printf("0x%02X%02X%02X%02X ", newbytes[i+3], newbytes[i+2], newbytes[i+1], newbytes[i]);
        		System.out.print("\nNew Signature: 0x");
        		for(int i = 12; i<32; i+=4)
        			System.out.printf("%02X%02X%02X%02X ", newbytes[i], newbytes[i+1], newbytes[i+2], newbytes[i+3]);

        		System.out.printf("\nLength: %04X", calcSize(newbytes));

        		// output compares to the two, highlight differences...
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
        while (offset < bytes.length
               && (numRead=is.read(bytes, offset, bytes.length-offset)) >= 0) {
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
                throw new RuntimeException((new StringBuilder()).append("unexpected digest write: ").append(amt).append(" bytes").toString());
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

    public static int calcSize(byte bytes[])
    {
    	return(bytes.length);
    }
}
```
This now dumps all the header information from the original file, and will recalculate the signature and checksum in case something has changed. A version should be available shortly to check for differences in all the values, hopefully soon being able to calculate the correct values if something is wrong.
Maybe this will be useful for someone? Otherwise, oh well it's just here in case I delete my files. Working on functions to find the new values after patching and to allow patching/injection of code. I'll have to write up more later as I don't have an overwhelming amount of time right now, busy day and I'm exhausted. Saw Sara play some volleyball, finished up solo campaign in COD5, spent a few hours reading and researching some dex related things and trying to get some more injection to work. Tomorrow I probably won't have time to post - but trust me, this stuff will be up sooner or later. It's a big puzzle I'm chipping away at, and it's bugging the heck out of me not having the answers.