---
layout: post
title: "React: High Order Components"
permalink: "react-high-order-components"
date: 2017-04-26 17:30:00 -0500
categories: web-development
---

I started using React almost a year ago and ran into the concept of high order
components (HOCs) a few months in. For a long time, whenever I came
across any libaries that used HOCs, I cringed and just sorta figured out
how to use it without a full understanding.

But as I've written and used HOCs, it's become one of my favorite types
of components/functions to write. What exactly is an HOC? As the React
documentation defines it:

> _A higher-order component is a function that takes a component and returns a new component._

I like to think of it as a function that intercepts a component you're
importing into another file and adds additional behavior. We can use the
HOCs to accomplish a few things:

1. Share behavior through composition rather than inheritance.
2. Create abstractions of common component behavior.
3. Layer multiple behavior together without issues of collision.

To read more about HOCs I recommend Dan Abramov's post [here](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750) and the
official documentation [here](https://facebook.github.io/react/docs/higher-order-components.html).

## Using a High Order Component
So now that I've talked about how awesome and useful HOCs are, let just
take a quick look at how you use a HOC. Let pretend we have a
`ChildComponent` that we want to render from a `ParentComponent`. Our
HOC will be called `HighOrderComponent`.

Apologies for the horrible horrible syntax highlighting. Apparently
Rogue does not support `jsx` style syntax yet...

{% highlight jsx %}
// ChildComponent.jsx
import React from 'react';

import HighOrderComponent from '../hoc/HighOrderComponent.jsx';

class ChildComponent extends React.Component {
  render() {
    return(
      <div>
        <div>Hello from ChildComponent!</div>
        <div>{this.props.parentMessage}</div>
      </div>
    )
  }
}

export default HighOrderComponent(ChildComponent);
{% endhighlight %}

The most important line of code is the `export default` where we are
invoking the HOC function and passing in our `ChildComponent`. The
return value of the HOC function, another component, will be exported
from this file.

{% highlight jsx %}
// ParentComponent.jsx
import React from 'react';

import ChildComponent from './ChildComponent.jsx';

class ParentComponent extends React.Component {
  render() {
    return(
      <div>
        <div>Hello from ParentComponent!</div>
        <ChildComponent parentMessage='Message from parent'/>
      </div>
    )
  }
}

export default ParentComponent;
{% endhighlight %}

Nothing too special in the implementation of `ParentComponent`. Its
great that we can use `ChildComponent` with no knowledge that it is
wrapped inside a high order component. The resulting html will look like
this:

{% highlight html %}
<div>
  <div>Hello from ParentComponent!</div>
  <div>
      <div>Hello from HOC!</div>
      <div>
        <div>Hello from ChildComponent!</div>
        <div>Message from parent</div>
      </div>
  </div>
</div>
{% endhighlight %}

Note that there in the layer between the Parent and Child render,
additional `div`s are present with a message from the HOC. So anytime
you wrap a component, you can add additional content or pass additional
props into the child component. Pretty neat.

## Building a High Order Component
In the previous example, I didn't show the implementation of the HOC
beacuse I wanted to show it to you here. I'll be using ES6 syntax for
this.

After building a few HOCs, I've come up with a template for building a
high order component:

{% highlight jsx %}
import React from 'react';

const HighOrderComponent = OriginalComponent => {
  class NewComponent extends React.Component {
    render() {
      return(
        <OriginalComponent
          {...this.props}
          {...this.state}
        />
      )
    }

    // Add other behavior here and pass in as props to OriginalComponent
  }

  return NewComponent;
}

export default HighOrderComponent;
{% endhighlight %}

You can see that we define a function called `HighOrderComponent` and
export it from this file. In the body of the function, we define a new
React component and return it. The boilerplate just renders the
`OriginalComponent` with no special behavior. One important thing to remember
is to pass the props that the `NewComponent` receives to the
`OriginalComponent`. If you forget to do this, you'll start seeing props
disappear!

If we take the previous example and use our template from above, our
high order component implementation will look something like this:

{% highlight jsx %}
import React from 'react';

const HighOrderComponent = OriginalComponent => {
  class NewComponent extends React.Component {
    render() {
      return(
        <div>
          <div>Hello from HOC!</div>

          <OriginalComponent
            {...this.props}
            {...this.state}
          />
        </div>
      )
    }
  }

  return NewComponent;
}

export default HighOrderComponent;
{% endhighlight %}

## Real world example
Equipped with the knowledge of how to build a HOC, let's examine a real
world example. I recently had to build a series of dropdown menus for
our website header. We wanted the menu to close if you clicked anywhere
outside of the component. In order to implement this, I
created a high order component called `OffClickWrapper` that will
invoke the callback function `onOffClick` that is supplied via props.

{% highlight jsx %}
// OffClickWrapper.jsx
import React from 'react';

const OffClickWrapper = Component => {
  class WrappedComponent extends React.Component {
    constructor(props) {
      super(props)
      this._addOffClickListener();
    }

    componentWillUnmount() {
      this._removeOffClickListener();
    }

    render() {
      return (
        <div ref={node => this.node = node}>
          <Component {...this.props}/>
        </div>
      )
    }

    _addOffClickListener() {
      document.addEventListener('click', this._handleOffClick.bind(this), false);
    }

    _removeOffClickListener() {
      document.removeEventListener('click', this._handleOffClick.bind(this), false);
    }

    _handleOffClick(e) {
      if(!this.node) { return; }
      if(this.node.contains(e.target)) { return; }

      if (typeof this.props.onOffClick === 'function') {
        this.props.onOffClick(e);
      }
    }
  }

  return WrappedComponent
}

export default OffClickWrapper;
{% endhighlight %}

The best part about this HOC is that we can use it in our modal
implementation so that when a user clicks off of the modal, we can close
it. And since not all modals need to close on an off click, we pick and
choose which ones have this behavior. Score one for Team Composition!

Theres a ton of other stuff you can do with HOCs like have it hold
state, but we'll save those for another time. Until next time, happy
coding!
