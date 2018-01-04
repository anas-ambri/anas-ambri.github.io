---
layout: post
title: "Chaining Mockito and Android instrumentation test rules"
comments: true
categories: android
published: false
disqus: y
tags:
 - testing
 - Android
 - Mockito
---

If you, like me, have been using the "new" [Mockito Strictness API](/android/the-other-mockitojunitrunners.html), you have probably ran into the problem of chaining the mocking with the rest of your Android instrumentation test rules. If yes, I got a solution for you. If you have no idea what I am referring to, read ahead.

## Android instrumentation tests

Unless you haven't done any Android development since 2013 (in which case, what on earth is bringing you back?), you probably heard that one can use the [Instrumentation API](https://developer.android.com/reference/android/app/Instrumentation.html) to interact with an app process and test its components as a *greybox*. Such an instrumentation test could, for example, look like this:

```
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {

    @Rule
    public ActivityTestRule<MainActivity> activityRule = 
	    new ActivityTestRule<>(MainActivity.class, false, false);

    @Before
    public void setup() {
        activityRule.launchActivity(null);
    }

    @Test
    public void testTextViewInActivity() {
        onView(withId(R.id.userName))
		    .check(matches(withText(containsString("john.doe"))));
    }
}
```

This test starts up a **MainActivity**, and verifies that a certain View, with id `textView`, is showing a certain string. As it happens, this string is retrieved from network after an HTTP call. Here is, for example, what said MainActivity would look like [1]:

```
public class MainActivity extends AppCompatActivity implements DataFetcher {

    private HttpClient client;

	@Override
	public void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		//...
		httpClient.getData(this);
	}

	@Override
	public void onData(@NonNull Data data) {
	    //...
	}
}
```

The problem with this approach is that the entire app is loaded, possibly making unnecessarily long operations for which results could just be mocked (network requests, yes, but also database reads, etc.). Instead of doing all that work (making our tests slower, relying on resources that might not be there), one could simply stub whatever behavior that depends on I/O.

## Mocks in instrumentation tests

Let's see how we can modify our previous test to mock the HTTP client used by **MainActivity**. We will do that by injecting the HttpClient in the **Application** object, and have **MainActivity** use it (kids, do not do this at home.Use [Dagger instead](https://engineering.circle.com/instrumentation-testing-with-dagger-mockito-and-espresso-f07b5f62a85b))

```
public class MyApplication extends Application {
    public static HttpClient httpClient;
	
	@Override
    public void onCreate() {
	    super.onCreate();
		//...
	    httpClient = new HttpClient();
	}
}

public class MainActivity extends AppCompatActivity implements DataFetcher {

    //...

	@Override
	public void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		httpClient = MyApplication.httpClient;
		//...
		httpClient.getData(this);
	}

}
```

(I hope I don't have to explain why you shouldn't do this in your production code. Ever.)


```
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {

    @Mock
	HttpClient httpClient;                              //(1)
	
	@Rule
	MockitoRule rule =                                  //(2)
	    MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS);
	
    @Rule
    public ActivityTestRule<MainActivity> activityRule = 
	    new ActivityTestRule<>(MainActivity.class, false, false) {
		    @Override
            protected void beforeActivityLaunched() {   //(3)
			     Context ctxt = Instrumentation.getTargetContext();
				 MyApplication app = (MyApplication) ctxt.getApplicationContext();
				 app.httpClient = httpClient;
			}
		}

    @Before
    public void setup() {
		doAnswer(new Answer() {                          //(4)
		    @Override
            public Object answer(InvocationOnMock invocation) throws Throwable {
			     DataFetcher fetcher = invocation.getArgument(0);
			     fetcher.onData(new Data("john.doe"));	 
			     return null;
            }
		}).when(httpClient).getData(any(DataFetcher.class));
        activityRule.launchActivity(null);
    }

    @Test
    public void testTextViewInActivity() {
        onView(withId(R.id.userName))
		    .check(matches(withText(containsString("john.doe"))));
    }
}
```

There are exactly 4 new things in this snippet, so let's go over them:

1- This is just the standard **@Mock** annotation, which can be used instead of calling the **Mockito.mock()** method manually for each mock.
2- This is the **MockitoRule** that will initialize our mocks. This is equivalent to using **@RunWith(MockitoJUnitRunner.class)**, but, since we already have to use the **AndroidJUnitRunner**, this rule will allow us to do the same thing.
3- This is where *some* of the magic happens. At the moment when [**beforeActivityLaunched**](https://developer.android.com/reference/android/support/test/rule/ActivityTestRule.html#beforeActivityLaunched()) is called, **MyApplication** is created, and is accessible through the [InstrumentationRegistry's target context](https://developer.android.com/reference/android/support/test/InstrumentationRegistry.html). However, **MainActivity**'s **onCreate** isn't called yet, and won't be until **launcActivity** is called.
4- We configure our mock, so that it returns a certain stub when **getData** is called. An alternative way of doing this is to use [**ArgumentCaptors**](https://fernandocejas.com/2014/04/08/unit-testing-asynchronous-methods-with-mockito/)

That might seem like a lot of work, but, the logic of the **ActivityTestRule** can be put in a subclass. Actually, if you are using Dagger, this [library from Fabio Collini](https://medium.com/@fabioCollini/android-testing-using-dagger-2-mockito-and-a-custom-junit-rule-c8487ed01b56) will do a lot of the setup for you.

So, we go ahead, run the test, go get a cup of coffee, answer emails, play a game of foosball, solve world peace (this is an instrumentation test, after all), and... surprise, it fails! More specifically, it throws an NPE here:

```
	@Override
	public void onCreate(Bundle savedInstanceState) {
	    super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		//...
		httpClient.getData(this);    //<---- NullPointerException
	}
```

The only way this could've happened is if our mock hadn't been initialized yet. Since the mock is initialized through the **@Rule** annotation, it is supposed to go through every single **@Mock**-annotated field, and [put a mock on it](https://www.youtube.com/watch?v=4m1EFMoRFvY).  Looking at the documentation for [**@Mock**](https://static.javadoc.io/org.mockito/mockito-core/2.13.0/org/mockito/Mock.html), one can see that the **MockitoAnnotations.initMocks(this)** won't be called until JUnit's **@Before**, which is way too late[2] (Actually, that is not true. The order in which the code is executed is undefined for reasons that will become clear in a second.)

## A Primer on JUnit Rules

The best explanation of JUnit Rules can actually be found in the doc (I know, I was surprised too!):

> A TestRule is an alteration in how a test method, or set of test methods, is run and reported. A TestRule may add additional checks that cause a test that would otherwise fail to pass, or it may perform necessary setup or cleanup for tests, or it may observe test execution to report it elsewhere. TestRules can do everything that could be done previously with methods annotated with Before, After, BeforeClass, or AfterClass, but they are more powerful, and more easily shared between projects and classes.

One should think of **Rule**s then as composition winning over inheritance when it comes to sharing test setup code. A good example of this is the [**@UiThreadTest**](https://developer.android.com/reference/android/support/test/annotation/UiThreadTest.html) rule, which ensures that the test code is run on the main thread (all so important for that button click). If you ever wondered [how it is implemented](https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/rules/src/main/java/android/support/test/internal/statement/UiThreadStatement.java?autodive=0%2F%2F%2F#41), it just runs the test inside [**runOnMainSync**](https://developer.android.com/reference/android/app/Instrumentation.html#runOnMainSync(java.lang.Runnable)).

## Back to your regularly scheduled programming

The only way for our mock to be **null** is if the **ActivityTestRule** 

#### Footnotes

[1]: Yes, the MainActivity is potentially [leaked to the Network worker thread](https://android.jlelse.eu/memory-leak-patterns-in-android-4741a7fcb570)
