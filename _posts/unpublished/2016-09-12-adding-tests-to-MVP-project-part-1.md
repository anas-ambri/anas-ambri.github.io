---
layout: post
title: "Adding tests to an MVP project, for fake internet points (part 1)"
comments: true
categories: android
published: false
disqus: y
tags: android testing
---

You know what's worse than creating a demo project about MVP and testing without actual tests? Posting said project to [/r/androiddev](https://www.reddit.com/r/androiddev/comments/526gds/stepbystep_introduction_to_mvp_on_android/). The only way to redeem myself after that mistake was to, not only add some tests to the project, but also write a post about the process. Hopefully, it will help other people out there, and will teach me a lesson never to try to fish for easy fake internet points.

The app under test
------------------

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

public class BooksListPresenter {
    private WeakReference<BooksListView> viewRef;
    
    public void attachView(BooksListView view) {
        viewRef = new WeakReference<BooksListView>(view);
    }
    
    public void detachView() {
        if (viewRef != null) {
            viewRef.clear();
            viewRef = null;
        }
    }
    
    public BooksListView getView() { return viewRef == null ? null : viewRef.get(); }

    public void loadBooks(final boolean pullToRefresh) {
        if (getView() != null) { getView().showLoading(pullToRefresh); }

        DataFetcher.getBooks(new DataCallback<Book[]>() {

            public void onSuccess(Book[] books) {
                if(getView() != null) {
                    getView().setData(books);
                    getView.showContent();
                }
            }
    
            public void onFailure(String reason) {
                Throwable t = new Throwable("Failed to load books: " + reason);
                if(getView() != null) {
                    getView().showError(t, pullToRefresh);
                }
            });
    }
}
```

When looking for dependencies, it is very important to understand how they affect our code. 

The first dependency, on `BooksListView`, is pretty obvious to spot. Because we need to wrap the refence in a `WeakReference`, we have introduced two methods `attachView()/detachView()` that allow us to easily swap it in and out painlessly.

The second dependency on `DataFetcher` is a bit harder to untangle, because we are using a static method. While unit testing a [static method is possible](https://blog.codecentric.de/en/2011/11/testing-and-mocking-of-static-methods-in-java/), it comes at a great expense on Android (due to its [byte-manipulating nature](https://lkrnac.net/blog/2014/01/using-powermock/)). It is almost always cheaper to transform the code to make it more testable (and, consequently, better). This brings us to our first point about testing:

> Don't be afraid to change the architecture of your app's code to make it more testable. (The opposite, generally, doesn't hold true: never change **failing** tests to make them fit your app's code).

The solution, in this case, is to simply transform the static method into an instance call. We inject a `DataFetcher` instance at construction, and just call the same code.

```java
public class BooksListPresenter<V> {

    private WeakReference<V> viewRef;
    private final DataFetcher dataFetcher;

    public BooksListPresenter(DataFetcher dataFetcher) {
        this.dataFetcher = dataFetcher;
    }
    
    //attachView(),detachView() & getView() don't change

    public void loadBooks(final boolean pullToRefresh) {

         //No changes

         dataFetcher.getBooks(new DataCallback<Book[]>() {
             //No changes here
         });
    }
}
```
### Fake it until you make it

Because we want to run our tests on the smallest unit of code possible, we would like to isolate the presenter class as much as possible. One way to do so is by "surrounding" it with [Test Doubles](http://www.martinfowler.com/bliki/TestDouble.html). Like stunt doubles in movies, they will replace the current implementation, and make the presenter believe he is dealing with the original implementation. The easiest kind of test doubles are Fakes (No, I am really not making this stuff up). According to [Martin Fowler](http://martinfowler.com/articles/mocksArentStubs.html):

> Fake objects actually have working implementations, but usually take some shortcut which makes them not suitable for production (an InMemoryTestDatabase is a good example).

This is perfect for the case of our network calls, because we already have an idea of what we want to return for response. In this case, we convert our `DataFetcher` class into an interface, and simply define a `FakeDataFetcher` class that implements it (in the laziest way possible):

```java
public interface DataFetcher {
    void getBooks(final DataCallback<Book[]> callback);
}

public class FakeDataFetcher implements DataFetcher {
    private String reason;
    private Book[] data;

    public FakeDataFetcher(String reason) {
        this.reason = reason;
    }

    public FakeDataFetcher(Book[] data) {
        this.data = data;
    }

    public void getBooks(DataCallback<Book[]> callback) {
        if (reason != null) {
            callback.onFailure(reason);
        } else {
            callback.onSuccess(data);
        }
    }
}
```

Now, when creating a `DataFetcher`, I can just tell it exactly how I want it to behave, and it will oblige. And since you have been great readers so far, I made surer to leave you [a snapshot](https://github.com/anas-ambri/MosbyBooksSampleApp/tree/refactor-data-fetcher) of exactly how the whole project looks like at this point.

### Testing the fake

Yep, I know, it sounds ludicrous, but you still gotta do: the fake implementation is still part of your production code that gets shipped to clients. Unless you can guarantee that it won't be called (e.g., can guarantee that the class is not included in the final package), then you have to write unit tests for it. So, go ahead, create a new folder called `test` under your main `src` folder. And create your `FakeDataFetcherTest` class:

```java
public class FakeDataFetcherTest {

    private static final Book[] data;
    private static final String reason = "Could not fetch books";       //Arrange

    static {
        Book b1 = new Book();
        b1.id = 1; b1.author = "Paula Hawkins"; b1.name = "The Girl on the Train";
        b1.imgUrl = "http://ecx.images-amazon.com/images/I/51-VcOHdoFL."
        + "_SL160_PIsitb-sticker-arrow-dp,TopRight,12,-18_SH30_OU15_SL150_.jpg";
        data = new Book[]{ b1 };                                        //Arrange
    }

    @Test
    public void dataFetcher_SuccessReturnsData() {
        DataFetcher dataFetcher = new FakeDataFetcher(data);            //Arrange

        dataFetcher.getBooks(new DataCallback<Book[]>() {               //Act

            public void onSuccess(Book[] data) {
                assertArrayEquals(data, FakeDataFetcherTest.data);      //Assert
            }

            public void onFailure(String reason) {
                assertTrue("Callback returned unexpected data", true); //Assert
            }
        });
    }

    @Test
    public void dataFetcher_FailureReturnsErrorMessage() {
        DataFetcher dataFetcher = new FakeDataFetcher(reason);          //Arrange

        dataFetcher.getBooks(new DataCallback<Book[]>() {               //Act
            @Override
            public void onSuccess(Book[] data) {
                assertTrue("Callback returned unexpected data", true); //Assert
            }

            @Override
            public void onFailure(String reason) {
                assertEquals(reason, FakeDataFetcherTest.reason);       //Assert
            }
        });
    }
}
```
So, we are testing both the success and the failure cases. When dealing with callbacks with multiple "returns", it is necessary to guard against the callback we don't expect. For example, when we are expecting a successful reponse in the first test, we need to ensure that the `onFailure` callback is never executed, and the best way to do that is to assert it.

Given our file structure, we can right-click > "Run 'FakeDataFetcherTest', or select it among the run configurations.

<div class="img-center"><img class="three-quarters" src="/images/MosbyBooksSampleApp/run_unit_tests_view.png" /></div>

Surely enough, our tests pass, which means, we can finally move on to testing the presenter.

<div class="img-center"><img class="half" src="/images/MosbyBooksSampleApp/unit_tests_results_1.png" /></div>


###  At last

Now that we know that our fake is just as good as the real deal, we can test against it. Our second dependency, however, We create a `BooksListPresenterTest` class:

```java
@RunWith(MockitoJUnitRunner.class)   //(1)
@SmallTest                           //(2)
public class BooksListPresenterTest {
    private static final String ERROR_DATA_FETCH = "Could not find data";
    private static final Book[] data;
    static {
        Book b1 = new Book();
        b1.id = 1; b1.author = "Paula Hawkins"; b1.name = "The Girl on the Train";
        b1.imgUrl = "http://ecx.images-amazon.com/images/I/51-VcOHdoFL."
        + "_SL160_PIsitb-sticker-arrow-dp,TopRight,12,-18_SH30_OU15_SL150_.jpg";
        data = new Book[]{ b1 };                                             //Arrange
    }

    @Mock BooksListView view; //(3)

    @Test
    public void failureDataFetcher_showError() {
        DataFetcher dataFetcher = new FakeDataFetcher(ERROR_DATA_FETCH);     //Arrange
        BooksListPresenter presenter = new BooksListPresenter(dataFetcher);
        presenter.attachView(view);
        boolean pullToRefresh = true;

        presenter.loadBooks(pullToRefresh);                                  //Act

        ArgumentCaptor<Throwable> throwable =       //(4)
            ArgumentCaptor.forClass(Throwable.class);
        ArgumentCaptor<Boolean> flag = 
            ArgumentCaptor.forClass(Boolean.class);
        verify(view).showLoading(pullToRefresh);    //(5)                    //Assert
        verify(view).showError(throwable.capture(), flag.capture());
        assertEquals(ERROR_DATA_FETCH, throwable.getValue().getMessage());
        assertEquals(pullToRefresh, flag.getValue());
    }

    @Test
    public void successDataFetcher_showContent() {
        dataFetcher = new FakeDataFetcher(data);                            //Arrange
        BooksListPresenter presenter = new BooksListPresenter(dataFetcher);
        presenter.attachView(view);
        boolean pullToRefresh = true;

        presenter.loadBooks(pullToRefresh);                                 //Act

        verify(view).showLoading(pullToRefresh);                            //Assert
        ArgumentCaptor<Book[]> books = 
            ArgumentCaptor.forClass(Book[].class);
        verify(view).setData(books.capture());
        verify(view).showContent();
        assertArrayEquals(data, books.getValue());
    }
}
```
This is way too much code for a single shot, so you would have to bear with me here. 

1- The [`MockitoJUnitRunner`](http://site.mockito.org/mockito/docs/current/org/mockito/runners/MockitoJUnitRunner.html): this is the part that runs your tests. We didn't need to specify one in our `FakeDataFetcherTest` class because JUnit uses `BlockJUnit4ClassRunner` by default. However, because we want to useIt is compatible with JUnit 4 [Here](http://www.codeaffine.com/2014/09/03/junit-nutshell-test-runners/) is a fantastic explanation of how JUnit test runners work.


Notes
-----
<sup>1</sup>This is not entirely true. One can argue that the HttpClient still has to rely on `HttpUrlConnection`, which is implemented somewhere in the OS.

For part 2, we discover a bug when selecting different books in landscape mode. We then write a regression test. Thanks to the best QA department in the world, obviously.

Morale of the story:

- Never try to fish for fake internet points 
- Support for tests is unbelievably good in Android Studio now: you have pretty much no reason not to include tests in your projects now (*cough*Googe IO app*cough*)
