---
id: 2076
title: 'Using JNIs (like OpenCV) in Flink'
date: '2017-08-14T11:09:48+00:00'
author: rawkintrevo
layout: post
guid: 'http://rawkintrevo.org/?p=2076'
permalink: /2017/08/14/using-jnis-like-opencv-in-flink/
jabber_published:
    - '1502708991'
email_notification:
    - '1502708993'
publicize_twitter_user:
    - rawkintrevo
publicize_google_plus_url:
    - 'https://plus.google.com/107169808969912762220/posts/UXacB8yhDnd'
publicize_linkedin_url:
    - 'https://www.linkedin.com/updates?discuss=&scope=94269141&stype=M&topic=6302818389966544897&type=U&a=E30q'
image: /wp-content/uploads/2017/08/native-jar.jpg
categories:
    - Engineering
    - flink
    - How-Tos
---

For a bigger project which I hope to blog about soon, I needed to get the OpenCV Java Native Library (JNI) running in a Flink stream. It was a pain in the ass, so I’m putting this here to help the next person.

### First thing I tried… Just doing it.

For OpenCV, you need to statically initialize (I’m probably saying this wrong) the library, so I tried something like this

val stream = env  
.addSource(rawVideoConsumer)  
.map(record =&gt; {  
System.loadLibrary(Core.NATIVE\_LIBRARY\_NAME)  
…

Well, that kicks an error that looks like:

\[code lang=text\]  
Exception in thread "Thread-143" java.lang.UnsatisfiedLinkError:  
Native Library /usr/lib/jni/libopencv\_java330.so already loaded in  
another classloader  
\[/code\]

Ok, that’s fair. Multiple task managers, this is getting loaded all over the place. I get it. I tried moving this around a bunch. No luck.

### Second thing I tried… Monkey see- monkey do: The RocksDB way.

In the [Flink-RocksDB connector](https://github.com/apache/flink/blob/master/flink-contrib/flink-statebackend-rocksdb/src/main/java/org/apache/flink/contrib/streaming/state/RocksDBStateBackend.java#L425), and other people have given this advice, the idea is to include the JNI in the resources/fat-jar, then write out a tmp one and have that loaded.

This, for me, resulted in seemingly one tmp copy being generated for each record processed.

\[code lang=text\]  
import java.io.\_

/\*\*  
\* This is an example of an extremely stupid way (and consequently the way Flink does RocksDB) to handle the JNI problem.  
\*  
\* DO NOT USE!!  
\*  
\* Basically we include libopencv\_java330.so in src/main/resources so then it creates a tmp version.  
\*  
\* I deleted from it resources, so this would fail. Only try it for academic purposes. E.g. to see what stupid looks like.  
\*  
\*/  
object NativeUtils {  
 // heavily based on <https://github.com/adamheinrich/native-utils/blob/master/src/main/java/cz/adamh/utils/NativeUtils.java>  
 def loadOpenCVLibFromJar() = {

 val temp = File.createTempFile("libopencv\_java330", ".so")  
 temp.deleteOnExit()

 val inputStream= getClass().getResourceAsStream("/libopencv\_java330.so")

 import java.io.FileOutputStream  
 import java.io.OutputStream  
 val os = new FileOutputStream(temp)  
 var readBytes: Int = 0  
 var buffer = new Array\[Byte\](1024)

 try {  
 while ({(readBytes = inputStream.read(buffer))  
 readBytes != -1}) {  
 os.write(buffer, 0, readBytes)  
 }  
 }  
 finally {  
 // If read/write fails, close streams safely before throwing an exception  
 os.close()  
 inputStream.close  
 }

 System.load(temp.getAbsolutePath)  
 }

}  
\[/code\]

### Third way: Sanity.

There were more accurately like 300 hundred ways I tried to make this S.O.B. work, I’m really just giving you the way points- major strategies I tried in my journey. This is the solution. This is the ‘Tomcat solution’ I’d seen referenced throughout my journey but didn’t understand what they meant. Hence why, I’m writing this blog post.

I created an entirely new module. I called it `org.rawkintrevo.cylons.opencv`. In that module there is one class.

\[code lang=text\]  
package org.rawkintrevo.cylon.opencv;

import org.opencv.core.Core;

public class LoadNative {

 static {  
 System.loadLibrary(Core.NATIVE\_LIBRARY\_NAME);  
 }

 native void loadNative();  
}  
\[/code\]

I compiled that as a fat jar and dropped it in `flink/lib`

Then, where I would have run `System.loadLibrary(Core.NATIVE_LIBRARY_NAME)`, I now put `Class.forName("org.rawkintrevo.cylon.opencv.LoadNative")`.

\[code lang=text\]  
 val stream = env  
 .addSource(rawVideoConsumer)  
 .map(record =&gt; {  
 // System.loadLibrary(Core.NATIVE\_LIBRARY\_NAME)  
 Class.forName("org.rawkintrevo.cylon.opencv.LoadNative")  
 …  
\[/code\]

Further, I copy the OpenCV Java wrapper (`opencv/build/bin/opencv-330.jar`), and library (`opencv/build/lib/opencv_java330.so`) in `flink/lib`

Then I have great success and profit.

Good hunting,

tg