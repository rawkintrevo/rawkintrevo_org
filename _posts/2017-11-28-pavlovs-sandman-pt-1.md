---
id: 2326
title: 'Pavlov&#8217;s Sandman pt. 1'
date: '2017-11-28T18:51:34+00:00'
author: rawkintrevo
layout: post
guid: 'http://rawkintrevo.org/?p=2326'
permalink: /2017/11/28/pavlovs-sandman-pt-1/
jabber_published:
    - '1511895096'
email_notification:
    - '1511895098'
publicize_twitter_user:
    - rawkintrevo
publicize_google_plus_url:
    - 'https://plus.google.com/107169808969912762220/posts/WHkTNS4Mx4m'
publicize_linkedin_url:
    - 'https://www.linkedin.com/updates?discuss=&scope=94269141&stype=M&topic=6341347702256730112&type=U&a=2wZr'
image: /wp-content/uploads/2017/11/pavlovs-sandman.png
categories:
    - Engineering
    - How-Tos
    - python
---

I snore. I’m perfect in <del datetime="2017-11-28T10:54:22-06:00">many</del> most ways, but this is the one major defect I am aware of. Like any good engineer, upon learning of a defect I set out to correct or at least patch it. The name of this little project stems from [Ivan Pavlov’s experiments with conditioning](https://en.wikipedia.org/wiki/Ivan_Pavlov#Reflex_system_research) as well as [Operation Sandman](http://www.sfgate.com/news/article/Lawyers-think-detainee-sleep-deprived-3276920.php), a CIA program for sleep deprivation torture of detainees at Gitmo.

The naming convention is evident when one looks at the strategy I am taking to correct my unpleasant snoring habit. I have an app on my phone ([Sleep as Android](https://sleep.urbandroid.org)) that tracks my sleep, is a cool alarm, and among other things, tracks my snoring. From this app I know:

1. It is possible to “detect” snoring.
2. I don’t rip logs all night, but in bursts.

I have tried a number of things to correct this throughout the last year including mouth piece, a jaw strap, a shirt with a tennis ball sewn in the back, video recording myself sleeping to see if I can detect a position where snoring occurs, essential oils/other alternative medicines, etc. Failing all of these, I now begrudgingly turn to “Data Science” the form of mysticism reserved for the exceptionally desperate.

The plan of attack on this endeavor is as follows:

1. Create a program that detects loud noises (preprocessing).
2. Differentiate between snoring and other nighttime noises (dog, furnace, coughs, etc).
3. When snoring is detected administer a small shock via a Bluetooth controlled shock collar for dogs which I will be wearing as an arm band.
4. Video record results of me electrocuting myself while trying to sleep and post to YouTube, elevating me to stardom.
5. Possibly train myself to stop snoring.

The title of this project should now be apparent, as I am hoping to “train” myself to not snore, and if I were developing this commercially, I’m fairly confident that any beta-testing I did would (rightly) classify me as a war criminal.

In part one of this series (I promise that all the time and have a bad habit of not following through), I present the code and methods I have developed for detecting loud noises / building my dataset.

In future parts, I hope to do some cool things with respect to signal processing and Bluetooth device hacking with Python.

[GitHub Repo](https://github.com/rawkintrevo/pavlovs-sandman)

### Loud Noises

The first step is to record sounds. The code presented is fairly elegant and easy to follow (for now).

We have a class `AudioHandler` which contains some variables and a few methods.

In addition to the `alsaaudio` handler, we have:

- `rawData` a list for holding caching the audio recorded
- `volume` which will be used to create a csv of timestamp, volume data
- and various thresholds to prevent getting multiple shocks in unison, a warm-up period, volume threshold for recording, etc.

The methods are:

##### `setThreshold`

This action listens to the mic for a number of seconds and attempts to dynamically set the threshold. It’s not great- for my first night of use I ended up doing it manually by observation and laying in bed and making some breathing / fake snoring sounds and seeing where it hit.

##### `dumpData`

This method writes the csv and audio to disk

##### `executeAction`

This is a place holder, and later will be used to call the “shock”

##### `run`

This is where all the fun happens. In short it

1. Attempts to set a baseline threshold considering mic sensativity, background noise, etc.
2. In a `while` loop it then 
    1. Listens to the mic
    2. If the volume is above the threshold: 
        1. Set a `recordingActive` flag to `True` if it wasn’t already
        2. If it wasn’t already, timestamp when this recording started
        3. Determine how long the current recording has been going on.
        4. If it has been going on for some duration, call the `executeAction` method and dump the recording to disk.
    3. If `recordingActive` is true, add the raw audio and volume levels to `rawData` and `volume`

And that’s about it. Again, [look at the code](https://github.com/rawkintrevo/pavlovs-sandman/blob/master/audio.py) for specifics, but all in all pretty straight forward.

Last night I recorded.

### Building a Training Set

As a priest of the mystic art of data science, the first part of any ceremonial ritual is to create a training / testing data set.

This was a very tedious part of my day. I went through the recordings and seperated them into two folders “snore” and “non-snore”. Well, I did this for about 30 minutes, and got approx 80 samples of each. Then I moved the rest into an “unlabled” folder… you know for testing purposes, not because I was super bored. Perhaps if I had an intern, this would have been a more robust set.

Finally I wrote a little python script that will copy the csvs over appropriately to all of the wav files you sorted out into the proper directories.

Stay tuned for part 2, where we’ll do some signal processing to differentiate the snores from the noises!