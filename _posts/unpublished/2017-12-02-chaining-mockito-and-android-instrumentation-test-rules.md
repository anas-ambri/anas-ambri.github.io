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



## The Mockito Strictness API

Since Mockito 2.3, the library ships with a **StrictStubs** mode, where unused stubs cause failed tests, as opposed to simple [hints](https://github.com/mockito/mockito/blob/v2.8.29/src/main/java/org/mockito/quality/MockitoHint.java). This mode can be achieved:

- Using the **MockitoJUnitRunner.StrictStubs** runner
- Using the **MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS)**

Because we are tied to the **AndroidJUnitRunner**, the easiest way to enable the **StrictStubs** mode is to use the rule. For example, let's say we write this test that checks that a certain textView on the **MainActivity** is displayed. Our **MainActivity** gets a mock presenter injected into it before being launched. This way, we can :

```
@RunWith(AndroidJUnit4.class)
public class MainActivityTest {

    @Mock
	

    @Rule
    public ActivityTestRule<MainActivity> activityRule = new ActivityTestRule<>(MainActivity.class, false, false);

    @Before
    public void setup() {
        activityRule.launchActivity(null);
    }

    @Test
    public void testTextViewInActivity() {
        onView(withId(R.id.textView)).check(matches(isDisplayed(());
    }
}
```


