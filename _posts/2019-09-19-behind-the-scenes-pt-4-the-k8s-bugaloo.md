---
id: 2399
title: 'Behind the Scenes Pt. 4: The K8s Bugaloo.'
date: '2019-09-19T20:19:41+00:00'
author: rawkintrevo
layout: post
guid: 'http://rawkintrevo.org/?p=2399'
permalink: /2019/09/19/behind-the-scenes-pt-4-the-k8s-bugaloo/
jabber_published:
    - '1568924383'
timeline_notification:
    - '1568924385'
email_notification:
    - '1568924385'
publicize_twitter_user:
    - rawkintrevo
image: /wp-content/uploads/2019/09/bad-plumbing.jpg
categories:
    - 'big data for noobs'
    - flink
    - K8s
---

Let me start off by saying how glad I am to be done with this post series.

I knew when I finished the project, I should have just written all four posts, and then timed them out for delayed release. But I said, “nah, writing blog posts is future-rawkintrevo’s problem and Fuuuuuuug that guy.” So here I am again. Trying to remember the important parts of a thing I did over a month ago, when what I really care about at the moment is [Star Trek Bots](http://www.startrekbots.com). But unforuntaly I won’t get to write a blog post on that until I haven’t been working on it for a month too (jk, hopefully next week, though I trained that algorithm over a year ago I think).

OK. So let’s do this quick.

## Setting up a K8s On IBM Cloud

Since we were using OpenWhisk earlier- I’m just going to assume you have an IBMCloud account. The bummer is you will now have to give them some money for a K8s cluster. I know it sucks. I had to give them money too (actually I might have done this on a work account, I forget). Anyway, you need to give them money for a 3 cluster “real” thing, because the free ones will no allow Istio ingresses, and we are going to be using those like crazy.

## Service Installation Script

If you do anything <del>on computers</del> in life, you should really make a script so next time you can do it in a single command line. Following that them, here’s [my (ugly) script](https://github.com/rawkintrevo/ibm-ai-a-thon/blob/master/k8s/setup-svcs.sh). The short outline is :

1. Install Flink
2. Install / Expose Elasticsearch
3. Install / Expose Kibana
4. Chill out for a while.
5. Install / Expose my cheap front end from a prior section.
6. Setup Ingresses.
7. Upload the big fat jar file.

### Flink / Elasticsearch / Kibana

The Tao of Flink On K8s has long been talked about (like since at least last Flink Forward Berlin) and is outlined nicely [here](https://ci.apache.org/projects/flink/flink-docs-stable/ops/deployment/kubernetes.html). The observant reader will notice I even left a little note to myself in the script. All in all, the Flink + K8s experience was quite pleasant. There is one little kink I did have to hack around, and I will show you now.

Check out [this line](https://github.ibm.com/trevor-grant/ibm-ai-a-thon/blob/master/k8s/flink/jobmanager-deployment.yaml#L28). The short of the long of it was, the jar we made is a verrrry fat boy, and blew out the limit. So we are tweaking this one setting to allow jars of any size to be uploaded. The “right way” to do this in Flink is to leave the jars in the local lib/ folder, but for &lt;reasons&gt; on K8s, that’s a bad idea.

Elasticsearh, I only deployed single node. I don’t think multi node is supposed to be that much harder, but for this demo I didn’t need it and was busy focusing on my trashy front end design.

Kibana works fine IF ES is running smoothly. If Kibana is giving you a hard time, go check ES.

I’d like to have a moment of silence for all the hard work that went in to making this such an easy thing to do.

```
kubectl apply -f ...
kubectl expose deployment ...
```

That’s life now.

### My cheap front end and establishing Ingresses.

A little kubectl apply/expose also was all it took to expose my bootleggy website. There’s probably an entire blog post on just doing that, but again, we’re keeping this one high level. If you’re really interested check out.

- Make a simple static website, then Docker it up. ([Example](https://github.ibm.com/trevor-grant/ibm-ai-a-thon/blob/master/frontend/Dockerfile))
- Make a yaml that runs the Dockerfile you just made ([Example](https://github.ibm.com/trevor-grant/ibm-ai-a-thon/blob/master/k8s/frontend/my-frontend.yaml))
- Make an ingress that points to your exposed service. ([Example](https://github.ibm.com/trevor-grant/ibm-ai-a-thon/blob/master/k8s/frontend/myingress.yaml))

Which is actually a really nice segway into talking about Ingresses. The idea is you K8s cluster is hidden away from the world, operating in it’s own little universe. We want to poke a few holes and expose that universe to the outside.

Because I ran out of time, I ended up just using the prepackaged Flink WebUI and Kibana as iFrames on my “website”. As such, I poked *several* holes and you can see how I did it here:

- [Flink Ingress](https://github.ibm.com/trevor-grant/ibm-ai-a-thon/blob/master/k8s/frontend/myingress4.yaml)
- [Kibana Ingress](https://github.ibm.com/trevor-grant/ibm-ai-a-thon/blob/master/k8s/frontend/myingress3.yaml)
- [Elasticsearch Ingress](https://github.ibm.com/trevor-grant/ibm-ai-a-thon/blob/master/k8s/frontend/myingress2.yaml) (I think I needed this for Kibana to work right?)

Those were hand rolled and have minimum nonsense, so I think they are pretty self explanatory. You give it a service, a port, and a domain host. Then it just sort of works, bc computers are magic.

## Conclusions

So literally as I was finishing the last paragraph I got word that my little project has been awarded 3rd place, but there were a lot of people in the competition so it’s not like was 3rd of 3 ( I have a lot of friends who read this blog (only my friends read this blog?), and we tend to cut at each other a lot).

More conclusively though, a lot of times when you’re tinkering like me, its easy to get off on one little thing and not build full end to end systems. Even if you suck at building parts, it helps illustrate the vision. Imagine you’ve never seen a horse. Then imagine I draw the back of one, and tell you to just imagine what the front is like. You’re going to be like, “WTF?”. So to tie this back in to Brian Holt’s “Full Stack Developer” tweet, this image is still better than “close your eyes and make believe”.

<figure aria-describedby="caption-attachment-2405" class="wp-caption alignnone" id="attachment_2405" style="width: 680px">[![fullstack - brian holt](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2019/09/fullstack-brian-holt.jpeg?resize=680%2C457&ssl=1)](https://twitter.com/holtbt/status/977419276251430912)<figcaption class="wp-caption-text" id="caption-attachment-2405">[Brian Holt, Twitter](https://twitter.com/holtbt/status/977419276251430912)</figcaption></figure>I take this even further in my next post. I made the Star Trek Bot Algorithm over a year ago and had it (poorly) hooked up to twitter. I finally learned some React over the summer and now I have a great way to kill hours and hours of time, welcome to the new Facebook. [startrekbots.com](http://www.startrekbots.com)

At any rate, thanks for playing. Don’t sue me.