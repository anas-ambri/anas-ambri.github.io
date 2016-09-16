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

<div class="img-center"><img class="third side-by-side center" src="/images/MosbyBooksSampleApp/master_view.png" /> <img class="third side-by-side" src="/images/MosbyBooksSampleApp/detail_view.png" /></div>

On tablet, it will look like this:

<div class="img-center"><img class="half" src="/images/MosbyBooksSampleApp/tablet_view.png" /></div>

### Applying MVP

<div class="img-center"><img class="half three-quarters" src="/images/MosbyBooksSampleApp/mvp_view.png" /></div>

We apply MVP by introducing two views: `BooksListView` and `BookDetailsView`.

The first view, responsible of displaying the books, will delegate the task to `BooksListPresenter`. The presenter, after instructing the view to set itself in the loading state, delegates the job of fetching the data to the `DataFetcher`, and passes itself for callback. Once a response is received, and the callbacks propagate, the presenter decide whether to show the error or the data it received.

The second view, however, doesn't need to do much. Since we already have all the data we need, there is no task to perform.

(If you need a refresher on MVP on Android, there is this [cool presentation](/presentations/mvp-on-android.html), by yours truly).

So, here are a few things to note in this diagram:

1. My drawing skills are amazing.
2. None of the components we described so far rely on Android<sup>1</sup>. This is arguably the *only* reason we go through all this hassle: to be able to run all this code on the JVM, including its unit tests. 

Unit testing the presenter
--------------------------



Notes
-----
<sup>1</sup>: This is not entirely true. One can argue that the HttpClient still has to rely on `HttpUrlConnection`, which is implemented somewhere in the OS.


For part 2, we discover a bug when selecting different books in landscape mode. We then write a regression test. Thanks to the best QA department in the world, obviously.

Morale of the story:

- Never try to fish for fake internet points 
- Support for tests is unbelievably good in Android Studio now: you have pretty much no reason not to include tests in your projects now (*cough*Googe IO app*cough*)
