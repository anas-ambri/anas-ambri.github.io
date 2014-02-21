---
layout: post
title: "A new way of teaching programming"
categories: blog
published: false
tags:
 -
---

Let's face it: universities do not teach programming. Yes, you learn, dependently on your degree, about computer science, software engineering, computer engineering, information systems or even communication systems, but you don't learn programming, for the simple fact that no programming happens in universities.

If you think academics are actually working on fixing this, you are probably right. But when every single paper starts with something like this:

    Three decades of active research on the teaching of introductory
    programming has had limited effect on classroom
    practice.

[1]
maybe you start to understand why it is taking so long.

How it should be done:

- Teach the syntax of the language (whatever the language, really, although the more concise the better) in a very Head-First style

- Start teaching methods and techniques that programmers (not designers, not architects, not engineers) use

- Teach debugging: what is the most transferable skill in the whole university curriculum? Communication? No, you can probably pass most courses without uttering a single word? Problem-solving? Maybe, but that is something inherent to any engineering discipline. The single skill that you will probably need for the rest of your career is debugging (or troubleshooting, if you are more into the kinky hardware stuff)

- Use analogies when explaining the whole concept is impossible: when talking about compiled versus interpreted languages, for example. An analogy helps get the point across, without delving too much into (platform-dependent) details. I know that most analogies make computers sound like they are the result of black magic, but, for a beginner, that is already the case. So why not using it?
- Use two introductory programming languages, not one : one for showing how programs can affect the world, develop their curiosity (JS for example: http://www.youtube.com/watch?v=OlkW5DKr6G8), and a second one where you can teach formal notation (anything that is concise enough to fit in 3/4 a semester, C or Scheme)

- Use a language that teaches debugging. I will return to debugging a bit later, but, the point I am trying to make here is that, do not use a language that fixes the bugs for its users (yes, I mean you, Java). As Stephen Draper noticed while teaching Pascal, students will quickly start to rely on their compilers to spot the errors for them, to the point where they  (For additional marks, design such a language)

- Create a team-oriented environment where a person is more likely to ask a friend to look into their code than to code by themselves

- Spend the rest of the semester listening to music while coding increasingly challenging problems in class, and watch students make progress
Programming is a craft; just like the apprentice spends countless hours watching his master and reproducing the same thing on a piece of wood, a programmer-to-be needs to sit and watch a more-experienced person (in this case, a professor) pound at a terminal, find problems, and fix them. This is what every programmer does, and I absolutely do not comprehend why people would do things differently and expect the same result. Just like insanity is doing the same thing, over and over again, but expecting different results (Einstein), one can only believe that absolute foolishness is to do things differently and expect the same result.

The only reason people consider short-circuiting university is simply because they do not learn how to program at universities.

A practical example of how to teach programming

Freshmen fancy nothing more than marks. This is simply due to the fact that they still carry the mentality from high school, where competition for more grades can only be topped by the maximum score achievable. Thus, the use of extra marks/grades is a very strong incentive, especially for people who are to begin university. Second, nerds are by nature very competitive, especially when it comes to solving problems. Finally, this would give the chance to the nerdiest to achieve a higher level of popularity, something they probably never got the chance to experiment before, and which would allow them to attract a new genre of people (hopefully a different gender altogether).

Give out daily programming assignments, in the form of a step-by-step guide to get some result. For example, ask your students to write a program that prints out the prime numbers smaller than 100.

Such a problem could be enunciated as following:

1- Write a function with parameters a and b that returns true if and only if a is divisible by b. Remember that, for a and b integers, a is divisible by b if there exists two numbers c and d such that b*c +d = a and d = 0 (I know this is a horrible definition, but I believe it is the correct one since it accounts for negative integers as well. Anyway, just keep reading, and you will get my point)

=> Give half mark if student uses the definition above to implement the function. Tell them that throughout their careers, they will always get obscure, if not incorrect, specifications, so they should probably get used to it. Skill acquired: Common sense (?), Research.

=> Give full mark if student solves problem by finding only c instead of c and d. Skill acquired: Logical reasoning

=> Give extra mark if student uses a%b == 0 (to reward best solution). Skill acquired: Aesthetic sense

Note that we would expect students with different level of experience to develop different set of skills. It seems natural, if not necessary, that a person who can't manage to get the modulo operator right still needs to learn some common sense, while the more advanced person is not only rewarded through extra grades, but also gets the chance to develop a skill very much advanced, while avoiding to fall asleep from sheer boredom.

Learning debugging Hogwarts-style

- Using points to reward debugging other people's code

- Types of debugging (from Stephen Draper)

"a) spotting errors that do not require understanding of the program e.g. missing declarations, redeclarations of the same identifier, code that can never be executed. b) Predicting what the code must do, and realising that this is not (cannot be) what is wanted. E.g. looking at a set of conditions (e.g. in a multi-branch if statement) and deciding if cases seem to overlap or be missed."

Notes:
[1] A survey of literature