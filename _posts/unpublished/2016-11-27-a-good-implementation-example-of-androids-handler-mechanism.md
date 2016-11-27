---
layout: post
title: "A good implementation example of Android's Handler mechanism"
comments: true
categories: blog
published: false
disqus: y
tags:
 -
---

Finally, this whole post was the consequence of my interest in **android-async-http**, a thread-aware Android HTTP client library. I will include that original research here as well, if you feel you didn't hear me repeat the word Handler enough. It's a good implementation for how Handlers can be used for stuff that needs to happen across threads.

## Epilogue - Practical example: android-async-http

**android-async-http** is an Android HTTP client library that ensures that any callback is called on the same thread as the one originating the HTTP request. While this can have some adverse effects on the [testability of your Android app](http://verybadalloc.com/android/what-your-http-client-says-about-your-app.html), it can sometimes be the perfect solution for a quick demo project, where we only care about quickly getting bytes straight off the network to the UI thread.

Looking into the library's source code, we can see that requests are simply Runnables that get submitted to a [ExecutorService](https://developer.android.com/reference/java/util/concurrent/ExecutorService.html), which acts as a thread pool for all requests.

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

These Runnables will defer their work to [Handlers](https://developer.android.com/reference/android/os/Handler.html), which are smart enough to know which thread to callback to: they use [Looper.myLooper()](https://developer.android.com/reference/android/os/Looper.html#myLooper()) to figure out their original thread.

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
    
    //Inner non-anonymous class because memory leaks
    private static class ResponderHandler extends Handler {
        private final AsyncHttpResponseHandler mResponder;

        ResponderHandler(AsyncHttpResponseHandler mResponder, Looper looper) {
            super(looper);
            this.mResponder = mResponder;
        }

        public void handleMessage(Message msg) {
            mResponder.handleMessage(msg);
        }
    }
}
```

Now, when any **sendXXXXMessage()** method is called on the **AsyncHttpResponseHandler**, it gets automatically processed by **AsyncHttpResponseHandler::handleMessage()** on the original thread in which the request was created.

<div class="img-center"><img src="http://i.imgur.com/YsbKHg1.gif" class="third"/></div>
