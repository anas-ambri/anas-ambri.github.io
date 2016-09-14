---
layout: post
title: Building the perfect Android app
comments: true
categories: android
published: false
disqus: y
tags: Android, libraries
---

-->Forget the comparison of stacks. Google architecture samples is doing exactly that. It would be a lot more interesting to make one app, and show to replace each component one at a time.
Basically, you are the quintessential FOMO Android developer, building the quintessential mobile app: email

I have been wanting to do a comparison of Android stacks for a while now. Android development is witnessing an explosion of architectural techniques, but there has not been any effort put in surveying, reviewing and analyzing all these techniques. The Android team at Google has been pretty [adamant](https://plus.google.com/+DianneHackborn/posts/FXCCYxepsDU) that it doesn't [promote one way over another](https://github.com/googlesamples/android-architecture). And while it is great that there are so many new libraries and tools introduced on a weekly basis, there doesn't seem to be as much work put into combining them, and trying to see which ones stick (at least, not that I am aware of). 

It seems necessary to do a reset of where we are, especially as we seem to start converging towards some solutions (MVP for easy testing, reactive programming, etc.). This will have the effect of avoiding reinventing the wheel (the whole a-new-JS-framework-every-week problem), and help consolidate the efforts in the problems that have not been solved yet. Additionally, it will help us avoid falling back into some convoluted solutions (Mortar + Flow, I am looking at you), and will educate newcomers to the platform to have a good starting point, instead of some arguable Google suggestions.

The best way to compare stacks is to implement them inside an app. 

Now, why should you trust me to do this review

the first in a series that will try to demonstrate different Android stacks that seem to be used within the community. It will go from the most simple (Fragments), to the most convoluted (Mortar & Flow), to the most master-race (RxJava). I will also try to include any esoteric ones, so don't hesitate to suggest yours.

The point of these posts will be to list the advantages and disadvantages of each stack. This is not meant as a way to find the best stack&trade; out there, nor it is to critique them. The idea is to understand the compromises involved in each decision, according to a defined set of requirements.

And while we are at it, here are those requirements:

- All 

It must be noted that I have absolutely no experience in any of 
