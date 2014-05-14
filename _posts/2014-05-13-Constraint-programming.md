---
layout: post
title: Constraint programming
categories: blog
tags: on-the-road, discipline
---

While going on my road trip, I decided to take a small laptop with me to do some programming/news browing. While the jury is still out there on whether that was a good idea or not, I thought I should write about the experience of using my notebook for software development during the trip so far. The machine in question is a 10"-screen 5-year old Internet-less [Dell notebook](http://www.dell.com/ca/p/inspiron-mini1012/pd), with a complete fresh Ubuntu 13.04 install. The title of this post is inspired by the fact that, like in [constraint programming](http://en.wikipedia.org/wiki/Constraint_programming), coding while on the road is defined mostly by what you can't do. (Hint: lack of Internet changes *everything*)


##The most basic need
<img src="/images/internet_maslow.png" />
You will never understand how scarce Internet access is until you are in a car for 8 hours a day. Wi-fi on campgrounds and motels will be inconsistent at best, so your only hope will lie on coffee-shops (Tim Hortons', in Canada). Typing `apt-get install` or `npm install` wil become a holy experience, where you are constantly praying to the Wi-fi gods that your download will succeed. Git pushes will be in themselves moments of celebration, independently of the quality of the code submitted. And let's not even mention those nights where you will leave your laptop on cloning a repo so that you could use it the next day.

Knowing that you might never be able to install your dependencies means that: a) you would need to choose them wisely, b) you need to be able to continue working even if they are not installed. This changes the way you program completely.


##Fake it until you install it
The idea here is to code your program in such a way that it can work both offline and without the dependency you are supposed to rely on. This means, for example:

- Writing a script that generates the HTML page that your web scraper is supposed to scrape, so that you can test said scraper.
- Refactoring your data layer in such a way that you can save to files instead of databases, while you find enough bandwidth to download and install mongodb.
- Reusing dependencies that come bundled with packages on other projects. I never thought I would be so happy that my dependencies had dependencies.


##So you are saying I can't google this?
One thing every programmer has grown accustomed to is the use of Google/Stackoverflow to answer the most random queries. While this comes with its benefits (faster development, anyone?), it is simply not a possibility while driving down [Saskatchewan](http://en.wikipedia.org/wiki/Saskatchewan). Thus becomes a fight for information, where digging into the source code of your stack becomes a mundane task. 
Don't know how to start a shell command from a python script? Just try `whereis python2.7 | xargs grep -rl 'shell command' --exclude=*.pyc` until you find a well-documented `commands.py` file that contains examples on how to run it.
Want to know how many parameters are passed to that function? How about you grep every use of that function in your favorite library? You might find some pretty interesting hacks on the way.


##Power is everything
This one goes without saying: the fact that the notebook's battery can't hold up more than 2 hours is a huge bummer. My biggest regret was not buying a new battery before leaving Montreal. However, the challenge is certainly to squeeze as much work as possible from those few hours. One of the coolest tricks I picked up is to write most of the code on paper, and only open the editor when it has been fully debugged. Obviously, the process is just more tedious, but ultimately, you are just trading development time for power, and in this case, there is plenty of time while driving across a whole continent.


##Epilogue
The moral of the story is that, having the most powerful development environment is not always necessary when writing software. My general feeling is that you will also write better code, simply because you will constantly be reminded that everything you always relied on is gone.
And yes, you have no idea how difficult it was to get this post uploaded...



PS: I know I am not supposed to tell you what to do, but, you know, you could always <a href="https://twitter.com/AnasAmbri">follow me on Twitter</a>