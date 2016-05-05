---
layout: post
title:  "React-ion May"
permalink:  "react-ion-may"
date:   2016-05-04 20:04:00 -0500
categories: web-development
---
I decided to dedicate the month of May to studying the new hotness that
is React.js.

A lot of people are excited about it. I've heard good things about
it. I want to learn it to an extent that I'll be able to make
intelligent decisions on why it should be used on a project or not.

So, I haven't exactly figured out the format or schedule of how I'm
going to learn React, but I figured I should start with an overview of
what it is:

React.js is the brain child of Facebook (ooos and ahhs) and was
open-sourced back in 2013. From what I've seen so far, the React
library/framework is primariy responsible for how data is presented and
updated on the DOM. That is to say the view of the Model-View-Controller paradigm.

From a high-level, when using React, there are a bunch of components
that are configured and rendered into different parts of the DOM. Each
component is specified with properties like date, name, etc. These properties could change over time and as it does, React will rerender the component as needed. One of the buzzwords around this is React's usage of a virtual DOM to efficiently identify changes and update the real DOM.

Another aspect of a component is that it gets a `render` function
which is HTML templating written most commonly in JSX. This part makes
me cringe because I'm having to write HTML into my JS files, but we'll
see how it shakes out.

It seems like there are a bunch of advanced ways of using React.js with
your website, but I think that lays down the ground work for now.

I'll be starting out by following
[this](https://www.airpair.com/reactjs/posts/reactjs-a-guide-for-rails-developers)
tutorial written for React + Rails. Crossing my fingers that this
doesn't go over my head too much.
