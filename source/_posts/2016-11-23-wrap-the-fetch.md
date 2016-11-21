---
layout: post
title: "Wrap the Fetch"
permalink: "wrap-the-fetch"
date: 2016-11-23 07:30:00 -0500
categories: web-development
---

Alright, here's my promised post about the first library I published on
npm. If you missed my post about how to publish a library to npm, check
out the [previous post here]({% post_url 2016-11-20-creating-an-npm-library-with-webpack %}).

## Origin of `wtfetch`
The library I published is called `wtfetch` short for "wrap the fetch".
Yeah I gotta work on my name creativity, but Mean Girls references were
mostly taken along with Farfetch references...

Anyway, the library is a wrapper I created for the (somewhat) new Javascript `fetch` API
for [Learn.co](https://learn.co). In our codebase, we were using
jQuery's `$.ajax()` function to make API calls from our Alt sources (an
implementation of Flux), which seemed pretty unnecessary since we were
requiring an entire library to make our AJAX request.

Upon further research, there are libraries that are solely creating for
making HTTP request from the client ([`axios`](https://github.com/mzabriskie/axios), [`superagent`](https://github.com/visionmedia/superagent) to name a few). However, all of these libraries are just convenience wrappers around the underlying Javascript `XMLHttpRequest` API.

My research eventually led me to the new Javascript `fetch` API that
have been implemented in newer web browsers (see [this chart](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API#Browser_compatibility) for compatible browsers) and of course the brilliant people at Github created a polyfill that would make this functionality available on unsupported browsers.

The usage and understandability of `fetch` is a million times easier than `XMLHttpRequest` and [this post](https://developers.google.com/web/updates/2015/03/introduction-to-fetch)
does a great job of explaining the differences in the usage of the two
APIs.

## About `wtfetch`
The `fetch` API was so easy to understand that I was able to
create a wrapper for it and publish it to npm as a library called
`wtfetch`. This library does a few things for the developer:

1. It exposes a API for specific HTTP request type: `get`, `post`,
   `patch`, `deleteRequest` (`delete` is a keyword in Javascript so I
   wanted to avoid the name conflict)
2. It sets the default header to request/send JSON. These headers can be
overridden by the developer.
3. The HTTP request will return a Promise.
3. It automatically `throw`s an error if the request was unsuccessful.
   I think that this is something that should automatically be handled
   in `fetch` but it's not.
4. On successful responses, it returns the parsed response body in the
   promise chain.
5. On `get` requests, it automatically encodes data as the query string
   before making the request.

The only dependency of the library is the `whatwg-fetch` polyfill. Lastly, `wtfetch` is fully tested using Mocha + Chai (woooo!).

## Example Usage
To use the library, here is an example of making a `get` request with
query strings:

{% highlight js %}
// Import or require functions from library
import { get, post, patch, deleteRequest } from 'wtfetch'

// Use the function to make HTTP request
get('www.google.com', { data: {q: 'search term'}})
{% endhighlight %}

More examples can be found on the README of the repo [here](https://github.com/snags88/wtfetch).

## Conclusion
I think this was a good first library since I didn't try to oversolve
an existing problem. I liked that it got me introduced to publishing to
npm. I also liked that it got me thinking about some good practices to follow like tagging release commits on
git and writing more JS specs. I feel like there's a few more libraries
I can pull out of our codebase, so be on the look out for those. But for
now, [here is the library on
npm](https://www.npmjs.com/package/wtfetch).
