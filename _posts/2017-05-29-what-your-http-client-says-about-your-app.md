---
layout: post
title: "What your HTTP client says about you"
comments: true
categories: android
published: true
disqus: y
tags:
 - android
 - okhttp
---

### Foreword

Yes, if you are starting a new Android project today, you would just use OkHttp. 

## Android HTTP client 101

[android-async-http](https://github.com/loopj/android-async-http) is a callback-based HTTP client library for Android. It has been my go-to http client library since I started Android, up until about 3 years ago. It is perfect for beginners, because:

- Asynchronous callbacks are much easier to reason about than the alternatives.
- It handles calling back on the original thread. From [the author's website](http://loopj.com/android-async-http/):

> All requests are made outside of your app’s main UI thread, but any callback logic will be executed on the same thread as the callback was created [...]

It might be easier to explain this with a practical example. You will find [here](https://github.com/anas-ambri/MosbyBooksSampleApp/tree/master) a demo project that uses this library to retrieve a list of books, and display it in a list, using MVP. When the app is launched, a call is started from the UI to fetch the data from a specific URL, and it eventually propagates back:

<div class="img-center"><img src="/images/NetworkingLibrary/call_propagation.png"/> </div>

A few things to note here:

- The work of creating an [**HttpURLConnection**](https://developer.android.com/reference/java/net/HttpURLConnection.html), connecting to it, retrieving a response, retrying, and cancelling said request will be done in the background. This is taken care of by **android-async-http**
- Because the code that initiated the HTTP request was on the main thread, the response will be executing on the main thread too.

### Why is this important?

Starting from Honeycomb, it was no longer possible to make network calls on the main thread (or else, the app would crash with a [NetworkOnMainThreadException](https://developer.android.com/reference/android/os/NetworkOnMainThreadException.html)). The goal of this change was to encourage developers to put long-running operations [off the main thread](http://android-developers.blogspot.ca/2010/12/new-gingerbread-api-strictmode.html). Ultimately, it was this restriction (along with the rise of multi-core processors) that forced Android developers to start looking into ways to do multi-threading better on Android. This has started a long journey, moving from single-threaded apps, to AsyncTask & Intent Services, and finally to RxJava (Some even say [the journey hasn't ended yet](http://www.reactive-streams.org/)).

## There is always a 'but'

Let's say that we used MVP throughout our app. Using **android-async-http**, this is how the request/callback flow will behave across our classes:

<div class="img-center"><img src="/images/NetworkingLibrary/diagram_async.png" class="three-quarters"/> </div>

### One serious limitation

Why is this a problem? For the same reason why the community has been pushing for MVP in Android, or why [Espresso test recorder](https://developer.android.com/studio/test/espresso-test-recorder.html) is the coolest thing since sliced bread: **Testing**.

As it turns out, the only (sensible) way to test this HTTP client is to use [Robolectric](https://gist.github.com/Axxiss/7143760), which isn't [*really*](https://www.reddit.com/r/androiddev/comments/6d0or6/what_are_the_immediate_disadvantages_of_using/dhywu7q) ideal. The trick is to provide a custom **ExecutorService** that executes synchronously, and use it instead of the [default thread pool inside the library](/android/a-good-implementation-example-of-android-handler-mechanism.html).
  
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

What is happening here? Because the callback is executed on the background thread, any code executed inside the callback is also executed on that thread, including our calls to update the UI. This is not good because Android doesn't like that: only the UI thread can touch its UI. Ultimately, this is what we want:

<div class="img-center"><img src="/images/NetworkingLibrary/diagram_okhttp_improved.png"/></div>

Many solutions can be used to resolve this problem. If this was 2012, you would use an eventbus that posts an event [across threads](https://stackoverflow.com/questions/15431768/how-to-send-event-from-service-to-activity-with-otto-event-bus) to inform the UI that it needs to refresh itself. Today, you would just use everyone's favorite ["library for composing asynchronous and event-based programs using observable sequences for the JVM"](https://github.com/ReactiveX/RxJava). For the sake of beating RxJava at its own game, I would like to introduce an even more edgy solution: Handlers.

(For a more detailed explanation of what Handlers are, you can check [this other article by yours truly](http://anasambri.com/android/introduction-handler-android.html). It has gifs and stuff, so you know it's good).

Using Handlers, one can pass a Runnable that will refresh the UI. It turns out, however, that passing an anonymous Runnable class to a Handler is a [very good way to leak your Activity](https://techblog.badoo.com/blog/2014/08/28/android-handler-memory-leaks/), so we wrap the Handler in a [WeakReference](https://github.com/badoo/android-weak-handler), and voila!

### Epilogue

A few added notes:

- No, passing **Runnables** around is not a solution that scales well. Ultimately, if you are to build an app with the best Engineering Practices™ in mind, you have to resort to Rx. But, as they say in the *biz*, demo apps can get away with a lot.
- Another solution to the `CalledFromWrongThreadException`, in the context of MVP, is to use [`@CallOnUiThread`](https://github.com/grandcentrix/ThirtyInch/blob/master/thirtyinch/src/main/java/net/grandcentrix/thirtyinch/callonmainthread/CallOnMainThreadInvocationHandler.java#L76) annotation from ThirtyInch.

Finally, if you've been an Android developer for a little while now, none of what is mentioned in the article should be news to you. But, amidst all the exciting new stuff coming every week, it is sometimes easy to forget the basics. The goal here was to start with a simple problem (choosing a HTTP client library), and try to explore all its implications.

