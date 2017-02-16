---
layout: post
title: "Getting Wasted on 99 Bottles of OOP"
permalink: "getting-wasted-on-99-bottles-of-oop"
date: 2017-02-15 07:30:00 -0500
categories: web-development
---

Excuse the title of this post, but it's beer week here in San
Francisco and I've cracked open a bottle of beer from Deschutes Brewery.
Oh yes, I also finished the first 4 chapters of [99 Bottles of OOP](https://www.sandimetz.com/99bottles/) by
Sandi Metz & Katrina Owen so just want to jot down a few things I
learned.

## Self-evaluation
There is a fun exercise at the very beginning of the book that every
reader should attempt regardless of their experience. The directions for
the exercise can be found [here](https://www.sandimetz.com/99bottles/sample/#appendix-exercise).
The exercise is test-driven and you should spend maybe like 30 minutes
on it.

The value of doing this exercise before getting too far into the book is
that in the first chapter, the authors examine 4 different approaches to
the exercise and determines the "best" solution based on a few criteria
such as the information currently available and quantifiable metrics
like ABC (assignments, branches, and conditions).

I really liked this exercise for the simple fact that you get feedback
on the code you write from a seasoned veteran like Sandi and Katrina. I
mean it's not a direct 1 on 1 code review session, but still, it's great
to get some sort of feedback. If you're in a similar boat like me, where
you know with some certainty what bad code looks like, but don't quite
know what great code looks like, this will be good for you.

## Yin & yang of changeability & understandability
One of the points that Sandi and Katrina nail is that great code lies
somewhere between extremely changeable and extremely understandable.
There are some important ideas shared in this book like:

- Experienced programmers try to optimize for changeability because they have experienced code change before and their code loses understandability.
- Object-oriented design is making the right trade-offs to optimize
  flexibility and complexity.
- Great design is about picking the **right abstractions at the right time**. Don't reach for the abstractions, let them present themselves in the code you write.

What ended up happening for me for while I went through the initial exercise
was that after the first couple of tests, I was going down a path of
writing clever code. About half way through, I took a step back and
looked at my text editor and thought to myself "Where do I make my next
change?". That's a bad sign when you wrote the code literally 5 minutes
ago. Instead of continuing down this road, I gave up on DRY code for now
and focused on getting the tests to pass. Which leads to the next point...

## Shameless Green
I've been told to write tests for my code since day 1. But have I
been doing that... mmm nope. In the first few months of being a
developer, I was so overwhelmed with learning the domain, tools in our
stack, and the processes on our engineering team, I didn't really
practice TDD. I'm here to tell you that's ok. But don't make it a habit.

Now that we've gotten that out of the way, let me just say the concept
of shameless green is great. Write the specs for your code before you
write anything, make your tests pass (turn green), then move on or
refactor if the code you wrote is too shameful (at your own discretion).
This is sort of a balancing act too in that you don't want to litter
your codebase with magic strings or repeated code all over the place.
But at the same time, the authors make a great point in that the time
you spend refactoring has an opportunity cost of you being able to work
on other, more important matters.

One thing I will say from my own experience is that learning how to TDD
properly doesn't happen overnight. Though I've gotten much better since
I first started, I'm still working on my TDD skills. The one thing I'm
most guilty of is writing code first then tests, which opens up your
tests to know too much about the implementation of your code since you
already wrote it. I think it requires a lot of practice to get it right
so keep on writing those tests.

## Final words...
There are way too many learning points in this book to list them all so
I encourage you to read the book. I'm going to leave you with 2 of my
favorite quotes from 99 Bottles of OOP:

> Writing code is the *process* of working your way to the next stable end point, not the end point itself. You don't know the answer in advance, instead you are seeking it.

> Code is read many more times than it is written. Writing code is like writing a book; your efforts are for *other* readers... Intention-revealing code is built from the accumulation of [such] thoughtful acts.

I love that coding is a mental journey and you're writing code not for
yourself, but for others.
