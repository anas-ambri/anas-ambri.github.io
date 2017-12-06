---
layout: post
title: "The other MockitoJUnitRunners"
comments: true
categories: android
published: true
disqus: y
tags:
 - Mockito
 - testing
---

The main benefit of using the `@RunWith(MockitoJUnitRunner.class)` annotation, as opposed to manually calling `MockitoAnnnotations.initMocks()`, is the extra validation that the runner [does for you](http://stackoverflow.com/questions/10806345/runwithmockitojunitrunner-class-vs-mockitoannotations-initmocksthis) for free. This validation runs as an **@After** hook, making sure that the errors are reported in the same test as they occur. The [javadocs](https://static.javadoc.io/org.mockito/mockito-core/2.7.22/org/mockito/Mockito.html#validateMockitoUsage()) list a bunch of examples that would fail, thanks to this validation:

```java
 //Oops, thenReturn() part is missing:
 when(mock.get());

 //Oops, verified method call is inside verify() where it should be on the outside:
 verify(mock.execute());

 //Oops, missing method to verify:
 verify(mock);
 ```
 
## New runner
 
Mockito 2.1 introduced a new runner, capable of one extra level of verification, which has since become the default behavior of the **MockitoJUnitRunner**. This runner, called [**MockitoJUnitRunner.Strict**](https://github.com/mockito/mockito/blob/v2.8.29/src/main/java/org/mockito/junit/MockitoJUnitRunner.java#L120), is able to do two things:

- detect unused stubs
- detect incorrect uses of stubbing

Unused stubs happen when the mock is expecting a certain call to happen, but never receives it. Consider this example:

```java
//Enables the strict runner
@RunWith(MockitoJUnitRunner.Strict.class)
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
        doReturn(true).when(mock).isChecked(); //Unused stubbing
	}
	
	@Test
	public void testDoSomethingCalled() {
	    example.doSomething();
        verify(example).doSomething();
	}
}

```

While this test passes, you will get a **MockitoHint** stating that the stubbing is unused, going as far as marking exactly where the unused stubbing is created.

### Multiple flavours

This runner can be used in different ways:
- As a runner, as shown above. (Actually, since this is the default behavior, putting **@RunWith(MockitoJUnitRunner.class)** suffices
- As a rule. In the case where we can't use the MockitoRunner inside the **@RunWith** annotation (as when using `AndroidJUnitRunner`), one can just attach a **@Rule** to the test, like so: `@Rule public MockitoRule rule = MockitoJUnit.rule();`
- As a configuration for the mocking session. This would almost never be used (first two ways are usually enough), so I won't show an example of it here.

### StrictStubs

Since Mockito 2.3, a new level of strictness has been added. This `StrictStubs` level will actually fail the tests if an unused stubbing is detected. This will force you to clean up your code, which is why the Mockito team is planning to make it default in Mockito 3. As before, this can be used as:

- A runner, using **@RunWith(MockitoJUnitRunner.StrictStubs.class)**
- A rule, using `@Rule public MockitoRule rule = MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS);`
