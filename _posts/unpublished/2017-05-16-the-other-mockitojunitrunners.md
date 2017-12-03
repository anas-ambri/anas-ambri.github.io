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

The main benefit of using the `@RunWith(MockitoJUnitRunner.class)` annotation, as opposed to manually calling `MockitoAnnnotations.initMocks()`, is the extra validation that the runner [does for you](http://stackoverflow.com/questions/10806345/runwithmockitojunitrunner-class-vs-mockitoannotations-initmocksthis) for free. This validation runs as an `@After` hook, making sure that the errors are reported in the same test as they occur. The [javadocs](https://static.javadoc.io/org.mockito/mockito-core/2.7.22/org/mockito/Mockito.html#validateMockitoUsage()) list a bunch of examples that would fail, thanks to this validation:

```java
 //Oops, thenReturn() part is missing:
 when(mock.get());

 //Oops, verified method call is inside verify() where it should be on the outside:
 verify(mock.execute());

 //Oops, missing method to verify:
 verify(mock);
 ```
 
## New runner
 
Mockito 2.1 introduced a new runner, capable of one extra level of verification, which has since become the default behavior of the `MockitoJUnitRunner`. This runner, called [`MockitoJUnitRunner.Strict`](https://github.com/mockito/mockito/blob/v2.8.29/src/main/java/org/mockito/junit/MockitoJUnitRunner.java#L120), is able to do two things:

- detect unused stubs
- detect incorrect uses of stubbing

Unused stubs happen when the mock is expecting a certain call to happen, but never receives it. Consider this example:

```java
@RunWith(MockitoJUnitRunner.class)
public class ExampleUnitTest {

    interface Dependency {
	    boolean isChecked();
		void doSomething();
	}
	
	public class Example {
	    public Example(Dependency dep) { 
		    this.dep = dep;
		}
		public void doSomething() {
		    dep.doSomething();
		}
		private final Dependency dep;
	}
	
	@Mock
	Dependency mock;
	
	private Example example;
	
	@Before
	public void setup() {
	    example = new Example(mock);
		doReturn(true).when(mock).isChecked();
	}
	
	@Test
	public void testDoSomethingCalled() {
	    
	}
}

```

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
