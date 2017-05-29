---
layout: post
title: "A good implementation example of Android's Handler mechanism"
comments: true
categories: android
published: true
disqus: y
tags:
 - Android
 - Handler
---

The [previous post](/android/introduction-handler-android.html) was the result of my interest in **android-async-http**, a thread-aware Android HTTP client library. I will include that original research work here as well, if you feel you didn't hear me repeat the word Handler enough. It's a good introduction to how Handlers can be used for stuff that needs to communicate across threads.

**android-async-http** is an Android HTTP client library that ensures that any callback is called on the same thread as the one originating the HTTP request. While this can have some adverse effects on the testability of your app, it can sometimes be the perfect solution for a quick demo project, where we only care about quickly getting bytes straight off the network to the UI thread.

Looking into the library's source code, we can see that requests are just **Runnables**:

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

This Runnable is special because it has a reference to an (Android) Handler, through which it can inform the **ResponseHandler** of any progress (start & finish messages, for example).

When making a request, this Runnable gets submitted to a [ExecutorService](https://developer.android.com/reference/java/util/concurrent/ExecutorService.html), which acts as a thread pool for all requests. This pool is able to create new threads as needed, and reuse existing ones. Here is how a `get` call is handled:


```java
public class AsyncHttpClient {
    private ExecutorService threadPool; // = Executors.newCachedThreadPool();
    
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

Remember that **ResponseHandlerInterface** we saw in the first snippet? Here it is:

```java

public abstract class AsyncHttpResponseHandler implements ResponseHandlerInterface {
    private Looper looper;
    private Handler handler;

    public AsyncHttpResponseHandler(Looper looper) {
        this.looper = looper == null ? Looper.myLooper() : looper;  // (1)
        //...
        handler = new ResponderHandler(this, looper);
    }
	
    //This is the method that processes all data returned from the request
    protected void handleMessage(Message message) {      // (2)
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

Two things to note here:

- This class gets initialized with the **Looper** of the thread where it is created. `Looper.myLooper()` will return the main thread's looper, if the instance is constructed on the UI thread, for example.
- When any **sendXXXXMessage()** method is called on the **AsyncHttpResponseHandler**, it gets automatically processed by **AsyncHttpResponseHandler::handleMessage()** on the original thread in which the request was created, because, Loopers!

<div class="img-center"><img src="http://i.imgur.com/YsbKHg1.gif" class="third"/></div>
