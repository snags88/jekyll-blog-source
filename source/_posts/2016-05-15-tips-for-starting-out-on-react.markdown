---
layout: post
title: "Tips for Starting out on React"
permalink: "tips-for-starting-out-on-react"
date: 2016-05-15 13:04:00 -0500
categories: web-development
---
Ok, its been about half the month and I've made it through handful of
React tutorials and I feel pretty comfortable using React as a
client-side framework.

## What I wish I knew 15 days ago...
As I made my way through some of the tutorials, I found that if
someone told me to read them in a certain order, it would have helped a
lot more than the random order I ended up going in. Here's the order I
would recommend:

1. [React.js: Introduction for People Who Know Just Enough JQuery to get by](http://reactfordesigners.com/labs/reactjs-introduction-for-people-who-know-just-enough-jquery-to-get-by/)
  - Covers basics of how React works by comparing it to something you
    already know.
  - Build as you go tutorial.
  - No local development environment need.
  - Also covers React vs JQuery at a very fundamental level.
2. [React.js: A Guide for Rails Developers](https://www.airpair.com/reactjs/posts/reactjs-a-guide-for-rails-developers)
  - Teaches you how to work with data + React.js without introducing
    Flux.
  - Build as you go tutorial.
  - You get to use coffeescript.
  - Local setup required is a Rails dev environment.
3. [Learning React: Getting Started and Concepts](https://scotch.io/tutorials/learning-react-getting-started-and-concepts)
  - Not a step-by-step tutorial.
  - Provides a nice overview of the capabilities of React with links out
    to the documentation.
4. [React Documentation](https://facebook.github.io/react/index.html)
  - Pretty in depth.
  - Provides code samples that makes sense.
  - Gives you an idea of what else is possible with React.

I'm trying out a few more tutorials before I move onto Flux, but the
above 4 will get a beginner in React onto the right path.

## Backbone/Marionette vs React
As I've been going through the tutorials, I've really enjoyed thinking about how to
architect the JS view/component objects since the approach is
different than Backbone/Marionette vs React.

1. React tends to require more granular components. This means more
building out parts, but I think what's going to end up happening is
that a lot of reusable components will be created.

2. Controlling the behavior of the DOM elements is much easier in React.
No more weird JQuery stuff to manipulate the DOM. Change the state and
let React take care of the rest.

3. State control is straight forward. One of the tougher parts of using Marionette was knowing when to
rerender certain views if it didn't do it automatically. It's supposed
to rerender when the collection or model is updated, but sometimes you
had to rerender the associated view or some of the other views to keep
the state of the data in sync with the view. In React, if you make sure
that the state is set on the parent component and pass it down to the
subcomponents as its props, then the DOM and the state stay in sync.

4. Learning React is a little bit easier because it only impacts the
   view. Backbone/Marionette has a steeper learning curve because it
also handles communication with the API and can wrap the data up in
models and collections. Also, Backbone/Marionette is not opiniated at
all so if you don't know what you're doing, you can end up with
spaghetti code.

That's all I got for now. Hopefully more updates in the weeks to come.
Cheers!
