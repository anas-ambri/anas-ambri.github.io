---
layout: post
title: "Prompt for PIN in Android when coming from background"
comments: true
categories: android
published: false
disqus: y
tags:
 - Android
 - Activity lifecycle
---

Say, you have a screen that shows some confidential information (like a user's bank information), and say that, for the sake of ensuring that this information is not seen by some malicious third-party, you are required to display a PIN-input screen, everytime the app is brought back from the background. How would you go about implementing this in Android?

## Option 1

Since the screen has to be shown first every time the app is launched, the natural thing to do is to simply implement the PIN-input screen as the entry point of your app. Placing the entry point is as easy as changing the `AndroidManifest.xml`:

```java

<activity
    android:name="PinScreenActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

The problem with this approach is pretty obvious. What happens when 

## Improvements

This solution does not handle the following:

- Covering the screens that require protected when the 'Recent' button is clicked
- Multi-window support

