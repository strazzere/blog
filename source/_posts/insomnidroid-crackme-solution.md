---
title: InsomniDroid - crackme solution
tags:
  - android
  - crackme
  - insomnidroid
  - insomnihack ctf
  - reverse engineering
  - solution
url: insomnidroid-crackme-solution.html
id: 488
categories:
  - android
  - reverse engineering
date: 2012-03-07 05:00:39
---

![not-done](http://www.strazzere.com/blog/wp-content/uploads/2012/03/not-done.png "not done")
It's a bit amusing that this solution for the _InsomniHack_ CTF challenge named _"InsomniDroid"_ was written up past midnight because I couldn't sleep. Regardless, this was typed up as a play-by-play analysis taken from my crib notes when I actually was solving the crackme, so try to bare with me as this may read a bit odd. While some of the steps might seem odd - I find it is often just easier to tackle each APK in the same manner. This sets up a nice way to quickly find the call-stack and how things get executed and where. When tackling this challenge, I attempted to only use _baksmali_, opposed to any other tools for simplicity and since not everyone has IDA Pro. Anyway, if you haven't downloaded the challenge yet you can get it from [root-me.org](http://www.root-me.org/en/Challenges-148/Realist/Insomni-Droid.html) or from this [local mirror](http://www.strazzere.com/crackmes/insomnidroid.apk) (MD5: c2f94fd52c8f270e66d8b16e07fe93e4). If you haven't solved the challenge yet, I'd recommend to stop reading here and give it a good try. Starting off, we shoudl take a quick look at the AndroidManifest.xml through AXMLPrinter shows which is the main activity;
```
<activity android:label="@7F040001" android:name=".InsomniActivity">
	<intent-filter>
		<action android:name="android.intent.action.MAIN"></action>
		<category android:name="android.intent.category.LAUNCHER">
		</category>
	</intent=filter>
</activity>
```
Now we see what where to start our search without opening the app yet, toss that crackme into baksmali and get ready for the output. No baksmali tricks in place, so we can just take a look right at the main activity, _InsomniActivity_. The only interesting bits for us is the call to _compute_ method on _keyBytes_ and setting an _onClick_ listener for the validate button;

```
.line 24
sget-object v0, Lcom/fortiguard/insomnihack2012/challenge/Compute;->keyBytes:[B
invoke-static {v0}, Lcom/fortiguard/insomnihack2012/challenge/Compute;->compute([B)V

.line 26
iget-object v0, p0, Lcom/fortiguard/insomnihack2012/challenge/InsomniActivity;->validateBtn:Landroid/widget/Button;
new-instance v1, Lcom/fortiguard/insomnihack2012/challenge/InsomniActivity$1;
invoke-direct {v1, p0}, Lcom/fortiguard/insomnihack2012/challenge/InsomniActivity$1;-><init>(Lcom/fortiguard/insomnihack2012/challenge/InsomniActivity;)V
invoke-virtual {v0, v1}, Landroid/widget/Button;->setOnClickListener(Landroid/view/View$OnClickListener;)V
```
Seems weird that we're computing something, though never doing anything with the returned result. Let's just take note of that and come back later. If we look into the _onClick_ listener (_InsomniActivity$1_) we can see what is going on deeper in the app;

```
// boolean b = checkSecret(editTxt.getText.toString());
.line 30
iget-object v1, p0, Lcom/fortiguard/insomnihack2012/challenge/InsomniActivity$1;->this$0:Lcom/fortiguard/insomnihack2012/challenge/InsomniActivity;
iget-object v1, v1, Lcom/fortiguard/insomnihack2012/challenge/InsomniActivity;->editTxt:Landroid/widget/EditText;
invoke-virtual {v1}, Landroid/widget/EditText;->getText()Landroid/text/Editable;
move-result-object v1
invoke-interface {v1}, Landroid/text/Editable;->toString()Ljava/lang/String;
move-result-object v1
invoke-static {v1}, Lcom/fortiguard/insomnihack2012/challenge/Compute;->checkSecret(Ljava/lang/String;)Z
move-result v0

// if(b == true) good_boy else bad_boy
.line 31
.local v0, b:Z
if-eqz v0, :cond_24
```
Here we can see the bulk of this _onClick_ listener just grabs the text from the _EditText_ widget and uses it as a parameter for the _checkSecret_ function. Now onward to the _checkSecret_ function;

```
.method public static checkSecret(Ljava/lang/String;)Z
    .registers 7
    .parameter "input"

    .prologue
    .line 63
    :try_start_0
    // Get a sha-256 messagedisgest
    const-string v3, "SHA-256"
    invoke-static {v3}, Ljava/security/MessageDigest;->getInstance(Ljava/lang/String;)Ljava/security/MessageDigest;
    move-result-object v1

    // reset
    .line 64
    .local v1, digest:Ljava/security/MessageDigest;
    invoke-virtual {v1}, Ljava/security/MessageDigest;->reset()V

    // get bytes of passed in string
    .line 65
    invoke-virtual {p0}, Ljava/lang/String;->getBytes()[B
    move-result-object v3

    // create new digest
    invoke-virtual {v1, v3}, Ljava/security/MessageDigest;->digest([B)[B
    move-result-object v0

    // get secret hash that appears to be precomputed
    .local v0, computedHash:[B
    sget-object v3, Lcom/fortiguard/insomnihack2012/challenge/Compute;->secretHash:[B

    // compare digest's hash to the precomputed one
    invoke-static {v3, v0}, Ljava/util/Arrays;->equals([B[B)Z
    :try_end_16
    .catch Ljava/lang/Exception; {:try_start_0 .. :try_end_16} :catch_1b

    move-result v3

    if-eqz v3, :cond_35

    .line 67
    const/4 v3, 0x1

    .line 73
    .end local v0           #computedHash:[B
    .end local v1           #digest:Ljava/security/MessageDigest;
    :goto_1a
    return v3

    // catch hashing exception
    .line 70
    :catch_1b
    move-exception v3
    move-object v2, v3
    .line 71
    .local v2, exp:Ljava/lang/Exception;
    const-string v3, "InsomniDroid"
    new-instance v4, Ljava/lang/StringBuilder;
    const-string v5, "checkSecret: "
    invoke-direct {v4, v5}, Ljava/lang/StringBuilder;-><init>(Ljava/lang/String;)V
    invoke-virtual {v2}, Ljava/lang/Exception;->toString()Ljava/lang/String;
    move-result-object v5
    invoke-virtual {v4, v5}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;
    move-result-object v4
    invoke-virtual {v4}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;
    move-result-object v4
    invoke-static {v3, v4}, Landroid/util/Log;->w(Ljava/lang/String;Ljava/lang/String;)I
    .line 73
    .end local v2           #exp:Ljava/lang/Exception;
    :cond_35
    const/4 v3, 0x0
    goto :goto_1a
.end method
```
Ok, this doesn't help us much. As we can see from the inlined comments I put above, all it's doing is a SHA-256 hash for our input and comparing it to _secretHash_ \- which obviously must also be a SHA-256 digest. Let's see if we can find this being set anywhere;

```
 different:insomnidroid tstrazzere$ grep -r "secretHash" * | grep "sput-object" com/fortiguard/insomnihack2012/challenge/Compute.smali: sput-object \ v0, Lcom/fortiguard/insomnihack2012/challenge/Compute;->secretHash:[B 
```
Only one hit - this is good. Let's go back into _Compute.smali_ and check out whats actually going on. Once we get in here we can see what appears to be, some left over code, along with an array-fill for the _secretHash_ which is trigger the _sput-object_ search we just performed. Let's get the hash;
` 6152587ede8a26f53fd391b055d4de501ee8b2497fe74f8fd69f2c72e2f3e37a ` And toss that into a hashcat... Maybe it's an easy one to get and I won't have to do any other work... Probably not - that would seem like a brute force challenge and not a crackme one, definitely not something simple like that for someone named "crypto girl". So lets keep looking! Looking back at _Compute.smali_ lets looks back into that function, _compute([B)_ we noticed being called earlier in the APK - but never had the return value used. This one looks interesting because (1) it's seems left over, is called once but not using the return value and (2) it's the only place certain left over variables are being touched like _c1_null_, _c2_ and is also expecting the parameter _keyBytes_ which we also have a variable for. 
```
.method public static checkSecret(Ljava/lang/String;)Z
    .registers 7
    .parameter "input"

    .prologue
    .line 63
    :try_start_0
    // Get a sha-256 messagedisgest
    const-string v3, "SHA-256"
    invoke-static {v3}, Ljava/security/MessageDigest;->getInstance(Ljava/lang/String;)Ljava/security/MessageDigest;
    move-result-object v1

    // reset
    .line 64
    .local v1, digest:Ljava/security/MessageDigest;
    invoke-virtual {v1}, Ljava/security/MessageDigest;->reset()V

    // get bytes of passed in string
    .line 65
    invoke-virtual {p0}, Ljava/lang/String;->getBytes()[B
    move-result-object v3

    // create new digest
    invoke-virtual {v1, v3}, Ljava/security/MessageDigest;->digest([B)[B
    move-result-object v0

    // get secret hash that appears to be precomputed
    .local v0, computedHash:[B
    sget-object v3, Lcom/fortiguard/insomnihack2012/challenge/Compute;->secretHash:[B

    // compare digest's hash to the precomputed one
    invoke-static {v3, v0}, Ljava/util/Arrays;->equals([B[B)Z
    :try_end_16
    .catch Ljava/lang/Exception; {:try_start_0 .. :try_end_16} :catch_1b

    move-result v3

    if-eqz v3, :cond_35

    .line 67
    const/4 v3, 0x1

    .line 73
    .end local v0           #computedHash:[B
    .end local v1           #digest:Ljava/security/MessageDigest;
    :goto_1a
    return v3

    // catch hashing exception
    .line 70
    :catch_1b
    move-exception v3
    move-object v2, v3
    .line 71
    .local v2, exp:Ljava/lang/Exception;
    const-string v3, "InsomniDroid"
    new-instance v4, Ljava/lang/StringBuilder;
    const-string v5, "checkSecret: "
    invoke-direct {v4, v5}, Ljava/lang/StringBuilder;-><init>(Ljava/lang/String;)V
    invoke-virtual {v2}, Ljava/lang/Exception;->toString()Ljava/lang/String;
    move-result-object v5
    invoke-virtual {v4, v5}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;
    move-result-object v4
    invoke-virtual {v4}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;
    move-result-object v4
    invoke-static {v3, v4}, Landroid/util/Log;->w(Ljava/lang/String;Ljava/lang/String;)I
    .line 73
    .end local v2           #exp:Ljava/lang/Exception;
    :cond_35
    const/4 v3, 0x0
    goto :goto_1a
.end method
```
This seems interesting... It's taking in the _keyBytes_ as a parameter, using them to create a _SecretKeySpec_ then using the _ivBytes_ to initialize an _IvParameterSpec_. Then it attempts to decrypt the _c1_null_ variable, though it does nothing with this return value. After that, we decrypt something that is much larger and again, don't do anything with the value. Essentially this maps out to the following java code;

```
byte[] key_bytes = new byte[] {
  0, 1, 2, 3, 0, 1, 2, 3, 0, 1, 2, 3, 0, 1, 2, 3
};
byte[] iv_bytes = new byte[] {
  0, 1, 2, 3, 0, 1, 2, 3, 0, 0, 0, 0, 0, 0, 0, 1
};
byte[] c1_null = new byte[] {
  (byte) 0xec, (byte) 0x34, (byte) 0x27, (byte) 0x1d, (byte) 0x0f, (byte) 0x97, (byte) 0x40, (byte) 0x4e,
  (byte) 0xf5, (byte) 0xe9, (byte) 0x5c, (byte) 0xa2, (byte) 0xfe, (byte) 0x0d, (byte) 0x84, (byte) 0x21
};
byte[] c2 = new byte[] {
  (byte) 0xaf, (byte) 0x5b, (byte) 0x49, (byte) 0x7a, (byte) 0x7d, (byte) 0xf6, (byte) 0x34, (byte) 0x3d,
  (byte) 0xd4, (byte) 0xc9, (byte) 0x18, (byte) 0xcd, (byte) 0x90, (byte) 0x79, (byte) 0xa4, (byte) 0x53,
  (byte) 0x89, (byte) 0x19, (byte) 0x52, (byte) 0x6e, (byte) 0x6a, (byte) 0xb7, (byte) 0x01, (byte) 0x0b,
  (byte) 0xa6, (byte) 0xc9, (byte) 0x1f, (byte) 0xf6, (byte) 0xac, (byte) 0x2d, (byte) 0xe7, (byte) 0x4e,
  (byte) 0x99, (byte) 0x5a, (byte) 0x53, (byte) 0x78, (byte) 0x7d, (byte) 0xe4, (byte) 0x60, (byte) 0x75,
  (byte) 0xdc, (byte) 0xc9, (byte) 0x0f, (byte) 0xc7, (byte) 0x9d, (byte) 0x7f, (byte) 0xe1, (byte) 0x55,
  (byte) 0xcc, (byte) 0x77, (byte) 0x48, (byte) 0x79, (byte) 0x6a, (byte) 0xb7, (byte) 0x29, (byte) 0x3d,
  (byte) 0xcf, (byte) 0xc9, (byte) 0x6e, (byte) 0xcf, (byte) 0x95, (byte) 0x6b, (byte) 0xe9, (byte) 0x49,
  (byte) 0xde, (byte) 0x46, (byte) 0x17, (byte) 0x75, (byte) 0x64, (byte) 0xf6, (byte) 0x2b, (byte) 0x2b,
  (byte) 0xaa, (byte) 0x84, (byte) 0x6d, (byte) 0x90, (byte) 0xcd, (byte) 0x39, (byte) 0xb1, (byte) 0x17
};
SecretKeySpec key_spec = new SecretKeySpec(key_bytes, "AES");
IvParameterSpec iv_parameter = new IvParameterSpec(iv_bytes);
try {
  Cipher cipher = Cipher.getInstance("AES/CTR/NoPadding");
  cipher.init(Cipher.DECRYPT_MODE, key_spec, iv_parameter);
  byte[] result = cipher.doFinal(c1_null);
  byte[] p2 = new byte[c2.length];
  for(int i = 0; i < c2.length; i += 0x10) {
    cipher.init(Cipher.DECRYPT_MODE, key_spec, iv_parameter);
    cipher.doFinal(c2, i, 0x10, p2, i);
  }
} catch(Exception e) {
  System.out.println("ops: " + e.toString());
} 
```
At this point I'm assuming I just solved this -- run the app, look at the output... Garbage -- it's just not UTF-8 (or UTF-16) characters. That can't be it -- I don't think the challenge would be to include non-ascii characters! After staring at this a bit longer I decided to look at the lengths of what is being output. _result_ has a length that will always going to be 16. _p2_ will end up having a length that is always being divisible by 16... Well - lets try some operations between the two resulting arrays that aren't being used. First one I tried was XOR'ing them together (everyone loves XOR, malware writers, crackme writers, etc, etc) -- and this ended up being exactly what needed to be done. 
```
byte[] key_bytes = new byte[] {
  0, 1, 2, 3, 0, 1, 2, 3, 0, 1, 2, 3, 0, 1, 2, 3
};
byte[] iv_bytes = new byte[] {
  0, 1, 2, 3, 0, 1, 2, 3, 0, 0, 0, 0, 0, 0, 0, 1
};
byte[] c1_null = new byte[] {
  (byte) 0xec, (byte) 0x34, (byte) 0x27, (byte) 0x1d, (byte) 0x0f, (byte) 0x97, (byte) 0x40, (byte) 0x4e,
  (byte) 0xf5, (byte) 0xe9, (byte) 0x5c, (byte) 0xa2, (byte) 0xfe, (byte) 0x0d, (byte) 0x84, (byte) 0x21
};
byte[] c2 = new byte[] {
  (byte) 0xaf, (byte) 0x5b, (byte) 0x49, (byte) 0x7a, (byte) 0x7d, (byte) 0xf6, (byte) 0x34, (byte) 0x3d,
  (byte) 0xd4, (byte) 0xc9, (byte) 0x18, (byte) 0xcd, (byte) 0x90, (byte) 0x79, (byte) 0xa4, (byte) 0x53,
  (byte) 0x89, (byte) 0x19, (byte) 0x52, (byte) 0x6e, (byte) 0x6a, (byte) 0xb7, (byte) 0x01, (byte) 0x0b,
  (byte) 0xa6, (byte) 0xc9, (byte) 0x1f, (byte) 0xf6, (byte) 0xac, (byte) 0x2d, (byte) 0xe7, (byte) 0x4e,
  (byte) 0x99, (byte) 0x5a, (byte) 0x53, (byte) 0x78, (byte) 0x7d, (byte) 0xe4, (byte) 0x60, (byte) 0x75,
  (byte) 0xdc, (byte) 0xc9, (byte) 0x0f, (byte) 0xc7, (byte) 0x9d, (byte) 0x7f, (byte) 0xe1, (byte) 0x55,
  (byte) 0xcc, (byte) 0x77, (byte) 0x48, (byte) 0x79, (byte) 0x6a, (byte) 0xb7, (byte) 0x29, (byte) 0x3d,
  (byte) 0xcf, (byte) 0xc9, (byte) 0x6e, (byte) 0xcf, (byte) 0x95, (byte) 0x6b, (byte) 0xe9, (byte) 0x49,
  (byte) 0xde, (byte) 0x46, (byte) 0x17, (byte) 0x75, (byte) 0x64, (byte) 0xf6, (byte) 0x2b, (byte) 0x2b,
  (byte) 0xaa, (byte) 0x84, (byte) 0x6d, (byte) 0x90, (byte) 0xcd, (byte) 0x39, (byte) 0xb1, (byte) 0x17
};
SecretKeySpec key_spec = new SecretKeySpec(key_bytes, "AES");
IvParameterSpec iv_parameter = new IvParameterSpec(iv_bytes);
try {
  Cipher cipher = Cipher.getInstance("AES/CTR/NoPadding");
  cipher.init(Cipher.DECRYPT_MODE, key_spec, iv_parameter);
  byte[] result = cipher.doFinal(c1_null);
  byte[] p2 = new byte[c2.length];
  for(int i = 0; i < c2.length; i += 0x10) {
    cipher.init(Cipher.DECRYPT_MODE, key_spec, iv_parameter);
    cipher.doFinal(c2, i, 0x10, p2, i);
  }
  byte[] solution = new byte[p2.length];
  for(int x = 0; x < p2.length / result.length; x++){
    for(int i = 0; i < result.length; i++) {
      solution[i + x * result.length ] = (byte) (result[i] ^ p2[i + (x * result.length)]);
    }
  }
  System.out.println(new String(solution));
}
catch(Exception e) {
  System.out.println("ops: " + e.toString());
} 
```
After running this we get the output;
```
Congrats! Dont re-use AES CTR counters ;) Secret Code is: 2mkfmh2r0hkake_m123456
``` 
[![](http://www.strazzere.com/blog/wp-content/uploads/2012/03/insomnidroid-congrats.png "insomnidroid-congrats")](http://www.strazzere.com/blog/wp-content/uploads/2012/03/insomnidroid-congrats.png) Ah - Crypto girl left us a nice crypto related message. Essentially explaining what went on here. Fun stuff and definitely glad I didn't wait for hashcat to try and spit out that password.