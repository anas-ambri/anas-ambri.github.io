---
layout: post
title:  "The road to Javascript mastery"
date:   2014-02-19 13:24:43
categories: blog
disqus: y
tags: 
 - JS
redirect_from:
 - "blog/2014/02/19/the-road-to-JS-mastery"
 - "blog/2014/02/19/the-road-to-JS-mastery/"
---

I like to think that Javascript is a very unique programming language. While it is super easy to get started in it (thanks to the [power of its View Source](http://www.codinghorror.com/blog/2006/08/the-power-of-view-source.html)), it takes years of practise to master the language. Indeed, the [wat effect](https://www.destroyallsoftware.com/talks/wat) of JS does make it difficult to develop any expertise in the language. 

One strategy I decided to follow while learning Javascript is to force myself to write in the semantics of the language. JS is a functional language, meaning that computation should be treated as a by-product of the execution of one or multiple functions. This is certainly very different from the imperative perspective, where it is about collecting information and transforming it into decisions. Take this example, that occured to me today.

Consider a sizeable array of events, which are modelled as objects of the form:

```javascript
var event = {
    action : "tap",
    timestamp : 1393019607
};

events = [{
        "action" : "tap",
        "timestamp" : 1393019607
      }, {
        "action" : "swipe",
        "timestamp" : 1393019627
      }
      // a long series of events
      ];
```

Given a certain priority in the events' type, we would like to be able to find the index of the event with highest priority in the list. Nothing too complicated; a simple linear search that updates the value of an index variable to point to the event with highest priority.

```javascript
var actionPriority = {
    "drag" : 0,
    "tap" : 1,
    "swipe" : 2,
    "long_press": 3
}; //0 is the highest priority

var indexBest = 0;
for(var i = 0; i<events.length; ++i){
    var event = events[i];
    var actionBest = events[indexBest].action;
    if( actionPriority[actionBest] > actionPriority[event.action] ){
        indexBest = i;
    }
}
//indexBest is the index of the event with highest priority in the list
```

While this code seems familiar to any C++ or Java programmer, this is very unidiomatic JS code. The proper way to do this is to actually use the ECMAScript 5 [forEach method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach), which, you guessed it, uses an iterator. Furthermore, `forEach` is so cool that it passes the `index` variable to us, saving us the trouble of maintaining it ourselves.

```javascript
var indexBest = 0;
events.forEach(function(event, index){
    var actionbest = events[indexBest].action;
    if( actionPriority[actionBest] > actionPriority[event.action] ){
        indexBest = i;
    }
});
```

While this code looks a bit less clustered with C++-like code, it still uses the same core idea: that of checking and updating. To make this better, one has to think about the problem differently.

Indeed, what we are trying to achieve is rather simple: we would like to transform a set of similar entities, into one instance of these entities based on some characteristic. If you have ever played with a functional language before, you will notice that this is a `reduce` operation, from the infamous [MapReduce](http://en.wikipedia.org/wiki/MapReduce) paradigm.

How does reduce work? As its name suggests, it reduces a lot of data into one element. In our case, we are reducing all the events into one event with highest priority. (`_` is the [underscore library](http://underscorejs.org/), which provides the `reduce` utility).

```javascript
var best = _.reduce(events, function(best, event, index){
    var currentRank =  actionPriority[event.action];
    best = ( best.actionRank > currentRank)? 
            {index: index, actionRank: currentRank} : 
            best;
}, {
    index:0,
    actionRank: actionPriority[events[index].action])
});

//best.index contains the index of the event with highest priority in the list
```

And here lies a central difference: instead of updating just the `indexBest` variable, we are pretty much constructing a new reduced object every time. Note that this object contains not only the answer to our question (what index are we looking for?), but also any information we need to make the reduction (in this case, the `actionRank`).

This brings us back to the original idea behind this post: in order to learn a new language, one has to learn to think in the tools provided by the language. Because JS is very malleable, developers can still continue to write in their previous language's paradigm. No wonder it is so hard to become good at Javascript!
