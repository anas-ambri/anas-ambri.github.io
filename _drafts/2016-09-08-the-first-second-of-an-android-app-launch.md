---
layout: post
title: "The first second of an Android app launch"
comments: true
categories: android
published: false
disqus: y
tags: android system
---

Lately, I caught myself asking this question:

*What happens from the moment you clicked on an app's icon on the home screen, to the moment said app is displayed?*

I mean, sure enough, the app does get displayed, but how does it *really* happen? And don't tell me we should go wavey-hands-in-the-air again, I already did that with RxJava.

I agree that this is the kind of question that an Android developer need not to bother with for day-to-day development (other such questions can be [Where to put a TabBar with Material Design?](https://material.google.com/components/bottom-navigation.html#bottom-navigation-usage), or [when is `onDestroy()` called?](https://developer.android.com/reference/android/app/Activity.html#onDestroy())). Yet, I feel that it helps to sometimes step back, and stare down at all the layers that service our apps, if not to just enjoy the complexity bundled in our palm-sized devices (or head-sized ones, for you [phablet fans out there](https://www.reddit.com/r/Android/comments/3ewh0n/is_a_55inch_flagship_phone_too_much_to_ask_for/). Plus, who wouldn't wanna pull out this question for those phone interviews?


##On the first day, it was just a screen


##Touchscreen effect: what a simple touch can do


##Your home screen is an app

##Resources

[1]: https://source.android.com/devices/input/touch-devices.html
[2]: http://processors.wiki.ti.com/index.php/Capacitive_touch_integration_with_android
