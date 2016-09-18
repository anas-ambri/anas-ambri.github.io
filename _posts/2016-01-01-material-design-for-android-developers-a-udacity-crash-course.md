---
layout: post
title: "Material Design for Android Developers: a Udacity crash course"
comments: true
categories: android
published: true
disqus: y
tags:
 - Material Design
 - Android
 - Udacity
redirect_from: 
 - "android/2016/01/01/material-design-for-android-developers-a-udacity-crash-course/"
 - "android/2016/01/01/material-design-for-android-developers-a-udacity-crash-course"
---

I never really had a chance to board the Material Design bandwagon when it started out. Working mostly with clients that go iOS first, I always just had to integrate iOS-minded designs and make them work on Android. To be honest, I never thought Material Design would catch on. I kinda felt it was a botched up attempt by Google to catch up on the look-ma-I-too-can-make-pretty-things wave (I am not saying design is not important; to the contrary, it's just that, maybe, there are more pressing matters for the Android platform). Anyway, I am glad I was proven wrong.

Material Design just grows on you: it's one of those things you just get used to. Seeing as more and more apps are transitioning into Material, I decided it was my duty to pick it up. It just happened that a link to the Udacity course about Material Design for Android Developers appeared on my reddit frontpage. This article is a collection of notes taken while completing the course.

Overall, the class was actually pretty good; very pragmatic, and very useful for someone like me who has no design skills whatsoever. It even made me wanna purchase a license of Sketch!

---

## Lesson 1: Introduction to Material Design

### Pixels vs DIP

- Just like everything in life, start design on small screens
- Android 1.6 introduced support for varying densities
- Density-independent pixels are a construct that transcends space, time, and resolution. Unit is dp

A device has a screen resolution, unit is dpi (dots-per-inch)

>  # pixels / # dp = device_dpi/160 dpi

```
ldpi    <--> 120dpi <--> .75x
mdpi    <--> 160dpi <--> 1x
hdpi    <--> 240dpi <--> 1.5x
xhdpi   <--> 320dpi <--> 2x
xxhdpi  <--> 480dpi <--> 3x
xxxhdpi <--> 640dpi <--> 4x
```

### Assets

- If including only one, use the highest dpi
- Don't use raster graphics, use vector graphics apps like Sketch, where you can export in svg

###Layouts
 
- FrameLayout should be used when children are overlapping. Last added child overlaps the ones before it
- Avoid putting RelativeLayout as root
- ScrollView must contain only one child (applies to iOS too)
- If layout_weight_child == weightSum_parent ==> child_height = parent_height - total_other_children_fixed_height (useful for making fitting view)
- A ScrollView's child layout_height should be wrap_content

###Common patterns

- Height of toolbar can be queried at `?actionBarSize`
- "Scrolling a list and horizontal paging of content are incompatible", Nick Butcher. You hear that, Gmail?

##Lesson 2: Surfaces

###Overview

A surface is a container for content, it provides grouping and separation from other elements.

- In a listview, putting each item in their surface slows your ability to scan the items. Use one surface with separators
- In heterogenous items, use different surfaces (for Masonry-like layout, for example)
- In general, not more than 5 surfaces

Elevated objects cover other surfaces with shadow. The closer the object, the more shadow it casts.

> android:elevation="4dp"

###FAB

The FAB can be either 40 or 56 dp diameters, with 6dp elevation at rest, with 12dp elevation when pressed
Use the design support lib

    <FloatingActionButton
	app:fabSize="normal|mini"
	app:elevation="6dp"
	app:layout_gravity="end" <-- right end -->
	app:pressedTranslationZ="12dp"
	/>

When using appcompat for design, switch `android:Theme.Material.x` to `Theme.AppCompat.x`

####Example implementation
[FabDemo](https://github.com/udacity/ud862-samples/tree/master/FabDemo)

###Ripples

If a surface has bounds, ripple affects the whole area
If a surface is boundless, ripple creates a circle

###Paper metaphor

A list item in a list view can transform and expand to create the details view, and overlay a new surface

###Seam-to-step

This effect happens when a view is elevated above a sibling with which it was sharing the same elevation.
[Scrolling toolbar demo](https://github.com/udacity/ud862-samples/tree/master/ScrollEventsDemo) 

When using AppBarLayout, remember that it expects a sibling with `nestedScrollingEnabled=true`

##Lesson 3: Bold graphic design

###Getstalt principles

- Law of past familiar experience: use familiar objects
- Law of proximity: close stuff are related
- Law of similarity: similar objects are related

###Grids

- All spacing values are multiple of 8dp
- All line heights are multiple of 4 dp

[Sketch template with Material Design](https://www.google.com/design/spec/resources/sticker-sheets-icons.html#sticker-sheets-icons-product-icons)

###Colors

You can override the Theme (AppCompat or Material) default colors using the values:

- colorPrimary: for app bar color
- colorPrimaryDark: for status bar color
- textColorPrimary: for app bar text color
- windowBackground: for background color of window
- navigationBarColor: for software buttons bar background color
- colorAccent: for stuff that sticks out

[Choose a palette of colors](https://www.google.com/design/spec/style/color.html)

###Typography

Use Roboto. Enough said

###Icons

[Material Design Icons](https://materialdesignicons.com/)

##Lesson 4: Meaningful motion

###Animations in Android 4.4

New TransitionManager class that transitions between scenes. TransitionInflater for custom xml <transitionSet>

###Animations in Android 5.0

- Content transition for transition in master/detail views
- Shared element transition

[Demo](https://github.com/udacity/ud862-samples/tree/master/Unsplash)

To customize animation for shared element transition, change the `windowSharedElementEnterTransition` theme property


###Instructive motion

- Animation length ~300ms
- Use fast-out slow-in for movement within the screen
- Use linear-out slow-in for objects entering the screen
- Use fast-out linear-in for objects leaving the screen
- Use Interpolator to control movement: http://developer.android.com/reference/android/view/animation/Interpolator.html
- When doing a shared element transition, use a curved motion
- When doing an increase in size, change width then height

[Demo](https://github.com/udacity/ud862-samples/tree/master/CoordinatedMotion)

###AnimatedVectorDrawable

[TickCross Demo](https://github.com/udacity/ud862-samples/tree/master/TickCross)


##Lesson 5: Adaptive design

Always inherit your theme twice, so you can override the base theme in your sw600-dp style


