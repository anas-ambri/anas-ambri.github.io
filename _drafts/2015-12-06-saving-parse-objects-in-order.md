---
layout: post
title: "Saving Parse objects in order"
categories: ios
published: false
disqus: y
tags: ios Parse 
---

At [Guaraná](guarana-technologies.com), we use [Parse](https://parse.com/) as the technology stack when building mobile apps. Parse is a [Mobile backend as a service](https://en.wikipedia.org/wiki/Mobile_Backend_as_a_service) that allows you to abstract away the complexities of building a backend service for your iOS or Android app.

As part of the Parse SDK, you get an [Active Record](http://www.martinfowler.com/eaaCatalog/activeRecord.html)-like object that is capable of syncing itself automatically with the backend. This functionality is exposed by the [PFObject](https://parse.com/docs/ios/api/Classes/PFObject.html), and the [ParseObject](https://parse.com/docs/android/api/com/parse/ParseObject.html) in iOS and Android, respectively.

For example, if your app