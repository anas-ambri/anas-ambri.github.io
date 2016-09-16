---
layout: post
title: "Adding tests to an MVP project, for fake internet points (part 1)"
comments: true
categories: android
published: false
disqus: y
tags: android testing
---

You know what's worse than creating a demo project about MVP and testing without actual tests? Posting said project to [/r/androiddev](https://www.reddit.com/r/androiddev/comments/526gds/stepbystep_introduction_to_mvp_on_android/). The only way to atone for my mistake was to, not only add some tests to the project, but also write a post about the process. Hopefully, it will help other people out there, and will teach me a lesson never to try to fish for easy fake internet points. The process will be divided into 3 parts:

- [Part 1](http://verybadalloc.com/android/2016/09/12/adding-tests-to-MVP-project/): a step-by-step guide into adding unit tests to an existing project
- Part 2: where we introduce Espresso integration tests, and discuss ways to test ImageViews loaded off the network.
- Bonus Part: where we deploy the tests to Firebase Test Lab (because why not).

The app
-------

The app under test was developed to demonstrate how to use mosby, an [MVP library](http://hannesdorfmann.com/mosby/) developed by [Hannes Dorfmann](https://twitter.com/sockeqwe). It emphasizes simplicity, so it tries as few libraries as possible (sorry for over-achievers out there: no Rx, no Retrofit, no Dagger). It displays a list of books (title, author name and an image preview) in a recyclerview, and then, upon selection of an item, show a more detailed view. See for yourself:

<div class="img-center"><img class="half" src="/images/MosbyBooksSampleApp/animation_phone.gif" /></div>

On landscape, it will look like this:

<div class="img-center"><img class="half" src="/images/MosbyBooksSampleApp/tablet_view.png" /></div>

### Applying MVP

<div class="img-center"><img class="half three-quarters" src="/images/MosbyBooksSampleApp/mvp_view.png" /></div>

We apply MVP by introducing two views: `BooksListView` and `BookDetailsView`.

The first view, responsible of displaying the books, will delegate the task to `BooksListPresenter`. The presenter, after instructing the view to set itself in the loading state, delegates the job of fetching the data to the `DataFetcher`, and passes itself for callback. Once a response is received, and the callbacks propagate, the presenter decide whether to show the error or the data it received.

The second view, however, doesn't need to do much. Since we already have all the data we need, there is no task to perform.

So, here are a few things to note from the diagram:

1. My drawing skills are amazing.
2. The views are responsible for creating the presenters. We need the Android lifecycle to know when to do that, thus the need for the views to be implemented by Fragments/Activities/ViewGroups.
3. Once we delegate to the presenter, all the work is done in pure Java code<sup>1</sup>. This is arguably the *only* reason we go through all this hassle: to be able to run all this code on the JVM, including its unit tests. (If this last sentence didn't make sense to you, I suggest you look at this [quick presentation](http://slides.com/anasambri/mvp-android), by yours truly. I'll wait).

Unit testing the presenter
--------------------------

The basic idea behind unit tests can be summarized in: [Arrange-Act-Assert](http://c2.com/cgi/wiki?ArrangeActAssert) (or AAA). First, you arrange the conditions to execute what you want to do. Then, comes the time to act by performing the function we want to test. Finally, assert that the results match what we expect.

In the case of our app, the `BooksListPresenter` depends on two things:

- The `DataFetcher`: this is the component that the presenter uses. This is the part that we arrange.
- The `BooksListView`: this is the view that the presenter has to "act upon". In our AAA cycle, this is the point where we want to assert.

### Pulling out dependencies

It is important, at this point, to take a look at the code. I promise to show as little as needed, but, if you wanna have a more complete view (read, that compiles), [here](https://github.com/anas-ambri/MosbyBooksSampleApp/tree/before-tests) is a snapshot of what we are starting with.

```java

public interface DataCallback<T> {
    void onSuccess(T data);
    void onFailure(String reason);
}

public class BooksListPresenter<V> {

    private WeakReference<V> viewRef;
	
	public void attachView(V view) {
        viewRef = new WeakReference<V>(view);
    }
	
	public void detachView() {
  	    if (viewRef != null) {
            viewRef.clear();
            viewRef = null;
        }
	}

    public void loadBooks(final boolean pullToRefresh) {

         //Set the view in loading state

         DataFetcher.getBooks(new DataCallback<Book[]>() {

             public void onSuccess(Book[] books) {
		 	     //Show data on view
             }

             public void onFailure(String reason) {
			     //Show error on view
             }
         });
    }
}
```
Two things, right off the bat:

- The presenter keeps a weak reference to the view. This is the standard way to handle the fact that the view can disappear at any moment.
- The presenter uses a static method to retrieve the books. While it is fun to try to mock static methods, we really don't have to in this case. So we just replace it with an instance, which gives us:

```java
public class BooksListPresenter<V> {

    private WeakReference<V> viewRef;
	private DataFetcher dataFetcher;

    public BooksListPresenter(DataFetcher dataFetcher) {
        this.dataFetcher = dataFetcher;
    }
	
	//attachView() & detachView() don't change

    public void loadBooks(final boolean pullToRefresh) {

         //Set the view in loading state

         dataFetcher.getBooks(new DataCallback<Book[]>() {
		     //No changes here
         });
    }
}
```

### Fake it until you make it

Because we want to run our tests on the smallest unit of code possible, we would like to isolate the presenter class as much as possible. This means that we need to control both 



Notes
-----
<sup>1</sup>This is not entirely true. One can argue that the HttpClient still has to rely on `HttpUrlConnection`, which is implemented somewhere in the OS.


For part 2, we discover a bug when selecting different books in landscape mode. We then write a regression test. Thanks to the best QA department in the world, obviously.

Morale of the story:

- Never try to fish for fake internet points 
- Support for tests is unbelievably good in Android Studio now: you have pretty much no reason not to include tests in your projects now (*cough*Googe IO app*cough*)
