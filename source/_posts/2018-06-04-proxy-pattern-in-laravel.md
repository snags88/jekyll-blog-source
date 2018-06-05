---
layout: post
title:  "The Proxy Pattern in Laravel"
permalink:  "proxy-pattern-in-laravel"
date:   2018-06-04 17:51:00 -0800
categories: general
---

Proxy is one of those words that makes you sound technically smart and
it's seen an increase in usage over the last 20 years
according to [Google Books Ngram Viewer](https://books.google.com/ngrams/graph?content=proxy&case_insensitive=on&year_start=1940&year_end=2018&corpus=15&smoothing=7&share=&direct_url=t4%3B%2Cproxy%3B%2Cc0%3B%2Cs0%3B%3Bproxy%3B%2Cc0%3B%3BProxy%3B%2Cc0%3B%3BPROXY%3B%2Cc0#t4%3B%2Cproxy%3B%2Cc0%3B%2Cs0%3B%3Bproxy%3B%2Cc1%3B%3BProxy%3B%2Cc0%3B%3BPROXY%3B%2Cc0){:target="_blank"}.

Well what does proxy even mean? Lets take a look in more detail.

## Proxy Pattern
The word _proxy_ often means something that replaces
something else and behaves similar to the original object. An example of
a proxy is when you give the authority to someone to vote on your
behalf; it's called proxy voting.

The proxy pattern in OOP follows a similar concept in that a proxy object
stands in place of a real object. That proxy object generally forwards
the instuctions on to the real object, but could add additional
functionality as needed.

The simplest implementation of a proxy object just uses the magic
`__call` function to dynamically pass along missing methods to the real
object:


{% highlight php %}
<?php
class Proxy
{
  public $obj;

  public function __construct($obj)
  {
    $this->obj = $obj;
  }

  public function __call($method, $parameters)
  {
    return $this->obj->{$method}(...$parameters);
  }
}
{% endhighlight %}

Well that's easy enough in PHP. Gotta love that syntactic sugar.

## Proxy Pattern in the Wild 🕵️
There are various uses for the proxy pattern. People give examples of
lazy loading files to authorizing method calls in proxy objects.
Interestingly, I found a place in the Laravel framework that uses the
proxy pattern for neither of these purposes.

Remember that time Taylor Otwell wrote about the `tap` function that was
inspired by the same named method in Ruby? Well [here](https://medium.com/@taylorotwell/tap-tap-tap-1fc6fc1f93a6){:target="_blank"} is his original post. In it, he describes two ways to 
use the `tap` helper function. The first way has 2 arguments:

{% highlight php %}
<?php
tap($object, function ($object) {
    $object->save();
});
{% endhighlight %}

The second passes along only 1 argument, then chains a method, and he
claims that it returns the original object:

{% highlight php %}
<?php
return tap($user)->update([
    'name' => $name,
    'age' => $age,
]);
{% endhighlight %}

I was digging around and found where the `tap` helper function is
defined and this is what the code looks like:

{% highlight php %}
<?php
if (! function_exists('tap')) {
    // Call the given Closure with the given value then return the value.

    function tap($value, $callback = null)
    {
        if (is_null($callback)) {
            return new HigherOrderTapProxy($value);
        }
        $callback($value);
        return $value;
    }
}
{% endhighlight %}

So the `if` block handles when it's 1 argument and the remaining code
handles when there's 2 arguments. Interesting. So what is this
`HigherOrderTapProxy` that gets returned instead of the original object?
Let's take a look:

{% highlight php %}
<?php
namespace Illuminate\Support;
class HigherOrderTapProxy
{
    public $target;

    public function __construct($target)
    {
        $this->target = $target;
    }

    public function __call($method, $parameters)
    {
        $this->target->{$method}(...$parameters);
        return $this->target;
    }
}
{% endhighlight %}

Ahh I see. So this proxy object just intercepts the method call
on the object, calls the method with the parameters, but ignores the
return value of that method call, and then always returns the "target"
_aka_ the original object.

BAM proxy pattern in real life. Definitely did not think I would find
one in Laravel that would be easy to understand, but there you have it!
Until next time, happy coding!
