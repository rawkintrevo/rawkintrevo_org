---
id: 650
title: 'Big Data for n00bs: My first streaming Flink program (twitter)'
date: '2016-09-30T19:28:59+00:00'
author: rawkintrevo
excerpt: 'Fishing for tweet with Flink'
layout: post
guid: 'http://trevorgrant.org/?p=650'
permalink: /2016/09/30/big-data-for-n00bs-my-first-streaming-flink-program-twitter/
jabber_published:
    - '1475263741'
email_notification:
    - '1475263744'
publicize_twitter_user:
    - rawkintrevo
publicize_google_plus_url:
    - 'https://plus.google.com/107169808969912762220/posts/LaHydmnPAp7'
publicize_linkedin_url:
    - 'https://www.linkedin.com/updates?discuss=&scope=94269141&stype=M&topic=6187704693255319552&type=U&a=qYM1'
image: /wp-content/uploads/2016/09/photo-on-9-30-16-at-2-26-pm.jpg
categories:
    - flink
---

It’s learning Friday, and I’m getting this out late so I’ll try to make it short and sweet. In this post we’re going to create a simple streaming program that hits the twitter [statuses endpoint](https://dev.twitter.com/streaming/reference/post/statuses/filter).

This is based on a demo from [my recent talk at Flink Forward 2016](https://www.youtube.com/watch?v=oVDSi30U7rw). As always, I will be doing this in Apache Zeppelin notebooks, because I am lazy and don’t like to compile jars.

### Step 1. Create API Keys

Go to [twitter application management](https://apps.twitter.com/) and create a new app. After you create the application, click on it. Under the **Keys and Access Tokens Tab** you will see the `Consumer Key` and `Consumer Secret`. Scroll down a little ways and you will also see `Access Token` and `Access Token Secret`. Leave this tab open, you’ll need it shortly.

### Step 2. Open Zeppelin, create a new Interpreter for the Streaming Job.

In the first paragraph we are going to define our new interpreter. We need to add the dependency `org.apache.flink:flink-connector-twitter_2.10:1.1.2`. Also, if you’re running in a cluster you also need to download [this jar](http://central.maven.org/maven2/org/apache/flink/flink-connector-twitter_2.10/1.1.2/flink-connector-twitter_2.10-1.1.2.jar) to `$FLINK_HOME/lib` and restart the cluster.

![screen-shot-2016-09-30-at-12-40-36-pm](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2016/09/screen-shot-2016-09-30-at-12-40-36-pm.png?resize=685%2C268&ssl=1)

**NOTE** If you haven’t updated your Zeppelin in a while, you should do that. You need to have a version of Zeppelin that is using Apache Flink v1.1.2. This is important because that update to Zeppelin also introduced the Streaming context to Flink notebooks in Zeppelin. A quick way to test your if your Zeppelin is OK, is to run the following code in a paragraph, and see no errors.

\[code lang=”scala”\]  
%flink

senv  
\[/code\]

![Screen Shot 2016-09-30 at 12.44.51 PM.png](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2016/09/screen-shot-2016-09-30-at-12-44-51-pm.png?resize=685%2C93&ssl=1)

### Step 3. Set your authorization keys.

Refer to Step 1 for your specific keys, but create a notebook with the following code:

\[code lang=”scala”\]  
%flinkStreamingDemo  
//////////////////////////////////////////////////////  
// Enter our Creds  
import java.util.Properties  
import org.apache.flink.streaming.connectors.twitter.TwitterSource

val p = new Properties();  
p.setProperty(TwitterSource.CONSUMER\_KEY, &amp;quot;&amp;quot;);  
p.setProperty(TwitterSource.CONSUMER\_SECRET, &amp;quot;&amp;quot;);  
p.setProperty(TwitterSource.TOKEN, &amp;quot;&amp;quot;);  
p.setProperty(TwitterSource.TOKEN\_SECRET, &amp;quot;&amp;quot;);  
\[/code\]

Obviously, plug your keys/tokens in- I’ve deleted mine.

### Step 4. Create an end point to track terms

I’ve created a simple endpoint to capture all tweats containing the words `pizza` OR `puggle` OR `poland`. See [the docs](https://dev.twitter.com/streaming/overview/request-parameters#track) for more information on how twitter queries work.

\[code lang=”scala”\]  
%flinkStreamingDemo

import com.twitter.hbc.core.endpoint.{StatusesFilterEndpoint, StreamingEndpoint}  
import scala.collection.JavaConverters.\_

// val chicago = new Location(new Location.Coordinate(-86.0, 41.0), new Location.Coordinate(-87.0, 42.0))

//////////////////////////////////////////////////////  
// Create an Endpoint to Track our terms  
class myFilterEndpoint extends TwitterSource.EndpointInitializer with Serializable {  
 @Override  
 def createEndpoint(): StreamingEndpoint = {  
 val endpoint = new StatusesFilterEndpoint()  
 //endpoint.locations(List(chicago).asJava)  
 endpoint.trackTerms(List(&amp;quot;pizza&amp;quot;, &amp;quot;puggle&amp;quot;, &amp;quot;poland&amp;quot;).asJava)  
 return endpoint  
 }  
}  
\[/code\]

I’ve also commented out, but left as reference how you would do a location based filter.

### Step 5. Set up the endpoints

Nothing exciting here, just setting up and initializing the endpoints.

\[code lang=”scala”\]  
%flinkStreamingDemo

import org.apache.flink.streaming.api.scala.DataStream  
import org.apache.flink.streaming.api.windowing.time.Time  
import org.apache.flink.core.fs.FileSystem.WriteMode

val source = new TwitterSource(p)  
val epInit = new myFilterEndpoint()

source.setCustomEndpointInitializer( epInit )

val streamSource = senv.addSource( source );  
\[/code\]

### Step 6. Setup the Flink processing and windowing.

This is an embarrassingly simple example. Flink can do so much, but all we’re going to do is show off a little bit of its windowing capability (light years ahead of Apache spark streaming).

Everything else is somewhat trivial- this is the code of interest that the user is encouraged to play with.

\[code lang=”scala”\]  
%flinkStreamingDemo

streamSource.map(s =&amp;gt; (0,1))  
 .keyBy(0)  
 // sliding time window of 1 minute length and 30 secs trigger interval  
 .timeWindow(Time.minutes(2), Time.seconds(30))  
 .sum(1)  
 .map(t =&amp;gt; t.\_2)  
 .writeAsText(&amp;quot;/tmp/2minCounts.txt&amp;quot;, WriteMode.OVERWRITE)

senv.execute(&amp;quot;Twitter Count&amp;quot;)  
\[/code\]

We count the tweets in each window and the counts by window are saved to a text file. Not very exciting.

[More information on windowing in Flink](https://flink.apache.org/news/2015/12/04/Introducing-windows.html).

Exercises for the reader:  
– Different windowing strategies  
– Doing an action on the tweet like determining which keyword it contains  
— Count by keyword  
– Joining windows of different lengths (e.g. how many users tweeted in this window who have also tweeted in the current 24 hour window).

Etc.

### Step 7. Visualize results.

Because there is a two minute window, you need to give this some time collect a window and write the results to `2minCounts.txt`. So go grab a coffee, tweet how great this post is, etc.

When you come back, run the following code:

\[code lang=”scala”\]  
%flink  
import scala.io.Source

val twoMinCnt = Source.fromFile(&amp;quot;/tmp/2minCounts.txt&amp;quot;).getLines.toList

println(&amp;quot;%table\\ntime\\tff\_tweets\\n&amp;quot; + twoMinCnt.zipWithIndex.map(o=&amp;gt; o.\_2+&amp;quot;\\t&amp;quot;+o.\_1).mkString(&amp;quot;\\n&amp;quot;))  
\[/code\]

That code will read the text file and turn it in to a table that Zeppelin can understand and render in AngularJS. In this chart, the numbers on the *x* axis are the ‘epochs’ that is the 2 minute windows offset by 30 seconds each.

![Screen Shot 2016-09-30 at 1.54.11 PM.png](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2016/09/screen-shot-2016-09-30-at-1-54-11-pm.png?resize=685%2C283&ssl=1)

**UPDATE**  
Remember how we created a new interpreter called `%flinkStreamingDemo`?

So here is the thing with a Flink streaming job, and Zeppelin. When you ‘execute’ the streaming paragraph from Step 6, that job is going to run forever.

If you are running Flink in a cluster, you have three options:  
– Restart the interpreter. This will free the interpreter, but the streaming job will still be running- you can verify this in the Flink WebUI (also, the Flink WebUI is where you can stop the job). Now that control of the interpreter has been returned to you, you can run the visualization code with the `%flinkStreamingDemo` interpreter.  
– You can use another Flink interpreter such as `%flink`. This will also work fine.  
– You can use another Scala interpreter (such as the `%ignite` or `%spark` interpreter). This is fine, because the visualization code doesn’t leverage Flink in anyway, only pure Scala to read the file and convert it into a Zeppelin `%table`

If you’re in local mode (e.g. you don’t have a Flink Cluster running) you’ll likely see an error: `java.net.BindException: Address already in use` when you try to use another `%flink` interpreter. This is an opportunity for improvement in Zeppelin as the `FlinkLocalMiniCluster` always binds to `6123`. In this case, the only option is #3 from above, simply run the code from any other Scala based interpreter (or use Python or R, but you’ll need to alter the code for those languages). All we *need* to do here is read the file, give it two headers and create a tab-separated string that starts with `%table`.

For example, this would work:

\[code lang=”scala”\]  
%spark  
import scala.io.Source

val twoMinCnt = Source.fromFile(&amp;quot;/tmp/2minCounts.txt&amp;quot;).getLines.toList

println(&amp;quot;%table\\ntime\\tff\_tweets\\n&amp;quot; + twoMinCnt.zipWithIndex.map(o=&amp;gt; o.\_2+&amp;quot;\\t&amp;quot;+o.\_1).mkString(&amp;quot;\\n&amp;quot;))  
\[/code\]

Again, because for the visualization, we don’t need anything that Flink does. We’re simply loading a file and creating a string.

### Troubleshooting

If you see **NO DATA AVAILABLE** you either:  
– didn’t wait long enough (wait at least the length of the window plus a little)  
– you entered your credentials wrong  
– you have a rarely use term and there is no data

#### Bad Auth

`tail f*/log/*taskmanager*log`probably *won’t* work for you, but you need to check those logs for something that looks like this.

![Screen Shot 2016-09-30 at 1.46.48 PM.png](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2016/09/screen-shot-2016-09-30-at-1-46-48-pm.png?resize=685%2C139&ssl=1)

### The whole enchilad

\[code lang=”scala”\]  
%flinkStreamingDemo  
import java.util.Properties  
import org.apache.flink.streaming.connectors.twitter.TwitterSource  
import com.twitter.hbc.core.endpoint.{StatusesFilterEndpoint, StreamingEndpoint, Location}

import scala.collection.JavaConverters.\_

import org.apache.flink.streaming.api.scala.DataStream  
import org.apache.flink.streaming.api.windowing.time.Time  
import org.apache.flink.core.fs.FileSystem.WriteMode

//////////////////////////////////////////////////////  
// Enter our Creds  
val p = new Properties();  
p.setProperty(TwitterSource.CONSUMER\_KEY, &amp;quot;&amp;quot;);  
p.setProperty(TwitterSource.CONSUMER\_SECRET, &amp;quot;&amp;quot;);  
p.setProperty(TwitterSource.TOKEN, &amp;quot;&amp;quot;);  
p.setProperty(TwitterSource.TOKEN\_SECRET, &amp;quot;&amp;quot;);

val chicago = new Location(new Location.Coordinate(-86.0, 41.0), new Location.Coordinate(-87.0, 42.0))

//////////////////////////////////////////////////////  
// Create an Endpoint to Track our terms  
class myFilterEndpoint extends TwitterSource.EndpointInitializer with Serializable {  
 @Override  
 def createEndpoint(): StreamingEndpoint = {  
 val endpoint = new StatusesFilterEndpoint()  
 //endpoint.locations(List(chicago).asJava)  
 endpoint.trackTerms(List(&amp;quot;pizza&amp;quot;, &amp;quot;puggle&amp;quot;, &amp;quot;poland&amp;quot;).asJava)  
 return endpoint  
 }  
}

val source = new TwitterSource(p)  
val epInit = new myFilterEndpoint()

source.setCustomEndpointInitializer( epInit )

val streamSource = senv.addSource( source );

streamSource.map(s =&amp;gt; (0,1))  
 .keyBy(0)  
 // sliding time window of 2 minute length and 30 secs trigger interval  
 .timeWindow(Time.minutes(2), Time.seconds(30))  
 .sum(1)  
 .map(t =&amp;gt; t.\_2)  
 .writeAsText(&amp;quot;/tmp/2minCounts.txt&amp;quot;, WriteMode.OVERWRITE)

senv.execute(&amp;quot;Twitter Count&amp;quot;)  
\[/code\]