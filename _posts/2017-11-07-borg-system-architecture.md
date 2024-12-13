---
id: 2122
title: 'Borg System Architecture'
date: '2017-11-07T18:31:39+00:00'
author: rawkintrevo
layout: post
guid: 'http://rawkintrevo.org/?p=2122'
permalink: /2017/11/07/borg-system-architecture/
jabber_published:
    - '1510079501'
email_notification:
    - '1510079504'
publicize_twitter_user:
    - rawkintrevo
publicize_google_plus_url:
    - 'https://plus.google.com/107169808969912762220/posts/abBm5AGHkHN'
publicize_linkedin_url:
    - 'https://www.linkedin.com/updates?discuss=&scope=94269141&stype=M&topic=6333732554021048320&type=U&a=O1tJ'
image: /wp-content/uploads/2017/11/futurama.jpg
categories:
    - Engineering
    - flink
    - How-Tos
    - mahout
---

### or “how I accidentally enslaved humanity to the Machine Overlords”.

The Borg are a fictional group from the Sci-Fi classic, Star Trek, who among other things have a collective consciousness. This creates a number of problems for the poor humans (and other species) that attempt to resist the Borg, as they are extremely adaptive. When a single Borg Drone learns something, its knowledge is very quickly propagated through the collective, presumably subject to network connectivity issues, and latency.

Here we create a system for an arbitrary number edge devices to report sensor data, a central processor to use the data to understand the environment the edge devices are participating in, and finally to make decisions / give instructions back to the edge device. This is in essence what the Borg are doing. Yes, there are some interesting [biological / cybernetic integrations](http://nypost.com/2015/10/25/five-amputees-show-off-the-amazing-advances-in-artificial-limbs/), however as far as the “[hive mind](http://memory-alpha.wikia.com/wiki/Hive_mind)” aspect is concerned, this is basic principles in play.

I originally built this toy to illustrate that “A.I.” has three principle components: Real time data going into a system, an understanding of the environment is reached, a decision is made. (1) *Real Time* artifical intelligence, like the “actual” intelligence it supposedly mimics is not making E.O.D. batch decisions. (2) In real time the system is aware of what is happening around it- understanding its environment and then using that understanding to (3) make some sort of decision about how to manipulate that environment. [Read up on definitions of intelligence, a murky subject itself.](https://en.wikipedia.org/wiki/Intelligence#Definitions)

Another sweet bonus, I wanted to show that sophisticated A.I. can be produced with off-the-shelf components and a little creativity, despite what vendors want to tell you. Vendors have their place. It’s one thing to make something cool, another to productionalize it- and maybe you just don’t care enough. However, since you’re reading this- I hope you at least care a little.

Artificial Intelligence is by no means synonymous with Deep Learning, though Deep Learning can be a very useful tool for building A.I. systems. This case does real time image recognition, and you’ll note does not invoke Deep Learning or even the less buzz-worthy “neural nets” at any point. Those can be easily introduced to the solution, but you don’t need them.

Like the Great and Powerful Oz, once you pull back the curtain on A.I. you realize its just some old man who got lost and creatively used resources he had lying around to create a couple of interesting magic tricks.

![oz.gif](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2017/11/oz.gif?resize=438%2C291&ssl=1)


## System Architecture

[OpenCV](https://opencv.org) is the Occipital Lobe, this is where faces are identified in the video stream.

[Apache Kafka](https://kafka.apache.org) is the nervous system, how messages are passed around the collective. (If we later need to defeat the Borg, this is probably the best place to attack- presuming we of course we aren’t able to make the drones self aware).

![hugh.gif](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2017/11/hugh.gif?resize=348%2C271&ssl=1)

[Apache Flink](http://flink.apache.org) is the collective consciousness of our Borg Collective, where thoughts of the Hive Mind are achieved. This is probably intuitive if you are familiar with Apache Flink.

[Apache Solr](http://lucene.apache.org/solr/) is the store of the “memories” of the collective consciousness.

The [Apache Mahout](http://mahout.apache.org) library is the “higher order brain functions” for understanding. It is an ideal choice as it is well integrated with Apache Flink and Apache Spark

Apache Spark with Apache Mahout gives our creation a sense of conext, e.g. how do I recognize faces? It quickly allows us to bootstrap millions of years of evolutionary biological processes.

## A Walk Through

(1) Spark + Mahout used to calculate [eigenfaces](https://en.wikipedia.org/wiki/Eigenface) (see [previous blog post](http://rawkintrevo.org/2016/11/10/deep-magic-volume-3-eigenfaces/)).

(2) Flink is started, it loads the calculated eigenfaces from (1)

(3) A video feed is processed with OpenCV .

(4) OpenCV uses [Haar Cascade Filters](https://docs.opencv.org/trunk/d7/d8b/tutorial_py_face_detection.html) to detect faces.

(5) Detected faces are turned in to Java Buffered Images, greyscaled and size-scaled to the size used for Eigenface calculations and binarized (inefficient). The binary arrays are passed as messages to Kafka.

(6) Flink picks up the images, converts them back to buffered images. The buffered image is then decomposed into linear a linear combination of the Eigenfaces calculated in (1).

(7) Solr is queried for matching linear combinations. Names associated with best *N* matches are assigned to each face. I.e. face is “identified”… poorly. See next comments.

(8) If the face is of a “new” person, the linear combinations are written to Solr as a new potential match for future queries.

(8) Instructions for edge device written back to Kafka messaging queue as appropriate.

## Problems

A major problem we instantly encountered was that sometimes OpenCV will “see” faces that do not exist, as patterns in clothing, shadows, etc. To overcome this we use Flink’s sliding time window and Mahout’s Canopy clustering. Intuitively, faces will not momentarily appear and disappear within a frame, cases where this happens are likely errors on the part of OpenCV. We create a short sliding time window and cluster all faces in the window based on their X, Y coordinates. Canopy clustering is used because it is able to cluster all faces in one pass, reducing the amount of introduced latency. This step happens between step (6) and (7)

In the resulting clusters there are either lots of faces (a true face) or a very few faces (a ghost or shadow, which we do not want). Images belonging to the former are further processed for matches in step (7).

Another challenge is certain frames of a face may look like someone else, even though we have been correctly identifying the face in question in nearby frames. We use our clusters generated in the previous hack, and decide that people do not spontaneously become other people for an instant and then return. We take our queries from step 7 and determine who the person is based on the cluster, not the individual frames.

Finally, as our Solr index of faces grows, our searches in Solr will become less and less effecient. Hierarchical clustering is believed to speed up these results and be akin to how people actually recognize each other. In the naive form, for each Solr Query will scan the entire index of faces looking for a match. However we can clusters the eigenface combinations such that each query will first only scan the cluster centriods, and then only consider eigenfaces in that cluster. This can potentially speed up results greatly.

## Usecases

### Borg

This is how the Borg were able to recognize Locutus of Borg.

![](https://i0.wp.com/www.startrek.com/legacy_media/images/200508/ds9-401-locutus-at-wolf359/320x240.jpg?w=685)

### Cylons

This type of system also was imperative for Cylon Raiders and Centurions to recognize (and subsequently not inadvertently kill) the Final Five.

![](https://i0.wp.com/i.pinimg.com/originals/48/db/08/48db080c8b9d14500e65f8da56a3998c.jpg?w=685&ssl=1)

### Shorter Term

This toy was originally designed to work with the [Petrone Battle Drones](https://www.amazon.com/Quadcopter-Battle-Drone-Bundle-Multiplayer/dp/B01MDU3363/ref=sr_1_5?s=toys-and-games&ie=UTF8) however as we see the rise of [Sophia](http://sophiabot.com) and [Atlas](https://www.bostondynamics.com/atlas), this technology could be employed to help multiple subjects with similar tasks learn and adapt more quickly. Additionally there are numerous applications in security (think network of CCTV cameras, remote locks, alarms, fire control, etc.)

Do you want Cylons? Because that’s how you get Cylons.

Alas, there is no great and powerful Oz. Or- there is, and …

![oz2.gif](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2017/11/oz2.gif?resize=420%2C307&ssl=1)

## References

**<span style="text-decoration:underline;">Flink Forward, Berlin 2017</span>**  
[Slides](https://www.slideshare.net/FlinkForward/flink-forward-berlin-2017-trevor-grant-do-i-know-you-real-time-facial-recognition-with-an-apache-stack) [Video](https://www.youtube.com/watch?v=DEQzoclTzbY) (warning I was sick this day. Not my best work).

<span style="text-decoration:underline;">**Lucene Revolution, Las Vegas 2017** </span>[Slides](https://www.slideshare.net/lucidworks/solr-and-machine-vision-scott-cote-lucidworks-trevor-grant-ibm) [Video](https://www.youtube.com/watch?v=wHBs_7VaScU)

[My Git Repo](https://github.com/rawkintrevo/cylons)

[PR Donating this to Apache Mahout](https://github.com/apache/mahout/pull/347)  
If you’re interested in contributing, please start here.