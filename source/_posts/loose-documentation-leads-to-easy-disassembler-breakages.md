---
title: Loose Documentation Leads to Easy Disassembler Breakages
tags:
  - android
  - dex header
  - dex verification
  - documentation
  - IDA Pro
  - radare
  - reverse engineering
url: compiling-an-emulator-kernel-for-loadable-modules.html
id: 707
categories:
  - android
  - dex bytecode
  - reverse engineering
date: 2013-02-15 18:23:41
---

As people have seen in the past, I tend to have a fun time finding edge-cases which break tools. Often you can find these types of edge-cases while reading documentation and cross referencing the implementation of that in the systems your validating. A pretty good example of this is highlighted in my BlackHat 2012 talk, where I was looking at the header section, which is described as always have the value of 0x70. When looking at the open source tools, some checked to make sure this was true - others ignored it. The actual code in the [Dex Verifier](http://androidxref.com/4.2.2_r1/xref/dalvik/libdex/DexSwapVerify.cpp#2888) is as follows;

```
if (okay) {
    state.pHeader = pHeader;

    if (pHeader->headerSize < sizeof(DexHeader)) {
        ALOGE("ERROR: Small header size %d, struct %d",
                pHeader->headerSize, (int) sizeof(DexHeader));
        okay = false;
    } else if (pHeader->headerSize > sizeof(DexHeader)) {
        ALOGW("WARNING: Large header size %d, struct %d",
                pHeader->headerSize, (int) sizeof(DexHeader));
        // keep going?
    }
}
```

From this we can see the actual implementation doesn't care what the size of, as long as it is larger than the current structure size, which is 0x70. This allows for the verifier to be forward compatible, though if anyone was creating a tool and only read the documentation - this might not be fully understood or assumed. This leads me to two extremely easy breakages which I never mentioned in my talk, but noticed IDA Pro 6.4 and Radare would fail against. The issue that IDA Pro and Radare broke against, was a bad file magic. According to the documentation the magic bytes are the following;

```
DEX_FILE_MAGIC
embedded in header_item The constant array/string DEX_FILE_MAGIC is the list of bytes that must appear at the beginning of a .dex file in order for it to be recognized as such. The value intentionally contains a newline ("\n" or 0x0a) and a null byte ("\0" or 0x00) in order to help in the detection of certain forms of corruption. The value also encodes a format version number as three decimal digits, which is expected to increase monotonically over time as the format evolves.

ubyte[8] DEX_FILE_MAGIC = { 0x64 0x65 0x78 0x0a 0x30 0x33 0x35 0x00 } = "dex\n035\0"

Note: At least a couple earlier versions of the format have been used in widely-available public software releases. For example, version 009 was used for the M3 releases of the Android platform (November–December 2007), and version 013 was used for the M5 releases of the Android platform (February–March 2008). In several respects, these earlier versions of the format differ significantly from the version described in this document.
```

So one might assume that the currently accepted magic bytes will be exactly `dex\n035\00` - though, they would be wrong in assuming this. If we take a look at the code in [DexFile.h](http://androidxref.com/4.2.2_r1/xref/dalvik/libdex/DexSwapVerify.cpp#2785);

```
/* DEX file magic number */
#define DEX_MAGIC      "dex\n"

/* current version, encoded in 4 bytes of ASCII */
#define DEX_MAGIC_VERS  "036\0"

/*
 * older but still-recognized version (corresponding to Android API
 * levels 13 and earlier
 */
#define DEX_MAGIC_VERS_API_13  "035\0"

...

/* (documented in header file) */
bool dexHasValidMagic(const DexHeader* pHeader)
{
    const u1* magic = pHeader->magic;
    const u1* version = &magic[4];

    if (memcmp(magic, DEX_MAGIC, 4) != 0) {
        ALOGE("ERROR: unrecognized magic number (%02x %02x %02x %02x)",
            magic[0], magic[1], magic[2], magic[3]);
        return false;
    }

    if ((memcmp(version, DEX_MAGIC_VERS, 4) != 0) &&
            (memcmp(version, DEX_MAGIC_VERS_API_13, 4) != 0)) {
        /*
         * Magic was correct, but this is an unsupported older or
         * newer format variant.
         */
        ALOGE("ERROR: unsupported dex version (%02x %02x %02x %02x)",
            version[0], version[1], version[2], version[3]);
        return false;
    }

    return true;
}
```

We can see that there are constant magic bytes of `dex\n`, but the versioning afterwards - which is loosely explained in the documentation, has multiple options. Since API level 14 on, the verifier has accepted both "036\00" and "035\00" as valid versioning parts of the magic bytes. Since the magic bytes are not part of the checksum or the signature of the dex file, one can simply bump the version number without any specialized tools, just doing it with a hex editor would be fine. This lead to Radare failing to load the file and IDA Pro to thinking the file was corrupt with the following dialog and log output; [![Corrupt File Dialog](http://www.strazzere.com/blog/wp-content/uploads/2013/02/corrupt-file1.png)](http://www.strazzere.com/blog/wp-content/uploads/2013/02/corrupt-file1.png)
```
Loading file '/Users/tstrazzere/reverse/targets/ida/classes-test.dex' into database...
Detected file format: Android DEX file version 54.0
bad dex version (0x30 33 36 00)
```
I originally reported this issue to January 22nd, 2013 and received a thank you and a fix back from them only two days later on the 24th. I'm unsure if they sent this out to all their customers or have it totally bundled into their latest packages, but you should easily be able to request it if not. For Radare I [submitted a patch](https://github.com/strazzere/radare2/commit/3fcc031083016385fa201b3885896c758393d7f4) for this issue which was quickly merge upstream by the extremely proactive author of the tool. The second breakage, which only directly effected IDA Pro, was revolving around the file size as dictated by the dex_header vs the actual file size. IDA Pro was comparing the two, and if they where not actually equal - assumes the file is corrupt. The documentation states, "size of the entire file (including the header), in bytes", though the implementation of the code doesn't actually care - as seen from the [DexSwapVerify.cpp](http://androidxref.com/4.2.2_r1/xref/dalvik/libdex/DexSwapVerify.cpp#2836) file;

```
if (okay) {
    int expectedLen = (int) SWAP4(pHeader->fileSize);
    if (len < expectedLen) {
        ALOGE("ERROR: Bad length: expected %d, got %d", expectedLen, len);
        okay = false;
    } else if (len != expectedLen) {
        ALOGW("WARNING: Odd length: expected %d, got %d", expectedLen,
                len);
        // keep going
    }
}
```
As we can see from above, if the actual length must be at least as large as the expected length, most likely to avoid any truncated files. Though it can easily be larger, which will just produce a warning - though processing of the dex file will continue. However, the same corrupt file dialog with this logging message comes up when loaded in IDA Pro; [![Corrupt File Dialog](http://www.strazzere.com/blog/wp-content/uploads/2013/02/corrupt-file1.png)](http://www.strazzere.com/blog/wp-content/uploads/2013/02/corrupt-file1.png)
```
Loading file '/Users/tstrazzere/reverse/targets/ida/classes-test.dex' into database...
Detected file format: Android DEX file version 53.0
ERROR: stored file size (438844) != expected (438845)
```
This was also fixed on the same timeline as the other issue I reported to Hex-Rays, so if you run across any files like this you will be prompted with this dialog; [![Extra data](http://www.strazzere.com/blog/wp-content/uploads/2013/02/extra_info.jpg)](http://www.strazzere.com/blog/wp-content/uploads/2013/02/extra_info.jpg) Just two small little issues that came about when looking at the implementation of the file format. These edge-cases always seem to exist in ever system, especially when creating reversing/disassembling/analyzing tools.