---
layout: post
title: "Adding tests to an MVP project, for fake internet points"
comments: true
categories: android
published: false
disqus: y
tags: android testing
---

You know what's worse than creating a demo project about MVP and testing without actual tests? Posting said project to [/r/androiddev](https://www.reddit.com/r/androiddev/comments/526gds/stepbystep_introduction_to_mvp_on_android/). The only way to atone for my blunder was to, not only add some tests to the project, but also write a post about the process. Hopefully, it will help other people out there, and will teach me a lesson never to try to fish for cheap fake internet points. Plus, this would give me a chance to try out the Firebase test lab, so it's really a win-win for everybody here. The post is divided into 3 parts:

- [Part 1](http://verybadalloc.com/android/2016/09/12/adding-tests-to-MVP-project/): a step-by-step guide into adding unit and integration tests to an existing project
- Part 2: where we improve on our integration tests suite, and discuss ways to test ImageViews
- Part 3: where we explore some basic capabilities of Firebase test lab (including taking screenshots)

##The app

The app under test was developed to demonstrate how to use mosby, an [MVP library](http://hannesdorfmann.com/mosby/) developed by [Hannes Dorfmann](https://twitter.com/sockeqwe). The emphasis was on simplicity, so it tries as few libraries as possible (sorry for over-achievers out there: no Rx, no Retrofit, no Dagger). It displays a list of books (title, author name and an image preview) in a recyclerview, and then, upon selection of an item, show a more detailed view. See for yourself:

<img src="/images/master_view_MosbyBooksSampleDemo.PNG" />

<img src="/images/detail_view_MosbyBooksSampleDemo.PNG" />

Because we are using t

The data is downloaded off-the-network, without any caching mechanism. Here is the flow of data from the state 

Morale of the story:

- Never try to fish for fake internet points 
- Support for tests is unbelievably good in Android Studio now: you have pretty much no reason not to include tests in your projects now (*cough*Googe IO app*cough*)
