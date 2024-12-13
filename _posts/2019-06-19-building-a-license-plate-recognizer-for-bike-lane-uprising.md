---
id: 2357
title: 'Building a License Plate Recognizer For Bike Lane Uprising'
date: '2019-06-19T18:46:07+00:00'
author: rawkintrevo
layout: post
guid: 'http://rawkintrevo.org/?p=2357'
permalink: /2019/06/19/building-a-license-plate-recognizer-for-bike-lane-uprising/
jabber_published:
    - '1560969968'
timeline_notification:
    - '1560969970'
email_notification:
    - '1560969971'
publicize_twitter_user:
    - rawkintrevo
image: /wp-content/uploads/2019/06/6191097965_ae13290e1e_b.jpg
categories:
    - Engineering
    - How-Tos
    - openwhisk
---

**UPDATE 6/20/2019:** Christina of Bike Lane Uprising doesn’t work in software sales, and therefor is terrified of “forward-selling”, and wants to make sure its perfectly clear that automatic plate recognition, while in the road map, will not be implemented for some time to come.

I was recently at the [Apache Roadshow Chicago](https://www.apachecon.com/chiroadshow19) where I met the founder of [Bike Lane Uprising](https://www.bikelaneuprising.com), a project whose goal is to make bicycling more safe by reducing the occurrence of bike lane obstruction.

I actually met Christina when I was running for Alderman last fall (which is why I’ve been radio-silent on the blogging for the last 8 months- before you go Google it, I’ll let you know I dropped out of the race). As a biker, I identified with what she was doing. Chicago is one of the bike-y-est cities in North America, but it can still be a very… exciting… adventure commuting via bike. It also seemed like there was probably *something* I could do to help out. I promised after the election was done I’d be in touch.

Of course, I forgot.

Actually, I didn’t forget- I dropped out of the election because I got a book deal with O’Reilly Media to write on Kubeflow AND because I had already committed to producing the Apache Roadshow Chicago. So I “punted” and promised to help out after the Roadshow.

Christina was *still* nice enough to come out and speak at the roadshow, and was very well received. We were talking there, and I, remembering my oath to help out, finally got a hold of her a week or two later.

Her plan, was to do license plate recognition. The current model of Bike Lane Uprising uses user submitted photos, which are manually tagged and entered into a database with things like: License Plate Number, Company, City/State Vehicles, etc. She had found an open source tool called [OpenALPR](https://github.com/openalpr/openalpr) and wondered if BLU could use it somehow.

Now obviously this is going to have to be served somewhere, and if you hadn’t heard, I have fallen deeply in love with [Apache OpenWhisk-incubating](https://openwhisk.apache.org) over the last 18 months. And I’m not saying that with my IBM hat on, I genuinely think it is an amazing and horribly underrated product. Also it’s really cheap to run (note- I am running it as IBM Cloud Functions, which is just a very thin veil over OpenWhisk).

OK so OpenALPR has a Python API. Good news and bad news- good news because this project will take 20 minutes and I’ll be done, bad news because it’s too quick and easy to make a blog post out of. Considering you’re reading the blog post- obviously that didn’t work. If you look at OpenALPR, you’ll see its been over a year since any work has gone on with it. It’s basically a derelict, but a functional one…ish. The Python is broken- some people said they could make the Python work if they built from scratch- I could not. Gonna have to CLI this one.

[As a spoiler- here’s the code.](https://github.com/rawkintrevo/plates-as-a-service)

Well, that’s exciting because I’ve never built a Docker function before (only Python, and some pretty abusive python tricks that use `os.system(...` to build the environment at run time…

For this trick however, we do a multi stage build. This is because the Alpine Linux repos are unstable as hell, and if you want to version lock something you basically have to build from source.

OpenALPR depends on OpenCV and Tesseract. OpenWhisk expects its Docker functions to start off from the `dockerSkelaton` image.\*\* So if you go into the `docker/` directory- I’d like to first direct your attention to `opencv` which uses `openwhisk/dockerSkelaton` for a base image and builds OpenCV. Was kind of a trick. Not horrible. Then, we have the `tesseract` folder which builds an image using `rawkintrevo/opencv-whisk` as a base. Finally, `openalpr/` which builds an image using `rawkintrevo/tesseract-whisk`as a base. Now we have an (extremely overweight bc I was lazy about my liposuction) environment with `openalpr` installed. Great.

Finally, let me direct your attention to where the magic happens. `plate-recog-server-whisk/` has a number of interesting files. First there is an executable called `exec`. A silly little bash file with only one job- to call

python3 /action/openalpr-wrapper.py <span class="pl-s"><span class="pl-pds">“</span><span class="pl-smi">${1}</span><span class="pl-pds">“</span></span>

You see- the dockerSkelaton has a bunch of plumbing, but OpenWhisk expects there to be a file `/action/exec` that it will execute and the last line of stdout from that executable to be a json (which OpenWhisk will return).

So lets look at the code of `<a href="https://github.com/rawkintrevo/plates-as-a-service/blob/master/docker/plate-recog-server-whisk/openalpr-wrapper.py">openalpr-wrapper.p</a>y` elegant in it’s simplicity. This is a program that takes a single command line arg, a json (that’s how OpenWhisk passes in parameters), that json may have two keys, a required image url, and an optional top *n* license plates. `subprocess.call(` calls `alpr` with the image and the top *n* plates, and prints the response (in json from, which is what the `-j` flag is for). And that’s it. So simple, so elegant.

I’m trying to follow this design pattern more and more as I get older- their usually exists some open source package that does what I want, and I just need a little python glue code up top. In this case- I wanted

- License Plate Recognition
- As-a-Service (e.g. done via API call and scalable).

OpenALPR + Apache OpenWhisk-incubating.

I’m hoping to write more now that life has slowed down a bit. Stop back by soon.

\*\* **UPDATE 2:** The Apache OpenWhisk folks wanted me to provide some clarity around this statement. Specifically they said about this sentence, that it:

> isn’t quite right. You can use any image as your base as long as you implement the lifecycle/protocol. What you get from the skeleton is a built in solution. You could have started with our python image or otherwise.

Photo Credit: [Ash Kyd](https://www.flickr.com/photos/ashkyd/)