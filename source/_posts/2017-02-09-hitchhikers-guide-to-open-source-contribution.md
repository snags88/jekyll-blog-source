---
layout: post
title: "A Hitchhiker's Guide to Open Source Contribution"
permalink: "a-hitchhikers-guide-to-open-source-contribution"
date: 2017-02-09 07:30:00 -0500
categories: web-development
---


## Why do it?
First of all, why even motivate myself to contribute to open source
projects? I mean, as a professional developer, I write enough code as
already right?

> Wrong - _Donald Trump interrupting Hillary Clinton 50 times in a
> debate_

I like to think of it this way: if people didn't spend anytime at all
contributing to open source projects, let alone create them, then we'd
all be solving the same problems over and over, while probably coming
across the same gotchas and bugs. And if you really think about it, if
we just solve the same set of problems, how can we move on to solving the
harder and more complex problems of the world.

Imagine I start learning how to program at the age of 18 and continue to
hone my craft until I was 60, but never use open source libraries nor
contributed to any. I would spend a large chunk of the time
writing vanilla JS DOM selectors and manipulating the DOM directly,
until eventually I figure out that storing state on the DOM is a bad
idea. Or that when I interact with a database, it's easier to model a
row of data as an object, then eventually I might end up with my own
ORM.

Instead of worrying about that stuff, I can use libraries like
React.js or ActiveRecord that solve those problems and focus on building a product that can
address bigger problems.

That kind of gets into my second point, which is that someone poured
their sweat and tears into creating these open source libraries that
allows you to do your job (or hobby) more efficiently. Would it not be
the right thing to do to return that favor for the next young, budding
developer? Pay it forward my friend.

Lastly, I think contributing to open source projects builds empathy
towards other devs and hopefully that extends to your other work.
Because let's be honest, developers aren't known to be the most
empathetic or understanding group of people.

Ok rant over. Now for some more interesting, less opinionated part of
the post...

## How to do it?
Well there are a ton of open source projects out there so just on
Github, randomly pick one and write some code.

Just kidding.

First thing is to **start small and build confidence** on your open source
commits. Don't let your first attempt be a commit to the Rails repo or
jQuery. Pick a smaller library that you're familiar with to get in the
flow of things. I would even go as far as saying improve the
documentation if you find it hard to understand or add some examples.
The worst case scenario is that your pull request gets rejected for some
legitimate reason and the best case is that you helped someone else
undertand how to use this library.

Second thing is to **contribute to libraries you use**. Most of the time,
you will run into a limitation or pain point of the library and as an engineer, you
find a solution for it. If that solution is applicable beyond the domain
you're working in, I bet it's a good candiate to be added to the
library. Remember, if you ran into this problem, chances are high that
others will too! Help a brother (or sister) out!

Third thing is to **check the contribution guidelines on the README**
before you open a pull request. Each maintainer is different so in order
to avoid friction and get your PR merged in, try to follow the rules he
or she has set. Whether it's writing a spec for your new code or
updating the documentation, follow their guidlines!

Fourth thing is to **be clear about the problem you're solving**. Your PR
is a great place to engage in dialogue with the maintainer or other
devs. You might find some useful knowledge and more likely than not, if
your PR isn't ready to merge, they will tell you what's wrong with it.

Last thing: **commit and prosper**. You've done good for the
world. Give yourself a pat and buy yourself an extra drink the next time
you're out at a bar.

## Example
At work, we've been using a library called [`rollout`](https://github.com/fetlife/rollout) to manage feature flags. It has a lot of nice APIs, but I found a few convenience methods that were missing that I had added to our own instace of `rollout`

My first contribution to this library was literally adding an `#inactive?`
method that returned `!active?`. You can see it
[here](https://github.com/fetlife/rollout/pull/86/files).

Yes. That simple.

The next contribution I made was a little bit more complex, but I
described the use case in the body of the PR, added specs for the new
method, and voilà! [My PR](https://github.com/fetlife/rollout/pull/90) was merged in.

## Conclusion
As a newish developer, I get stage fright knowing that
my code will be scrutinized by experienced devs, but the more often you
put your code out there, the better you get. We should all try to be
helpful to each other and contribute to open source projects.
