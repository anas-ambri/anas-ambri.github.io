---
layout: post
title: There is a special place for Samsung in Android hell
comments: true
categories: android
published: true
disqus: y
tags:
 - Android
 - Rant
redirect_from: 
 - "android/2015/12/19/special-place-for-samsung-in-android-hell/"
 - "android/2015/12/19/special-place-for-samsung-in-android-hell"
---

**Disclaimer**: if you are here just because you heard someone is bashing Samsung, scroll to the bottom. Also, you came to the right place.

If you are using the support appcompat library (and, of course you are), you might have seen the following stack trace on Crashlytics, coming from Samsung 4.2.2 devices:

```
java.lang.NoClassDefFoundError: android.support.v7.internal.view.menu.MenuBuilder
at android.support.v7.app.ActionBarActivityDelegateBase.initializePanelMenu(ActionBarActivityDelegateBase.java:991)
at android.support.v7.app.ActionBarActivityDelegateBase.preparePanel(ActionBarActivityDelegateBase.java:1041)
at android.support.v7.app.ActionBarActivityDelegateBase.doInvalidatePanelMenu(ActionBarActivityDelegateBase.java:1259)
at android.support.v7.app.ActionBarActivityDelegateBase.access$100(ActionBarActivityDelegateBase.java:80)
at android.support.v7.app.ActionBarActivityDelegateBase$1.run(ActionBarActivityDelegateBase.java:116)
at android.os.Handler.handleCallback(Handler.java:725)
at android.os.Handler.dispatchMessage(Handler.java:92)
at android.os.Looper.loop(Looper.java:176)
at android.app.ActivityThread.main(ActivityThread.java:5299)
at java.lang.reflect.Method.invokeNative(Native Method)
at java.lang.reflect.Method.invoke(Method.java:511)
at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:1102)
at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:869)
at dalvik.system.NativeStart.main(Native Method)
```

Now, listen very carefully, because this is a story you gonna tell to your grandchildren. 

- It turns out that on some select devices (you can find the list [here](https://code.google.com/p/android/issues/detail?id=78377#c311)), the ROM image has bundled up its own version of the support MenuBuilder, and exported it on the classpath. Now, when you include your own version of the support library (and, of course you should), and used the support ActionBar, the app will crash because it doesn't find the class it wants.
- When the issue got originally reported to the Android project, the initial workaround was to obfuscate the MenuBuilder class name (this way, it wouldn't clash with Samsung's), or revert to v20. Obfuscation being the easiest route (obviously), this worked for the support library from v21.0.0, until...
- A year later, when the Google team decided to do something about it (the outrage on the issue tracker was deafening), they renamed the package that contains MenuBuilder, thus avoiding any clash. This change made it into v23.1.1 of the library, but wasn't enough because...
- The support design library is also using Menubuilder, and so we were back to square one.

And this is just the first Crashlytics report of the day.

At this point, you just want a solution, so you do a quick google search, and you run into this article, that tells you to use Proguard, and add the following line:

```
# Samsung ruining all nice things
-keep class !android.support.v7.view.menu.**,
 !android.support.design.internal.NavigationMenu,
 !android.support.design.internal.NavigationMenuPresenter,
 !android.support.design.internal.NavigationSubMenu, 
android.support.** {*;}
```

**Note that this should all be in one line**. Note also that the comment is not necessary for this change to work, but very encouraged for your own sanity.


Where to go from here
---------------------

Now that we got that out of the way, I would like you to grab a cup of coffee, and come back to this, because we got some ranting to do.

If you are an Android developer, your hatred for Samsung devices is probably boundless. More than an average user, for whom Samsung is synonymous with [silly Touchwiz](http://www.androidauthority.com/community/threads/why-most-of-the-people-hate-samsung.588/) and [excessive bloatware](https://www.reddit.com/r/Android/comments/1zyo91/why_the_hate_for_samsung/), you despise Samsung because you don't have a choice. Because of Samsung's [massive market share](http://www.cnet.com/news/samsung-continues-to-rule-over-apple-in-smartphone-market/), you simply cannot choose not to support Samsung devices. And that's what hurts the most; the fact that this choice is taken away from you!

Don't get me wrong, Samsung makes unbelievably good phones. But, everyone and their mum just wish they could put as much effort into their software. The fact that so few Android devs use Samsung as their personal phone definitely speaks for itself.

Of course, these problems are not specific to Samsung, and plague all the ROMs (except Google's Nexus master race). But, you gotta admit that once you become the leading Android phone manufacturer, there is some requirement of being able to do better than the average OEM (as they say in NYC, great power comes with great responsibility).

The point being, this is part of the reason why the iOS guys can't take us seriously: Android is in dire need of stability. I understand that fragmentation is a thing, but it would be so nice to be able to say that maintenance costs for an Android app won't include tracking down a Wiko phone that shouldn't have been called Wiko in the first place.

Instead of [letting the hate flow through you](https://www.youtube.com/watch?v=_Avn2nT16FA) (obligatory Star Wars reference), it would be very interesting to see if the Android community can do something about this. HTML5 has the [CanIUse](http://caniuse.com/) project, iOS devs have [SDK critic](http://www.sdkcritic.com/). Can we create a database of all the quirky bugs that have been piling up through generations of devices? Stackoverflow alone is just not enough.

