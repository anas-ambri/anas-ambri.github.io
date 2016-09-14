---
layout: post
title: "Adding unit and intests to an old demo project"
comments: true
categories: android
published: false
disqus: y
tags: android testing
---

You know what's worse than creating a demo project about MVP and testing without actual tests? Posting said project to [/r/androiddev](https://www.reddit.com/r/androiddev/comments/526gds/stepbystep_introduction_to_mvp_on_android/). While some people would consider that to be quite ironic, I decided that the best way to difuse the embarassment is to fix the situation. Plus, I haven't had the chance to This post is about the process of introducing unit and integration tests to a simple app from scratch. 

##The app

Morale of the story:

- Never try to fish for fake internet points 
- Support for tests is unbelievably good in Android Studio now: you have pretty much no reason not to include tests in your projects now (*cough*Googe IO app*cough*)
