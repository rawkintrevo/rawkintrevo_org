---
id: 2420
title: 'StarTrekBots.com: Part 1- The Introduction'
date: '2019-09-25T15:34:12+00:00'
author: rawkintrevo
excerpt: 'An introduction and some context setting around startrekbots.com , my new fun toy.  In this post we briefly compare  retrieval  vs generative chatbots, and consider how this method is more akin to improv comedy than traditional story construction.'
layout: post
guid: 'http://rawkintrevo.org/?p=2420'
permalink: /2019/09/25/startrekbots-com-part1-the-intro/
advanced_seo_description:
    - 'The first of a four part blog series on startrekbots.com on various bot forms, and automated improv. '
jabber_published:
    - '1569425654'
timeline_notification:
    - '1569425657'
email_notification:
    - '1569425658'
publicize_twitter_user:
    - rawkintrevo
image: /wp-content/uploads/2019/09/yes-and.png
categories:
    - startrekbots
---

Welcome to blog series where I talk about how I made my latest toy [startrekbots.com](http://www.startrekbots.com). It was a fun little project and I have already pissed away too much of my life watching it babble on (literally) mindlessly.

In this series of three blog posts I’m going to discuss at you (1) high level of the math (with links if you care to dive deeper) (you are here), (2) how I made the three iterations of “bot”, and (3) the technology required for training the Star Trek Bots, as well as for web hosting them. Odds on you will primarily care about one of these facets and the other two will give a passing read at best. Good News Reader! I wrote them all to be high level overviews with lots of links for you to get started on a dig in if you care to. But first I’m going to lay out some context.

# Varieties of Chat Bots.

**Retrieval Bots.** Odds on you have interacted with lots of Retrieval Bots, online and in some cases even on the phone (my bank uses a decision tree bot when I call in). A decision tree bot may have menu buttons or it may try to infer which path on the tree to take by asking you questions and looking for key words in your response. These are handy for common chat bot applications such as support chat bots, Alexa / Google Home, etc.

**Generative Bots.** Generative bots, you likely do not have as much experience with. They are harder to program, and like small children, you never know what they are going to say next. We will delve into the differences in part two, but suffice to say here, *a **generative chat bot** is a bot that “reads” the chat up to the current point and “generates” the next response (as opposed to retrieving it from a repository).*

In practice that means, after an initial few lines of text to “seed” the document, you’re always going to get new content, because the algorithm is “making it up” as it goes. Specifically, this bot takes the last 100 “words” of “script” and generates the next 15 “words”. More on this in a bit.

# Haters Haters Everywhere.

There’s always haters everywhere. And I’ve gotten plenty of hating on my dear little bots that they don’t develop complex plot structures. OK, that’s fair. But let’s take a bigger step back and look at how humans come up with stories. In general we “scaffold” that is we have an idea- maybe a one liner. Then we come up with an outline of what will happen in each act. Then we outline those acts into scenes. Then we fill in some actions and dialogue for each scene (I know I’ve oversimplified all this, but bear with me.). What we *don’t usually* do is tell the entire story on the first pass making it up as we go. That is, UNLESS we are doing improv comedy. I do some improv (at least I take classes because Chicago is the greatest improv city in the world and it would be a waste not to.)

In some long form improv, more specifically in [The Harold](http://improvencyclopedia.org/games/Harold.html), an entire story in three acts with three to five scenes per act is invented on the fly. (I’m also butchering the finer points of a Harold, but whatever, it works here sharp shoot me in the comments).

That said, to all my hater friends, it is probably more appropriate to compare startrekbots.com to something like [The Improvised Star Trek](https://www.theimprovisedstartrek.com) than professionally written story arcs of The Next Generation or Deep Space Nine. I think we can all agree however, that startrekbots.com has far superior writing to whatever interns did the later seasons of Voyager.

But in all seriousness, The Improvised Star Trek is one of my favorite pod casts, and considering you’re already this far in, you might as well start listening to it.

# Emergent Properties

There is ironically a Star Trek episode that deals with this explicitly. In the episode *[Emergence](https://memory-alpha.fandom.com/wiki/Emergence_(episode))*, the Enterprise is trying to express its self through Holodeck characters from various people’s programs on an old-timey train ride to New Vertiform City. It’s a fine episode and I recommend you watch it rather than take my hacky summation. A key point though is that Data succinctly describes emergent properties:

> *Complex systems can sometimes behave in ways that are entirely unpredictable. The Human brain for example, might be described in terms of cellular functions and neurochemical interactions, but that description does not explain Human consciousness, a capacity that far exceeds simple neural functions. Consciousness is an emergent property.*
> 
> *– Lt. Cmdr. Data*

You can play with the chat bots and argue they have a very short attention span, and seem to just sort of bable from one topic to the next.

But here’s the thing- the *have* a short attention span. They reference other characters from *their* universe. For the most part, Data doesn’t use contractions. If you try to do a “cross over” episode, they will quickly start using characters who spanned both episodes, and after a bit, you’ll be in the new series’s universe (Sometimes this even happens by accident). When you get into a battle, you will be in a battle for sometime (though they will do weird things like “lowering sheilds” just as they’re about to be fired upon)..

*This chat bot may be babbly nonsense, but it is **coherent** babbly nonsense.*

I don’t want to give the thing credit where none is due, but considering it learned English from Star Trek, and at the nuts and bolts level, it is nothing more than addition and multiplication of vectors and tensors that was trained with a small amount of calculus, I think all of this is a really amazing emergent property.

# Conclusion

The point of this post was to whet your appetite for things to come in future installments of this series. All of the math will be in one post and it will be intentionally high level and easy to understand. All of the tech will be in another (also high level), as will an entire post on differences between bots, and the various bots I fielded before startrekbots.com.

I think generative chatbots stand to be a really exciting disruptive technology, though probably not for another decade or so (maybe less). Something akin to *[The Diamond Age](https://en.wikipedia.org/wiki/The_Diamond_Age)* maybe? Thanks for reading and be sure to follow @rawkintrevo on twitter for updates on future posts in this series, as well as whatever fun new toy I invent next.

PS: Two last little points I want to make before you run off to play with my toy:

1. The algorithm needs a few rounds of dialogue to “warm up”. Don’t talk to it twice and then get miffed that it’s not doing anything fun.
2. There is a “Let it Ride” button in the top right. Hit that and just let it go for a while (especially good for warming up).