---
layout: post
title: "Backbone to React: Learn.co's Track Nav"
permalink: "backbone-to-react-learn-cos-track-nav"
date: 2017-03-13 07:30:00 -0500
categories: web-development
---

Last week, I attended a React meetup where I listened to three speakers (one
from Pinterest, another from NPR, and the last from OpenTable) talk
about their stories of migrating their web applications from Backbone.js
to React.js. The day after, I was put on a project to reskin/rewrite a
part of our site's navigation header: the Track Nav. It seemed like a
good chance to flex my newly acquired knowledge to determine if I should
rewrite our Track Nav in React.

## Background
A few things to know about our Track Nav is that:

- It's a 4 level tree hierarchy
- The node data is represented by a Backbone model, which has an
attribute called `children` that is a Backbone collection of Backbone
models.
- It shares the models with other Backbone apps on the website.
- It's rendered using nested Backbone/Marionette views with some custom JS scripts to
make it show up on our page.
- It also happens to be a bottleneck on our page's initial load because
Backbone is rendering all of the views for each node of the tree
upfront.

Here's what the nav looks like:

<img style="margin: 0 auto;" width="481" alt="screen shot 2017-03-13 at 8 47 30 pm" src="https://cloud.githubusercontent.com/assets/9381931/23885107/74d9dbd4-082e-11e7-9328-b9d2ae99dd1f.png">

As I dug into this part of our code more, it became apparent that we
have some of the Track Nav state on the DOM in data attributes as well
as in the Backbone models/collections.

Given the information above, I definitely wanted to do a spike on
rewriting the Track Nav in React to see if we could address some of the
pain points involved with maintaining and updating the Track Nav.

## Strategy
As I approached this, there were a few things I wanted to do and not do:

- I wanted to improve the page load time by deferring the rendering
until you opened the Track Nav.
- I did not want to maintain and sync two sets of data on the client side
for the Backbone apps and the React app.
- I wanted to make the source code easier to follow.

With the three objectives above, I decided to only rewrite the Backbone
views as React components. If we want to rewrite the rest of the site,
that is probably the right time to bring in the Flux architecture and
completely replace the Backbone infrastructure.

In order to swap out the Backbone template rendering with React
components, I just followed these steps:

- Find the entry point or the top level Backbone view and overwrite the
`render` function. For us this was in the `TrackNavView` view.
{% highlight javascript %}
import React from 'react';
import ReactDOM from 'react-dom';
import Backbone from 'backbone';
import AReactComponent from './a-react-component.jsx';

const TrackNavView = Backbone.View.extend({
  render: function render() {
    if (FEATURE IS ON) {
      const component = React.createElement(
        AReactComponent,
        { model: this.model } // Pass in your props here
      )

      // Unmount existing react component on rerenders
      ReactDOM.unmountComponentAtNode(this.el);

      // Render the React Component
      ReactDOM.render(component, this.el);
    } else {
      // Just default back to the original render funciton of Backbone View
      Backbone.View.prototype.render.apply(this, arguments)
    }
  }
})
{% endhighlight %}
- Rewrite your Backbone template in JSX. For any view related helpers,
like labels or class names, port them over to the React component.
- For any behavior related functions (`onClick`, etc.), you can pass
them in as props to the React component. This way, the only way you're
interacting with the external data is through the existing functions on
your Backbone view.

## Results
With the above steps, let's take a quick look at how we did against our goals.

#### Improve page load time
As I mentioned before, we configured Backbone to render the entire tree
view upfront. You can see that occuring on the large green section in
the screenshot below.

<img width="1524" alt="screen shot 2017-03-13 at 8 29 27 pm" src="https://cloud.githubusercontent.com/assets/9381931/23886574/49a36b50-0839-11e7-8a69-e98433298e16.png">

Here's the timeline after we rewrote it in React to defer the rendering
until the user clicks to open the nav. The entire process of rendering
the nav is no where to be seen! (Note that the timeline magnification is
large in the screenshot below)

<img width="1552" alt="screen shot 2017-03-13 at 8 27 40 pm" src="https://cloud.githubusercontent.com/assets/9381931/23886572/46a37206-0839-11e7-8a47-33c25ed555f5.png">

#### Maintain only 1 dataset
Since we passed in the Backbone model and collections as props, any
changes to them from other apps cause the React components to rerender
if necessary.

#### Easier source code
One of the features of Backbone/Marionette is that it can automatically
render collection of views based on a Backbone collection. This is
usually a great feature, but when its being used for a hierarchy where
some of the nodes like the leaf nodes are displayed and interacted with
differently, it can be hard to follow the code.

In the React rewrite, I ended up with components specific to each node
level with the different states of the node (i.e. active, open,
completed, in progress) written declaratively. It was much easier than
manipulating the data attributes of the DOM elements and having to match
up the CSS styles based on data attributes.

## Conclusion
When rewriting a part of your app, it's always important to consider the
time investment you're making and the benefits that will come with the
rewrite. In this particular case, I think the day to rewrite the code
was well worth it and hopefully we can extend this strategy to other
parts of our app to eventually have a fully React web app. That's all I
got for now. Until next time, happy coding!


