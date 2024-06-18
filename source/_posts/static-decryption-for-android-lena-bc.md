---
title: 'Static decryption for Android LeNa.[b/c] using IDA Pro'
tags:
  - android
  - decryption
  - IDA Pro
  - IDC
  - legacy native
  - LeNa
  - malware
  - reverse engineering
  - reversing
url: static-decryption-for-android-lena-bc.html
id: 549
categories:
  - android
  - reverse engineering
date: 2012-04-08 14:25:24
---

Recently at Lookout we blogged about a newer flavor of [Legacy Native (LeNa)](http://blog.lookout.com/blog/2012/04/03/security-alert-new-variants-of-legacy-native-lena-identified/). This variant is extremely similar to the other ones we've found in the past and [blogged about in October of 2011](http://blog.lookout.com/blog/2011/10/20/security-alert-legacy-makes-a-another-appearance-on-android-market-meet-legacy-native-lena/), the full tear down I wrote can be found in [pdf form here](http://blog.lookout.com/wp-content/uploads/2011/10/LeNa-Legacy-Native-Teardown_Lookout-Mobile-Security1.pdf). The samples for both LeNa.b and LeNa.c have been added to the [Contagio Mini Dump](http://contagiominidump.blogspot.com/2012/03/android-dkfbootkit-aka-lenab.html)

While going through some samples I decided to throw together a quick IDC script for IDA Pro to help decrypt the commands and variables without executing the code. The decryption isn't hard, in fact it actually ends up just being an XOR with 0xFF. Though if anyone hasn't ever dealt with IDA Pro or ARM, it might look a bit confusing at first. Below is the decryption function commented from one of specific samples of LeNa;
![Simple XOR](http://www.strazzere.com/blog/wp-content/uploads/2012/04/decryption-function.png "LeNa Decryption Function")
The IDC script is really simple, the current version can be found [on github](https://github.com/strazzere/LeNa-Decryption-Script) with the current version featured below;

```
/* LeNa-Decryption.idc
 
   Simple decryption script for the LeNa.b/.c variants.
 
   Designed for use on the elf binary that LeNa drops, it works by decrypting the
   encrypted string and dropping the decrypted bytes as a comment where ever the
   string's pointer is used.
 
   The decryption is really easy (a MVNS), basically a xor to 0xFF
 
   Timothy Strazzere <Tim.Strazzere@lookout.com>
   March 2012
*/
 
#include <idc.idc>
 
static decrypt_string(void)
{
    auto original_address, ptr, comment;
 
    // This is normally a pointer to a pointer, follow it
    original_address = ptr = Dword(ScreenEA());
 
    // Make sure we retrieved a good address
    if (ptr == BADADDR) {
       Message("Unable to find a actual encrypted string's address!\n");
       return;
    }
 
    // XOR until we hit the null byte of the string
    comment = "";
    while(Byte(ptr) != 0x00) {
        comment = comment + sprintf("%s", Byte(ptr) ^ 0xFF);
        ptr++;
    }
 
    // Drop a comment at the address we originally where pointing too
    MakeRptCmt(original_address, "Decrypted value:  " + comment);
 
    // Drop a comment on all xrefs to this
    auto xref = Dfirst(original_address);
    while(xref != BADADDR) {
        MakeRptCmt(xref, "Decrypted value: " + comment);
 xref = Dnext(original_address, xref);
    }
 
    // Drop a message in the output window
    Message("Decrypted string: %s\n", comment);
}
 
static main(void) {
    // Set hotkey for decryption routine
    if(AddHotkey('/', "decrypt_string") != IDCHK_OK) {
        Message("Something went wrong with adding a hotkey for the decryption routine");
    }
    Message("Hotkey added!\n");
}
```
The script should be really easy to follow, I tried to comment it well enough. Essentially when you load the script it will bind the function for decrypting to the "/" key. Find the pointer to the encrypted data like below;
![Highlighted pointer to the encrypted data](http://www.strazzere.com/blog/wp-content/uploads/2012/04/encrypted_pointer.png "Highlighted pointer to the encrypted data")
Next just follow the pointer to it's selection in the text section. Once the encrypted pointer is highlighted, simply hit the assigned hot-key "/" and let the script do it's job. The script will dump the decrypted message to the output window and also add a comment next to the pointer as seen below;
![](http://www.strazzere.com/blog/wp-content/uploads/2012/04/decrypted-1.png)
The script also traverses back to all the cross references (xrefs) and will add a comment to all those spots where this variable is used.
![The original usage in the decryption function](http://www.strazzere.com/blog/wp-content/uploads/2012/04/decrypted-2.png "The original usage in the decryption function")
![The main use of the decrypted data in another function](http://www.strazzere.com/blog/wp-content/uploads/2012/04/decrypted-3.png "The main use of the decrypted data in another function")
Nothing too fancy or complicated, though it was a nice way to get into IDC scripts for IDA Pro. Also it's a good way to start segwaying people who may only have used baksmali or dex2jar into using IDA Pro for ARM reversing.