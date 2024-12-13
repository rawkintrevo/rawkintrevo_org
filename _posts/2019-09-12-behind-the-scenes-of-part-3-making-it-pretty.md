---
id: 2392
title: 'Behind the Scenes of “&#8230;&#8221;: Part 3- Making it Pretty'
date: '2019-09-12T20:37:31+00:00'
author: rawkintrevo
layout: post
guid: 'http://rawkintrevo.org/?p=2392'
permalink: /2019/09/12/behind-the-scenes-of-part-3-making-it-pretty/
jabber_published:
    - '1568320653'
timeline_notification:
    - '1568320655'
email_notification:
    - '1568320655'
publicize_twitter_user:
    - rawkintrevo
image: /wp-content/uploads/2019/09/48632202606_e47baf96f8_z.jpg
categories:
    - 'big data for noobs'
    - Engineering
    - React
---

At my first job, I was hired to do something (for 40 hours a week) that I was able to write a script to take care of in my first week. I went to my boss and showed them how efficient I was (like an idiot). Luckily, my bosses were cool, and the industry was structured as such that my company charged our clients what they were paying me times 3. So my bosses wanted me to stick around, and so gave me free range to do things that would “wow” the client.

I learned in my second through 16th week, in general, no one cares how good your code/data science/blah is, if you can’t make it pretty (I was working in marketing, however I feel that the lesson carries to just about everything beyond analytics and code monkeying).

## Becoming one of the beautiful people

I started monkeying around with React and Javascript a year or two ago over the Chicago winter, but it turned to spring before I got too far with it. This summer, I got back into it. My first project was a little diddy I like to call <https://www.carbon-canary.com> (which actually started out at <http://carbon.exposed>). Carbon Canary started off with my hacking on the Core-UI for React framework, but fairly quickly out growing it and implementing many of my own hacks. It does web login with Gmail and Outlook, and will read your email for flight receipts and convert those into your carbon footprint. In these days of flight shaming, etc. a handy tool to have. It will also chart your foot print over time, and let you add a few things “by hand”.

Carbon-Canary.com was a good first step into playing with react.

I also made <http://www.2638belden.com> which is a little experiment for a property I own and am (currently) trying to rent out and soon will use as a portal for filing maintenance requests.

So why am I telling you about these? Obviously to boost my google search rank.

As a corollary though, it can also be pointed out, that I’m sort of getting the hang of writing React, and recommend all “back end devs” learn enough of this so you can tell the front end people how to do their job.

## Becoming ugly again…

So this whole streaming IoT engine thing was for an IBM hackathon. I figured it would be in bad taste to not make the front end with IBM’s favorite design framework, [Carbon](https://www.carbondesignsystem.com).

The hardest part about learning Carbon-react was that all of the examples seem to be for some functional version of React, which I have not really seen out in the wild (I’m using object oriented). Beyond that, it gets real weird with CSS, and other stuff.

I’m looking at my [github code](https://github.com/rawkintrevo/ibm-ai-a-thon/tree/master/frontend) now, and I see it’s been a month since I messed with this.

The main things that stand out in my mind are:

1. Dealing with Carbon was a bad experience.
2. There are a lot of poorly documented “features” run amok.
3. If you never knew anything else, this framework might seem passible, but then you would have a hard time learning any other frameworks (as is common in IBM products bc &lt;design choices&gt;).

My solution after a week of fighting with Carbon to make the simplest things work ended up being an accordion menu and some iFrames of the Flink WebUi and Kabana board.

If I were doing this project for real or if I had an infinite amount of time to learn Carbon, I would like to create charts using React (instead of Kibana), which could make it look nice and pro. Having continued on my React journey for another month, I think making a nice looking frontend (sans Carbon) could be fairly easy done, as well as the components for submitting new jobs, etc. (e.g. replacing the FlinkWeb UI i-frame).

## Conclusion / Next Time:

I know this was a short one where I mainly just plugged my other sites and dumped on Carbon, but it’s hard to write some deep cuts about a thing you’re just learning yourself. That said, I don’t really want to be a front end person, I just want to be able to hack some stuff enough to get a series A and pay one. If you know anyone who wants to give me a series A please have them contact me.

Next time, we’ll return to the land of Trevor having some passing idea of what he’s talking about as we discuss laying out the K8s ingresses to make all of this work on IBMCloud.

###### Photo Credit: Trevor Grant; [https://www.flickr.com/photos/182053726@N05/48632202606/in/dateposted-public/ ](https://www.flickr.com/photos/182053726@N05/48632202606/in/dateposted-public/ ) ; Subject “Robin Williams Mural” by Jerkface and Owen Dippie- Logan Square, Chicago IL