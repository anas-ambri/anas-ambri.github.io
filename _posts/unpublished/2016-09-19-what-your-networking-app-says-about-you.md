---
layout: post
title: "What your Android networking library says about you"
comments: true
categories: android
published: false
disqus: y
tags: okhttp android
---

[android-async-http](https://github.com/loopj/android-async-http) is a callback-based HTTP client library for Android. It has been my go-to http client library since I started out Android, up until last year. It is perfect for beginners, because:

- Asynchronous callbacks are the simplest form for making *functional* network calls, without relying on complicated concepts
- It handles calling back on the main thread for you. From [the author's website](http://loopj.com/android-async-http/):

> All requests are made outside of your app’s main UI thread, but any callback logic will be executed on the same thread as the callback was created using Android’s Handler message passing

This means that, if you initiate a network call in the same thread as the one that runs your UI code, you are guaranteed of two things:

- The work of creating an [**HttpUrlConnection**](https://developer.android.com/reference/java/net/HttpURLConnection.html), connecting to it, retrieving a response will be done in the background
- The 
