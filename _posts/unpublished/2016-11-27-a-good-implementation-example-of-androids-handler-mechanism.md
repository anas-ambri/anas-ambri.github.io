---
layout: post
title: "A good implementation example of Android's Handler mechanism"
comments: true
categories: blog
published: false
disqus: y
tags:
 - Android
 - Handler
---

The [previous post](/android/introduction-handler-android.html) was the result of my interest in **android-async-http**, a thread-aware Android HTTP client library. I will include that original research here as well, if you feel you didn't hear me repeat the word Handler enough. It's a good introduction to how Handlers can be used for stuff that needs to communicate across threads.

## Epilogue - Practical example: android-async-http

**android-async-http** is an Android HTTP client library that ensures that any callback is called on the same thread as the one originating the HTTP request. While this can have some adverse effects on the testability of your app (more on this in a bit), it can sometimes be the perfect solution for a quick demo project, where we only care about quickly getting bytes straight off the network to the UI thread.

Looking into the library's source code, we can see that requests are simply **Runnables**. A Runnable represents something that can be run. Here is what an **AsyncHttpRequest** looks like:

```java
public class AsyncHttpRequest implements Runnable {
    private ResponseHandlerInterface responseHandler;

    @Override
    public void run() {
        //...
        responseHandler.sendStartMessage();
        //...
        //Internally, this method calls sendResponseMessage()
        makeRequestWithRetries();
        //...
        responseHandler.sendFinishMessage();
        //...
    }
}

```

This Runnable is special because it has a reference to an (Android) Handler. Not only can it do the network stuff operations in the background (every Runnable executes in its own separate thread), it is also able to inform the **ResponseHandler** of any progress (start & finish messages, for example).

When making a request, this Runnable gets submitted to a [ExecutorService](https://developer.android.com/reference/java/util/concurrent/ExecutorService.html), which acts as a thread pool for all requests. This is, for example, the case for a `get` call:


```java
public class AsyncHttpClient {
    private ExecutorService threadPool;
    
    public RequestHandle get(/*...*/) {
        return sendRequest(/*...*/);
    }

    protected RequestHandle sendRequest(/*...*/) {
        //...
        AsyncHttpRequest request = newAsyncHttpRequest(/*...*/);
        threadPool.submit(request);
        //...
    }
}
```

What does the **ResponseHandler** look like? It is a class that gets initialized with a Looper. This way, it is able to communicate with the thread where it was created. This way, if a request object is created in the UI thread, the `this.looper` field gets the `Looper.getMainLooper()`:

```java

public abstract class AsyncHttpResponseHandler implements ResponseHandlerInterface {
    private Looper looper;
    private Handler handler;

    public AsyncHttpResponseHandler(Looper looper) {
        this.looper = looper == null ? Looper.myLooper() : looper;  <=== Right here
        //...
        handler = new ResponderHandler(this, looper);
    }
	
    //This is the method that processes all data returned from the request
    protected void handleMessage(Message message) {
        //...
        switch(message.what) {
            case SUCCESS_MESSAGE:
            //...
            case FAILURE_MESSAGE:
            //...
        }
    }
}
```

Now, when any **sendXXXXMessage()** method is called on the **AsyncHttpResponseHandler**, it gets automatically processed by **AsyncHttpResponseHandler::handleMessage()** on the original thread in which the request was created.

<div class="img-center"><img src="http://i.imgur.com/YsbKHg1.gif" class="third"/></div>
