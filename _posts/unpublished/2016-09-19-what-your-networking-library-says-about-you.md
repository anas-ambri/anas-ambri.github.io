---
layout: post
title: "What your Android networking library says about your app"
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

It might be easier to explain this with a practical example (You can find the code [here](https://github.com/anas-ambri/MosbyBooksSampleApp/tree/okhttp). Let's assume that we have an app that fetches a list of books off the network. When the app is launched, a call is started from a **BooksListFragment** to fetch the data from a specific URL, and it eventually propagates back to the UI:

<div class="img-center"><img src="/images/NetworkingLibrary/call_propagation.png"/> </div>

A few things to note here:

- The work of creating an [**HttpUrlConnection**](https://developer.android.com/reference/java/net/HttpURLConnection.html), connecting to it, retrieving a response, retrying and cancelling said request will be done in the background. This is taken care of by **android-async-http**
- Because the code that initiated the HTTP request was on the main thread, the response will be executing on the main thread too.

## Why is this important?

Starting from Honeycomb, it was no longer possible to make network calls from the main thread (or else, the app would crash with a [NetworkOnMainThreadException](https://developer.android.com/reference/android/os/NetworkOnMainThreadException.html). The goal of this change was to force developers to put long-running operations off the main thread (other such operations are disk read & writes, or even [custom slow calls](https://developer.android.com/reference/android/os/StrictMode.ThreadPolicy.Builder.html)). Ultimately, it was this restriction (along with multi-core processors) that forced Android developers to start looking into ways for solving multi-threading on Android. This is the reason we ended up moving from single-threaded apps, to AsyncTask & Intent Services, and finally to RxJava. (Who said [Honeycomb was useless?](https://www.reddit.com/r/AndroidMasterRace/comments/41g74s/what_would_you_say_are_the_best_and_worst/))

If you are a visual kind of person, here is a diagram for you:

<div class="img-center"><img src="/images/NetworkingLibrary/diagram_async.png" class="three-quarters"/> </div>
