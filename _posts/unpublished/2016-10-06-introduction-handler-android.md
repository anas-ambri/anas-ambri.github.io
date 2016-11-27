---
layout: post
title: "Introduction to Android's multi-threading mechanism"
comments: true
categories: android
published: true
disqus: y
tags:
 - android-async-http
 - Internals
---

If you ever done any kind of multi-threaded programming on Android, then you probably had to deal with Handlers at some point. Maybe it was when trying to post events across [multiple threads](http://stackoverflow.com/questions/15431768/how-to-send-event-from-service-to-activity-with-otto-event-bus) using [Otto](http://square.github.io/otto/). Maybe it was while trying to return some data from a background thread after making an [OkHttp call](http://stackoverflow.com/questions/24246783/okhttp-response-callbacks-on-the-main-thread). The point is: you never really had to delve into them, since the solution was usually just one Stackoverflow answer away (OK, maybe you did delve into them, I don't really know you).

Well, you are in luck. Today, you will find out what Handlers are all about. And, if you read through this, you will learn a thing or two to show off to your coworkers tomorrow.


## Intro to the Android threading model

[Handlers](https://developer.android.com/reference/android/os/Handler.html) are a mechanism for scheduling tasks (also known as [Runnables](https://developer.android.com/reference/java/lang/Runnable.html)) across threads. While Runnables are part of the Java language, Handlers, [MessageQueues](https://developer.android.com/reference/android/os/MessageQueue.html) and [Loopers](https://developer.android.com/reference/android/os/Looper.html) are part of the **android.os** package, so it must be understood that they are not part of your standard run-of-the-mill JVM.

When an application is started, it is launched into its [own separate process](https://en.wikipedia.org/wiki/Process_(computing)). The semantics are technically equivalent to starting a Java program on your machine (by typing `java MyProgram`). In fact, even your application's main thread is no different than the standard Java thread that executes the **main** method on any Java app.

Don't believe me? Check for yourself. Here is the code inside the [main method](http://androidxref.com/7.0.0_r1/xref/frameworks/base/core/java/android/app/ActivityThread.java#6041), as of Nougat 7.0.0_r1 (it's okay if the code doesn't make much sense):

```java
public static void main(String[] args) {
    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
    SamplingProfilerIntegration.start();
    
    // CloseGuard defaults to true and can be quite spammy.  We
    // disable it here, but selectively enable it later (via
    // StrictMode) on debug builds, but using DropBox, not logs.
    CloseGuard.setEnabled(false);
    
    Environment.initForCurrentUser();
    
    // Set the reporter for event logging in libcore
    EventLogger.setReporter(new EventLoggingReporter());
    
    // Make sure TrustedCertificateStore looks in the right place for CA certificates
    final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
    TrustedCertificateStore.setDefaultUserDirectory(configDir);
    
    Process.setArgV0("<pre-initialized>");
    
    Looper.prepareMainLooper();
    
    ActivityThread thread = new ActivityThread();
    thread.attach(false);
    
    if (sMainThreadHandler == null) {
        sMainThreadHandler = thread.getHandler();
    }
    
    if (false) {
        Looper.myLooper().setMessageLogging(new
        LogPrinter(Log.DEBUG, "ActivityThread"));
    }
    
    // End of event ActivityThreadMain.
    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
    Looper.loop();
    
    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

The point is: when an application is launched, a main thread is created, and the system marks it as [The One](https://en.wikipedia.org/wiki/The_One_(2001_film)) (+1 if you knew that was a Jet Li reference before clicking). Additionally, it prepares a Looper for it.


### What goes around, comes around

Each main thread comes with three objects:

- A MessageQueue, which, as its name suggests, is a queue of Messages. Any action performed by your app is, at its core, a simple Message that was enqueued to the MessageQueue. Don't believe me? Here is the method that pauses an Activity:
- A Handler: Messages are not directly added to the MessageQueue. Instead, a Handler is created to take care of two things: deliver messages to the MessageQueue, and execute them as they come out of the queue.
- A Looper: the main job of the Looper is... to loop, [infinitely](http://androidxref.com/7.0.0_r1/xref/frameworks/base/core/java/android/os/Looper.java#123):

```java

public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }
		//...
        try {
            msg.target.dispatchMessage(msg);
        } finally {
		    //...
        }
		//...
        msg.recycleUnchecked();
    }
}
```
Remember the **main** method from ActivityThread that I showed you earlier? Yeah. All it really does is call `Looper.loop()`

<div class="img-center"><img src="http://i.imgur.com/tZQDRWb.gif" class="third"/></div>


### The Handler

Out of all the classes we've covered, the Handler is probably the one we developers have to deal with the most. It exposes many methods for message queueing, but we can divide them into two groups:

- The **sendMessage** version: enqueues Messages. You get all the different flavors: send at specific time, delayed, and, everyone's favorite: right now. It also allows for specifiying extra data to be used during the processing of the message.
- The **post** version: Enqueues Runnables. Because Runnable is an interface, this will wrap the task into a Message, and defer to **sendMessage**.

Sending a message means adding it to the queue. The message will then wait there patiently, until the Looper gets to it, and then it will forward it to its target Handler.

When it is time to process a message, it is simply passed back to the Handler, by calling its **handleMessage** method. In the case of the ActivityThread, the hanleMessage is a [~300 lines switch statement](http://androidxref.com/7.0.0_r1/xref/frameworks/base/core/java/android/app/ActivityThread.java#1451), that handles anything from stopping a service, to SUICIDE:

```java
switch(msg.what):
    //...
    case SUICIDE:
        Process.killProcess(Process.myPid());
        break;
```

(Yes, this is real, killing an application does require asking [the thread to commit suicide](http://androidxref.com/7.0.0_r1/xref/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java#5851))


### The Handler is everywhere

It is important to understand that all work done by the app has to go through this Handler mechanism. [**runOnUiThread**](http://androidxref.com/7.0.0_r1/xref/frameworks/base/core/java/android/app/Activity.java#5845) uses it, [RxAndroid](https://github.com/ReactiveX/RxAndroid/blob/1.x/rxandroid/src/main/java/rx/android/schedulers/LooperScheduler.java) uses it<sup>1</sup>. The whole world knows about it, and, now, you do too!


### Legal

Just kidding, this blog is the last place where you will find legal stuff. However, I wanna include a few additional points, for those of you who made it this far:

- Only the main thread comes equipped with the Looper. Other "normal" threads have to either do it themselves, or use [HandlerThread](http://androidxref.com/7.0.0_r1/xref/frameworks/base/core/java/android/os/HandlerThread.java), which is your out-of-the-box implementation of a Thread with a Handler (and a Looper, and a MessageQueue).
- When trying to forward a Message to the main thread, presumably from outside said thread, one easy way to do so when `runOnUiThread` is not possible, is to instantiate a Handler, and pass the message to it:

```java
Handler handler = new Handler(Looper.getMainLooper());
handler.post(new Runnable() {
    //...
});
```

All these handlers will reference the same Looper, so you are writing into the main thread's MessageQueue.

## References

[A good rundown of the Handler mechanism on Android](http://codetheory.in/android-handlers-runnables-loopers-messagequeue-handlerthread/)
[Source code of ActivityThread](http://androidxref.com/7.0.0_r1/xref/frameworks/base/core/java/android/app/ActivityThread.java)

## Footnotes
<sup>1</sup>: Though they are [phasing out its use](https://github.com/ReactiveX/RxAndroid/commit/0bb9b4e63f5560ca238233f1c4d692d9bc8b5ba0)
