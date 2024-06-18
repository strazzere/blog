---
title: An update on the Archos 5 Android rooting...
tags:
  - android
  - Android root
  - androidroot.cramfs
  - androidroot.cramfs.secure
  - Archo 5 Android
  - archos
  - Archos 5 Internet Tablet
  - archos init
  - archos recovery
  - Archos root
  - bootloader
  - cramfs
  - cramfs.secure
  - GPL
  - kernel
  - mtd_partition
  - MTD_WRITEABLE
  - root
  - stage0
  - stage1
url: an-update-on-the-archos-5-android-rooting.html
id: 320
categories:
  - android
  - archos
  - coding
  - reverse engineering
  - updating
date: 2009-11-20 17:00:08
---

![Penguin inside ;)](http://173.230.150.16/blog/wp-content/uploads/2009/11/archosloading-300x225.jpg "Penguin inside ;)")
Progress has been quiet slow as of lately, albeit progress none the less though. As normal, I figured I'd attempt to document this so that anyone attempting to root devices in the future that might be similar to the Archos might have an easier time :)

Thus far when rooting phones we have been coming in contact with less protection. After gaining root we have been able to remount the partitions and edit them directly - making the changes persistent immediately. Though there is a fundamental difference with the Archos that comes into play. The Archos stores everything on a (maybe two or three?) flash chips. The partitions of the chips are like normal phones and mounted as read-only. The problem comes into play since what is mounted is not a filesystem, but a cramfs file. This cramfs file is what needs to be modified to create any changes to the system files.

So just modify the cramfs file, right? Nope - Archos caught you on this one. Simply modifying the cramfs, which is actually a `cramfs.secure` file will lead to a bad signature error. What the heck is that all about? Essentially when Archos creates a firmware update and flashes a new `androidroot.cramfs.secure` to the device. This file is signed with a signature of itself. Sadly we cannot recreate the signature for the file since it's an RSA/MD5 signature of the contents. This means that they basically run a program after the have a androidroot.cramfs to append the signature like the following:
```
secure = RSA(MD5(cramfsFile)) + cramfsFile;
```
The RSA function uses a private key that we do not, and most likely will never have. Essentially in the bootloader there is a program called cramfschecker that does something like the following pseudo code:
```
if(RSADecrypt(signature) == MD5(file)) ? return goodFile : return badFile;
```
The RSADecrypt function uses a public key that we have [found](http://strazzere.com/blog/?p=311), but is no help to us.
Alright, know that we know what files need to get modified and how we are locked out of them, how do we actually get past this? This is where we have to get into modifying the kernel and boot loader. The flash memory is a little tricky but essentially is mapped out like the following:
```
mtd device – name —– nickname – description
mtd0 ——- stage1 — boot0 —- bootloader part 1, also contains keystore
mtd1 ——- stage2 — boot1 —- bootloader part 2, also contains boot image
mtd2 ——- recovery – recovery – recovery kernel and recovery.cpio (filesystem)
mtd3 ——- init —– init —– init (main) kernel and init.cpio (filesystem)
```
Now here is where the cool stuff comes into play. Stage1, among other things, checks the signature of stage2 to verify that it has not been modified. When stage2 begins it performs the same type of check on recovery and init. Inside recovery and init a program called cramfschecker is called, which checks the actual cramfs.secure files that we want to change. So the chain of trust is as follows:
`Stage1 -> Stage2 -> recovery/init -> cramfs.secure`
We need to modify Stage1 to accept any stage2, stage2 to accept any recovery/init and then remove the cramfschecker call so we can execute anything we'd like without worrying about if it is signed or not.

Now we know everything that needs to be done, so lets do it! Well, it's sadly not _that_ easy. We know how and can modify the cramfs files, that's not hard. We can flash new a recovery/init, and even flash a new stage2. The problem is that we cannot currently flash a stage1 since it is marked as read-only after boot by the kernel. Yes, it is marked read-only, not locked - which if it where we could simply use a `flash_unlock` tool on it.

Currently I've been diving into the init kernel, which is at the beginning part of the init section, gzipped. This has been pretty tough trudging and I've enlisted the help of [EiNSTeiN_](http://archos.g3nius.org). This is still pretty ugly stuff to look through though - we are basically looking for a small struct that make uses the kernel module to set the partition to read-only. The struct should look something like this:
```
struct mtd_partition {
	char name; 				/* identifier string */
	uint64_t size; 				/* partition size */
	uint64_t offset; 			/* offset within the master MTD space */
	// Probably set to MTD_WRITEABLE (0x0400), since it is MASKING this flag
	uint32_t mask_flags; 			/* master MTD flags to mask out for this partition */
	struct nand_ecclayout *ecclayout; 	/* out of band layout for this partition (NAND only)*/
};
```
Though this should all be GPLed code - since it is the kernel! Ah hah! That could make things _so much easier_. Sadly Archos has not yet posted the [GPLed kernel source code](http://www.archos.com/support/support_tech/updates.html?country=us&lang=en) for the Gen7 devices (which the Android model is).

After about two to three weeks of trying to track down someone, anyone with an Archos contact, or even just someone at Archos who isn't outsourced technical support I finally got an answer! Prior to reaching this person I mainly got the run around, saying it will be up soon or that it is already posted. For some reason it also kept getting lost in the translation that I was requesting the GPLed \*kernel\* source code from the Archos 5 Android model. Someone in France apparently kept seeing "Android" and said "NO! It's Apache, we don't have to release that, Google hosts the source, goto them!" _Finally_ I got a response, rather promptly I might add, from a USA Archos representative saying that Google hosts the code. After exchanging a few more emails they finally understood that I was requesting the Gen7 kernel source code, which is under the GPL license - NOT the Android source code which is under Apache. PHEW!

So the latest update is that we are essentially in a holding pattern, waiting for next week to come. I've been promised that the GPLed source code for the kernel _will_ be posted by the end of next week, though I shouldn't hold my breath until Friday. If it doesn't appear on the site by Friday evening EST then I can start calling and complaining again... This time someone can actually be help responsible though, so I feel like it will actually happen. Once we get this code, it's only a matter of time before EiNSTeiN_ and myself track down the right code which should help us in creating a program to patch the mtd partitions into being read/writable.

If you feel like you can help us with this, feel free to post here, email me or send a reply on twitter. Also if you just want to get the most updated information, I'd recommend you follow me on twitter [@timstrazz](http://www.twitter.com/timstrazz).