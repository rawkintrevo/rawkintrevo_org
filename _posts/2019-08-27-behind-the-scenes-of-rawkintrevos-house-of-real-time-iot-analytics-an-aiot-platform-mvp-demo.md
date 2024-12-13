---
id: 2364
title: 'Behind the Scenes of &#8220;Rawkintrevo&#8217;s House of Real Time IoT Analytics, An AIoT platform MVP Demo&#8221;'
date: '2019-08-27T21:04:33+00:00'
author: rawkintrevo
layout: post
guid: 'http://rawkintrevo.org/?p=2364'
permalink: /2019/08/27/behind-the-scenes-of-rawkintrevos-house-of-real-time-iot-analytics-an-aiot-platform-mvp-demo/
jabber_published:
    - '1566939875'
timeline_notification:
    - '1566939876'
email_notification:
    - '1566939877'
advanced_seo_description:
    - 'Apache Flink, Watson IoT, Apache OpenWhisk, IoT, Kubernetes K8s'
publicize_twitter_user:
    - rawkintrevo
image: /wp-content/uploads/2019/08/dsc00721_md.jpg
categories:
    - 'big data for noobs'
    - Engineering
    - flink
    - How-Tos
    - IoT
    - K8s
    - openwhisk
    - python
    - React
    - video
---

Woo, that’s a title- amirigh!?

It’s got everything- buzzwords, a corresponding [YouTube video](https://www.youtube.com/watch?v=EKzv1YGukEk), a Twitter handle conjugated as a proper noun.

## Introduction

Just go watch the [video](https://www.youtube.com/watch?v=EKzv1YGukEk)– I’m not trying to push traffic to YouTube, but it’s a sort of complicated thing and I don’t do a horrible job of explaining it in the video. You know what, I’m just gonna put it in line.

<div class="jetpack-video-wrapper"><span class="embed-youtube" style="text-align:center; display: block;"><iframe allowfullscreen="true" class="youtube-player" height="386" loading="lazy" sandbox="allow-scripts allow-same-origin allow-popups allow-presentation allow-popups-to-escape-sandbox" src="https://www.youtube.com/embed/EKzv1YGukEk?version=3&rel=1&showsearch=0&showinfo=1&iv_load_policy=1&fs=1&hl=en-US&autohide=2&wmode=transparent" style="border:0;" width="685"></iframe></span></div>Ok, so Now you’ve see that. And you’re wondering? How in the heck?! Well good news- because you’ve stumbled to the behind the scenes portion where I explain how the magic happened. There’s a lot of magic going on in there, and some you probably already know and some you’ve got no idea. But this is the story of my journey to becoming a full stack programmer. As it is said in the [Tao of Programming](http://www.mit.edu/~xela/tao.html):

> There once was a Master Programmer who wrote unstructured programs. A novice programmer, seeking to imitate him, also began to write unstructured programs. When the novice asked the Master to evaluate his progress, the Master criticized him for writing unstructured programs, saying, “What is appropriate for the Master is not appropriate for the novice. You must understand Tao before transcending structure.”

I’m not sure if I’m the Master or the Novice- but this program is definitely unstructured AF. So here is a companion guide that maybe you can learn a thing or two / Fork my repo and tell your boss you did all of this yourself.

## Table of Contents

Here’s my rough outline of how I’m going to proceed through the various silliness of this project and the code contained in my [github repo](https://github.com/rawkintrevo/ibm-ai-a-thon) .

1. **YOU ARE HERE**. A sarcastic introduction, including my dataset, WatsonIoT Platform (MQTT). Also we’ll talk about our data source- and how we shimmed it to push into MQTT, but obviously could (should?) do the same thing with Apache Kafka (instead). I’ll also introduce the chart- we might use that as a map as we move along.
2. In the second post, I’ll talk about my Apache Flink streaming engine- how it picks up a list of REST endpoints and then hits each one of them. In the comments of this section you will find people telling me why my way was wrong and what I should have done instead.
3. In this post I’ll talk about my meandering adventures with React.js, and how little I like the [Carbon Design System](https://www.carbondesignsystem.com). In my hack-a-thon submission, I just iFramed up the Flink WebUI and Kibana, but here’s where I would talk about all the cool things I would have made if I had more time / Carbon-React was a usable system.
4. In the last post I’ll push this all on IBM’s K8s. I work for IBM, and this was a work thing. I don’t have enough experience on any one else’s K8s (aside from microK8s which doesn’t really count) to bad mouth IBM. They do pay me to tell people I work there, so anything to rude in the comments about them will most likely get moderated out. F.u.

## Data Source

See [README.md](https://github.com/rawkintrevo/ibm-ai-a-thon/blob/master/README.md) and scroll down to Data Source. I’m happy with that description.

As the program is currently, right about [here](https://github.com/rawkintrevo/ibm-ai-a-thon/blob/master/flink-runtime/src/main/java/org/rawkintrevo/aiathon/DeviceDataAnalysis.java#L60) the schema is passed as a string. My plan was to make that an argument so you could submit jobs from the UI. Suffice to say, if you have some other interesting data source- either update that to be a command line parameter (PRs are accepted) or just change the string to match your data. I was also going to do something with Schema inference, but my Scala is rusty and I never was great at Java, and tick-tock.

## Watson IoT Platform

I work for IBM, specifically Watson IoT, so I can’t say anything bad about WatsonIoT. It is basically based on [MQTT](https://en.wikipedia.org/wiki/MQTT), which is a pub-sub thing IBM wrote in 1999 (which was before Kafka by about 10 years, to be fair).

If you want to see my hack to push data from the Divvy API into Watson IoT Platform, you can see it [here](https://github.com/rawkintrevo/ibm-ai-a-thon/blob/master/divvy-shim-openwhisk/inject_20_divvy_events.py). You will probably notice a couple of oddities. Most notably, that only 3 stations are picked up to transmit data. This is because the Free account gets shut down after 200MB of data and you have to upgrade to a $2500/mo plan bc IBM doesn’t really understand linear scaling. /shrug. Obviously this could be easily hacked to just use Kafka and update the Flink Source [here](https://github.com/rawkintrevo/ibm-ai-a-thon/blob/master/flink-runtime/src/main/java/org/rawkintrevo/aiathon/DeviceDataAnalysis.java#L99).

## The Architecture Chart

That’s also in the github, so I’m going to say just look at it on README.md.

## Coming Up Next:

Well, this was just about the easiest blog post I’ve ever written. Up next, I may do some real work and get to talking about my Flink program which picks up a list of API endpoints every 30 seconds, does some sliding window analytics, and then sends each record and the most recent analytics to each of the end points that were picked up, and how in its way- this gives us dynamic model calling. Also- I’ll talk about the other cool things that could/should be done there that I just didn’t get to. /shrug.

See you, Space Cowboy.