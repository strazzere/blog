---
title: Using Dedexer by Gabor...
tags:
  - android
  - bytecode
  - ddx
  - Dedexer
  - dex
  - g1
  - Gabor
  - reverse engineering
url: using-dedexer-by-gabor.html
id: 171
categories:
  - android
date: 2009-01-21 00:00:39
---

Was playing around with dedexer, mention in this [previous post](http://strazzere.com/blog/?p=165), and noticed it wasn't working well on my ubuntu dev. machine. Turns out it just didn't play well with the default ubuntu java - so switching it made all the difference. So if your getting the following error or something like this when running:
```
tstrazze@strazz-workstation:~/Desktop$ java -jar ddx.jar -d dump classes.dex
Processing com/android/im/util/QueryUtils
Exception in thread “main” java.lang.NoSuchMethodError: method java.io.PrintStream. with signature (Ljava.io.File;)V was not found.
at hu.uw.pallergabor.dedexer.JasminStyleCodeGenerator.generate(JasminStyleCodeGenerator.java:29)
at hu.uw.pallergabor.dedexer.Dedexer.run(Dedexer.java:116)
at hu.uw.pallergabor.dedexer.Dedexer.main(Dedexer.java:12)
```

Then run the following command;
```
tstrazze@strazz-workstation:~/Desktop$ java -version
java version “1.5.0”
gij (GNU libgcj) version 4.2.4 (Ubuntu 4.2.4-1ubuntu3)

Copyright (C) 2007 Free Software Foundation, Inc.
This is free software; see the source for copying conditions. There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
tstrazze@strazz-workstation:~/Desktop$ sudo update-java-alternatives -l
java-6-sun 63 /usr/lib/jvm/java-6-sun
java-gcj 1042 /usr/lib/jvm/java-gcj
```

We want to be using _java-6-sun_, not _java-gcj_ so we'll do the following;
```
tstrazze@strazz-workstation:~/Desktop$ sudo update-java-alternatives -s java-gcj
No alternatives for apt.
No alternatives for extcheck.
No alternatives for firefox-3.0-javaplugin.so.
No alternatives for HtmlConverter.
No alternatives for idlj.
No alternatives for javap.
No alternatives for java-rmi.cgi.
No alternatives for jconsole.
No alternatives for jdb.
No alternatives for jhat.
No alternatives for jinfo.
No alternatives for jmap.
No alternatives for jps.
No alternatives for jrunscript.
No alternatives for jsadebugd.
No alternatives for jstack.
No alternatives for jstat.
No alternatives for jstatd.
No alternatives for jvisualvm.
No alternatives for schemagen.
No alternatives for wsgen.
No alternatives for wsimport.
No alternatives for xjc.
Using ‘/usr/lib/jvm/java-gcj/bin/appletviewer’ to provide ‘appletviewer’.
Using ‘/usr/lib/jvm/java-gcj/bin/jarsigner’ to provide ‘jarsigner’.
Using ‘/usr/lib/jvm/java-gcj/bin/javac’ to provide ‘javac’.
Using ‘/usr/lib/jvm/java-gcj/bin/javadoc’ to provide ‘javadoc’.
Using ‘/usr/lib/jvm/java-gcj/bin/javah’ to provide ‘javah’.
Using ‘/usr/lib/jvm/java-gcj/bin/native2ascii’ to provide ‘native2ascii’.
Using ‘/usr/lib/jvm/java-gcj/bin/rmic’ to provide ‘rmic’.
Using ‘/usr/lib/jvm/java-gcj/bin/tnameserv’ to provide ‘tnameserv’.
Using ‘/usr/lib/jvm/java-gcj/jre/bin/jar’ to provide ‘jar’.
Using ‘/usr/lib/jvm/java-gcj/jre/bin/java’ to provide ‘java’.
Using ‘/usr/lib/jvm/java-gcj/jre/bin/keytool’ to provide ‘keytool’.
Using ‘/usr/lib/jvm/java-gcj/jre/bin/orbd’ to provide ‘orbd’.
Using ‘/usr/lib/jvm/java-gcj/jre/bin/rmid’ to provide ‘rmid’.
Using ‘/usr/lib/jvm/java-gcj/jre/bin/rmiregistry’ to provide ‘rmiregistry’.
Using ‘/usr/lib/jvm/java-gcj/jre/bin/serialver’ to provide ‘serialver’.
update-java-alternatives: plugin alternative does not exist: /usr/lib/gcj-4.2/libgcjwebplugin.so
update-java-alternatives: plugin alternative does not exist: /usr/lib/gcj-4.2/libgcjwebplugin.so
update-java-alternatives: plugin alternative does not exist: /usr/lib/gcj-4.2/libgcjwebplugin.so
update-java-alternatives: plugin alternative does not exist: /usr/lib/gcj-4.2/libgcjwebplugin.so
update-java-alternatives: plugin alternative does not exist: /usr/lib/gcj-4.2/libgcjwebplugin.so
update-java-alternatives: plugin alternative does not exist: /usr/lib/gcj-4.2/libgcjwebplugin.so
update-java-alternatives: plugin alternative does not exist: /usr/lib/gcj-4.2/libgcjwebplugin.so
```

Ta-da! A simple (and probably obvious for most) work around. Just figured I'd throw it up here to help anyone who might bump into the problem.