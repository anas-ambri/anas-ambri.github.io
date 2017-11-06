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

It is often argued that the main benefit of using the `@RunWith(MockitoJUnitRunner.class)` annotation is the extra validation that the runner [does for you](http://stackoverflow.com/questions/10806345/runwithmockitojunitrunner-class-vs-mockitoannotations-initmocksthis) for free. In addition to calling `MockitoAnnotations.initMocks()` for you at the beginning of each test, it also does some validation at the end of each test. Let's take a look at an example:

```java

@RunWith(MockitoJUnitRunner.class)
public class ExampleUnitTest {

    interface Dependency {
	    boolean isChecked();
		void doSomething();
	}
	
	public class Example {
	    public Example(Dependency dep) {this.dep = dep;}
	    public void doSomething() {
		    dep.doSomething();
		}
	}
    
	@Mock
	Dependency mock;
	
	@Test
	public void testExample() {
	    Example example = new Example(mock);
		example.doSomething();
		verify(mock).hasBeenCalled();
	}
}
```

In this example, the class under test `Example`, uses an instance of `Dependency` to delegate its work. We would like to test that, when we call `doSomething()` on an `Example` instance, it forwards said call to its dependency. This is simply done by verifying that, after acting on our class under test, the mock has also been called.

The equivalent, if one chooses not to use the runner, is:

```java

@RunWith(JUnit4.class)
public class ExampleUnitTest {

    interface Dependency {
    	//...
    }
	
	public class Example {
	    //...
	}

    @Mock
    ExampleMock mock;
	
	@Before
	public void setup() {
	    MockitoAnnotations.initMocks();
	}
	
	@Test
	public void testExample() {
	    //...
	}
```

Basically, we saved 3 lines of code. In reality, since Mockito 2, the runner does one extra thing. Let's look at an example when we do some [stubbing](https://martinfowler.com/articles/mocksArentStubs.html):


```java
@RunWith(MockitoJUnitRunner.class)
public class ExampleUnitTest {    
    
	@Mock
	ExampleMock mock;
	
	@Before
	public void setup() {
	    doReturn(true).when(mock).isChecked();
	}
	
	@Test
	public void testExample() {
	    Example example = new Example(mock);
		example.doSomething();
		verify(mock).hasBeenCalled();
	}
}
```


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
 
 Mockito 2.1 introduced two runners to be used with a bunch of extra runners.They are are all part of the [MockitoJUnitRunner](https://github.com/mockito/mockito/blob/v2.8.29/src/main/java/org/mockito/junit/MockitoJUnitRunner.java#L76) class:
 
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
