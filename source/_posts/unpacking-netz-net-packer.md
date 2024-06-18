---
title: Unpacking "netz .Net packer"
tags:
  - .net unpacking
  - netz
  - reverse engineering
  - reversing
  - unpacker
  - unpacking
url: unpacking-netz-net-packer.html
id: 370
categories:
  - random
  - reversing
  - windows
date: 2010-09-01 17:11:27
---

It's actually pretty trivial to do this, though it seemed like something that could be mildly interesting to post on. While it's written in C# .Net itself, it means we can pretty much see exactly whats going on - so it's lends itself to be a pretty easy example for people who have never actually done any unpacking before. It also lets to understand the concept of unpacking from a programmers prospective of actually packing it.

A little backstory to this is, someone sent me a "hack" for diablo2 and said it wasn't working for them. Why they sent me this? I have no idea, but I figured I'd just take a look at it and see what was going on. Turns out it was a password stealer for d2 written in .Net - pretty cool looking stuff. The funny part about it all was that since this was "packed" with netz the person who sent me it couldn't use reflector on it (and succeed) - thus why they sent it to me I guess. Anyway, let's get down to the actual meat of the post. 

Identification of netz is pretty simple - if you open up the application in Reflector, you'll see the namespace `nets`. Also it will almost always be accompanied by a `zip.dll`, which is used for decompressing the resource. Essentially the main function we want to look at is `StartApp`:
```
public static int StartApp(string[] args)
{
    byte[] resource = GetResource("213213-2131223-2134234-234");
    if (resource == null)
    {
        throw new Exception("application data cannot be found");
    }
    int num = InvokeApp(GetAssembly(resource), args);
    resource = null;
    return num;
}
```
From here we can see that a resource of bytes is going to be loaded, and `GetAssembly` is called with it as an argument. Go dump the resource listed here into some directory, I named mine `virgin.dump`. `GetAssembly` is a pretty simple function that I'm not going to dive into, it essentially calls `UnZip(byte[] data)`. This code is a little more interesting and is doing the most work we're interested in, here is small snippet;
```
private static MemoryStream UnZip(byte[] data) {
    MemoryStream baseInputStream = null;
    MemoryStream stream2 = null;
    InflaterInputStream stream3 = null;

    baseInputStream = new MemoryStream(data);
    stream2 = new MemoryStream();
    stream3 = new InflaterInputStream(baseInputStream);
    byte[] buffer = new byte[data.Length];
    while (true)
    {
       int count = stream3.Read(buffer, 0, buffer.Length);
        if (count <= 0)
        {
           break;
        }
        stream2.Write(buffer, 0, count);
    }
    stream2.Flush();
    stream2.Seek(0L, SeekOrigin.Begin);
}
```
Ah, this looks familiar! In fact - thanks to Reflector it's pretty much almost exactly Java code. All we need to do is inflate the resource and dump it to a file - there isn't any encryption or anything special about it at all. Basically by looking at the code above, I through together a small little resource inflater:
```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.zip.InflaterInputStream;

public class inflater {
    public static void main(String[] args) {
        try {
            FileInputStream inStream = new FileInputStream(args[0]);
            InflaterInputStream inflaterStream = new InflaterInputStream(inStream);
            FileOutputStream outStreams = new FileOutputStream(args[0] + "_unpacked");
            for (int c = inflaterStream.read(); c != -1; c = inflaterStream.read()) {
                outStreams.write(c);
            }
            outStreams.close();
        } catch (IOException ex) {
            System.err.println(ex);
        }
    }
}
```
Is this anything special? No, but it worked for me and my quick little need for it. Could you use a Generic .Net unpacker? Of course, I just did this since I wasn't running Windows and didn't want to fire up a VM instance just to debug and dump a little .Net application. :)