---
id: 2341
title: 'Pavlov&#8217;s Sandman pt 3: Hacking a Shock Collar and the benefits of Propper Cargo Culting'
date: '2018-05-30T20:42:25+00:00'
author: rawkintrevo
layout: post
guid: 'http://rawkintrevo.org/?p=2341'
permalink: /2018/05/30/pavlovs-sandman-pt-3-hacking-a-shock-collar-and-the-benefits-of-propper-cargo-culting/
jabber_published:
    - '1527713115'
timeline_notification:
    - '1527713117'
email_notification:
    - '1527713117'
publicize_twitter_user:
    - rawkintrevo
publicize_google_plus_url:
    - 'https://plus.google.com/107169808969912762220/posts/YMELsQPGyVt'
publicize_linkedin_url:
    - 'https://www.linkedin.com/updates?discuss=&scope=94269141&stype=M&topic=6407693281618124801&type=U&a=fL_t'
image: /wp-content/uploads/2018/05/cargo-cult-plane_497x375.jpg
categories:
    - Engineering
    - How-Tos
    - Uncategorized
---

In the [last post](https://rawkintrevo.org/2018/05/22/pavlovs-sandman-pt-2-background-and-detecting-snores/) I laid out a super simple algorithm for detecting snores. You might have read that and thought, “dude, this *barely* counts as data science”. I would agree with you. I did that in January, the talk was in May. I figured I’d go hack the [shock collar](https://www.amazon.com/PetSafe-SMART-Bluetooth-Training-Collar/dp/B01K73MUTI/), and then work on fine tuning the algorithm with some more advanced magic. Turns out I had no idea what I was doing with bluetooth hacking in Python, and it was by luck in early March before I was able to make any communication with the shock collars / begin testing.

I had done some device hacking before (see [Cylons](http://rawkintrevo.org/2017/11/07/borg-system-architecture/)). I thought this would be easy…

\_ Author makes 1000-yard stare out of the window at restaurant where he is writing blog\_

… it wasn’t.

The idea was, I would be able to see what messages my phone was sending the device, then I would send the same messages and viola! I would have control. So step 1 is to snoop the communication between the phone and the device.

To do this, open your Android (if you’re using an iPhone, you’ll first need to smash it with a hammer and get a real cell phone), go to settings, open developer options (varies by phone look up a tutorial for your model), click on the “Enable HCI bluetooth snoop log”, and then go into the app to pair with the collar / send commands.

![Screen Shot 2018-05-22 at 2.37.58 PM](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2018/05/screen-shot-2018-05-22-at-2-37-58-pm.png?resize=685%2C247&ssl=1)

Too easy! Next step is to pull the log off your phone. For this step you’ll need Linux (actually you can theoretically do this without Linux, but you will *need* linux for the Python program that controls the audio of the app or significantly refactor it to run on your trash OS)- if you’re not using Linux you’ll need to format your hard drive and [install a grownup’s operating system](https://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-desktop#0), you may also need to install [Android Debugging Bridge](https://www.xda-developers.com/install-adb-windows-macos-linux/). Open the terminal and type in:

\[code lang=text\]  
adb pull /sdcard/btsnoop\_hci.log

\[/code\]

Now here’s a picture.

<figure aria-describedby="caption-attachment-2347" class="wp-caption aligncenter" id="attachment_2347" style="width: 928px">![Screen Shot 2018-05-23 at 3.24.01 PM](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2018/05/screen-shot-2018-05-23-at-3-24-01-pm.png?resize=685%2C217&ssl=1)<figcaption class="wp-caption-text" id="caption-attachment-2347">A screen shot of a screen shot, but you get the idea.</figcaption></figure>Ok, that’s going to pull the snoop file on to your computer. Next step, open up Wireshark or whatever your favorite log analyzer is and read through.

![Screen Shot 2018-05-23 at 3.26.02 PM.png](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2018/05/screen-shot-2018-05-23-at-3-26-02-pm.png?resize=685%2C189&ssl=1)

What we see here is what (I think) was a “buzz” command from my phone to the collar.

Cool- so now we see the commands my phone was sending to my device (shock collar), should be easy to mimic right? Well, no it wasn’t. Not because it was intrinsically difficult, but because I had no idea what I was doing. What happened next in my story was about two months of me (off and on) trying to make contact from my computer to my device. The first problem was there was no way to pair between my device and the computer, some of the linux bluetooth utils would see the device, others would not.

I was stabbing in the dark. Do you remember being a child, and seeing your parents order pizza. And then one day you want to order pizza, and you’ve seen your parents do it plenty of times and seems easy enough, so you walk to the phone press buttons, say “pizza please” and then wait forty five minutes for pizza to show up. You only know if you did it right because pizza either shows up or it doesn’t. That was basically my experience trying to hack this S.O.B.

I was cargo culting, and doing so with more concern as the deadline approached. Finally, the cargo gods smiled on me. I learned about Low Energy Bluetooth devices (which my device is). I found a somewhat obscure python library for dealing with them. But I still wasn’t getting signals through (that is to say, the device was not reacting to commands I sent). From there, I litterally cargo culted backwards. There are a lot of commands the phone sends to the device and I started going backwards through them all. In not much time- SUCCESS! The collar buzzed. It turns out there is a command which I roughly translate to mean “Hello device, I am a controller that will be sending you commands now, please do whatever you hear from me”, and then it does.

![cargo-cult-plane_497x375](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2018/05/cargo-cult-plane_497x375.jpg?resize=497%2C375&ssl=1)

The code that does all of this is so simple and straight forward its hardly worth giving a line by line.

You can find the controller class [here](https://github.com/rawkintrevo/pavlovs-sandman/blob/master/shock_collar/shock_collar.py).