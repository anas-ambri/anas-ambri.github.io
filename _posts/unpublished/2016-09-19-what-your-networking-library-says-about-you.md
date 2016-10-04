---
layout: post
title: "What your Android networking library says about you"
comments: true
categories: android
published: false
disqus: y
tags: 
 - android
---

[android-async-http](https://github.com/loopj/android-async-http) is a callback-based HTTP client library for Android. It has been my go-to http client library since I started out Android, up until last year. It is perfect for beginners, because:

- Asynchronous callbacks is much easier to reason about than the alternatives, when making network calls.
- It handles calling back on the original thread. From [the author's website](http://loopj.com/android-async-http/):

> All requests are made outside of your app’s main UI thread, but any callback logic will be executed on the same thread as the callback was created using Android’s Handler message passing

It might be easier to explain this in terms of code. Let's assume that we have an app that fetches a list of books off the network. [Here](https://github.com/anas-ambri/FromAsyncHttpToOkHttp) is the sample code for the app.



This means that, if you **initiate** a network call in the same thread as the one that runs your UI code, you are guaranteed of two things:

- The work of creating an [**HttpUrlConnection**](https://developer.android.com/reference/java/net/HttpURLConnection.html), connecting to it, retrieving a response, retrying and cancelling said request will be done in the background.
- The code that you want to execute when a response is returned, will be executing on the main thread.



If you are a visual kind of person, here is a diagram for you:

<div class="img-center"><img src="/images/FromAsyncToOkHttp/diagram_async.png" class="three-quarters"/> </div>
