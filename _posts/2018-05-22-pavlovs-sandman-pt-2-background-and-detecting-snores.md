---
id: 2340
title: 'Pavlov&#8217;s Sandman Pt 2: Background and Detecting Snores'
date: '2018-05-22T19:18:42+00:00'
author: rawkintrevo
excerpt: 'Where rawkintrevo learns some hard data science lessons. '
layout: post
guid: 'http://rawkintrevo.org/?p=2340'
permalink: /2018/05/22/pavlovs-sandman-pt-2-background-and-detecting-snores/
jabber_published:
    - '1527016724'
timeline_notification:
    - '1527016725'
email_notification:
    - '1527016726'
publicize_twitter_user:
    - rawkintrevo
publicize_google_plus_url:
    - 'https://plus.google.com/107169808969912762220/posts/Wwc5LPkiBfU'
publicize_linkedin_url:
    - 'https://www.linkedin.com/updates?discuss=&scope=94269141&stype=M&topic=6404772408208695296&type=U&a=5UTn'
image: /wp-content/uploads/2017/11/pavlovs-sandman.png
categories:
    - Engineering
    - How-Tos
    - python
---

Welcome to the long anticipated follow on post to [Pavlov’s Sandman: Pt 1](http://rawkintrevo.org/2017/11/28/pavlovs-sandman-pt-1/). In that post I admitted that I had a problem with snoring, and laid out a basic strategy and app to stop snoring. They say that admitting you have a problem is half of the battle. In this case that is untrue- hacking the shock collar was half of the battle, and the other half of the battle was detecting snores.

I had a deadline for completing this project, ODSC East in Boston, at the beginning of May. What the talk ended up being about (and in turn what this blog post will now be about) is how in “data science” we usually start with a problem, that based on some passing knowledge seems very solvable and then the million little compromises we make with ourselves on our way to solving the problem/completing a product.

This blog post and the one or two that follow will be an exposition / journal of my run-and-gun data science as I start with assuming a problem is easy, have some trouble but assume I have plenty of time, realize time is running out, panic, be sloppy with experiments, make a critical break through in the 11th hour, and then deliver an OK product / talk / blog post series.

## A brief history of the author as a snorer.

As best as I can tell, I began snoring in Afghanistan. This isn’t suprising, the air in Kabul was so bad the army gave me a note saying that if I ever had respitory issues, it was probably their fault (in spite of the fact that everyone was smoking a pack per day of Luckys / Camel Fulls / Marlboro Reds ). This is to say nothing of the burn pits I sat next to to keep warm while on gate duty from December until March, or the five thousand years of sheep-poo-turned-to-moon-dust always blowing around in the country side west of Kabul City.

As a brief aside, do you know what’s really fun about trying to “research” when you started snoring? Making a list of ex’s and then contacting each of them out of the blue with a “hey, weird question but…”, it’s like a much more fun version of the “Who gave me the STD?” game.

After Afghanistan, girls I was sleeping with would occasionally complain of my snoring. This came to a head with my ex though. She would complain, elbow me, sleep on the couch, etc. But I was more concerned when the issue came up with my new girl friend (a very light sleeper, or I’ve gotten much worse about snoring). This was especially concerning, because I had tried every snoring “remedy” on the internet and had no success.

## Break throughs on other fronts.

I have a puppy named Apache, and was at the trainers. They convinced me that I should start using a wireless collar ( a shock collar ). The guy who trained me taught me that you don’t want to hurt the dog, you want to deliver the lightest shock they can feel and just keep tapping them with that until they stop doing what they should be doing. The shock should be uncomfortable, not painful.

One of the “remedies” I had tried for my snoring before was an app called Sleep as Android. In this app there was an “anti-snoring” function where the phone would buzz or make a noise when you were snoring- this had no effect, but I had always wished I could rig it out to a shock collar.

Finally, in November of last year- I discovered that you can buy a shock collar which is controlled via Bluetooth on Amazon for about $50. (PetSafe).

I have done some device hacking, I figured I could figure out the shock collar easily enough. Detecting snores also seemed easy enough. So I wrote a paper proposal for ODSC and started working on the issue. (And wrote the last blog post which recorded me snoring).

## Snore Detection and the Data Science Process of the Author

Traditionally when I start on a project, I try to come up with an exceptionally simple-enough-to-explain-to-a-5-year-old type of algorithm just so I have a baseline to test against. The benefits to this are 1) having a baseline to train against, but 2) to become “familiar with the data”.

My first attempt at a “simple snore detector” was to attempt to fit a sin curve to the volume of the recorded noises. This got me used to working with Python audio controls and sound files. I also learned right away this wasn’t going to work because the “loud” part of the snore happens then there is a much longer quiet portion. That is to say we don’t breath evenly. I don’t have sleep apnea (that is to say I don’t stop breathing), so the snores are relatively evenly spaced apart, but there are also “other noises” and various other reasons, the sin wave curve fitting just wasn’t ideal.

At this point I went back and read some academic literature on snore detection. There isn’t a lot, but there is a bit.

[Automatic Detection of Snoring Events Using Gaussian Mixture Models](https://pdfs.semanticscholar.org/05a7/c2c7a37fdccd2942db920d24be0aa295c159.pdf) by Dafna et. al.

[Automatic detection, segmentation and assessment of snoring from ambient acoustic data ](http://iopscience.iop.org/article/10.1088/0967-3334/27/10/010/meta)by Duckitt et. al.

<div class="gs_ri">[An efficient method for snore/nonsnore classification of sleep sounds](http://iopscience.iop.org/article/10.1088/0967-3334/28/8/007/meta) Cavusoglu et. al.

#### My Synopsis

Dafna reconstructed when the patient was snoring by looking at the entire night of data and looking at how volume compared to the average. Following his method and converting it to “real-time” detection however, was going to be problematic.

Duckitt created a Hidden Markov Model (mid 2000s speak for LSTM) (yes I know they’re not that same) with the states **snoring**, **not-snoring**, **other-noises**, **breathing**, **duvet-noise**. An interesting idea, one I might visit for a “real version”.

Cavusoglu looked at subband energy distributions, inter and intra individual spectra energy distributions, some principal component analysis, in other words- **MATH**. I liked this guys approach and decided to mimic it next.

#### PyAudioAnalysis

[pyAudioAnalysis](https://github.com/tyiannak/pyAudioAnalysis) is a package created and maintained(ish) by Theodoros Giannakopoulos. It will break audio files down into 32 features, including the ones used by Cavusoglu. From there I tried some simple logistic regression, random forrest classification, and K-nearest-neighbors classification.

The results weren’t bad, but I was **VERY opposed** to false positives (e.g. getting shocked when I didn’t snore. The numbers I was getting on validation just didn’t inspire me (though looking back I think I could probably have been ok.)

![Screen Shot 2018-05-22 at 1.59.18 PM](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2018/05/screen-shot-2018-05-22-at-1-59-18-pm.png?resize=685%2C149&ssl=1)

#### Back to Basics

A quick note on the “equipment” I am using to record, it is a laptop mic, which is basically trash. Lots of back ground noise. At this point, I had been playing with audio files for a while. I decided to see if I could isolate the frequency bands of my snoring.

In short I found that I normally snore at 1500-2000Hz, 5khz-7khz, and occasionally at 7khz-15khz. I decided to revisit the original loud noises idea, but this time, I would filter the original recording (for the last 5 seconds) into 1500-2khz and 5-7khz. If there was a period which was over the average + 1 standard deviation for the clip, which lasted longer than 0.4 seconds, but less than 1.2 seconds, then there was a pause in which the intensity (volume at that frequency band) was less than the threshold (mean + 1.5 stdevs) for 2.2-4.4 seconds and then another period where the intensity was above the threshold for 0.4 to 1.2 seconds, then we would be in a state of snoring.

This worked exceptionally well, except when I deployed it, I accidentally set the bands at 1500-2khz and 5khz-7khz, which missed a lot of snores. I will be updating shortly.

![Screen Shot 2018-05-22 at 2.13.08 PM.png](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2018/05/screen-shot-2018-05-22-at-2-13-08-pm.png?resize=685%2C259&ssl=1)

In the upper image above, we see the original audio file intensity (blue) over time, and in orange is the intensity on the 5-7khz band. The black line is the threshold. This would have been classified as a snore (except it wouldn’t because I wasn’t watching 5-7khz on accident).

## Conclusions

So that is the basics of detecting snores in real time. Record a tape of the last 5 seconds of audio, analyze it- and if thresholds are surpassed, then we have a “snore”. Fire the shock. but oh, firing the shock and hacking the shock collar- that was a whole other adventure. To be continued…

</div>