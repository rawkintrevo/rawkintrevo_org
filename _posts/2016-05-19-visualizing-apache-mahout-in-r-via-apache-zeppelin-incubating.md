---
id: 299
title: 'Deep Magic Volume 1: Visualizing Apache Mahout in R via Apache Zeppelin (incubating)'
date: '2016-05-19T03:18:28+00:00'
author: rawkintrevo
layout: post
guid: 'http://trevorgrant.org/?p=299'
permalink: /2016/05/19/visualizing-apache-mahout-in-r-via-apache-zeppelin-incubating/
jabber_published:
    - '1463627910'
email_notification:
    - '1463627913'
publicize_twitter_user:
    - rawkintrevo
publicize_google_plus_url:
    - 'https://plus.google.com/107169808969912762220/posts/BQ2WcekWA8Y'
publicize_linkedin_url:
    - 'https://www.linkedin.com/updates?discuss=&scope=94269141&stype=M&topic=6138900449413058560&type=U&a=JK4Q'
image: /wp-content/uploads/2016/05/color.jpg
categories:
    - flink
    - How-Tos
    - mahout
    - zeppelin
tags:
    0: apache
    1: 'big data'
    2: 'distributed computing'
    3: ggplot2
    4: linearalgebra
    5: machinelearning
    7: matplotlib
    8: r
    9: spark
---

I was at Apache Big Data last week and got to talking to some of the good folks at the Apache Mahout project. For those who aren’t familiar, [Apache Mahout](http://mahout.apache.org/) is a rich Machine Learning and Linear Algebra Library that originally ran on top of Apache Hadoop, and as of recently runs on top of Apache Flink and Apache Spark. It runs in the interactive Scala shell but exposes a domain specific language that makes it feel much more like R than Scala.

Well, the Apache Mahout folks had been wanting to build out some visualization capabilities comparable to `matplotlib` and `ggplot2` (Python and R respectively). They had considered integrating with Apache Zeppelin and utilizing the AngularJS framework native to Zeppelin. We talked it out, and decided it made much more sense to simply *use* *the* matplotlib *and* ggplot2 *features of Python and R, and Apache Zeppelin could to facilitate that somewhat cumbersome pipeline.*

So I dinked around with it Monday and Tuesday, learning my way around Apache Mahout, and overcoming an issue with an upgrade I made when I rebuilt Zeppelin (in short I needed to refresh my browser cache…).

Without further ado, here is a guide on how to get started playing with Apache Mahout yourself!

## Step 1. Clone / Build Apache Mahout

At the bash shell (e.g. command prompt, see my other blog post on setting up [Zeppelin + Flink + Spark](http://trevorgrant.org/2015/11/03/apache-casserole-a-delicious-big-data-recipe-for-the-whole-family/)), enter the following:

```

git clone https://github.com/apache/mahout.git
cd mahout
mvn clean install -DskipTests
```

That will install Apache Mahout.

## Step 2. Create/Configure/Bind New Zeppelin Interpreter

### Step 2a. Create

Next we are going to create a new [Zeppelin Interpreter](https://zeppelin.incubator.apache.org/docs/0.5.6-incubating/manual/interpreters.html).

In the interpreters page, at the top right, you’ll see a button that says: “+Create”. Click on that.

We’re going to name this ‘spark-mahout’ (thought the name is not important).  
On the interpreter drop-down we’re going to select Spark.

![create-terp2](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2016/05/create-terp2.png?resize=373%2C273&ssl=1)

### Step 2b. Configure

We’re going to add the following properties and values by clicking the “+” sign at the bottom of the properties list:

| Property | Value |
|---|---|
| spark.kryo.registrator | org.apache.mahout.sparkbindings.io.MahoutKryoRegistrator |
| spark.serializer | org.apache.spark.serializer.KryoSerializer |
| spark.kryo.referenceTracking | false |
| spark.kryoserializer.buffer | 300m |

And below that we will add the following artifacts to the dependencies (no value necessary for the ‘exclude’ field)

| Artifact | Exclude |
|---|---|
| /home/username/.m2/repository/org/apache/mahout/mahout-math/0.12.1-SNAPSHOT/mahout-math-0.12.1-SNAPSHOT.jar |  |
| /home/username/.m2/repository/org/apache/mahout/mahout-math-scala\_2.10/0.12.1-SNAPSHOT/mahout-math-scala\_2.10-0.12.1-SNAPSHOT.jar |  |
| /home/username/.m2/repository/org/apache/mahout/mahout-spark\_2.10/0.12.1-SNAPSHOT/mahout-spark\_2.10-0.12.1-SNAPSHOT.jar |  |
| /home/username/.m2/repository/org/apache/mahout/mahout-spark-shell\_2.10/0.12.1-SNAPSHOT/mahout-spark-shell\_2.10-0.12.1-SNAPSHOT.jar |  |
| /home/username/.m2/repository/org/apache/mahout/mahout-spark\_2.10/0.12.1-SNAPSHOT/mahout-spark\_2.10-0.12.1-SNAPSHOT-dependency-reduced.jar |  |

Make sure to click ‘Save’ when you are done. Also, maybe this goes without saying, maybe it doesn’t… but

#### make sure to change `username` to your actual username, don’t just copy and paste!

![dependencies](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2016/05/dependencies.png?resize=685%2C351&ssl=1)

### Step 2c. Bind

In any notebook in which you want to use the `spark-mahout` interpreter, not the regular old Spark one, you need to *bind* correct interpreter.

Create a new notebook, lets call it “\[MAHOUT\] Binding Example”.

In the top right, you’ll see a little black gear, click on it. A number of interpreters will pop up. You want to click on the Spark one at the top (such that is becomes un-highlighted) then click on the “spark-mahout” one toward the bottom. Finally drag the “spark-mahout” one up to the top. Finally, as always, click on ‘Save’.

Now, this notebook knows to use the spark-mahout interpreter instead of the regular spark interpreter (and so, all of the properties and dependencies you’ve added will also be used). **You’ll need to do this for every notebook in which you wish to use the Mahout Interpreter!**

![terp binding](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2016/05/terp-binding2.png?resize=512%2C830&ssl=1)

### Step 2d. Setting the Environment

Back at the command prompt, we need to tweek the environment a bit. At the command prompt (assuming you are in the `mahout` directory still):

```

./bin/mahout-load-spark-env.sh 

```

And then we’re going to export some environment variables:

```

export MAHOUT_HOME=[directory into which you checked out Mahout]
export SPARK_HOME=[directory where you unpacked Spark]
export MASTER=[url of the Spark master]

```

If you are going to be using Mahout often, it would be wise to add those exports to `$ZEPPELIN_HOME/conf/zeppelin-env.sh` so they are loaded every time.

## Step 3. Mahout it up!

I don’t like to repeat other people’s work, so I’m going to direct you to another great article explaining how to do simple matrix based linear regression. <https://mahout.apache.org/users/sparkbindings/play-with-shell.html>

I’m going to do you another favor. Go to the Zeppelin home page and click on ‘Import Note’. When given the option between URL and json, click on URL and enter the following link:

`<a href="https://raw.githubusercontent.com/rawkintrevo/mahout-zeppelin/master/%5BMAHOUT%5D%5BPROVING-GROUNDS%5DLinear%20Regression%20in%20Spark.json" target="_blank">https://raw.githubusercontent.com/rawkintrevo/mahout-zeppelin/master/%5BMAHOUT%5D%5BPROVING-GROUNDS%5DLinear%20Regression%20in%20Spark.json</a>`

That should run, and is in fact the Zeppelin version of the above blog post.

The key thing I will point out however is the top of the first paragraph:

```

import org.apache.mahout.math._
import org.apache.mahout.math.scalabindings._
import org.apache.mahout.math.drm._
import org.apache.mahout.math.scalabindings.RLikeOps._
import org.apache.mahout.math.drm.RLikeDrmOps._
import org.apache.mahout.sparkbindings._

implicit val sdc: org.apache.mahout.sparkbindings.SparkDistributedContext = sc2sdc(sc)

```

That is where the magic happens and introduces Mahout’s *SparkDistributedContext* and the R-like Domain Specific Language.

You know how in Scala you can pretty much just write whatever you want (syntactic sugar run-amok) well a domain specific language (or DSL) lets you take that even further and change the syntax even further. This is not a precisely accurate statement, feel free to google if you want to know more.

The moral of the story is: what was Scala, now smells much more like R.

Further, for the rest of this notebook, you can now use the Mahout DSL, which is nice because it is the same for Flink and Spark. What *that* means is you can start playing with this right away using Spark-Mahout, but when the Flink-Mahout comes online soon (and I promise to update this post showing how to hook it up) you can copy/paste your code to your Flink-Mahout paragraphs and probably run it a bunch faster.

## The Main Event

So the whole point of all of this madness was to monkey-patch Mahout into R/Python to take advantage of those graphics libraries.

I’ve done you another solid favor. Import this notebook:  
`<a href="https://raw.githubusercontent.com/rawkintrevo/mahout-zeppelin/master/%5BMAHOUT%5D%5BPROVING-GROUNDS%5DSpark-Mahout%2Bggplot2.json" target="_blank">https://raw.githubusercontent.com/rawkintrevo/mahout-zeppelin/master/%5BMAHOUT%5D%5BPROVING-GROUNDS%5DSpark-Mahout%2Bggplot2.json</a>`

**UPDATE 5-29-16:** Originally, I had accidentally re-linked the first notebook (sloppy copy-paste on my part)- this one shows ggplot2 integration, e.g. the entire point of this Blog post…

Ignore the first couple of paragraphs (by the time you read this I might have (unlikely, lol) cleaned this notebook up and deleted).

There is a paragraph that **Creates Random Matrices**

![terp setup and create random matrices](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2016/05/terp-setup-and-create-random-matrices.png?resize=685%2C391&ssl=1)

…yawn. You can grok it later. But again, notice those imports and creating the SparkDistributedContext. We’re using our SparkContext (`sc` ) that Zeppelin automatically creates in the paragraph to initialize this.

In the next paragraph we sample 1000 rows from the matrix. Why a sample? Well in theory the whole point of using Mahout is you’re going to be working with matrices much to big to fit in the memory of a single machine, much less graph them in any sort of meaningful way (think millions to trillions to bajillions of rows). How many do you really need. If you want to get a feel for the matrix as a whole, random sample. Depending on what you’re trying to do will determine how exactly you sample the matrix, just be advised- it is a nasty habit to think you are just going to visualize the whole thing (even though it is possible on these trivial examples). If that were possible in the first place on your real data, you’d have actually been better served to just used R to begin with…

The next paragraph basically converts the matrix into a tab-separated-file, except it is held as a string and never actually written to disk. This loop is effective, but not ideal. In the near future we hope to wrap some syntactic sugar around this, simply exposing a method on the matrix that spits out a sampled \*.tsv. Once there exists a tab-separated string, we can add `%table` to the front of the string and print it- Zeppelin will automatically figure out this is supposed to be charted and you can see here how we could use Zeppelin’s predefined charts to explore this table.

![angular vis](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2016/05/angular-vis.png?resize=685%2C463&ssl=1)

Keeping in mind this matrix was a sine function, this sampling looks more or less accurate. The Zeppelin graph is trying to take some liberties though and do aggregations on one of the columns. To be fair, we’re trying to do something weird here; something for which this chart wasn’t intended for.

Next, the tsv string is then stored in something known to Zeppelin as the *ResourcePool*. Almost any interpreter can access the resource pool and it is a great way to share data between interpreters.

Once we have a \*.tsv in memory, and it’s in the resource pool, all that is left is to “fish it out” of the resource pool and load it as a dataframe. That is an uncommon but not altogether unheard of thing to do in R via the `read.table` function.

Thanks to all of the work done on the SparkR-Zeppelin integration, we can now load our dataframe and simply use ggplot2 or a host of other R plotting packages (see the R tutorial).

![handoff to r](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2016/05/handoff-to-r.png?resize=571%2C797&ssl=1)

#### A post thought

Another way to skin this cat would be to simply convert the Mahout Matrix to an RDD and then register it as a DataFrame in Spark. That is correct, however the point of Mahout is to be engine agnostic, and as Flink is mainly focused on streaming data and not building out Python and R extensions, it is unlikely a similar functionality would be exposed there.

However, you’re through the looking-glass now, and if doing the distributed row matrix -&gt; resilient distributed data set -&gt; Spark data frame -&gt; read in R makes more sense to you/your use case, go nuts. Write a blog of your own and link back to me 😉