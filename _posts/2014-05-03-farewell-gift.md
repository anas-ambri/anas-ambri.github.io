---
layout: post
title: Open-source away
categories: blog
disqus: y
tags: tech, non-tech, work
---

Last Friday was my last day at [Radialpoint](http://radialpoint.com). I spent the last 16 months working as a software developer at one of [Montreal's finest places to work](http://www.glassdoor.com/Reviews/Radialpoint-Reviews-E11266.htm), and the ride has been thrilling. While I will dedicate a full blog post to reflect on the time I spent there, I wanted to focus here on a very cool thing I got to do before wrapping up: open-sourcing some of my code.

##Necessity is the mother of invention
Last October, we were in the midst of defining a new product to build (this product later turned out to be [SupportKit](https://github.com/radialpoint/SupportKit). To fill our customer development needs, we wished to have a sample list of current Zendesk users who would potentially want to try SupportKit. Since the product is meant only for mobile, it was necessary to target only Zendesk users with iOS or Android apps on the market. Thus was born [Store Scraper](https://github.com/radialpoint/store-scraper). 

This scraper is a Nodejs script that, once fed with a list of website URLs, collects information regarding the price of the app, the latest version number, the date of its release, as well as the app page URL. On the Android side, it can go as far as finding the average rating of the app, the number of downloads, its rating, as well as the contact information of the developer. A pretty cool hack, if you ever get the chance to [look into it](https://github.com/radialpoint/store-scraper)!

##All you need to do is ask
Fast forward to a few weeks ago, where I decide to send an email to my manager, and ask him if it would be possible to open-source the project. Until that moment, the scraper has been used a couple of times internally, and proved to fulfill its original use case. Open-sourcing it, however, meant putting it up publicly, where everyone, and their [rubber ducks](http://c2.com/cgi/wiki?RubberDucking), would get to look at it and criticize [every](https://github.com/radialpoint/store-scraper/blob/master/lib/playStore/index.js#L47) [one](https://github.com/radialpoint/store-scraper/blob/master/lib/playStore/index.js#L91) [of](https://github.com/radialpoint/store-scraper/blob/master/config/config.json#L7) [its](https://github.com/radialpoint/store-scraper/blob/master/lib/query/index.js#L63-L66) [hacks](https://github.com/radialpoint/store-scraper#to-do).

My manager's response, however, was quite eye-opening. Not only did he welcome the idea, but he tried his best to get the thing up and running before my leaving. In my manager's mind, the simple fact that the code was working and somewhat useful completely justified open-sourcing it. Code didn't have to be perfect, or even have proper testing infrastructure, to be shared.

##Open-source code does not have to be perfect
As with every post on this blog, I cannot let you go without a lesson: never shy away from open-sourcing any piece of code you wrote, especially if it is the biggest hack in the world. You just solved a problem that has never publicly been attempted before, and, that is, in itself, of great value to someone out there. 

So, open-source away!

