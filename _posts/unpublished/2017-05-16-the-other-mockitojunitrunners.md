---
layout: post
title: "The other MockitoJUnitRunners"
comments: true
categories: blog
published: false
disqus: y
tags:
 - Mockito
---

The main benefit of using the `@RunWith(MockitoJUnitRunner.class)` annotation is the extra validation that the runner [does for you](http://stackoverflow.com/questions/10806345/runwithmockitojunitrunner-class-vs-mockitoannotations-initmocksthis), for free.

The [javadocs](https://static.javadoc.io/org.mockito/mockito-core/2.7.22/org/mockito/Mockito.html#validateMockitoUsage()) list a bunch of examples that would fail, thanks to this validation:

```java
 //Oops, thenReturn() part is missing:
 when(mock.get());

 //Oops, verified method call is inside verify() where it should be on the outside:
 verify(mock.execute());

 //Oops, missing method to verify:
 verify(mock);
 ```
 
 The additional benefit of using `MockitoJUnitRunner`, then, is the fact that the runner `validateMockitoUsage()` for you, in the `@After` hook, making sure that these errors are reported in the same test as they occur.

 
 ## New runners
 
 It turns out that, since 2.1, Mockito has been shipping with a bunch of extra runners.They are are all part of the [MockitoJUnitRunner](https://github.com/mockito/mockito/blob/v2.8.29/src/main/java/org/mockito/junit/MockitoJUnitRunner.java#L76) class:
 
 - 

 ## Epilogue
 
 Did you know that these runners can be used as rules as well? This is useful in cases where a different runner is required (for example, [`RobotElectricTestRunner`](http://robolectric.org/getting-started/)). It looks like this:
 
 ```java
 @Rule
 public MockitoRule rule = MockitoJUnit.rule();
 ```
 This also comes in silent and strict variants:
 
 ```java
 @Rule
public MockitoRule rule = MockitoJUnit.rule().strictness(Strictness.SILENT);
@Rule
public MockitoRule rule = MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS);
```
Note that the `Scritness.SCRICT_STUBS` is available on 2.3+
