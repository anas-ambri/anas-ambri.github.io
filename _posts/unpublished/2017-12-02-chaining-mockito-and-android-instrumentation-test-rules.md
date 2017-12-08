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

If you, like me, have been using the "new" [Mockito Strictness API](/android/the-other-mockitojunitrunners.html), you have probably ran into the problem of chaining the mocking with the rest of your Android test rules. If yes, I got a solution for you. If you have no idea what I am referring to, read ahead.

## The Mockito Strictness API

Since Mockito 2.3, the library ships with a **StrictStubs** mode, where unused stubs cause failed tests, as opposed to simple [hints](https://github.com/mockito/mockito/blob/v2.8.29/src/main/java/org/mockito/quality/MockitoHint.java). This mode is offered in three ways:

- Using the **MockitoJUnitRunner.StrictStubs** runner
- Using the **MockitoJUnit.rule().strictness(Strictness.STRICT_STUBS)**
- Using the **MockitoSession** API.

