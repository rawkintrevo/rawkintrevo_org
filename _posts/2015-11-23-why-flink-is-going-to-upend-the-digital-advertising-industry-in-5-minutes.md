---
id: 116
title: 'Why Flink is going to upend the Digital Advertising industry in 5 minutes.'
date: '2015-11-23T16:58:02+00:00'
author: rawkintrevo
layout: post
guid: 'http://trevorgrant.org/?p=116'
permalink: /2015/11/23/why-flink-is-going-to-upend-the-digital-advertising-industry-in-5-minutes/
jabber_published:
    - '1448297883'
email_notification:
    - '1448297886'
publicize_twitter_user:
    - rawkintrevo
publicize_google_plus_url:
    - 'https://plus.google.com/107169808969912762220/posts/HE9w2xGbocb'
publicize_linkedin_url:
    - 'https://www.linkedin.com/updates?discuss=&scope=94269141&stype=M&topic=6074601654089428992&type=U&a=losh'
image: /wp-content/uploads/2015/11/flink_squirrel_1000.png
categories:
    - Uncategorized
tags:
    0: apache
    1: 'big data'
    2: 'digital advertising'
    4: 'internet of things'
    5: streaming
---

So, I was on a call for the November[ i-Com ](http://www.i-com.org/)Data Science Board meeting Thursday morning. There were supposed to be 5 minute presentations with discussions on a few topics, however a couple of the presenters couldn’t make the call, including the presenter on Streaming Data Processing, and Real Time Analytics. So I think to myself I think, “Hey, real time analytics and streaming data processing is kind of my wheel house. I could probably hip-shot a pretty solid 5 minute talk on the subject and how it relates to digital marketing.” So I hacked out a quick outline, and was all set and then the discussion following the first presentation went until the end of the meeting.

Waste not, want not; I am recycling the outline into a quick blog post because most of the people on my distribution list (e.g. Twitter, Facebook, and LinkedIn friends) are somehow tied to digital marketing, and this is relevant to their interests.

To do so properly I have to start with a quick primer on Big Data and real-time stream processing so that the reader understands where we were 5 years ago, where we are now, and why some up-and-coming Apache projects are about to change everything. Second I’ll briefly compare the current real-time streaming technology, and the two new comers. Finally I’ll give a couple of quick hits on how this will turn the digital ad space on its head.

## The history of Big Data / Streaming / Real Time Data processing in 120 seconds.

***A stream.*** Consider a stream of data coming at you. The most familiar type of streaming data to the audience at large is probably Twitter data. Consider a stream of all Tweets originating within a 100 kilometer radius of Chicago.

***Distributed computing.*** You might have heard this term floating around; more likely you have heard the tangent buzzwords, including but not limited to: Map-Reduce, Spark, Hadoop, Flink, Clusters, Cloud, etc. In the mid 2000s Google figured it was cheaper and wiser to make teams of cheap computers work together than keep buying big expensive servers. People had been doing this for a while, but Google did it really well and addressed a lot of the problems associated with it (for example machines randomly fail) and the best part was in the late 2000s Google released the technology open-source (read free) to the world, that’s when this whole mess started.

***[Apache Hadoop.](https://hadoop.apache.org/)*** So in a Big Data sense, real-time stream processing means having teams of computers working together to process streaming data. *Streaming* is the key here, because Google’s version worked on large data sets that were already there, think of a traditional database or even an Excel spreadsheet. Excel can handle 1 million rows. Hadoop (the name of Google’s baby) can handle several trillion and beyond.

***[Apache Storm.](http://storm.apache.org/)*** Twitter has been dealing with streaming data for a while. Similar to Google, Twitter developed a really nice tool for handling streaming data that solved a lot of the common problems and was generalizable to many use-cases. They also released it to the world open source in the early 2010s. This is the tool (or some hacked version of it) that many DSPs use today ([for example Rubicon](http://storm.apache.org/documentation/Powered-By.html)).

***[Apache Spark.](http://spark.apache.org/)*** Apache Spark is the hot new thing in Big Data. It has significant improvements over Hadoop, the Google product, and solves a lot of the Hadoop problems, a full discussion is beyond the scope of this article. To be fair though, Spark is really like 5 separate projects all kind of lumped into one. One of those side projects is Spark Streaming. The way Spark thinks of data is in batches. Spark “Streaming” is really mini-batch processing. In the Twitter example Spark creates ‘windows’ e.g. it would take all the records (e.g. tweets) that come in every 5 seconds, or every 100 records as a mini batch then process that.

***[Apache Flink.](http://flink.apache.org/)*** Flink, came into existence a little more recently than Spark, and was built specifically to take the place of the aging Storm (Twitter quit supporting Storm and it has lost a lot of development momentum). Flink, like Storm, was built to process streaming system. It can process batches like Spark, but it considers batches to be a special case of streams where you know the stream is only so long.

***Sidebar: What is Apache?*** The [Apache Software Foundation](http://www.apache.org/foundation/) is like Habitat for Humanity for software. Professionals and companies volenteer time to improve code of the open-source they use, and the software is then free for everyone. ASF is a non for profit that coordinates and provides a set of standards for some (but not all) larger open-source projects.

## Four key things when processing streaming data in 60 seconds.

There are four interrelated concepts when it comes to processing streaming data in a distributed (team of computers) way.

***Throughput.*** How many records can a given set of hardware (team of computers) process in a given length of time. I.e. How many records (tweets) per second can you process.

***Latency.*** A concept all gamers are familiar with- a record comes in now, how long until it is processed. E.g. time between the person clicking the ‘tweet’ button and the time it shows up in your feed.

***Functionality.*** What kinds of weird black-magic data-science stuff can you do on the records to analyze them. Simple function on tweets: Counting how many came out of Chicago today. Less simple function: real-time language processing algorithm to detect terrorist tweeting they’re about to go do some terror and then alerting the authorities. The higher the functionality, the lower the throughput.

***Statefulness.*** This is more of an under-the-hood concept but an important one that I will point out because it significantly impacts the others. Statefulness and more precisely ‘Exactly Once’ statefullness refers to the systems ability to make sure the team of computers process every record that comes in and they process it only once. Somewhat trivial for most Twitter applications, VERY important for banking transactions. Consider the team of computers- what if one computer crashes halfway through processing a record, how does the system recover? Also consider someone sending a tweet when their phone was in airplane mode, they clicked send at noon, but their plane didn’t land until 5pm and then reported the tweet, how does the system rectify this?

## Comparison of the technologies and implications for digital marketers in 120 seconds.

Storm and Hadoop were based on internal code released into the wild by benevolent companies. Flink and Spark were grad-student projects that took off, both still in their early phases but already being used in production (in real life, not just toys in the R&amp;D department) at numerous firms. Spark is considerably more popular at the moment (it had a two year head start).

|  | Functionality (good) | Latency (bad) | Throughput (good) |
|---|---|---|---|
| **Spark** | <span style="color:yellow;">Mid</span> | <span style="color:red;">High</span> | <span style="color:green;">High</span> |
| **Flink** | <span style="color:red;">Low</span> | <span style="color:green;">Low</span> | <span style="color:green;">Ultra-High</span> |
| **Storm** | <span style="color:yellow;">Mid</span> | <span style="color:green;">Ultra-Low</span> | <span style="color:red;">Low</span> |

I left Hadoop off this chart because it isn’t a viable option at all for streaming data. I also want to call out the throughput for Flink vs. Storm: A [study by Data Artisans](http://data-artisans.com/high-throughput-low-latency-and-exactly-once-stream-processing-with-apache-flink/) found that on a particular task and hardware set, Flink achieved average throughput of 690,000 records per second per core, compared to Storm’s 2,600 records per second on similar hardware. That is a mind-blowing increase in throughput, and that one thing you must remember or nothing that follows will seem wondrous.

#### Here in lies the crux of why any of this matters to digital marketers:

A sea change is coming. All of the RTB, targeting, digital hocus-pocus / etc. that you/your vendors currently do, is about to see a ***hardware cost reduction to the tune of 1000-fold***. Another way to look at it: using current hardware, digital advertisers are about to have 1000-fold more computational power (for black magic algorithms) at their disposal.

To further put this in perspective, consider Rubicon, a juggernaut in the industry, currently running on Apache Storm. ***A few employees could go rogue and with in a week or two setup a platform that could honestly out perform Rubicon, on a shoe-string budget.*** You as the buyer might think, “Oh there is no way this fly-by-night operation could really out perform Rubicon”… As with any start-up I’m sure there will be hiccups, but yes- they can. Also, I use Rubicon only as an example. Any one whose core business involves making high velocity decisions at scale (read: anyone in the RTB space) is at risk.

For a short list of companies who can be upset check out the [Companies using Storm](http://storm.apache.org/documentation/Powered-By.html). ***Paradoxically, these companies are actually in the best position to move forward***, because there are guides and tools readily available for migrating from Storm to Flink. Companies that have built custom in house solutions will basically have to start from scratch either rebuilding a new in house solution, or migrating to Flink. A quick moral of the story- it behooves a firm to use open-source when ever possible, you are out sourcing a non-core-competency.

This is literally the ***horse-and-buggy versus high-speed-trains***. In my 5-minute-hip-shot talk, this is the big change that jumps off the page at me, that the giants have to competitive advantage (and arguably a competitive disadvantage, in terms of no sunk cost in defunct technologies/organizational resistance to change) against the upstarts.

The second implication is a little more pleasant than the fall and rise of digital ad empires. The Internet of Things promises to increase the amount of streaming data exponentially. To date, there hasn’t been a great solution for processing all of this data. More is not more unless there is something useful to do with all of the data that is being kicked off.  ***Unlocking potential to cost-effectively process sensor data in real time frees up a current bottle neck slowing the progression of the internet of things.***

For example, my day job is working with off-line CPG transaction data. No one has ever merged the offline store-register data with online targeting (at scale). Mainly because doing so would be an expensive pain in the butt. Well, it’s getting to a point where it would be fairly cheap and trivial exercise. That’s going to give rise to entirely new sets of strategies, tactics, and KPIs. It’s also going to unfold differently for each industry. I can’t tell you what the future looks like, only that there is a storm coming.

You were probably already expecting to see significant gains in IoT and real-time streaming technologies over the next 5 years. Now you understand why it is happening, and as Virgil says, “Fortunate is he who understands the causes of things”.