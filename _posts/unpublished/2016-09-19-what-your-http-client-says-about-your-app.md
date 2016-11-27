---
layout: post
title: "What your HTTP client says about your (multi-threaded) app"
comments: true
categories: android
published: false
disqus: y
tags: 
 - android
---

I am experimenting with these objc.io style articles (e.g., longer than the average medium article, shorter than a book chapter). Let me know how you guys feel about them.

## Android HTTP client 101

[android-async-http](https://github.com/loopj/android-async-http) is a callback-based HTTP client library for Android. It has been my go-to http client library since I started Android, up until two years ago. It is perfect for beginners, because:

- Asynchronous callbacks are much easier to reason about than the alternatives for making network calls.
- It handles calling back on the original thread. From [the author's website](http://loopj.com/android-async-http/):

> All requests are made outside of your app’s main UI thread, but any callback logic will be executed on the same thread as the callback was created [...]

It might be easier to explain this with a practical example (You can find the demo project under discussion [here](https://github.com/anas-ambri/MosbyBooksSampleApp/tree/okhttp). Let's assume that we have an app that fetches some JSON off the network. When the app is launched, a call is started from the UI to fetch the data from a specific URL, and it eventually propagates back:

<div class="img-center"><img src="/images/NetworkingLibrary/call_propagation.png"/> </div>

A few things to note here:

- The work of creating an [**HttpURLConnection**](https://developer.android.com/reference/java/net/HttpURLConnection.html), connecting to it, retrieving a response, retrying and cancelling said request will be done in the background. This is taken care of by **android-async-http**
- Because the code that initiated the HTTP request was on the main thread, the response will be executing on the main thread too.

### Why is this important?

Starting from Honeycomb, it was no longer possible to make network calls on the main thread (or else, the app would crash with a [NetworkOnMainThreadException](https://developer.android.com/reference/android/os/NetworkOnMainThreadException.html)). The goal of this change was to encourage developers to put long-running operations [off the main thread](http://android-developers.blogspot.ca/2010/12/new-gingerbread-api-strictmode.html). Ultimately, it was this restriction (along with the rise of multi-core processors) that forced Android developers to start looking into ways to do multi-threading better on Android. This has started a long journey, moving from single-threaded apps, to AsyncTask & Intent Services, and finally to RxJava (Some even say [the journey hasn't ended yet](http://www.reactive-streams.org/)).

### Let's talk architecture

Let's say that we used MVP throughout our app. Using **android-async-http**, this is how the request/callback flow will behave across our MVP objects:

<div class="img-center"><img src="/images/NetworkingLibrary/diagram_async.png" class="three-quarters"/> </div>

### One serious limitation

Why is this a problem? For the same reason why the community has been pushing for MVP in Android, or why [Espresso test recorder](https://developer.android.com/studio/test/espresso-test-recorder.html) is the coolest thing since sliced bread: **Testing**.

As it turns out, the only (sensible) way to test this HTTP client is to use [Robolectric](https://gist.github.com/Axxiss/7143760), which isn't [*really*](https://www.reddit.com/r/androiddev/comments/54cjff/working_on_mvp_am_i_doing_it_right/d80xxzn) ideal. The trick is to provide a custom **ExecutorService** that executes synchronously, and [inject it in your code](verybadalloc.com/android/adding-unit-tests-to-MVP-project.html) during the tests.
  
## Beyond a beginner HTTP client

Ultimately, I felt that I have grown past the full-blown solution provided by this library. Substituting it with something less monolithic would allow for greater flexibility and configurability. In reality, I just couldn't resist the hype, and had to switch to [OkHttp](https://github.com/square/okhttp/). The main advantage is the fact that the whole HTTP stack can be tested using a [MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver) (yep, another Square project).

### Switching to OkHttp

While I won't go into details on how to make the switch (if you insist, you can find all the changes in this [diff](https://github.com/anas-ambri/MosbyBooksSampleApp/compare/8136ad378708b04f76383791ed776e5f5ea00067...99981)), I wanna discuss one small implication of removing **android-async-http**. Because OkHttp will run [callbacks on the background thread](http://stackoverflow.com/questions/24246783/okhttp-response-callbacks-on-the-main-thread), we end up with the following behavior:

<div class="img-center"><img src="/images/NetworkingLibrary/diagram_okhttp.png"/></div>

This also happens to be an incomplete solution: running the code, we get the following exception:

```
android.view.ViewRootImpl$CalledFromWrongThreadException: 
Only the original thread that created a view hierarchy can touch its views.
```


## Coming back full-circle
