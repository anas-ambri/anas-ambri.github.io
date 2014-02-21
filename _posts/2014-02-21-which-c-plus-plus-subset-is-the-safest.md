---
layout: post
title: "which C++ subset is the safest?"
categories: blog
published: false
tags:
 -
---

Yes, there is no right or wrong way of doing this in C++. However, it is very easy to find the 'most conventional way of doing things', and, since this way is something generations and generations of people have looked into, it is more likely to be right.

Use Steve Meyer' description of C++ in effective C+ (4 subgroups).
Parse github repo to look for which subset is the most used. How many issues each subset usually contains per number of lines?

Not a good study because:

- github is not representative of the SOEN community (need to add bitbucket, and any other repo website)

- People who keep repos on github usually do it on their spare time, thus less likely to use C++ (or, at least, not for its original purpose (high-performance and embedded systems coding)

- However, these who do will be representative of the practises they follow at work