---
layout: post
title: "Implementing Faraday Retry in Rails"
permalink: "implementing-faraday-retry-in-rails"
date: 2016-06-13 07:30:00 -0500
categories: web-development
---
Yesterday, I had the chance to tackle an interesting problem at work. We
use a lot of 3rd party APIs ranging from [Slack](https://slack.com/), [Github](https://github.com/), to [Ambassador](https://www.getambassador.com/) and one potential issue is that sometimes these services timeout for whatever reason (API is down or slow).

A common way to address `Timeout` errors is to have a retry system with an exponential backoff. Simply put, if you disconnect your internet while you're on your Gmail, it will have a message that says __Reconnecting in 1s...__ and if it fails to reconnect, the new message will say __Reconnecting in 2s...__, __Reconnecting in 4s...__, __Reconnecting in 8s...__. The algorithm of the backoff depends on the system, but you get the idea.

Most of our API wrappers use [Faraday](https://github.com/lostisland/faraday) to make HTTP requests. One benefit to using Faraday is that it uses middleware and luckily Faraday already has a built in retry middleware. You can see it in their source code [here](https://github.com/lostisland/faraday/blob/master/lib/faraday/request/retry.rb). We just need to add it to our list of middleware. For more info about middleware, here are some good resources: [Railscast](http://railscasts.com/episodes/151-rack-middleware?view=asciicast), [Blog](https://www.amberbit.com/blog/2011/07/13/introduction-to-rack-middleware/), [Another Blog](https://www.mobomo.com/2012/3/faraday-one-http-client-to-rule-them-all/).

For this particular post, I'm going to use an example with the
[Octokit](https://github.com/octokit/octokit.rb)
gem, a wrapper to the Github API. Octokit uses Sawyer/Faraday to make
HTTP requests so we just need to override the default Octokit middleware as seen in the source code [here](https://github.com/octokit/octokit.rb/blob/22846aec1140dac89945c8908ce62777bec37271/lib/octokit/default.rb#L26-L32).
If you are using a Rails app like me, then you just need to stick the
following code in an initializer file so that the Octokit defaults are
overridden before any instances are created during code execution.

{% highlight ruby %}
Octokit.middleware = Faraday::RackBuilder.new do |builder|
  # add the faraday retry middleware
  builder.use Faraday::Request::Retry

  # add the default Octokit middlewares
  builder.use Octokit::Middleware::FollowRedirects
  builder.use Octokit::Response::RaiseError
  builder.use Octokit::Response::FeedParser
  builder.adapter Faraday.default_adapter
end
{% endhighlight %}

Note that in the above example, I use the [default retry settings](https://github.com/lostisland/faraday/blob/master/lib/faraday/request/retry.rb#L74-L96). You
will be able to set your own configurations by following the code
sample [here](https://github.com/lostisland/faraday/blob/master/lib/faraday/request/retry.rb#L11-L16).

Sweet, now that we have that middleware in place, how do we know this is actually working?

**By writing a spec of course.**

The tools we'll be using are [Rspec](http://rspec.info/) and [Webmock](https://github.com/bblimke/webmock). The spec will be pretty straight forward: we'll use webmock to throw a `Timeout` error on the initial API request and a legit response on the second request. The expected behavior is that the webmock stub is requested exactly twice since that is the default number of retries from the Faraday middleware.

{% highlight ruby %}
it 'retries on a timeout exception' do
  # mock out the api request with responses
  stubbed_request = stub_request(:any, /github/).
    to_timeout.then.
    to_return({body: 'abc'})

  # OCTOKIT here is an authenticated Octokit client
  OCTOKIT.user

  # Provide expected behavior
  expect(stubbed_request).to have_been_requested.twice
end
{% endhighlight %}

This implementation can be applied to other API wrappers using Faraday. Surprisingly, I couldn't find a lot of resources online about the Faraday retry middleware so hopefully this helps someone in the future.

Also, big kudos to [this SO post](https://stackoverflow.com/questions/25552239/webmock-simulate-failing-api-no-internet-timeout) for providing a starting point and my co-worker Spencer for guidance on the implementation.
