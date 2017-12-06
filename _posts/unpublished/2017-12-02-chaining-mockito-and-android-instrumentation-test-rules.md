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

If you, like me, have been using the "new" [Mockito Strictness API](https://github.com/mockito/mockito/issues/769), you have probably ran into the problem of chaining the mocking with the rest of your Android test rules. If yes, I got a solution for you. If you have no idea what I am referring to, read ahead.

## The Mockito Strictness API
