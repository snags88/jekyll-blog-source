---
layout: post
title:  "The Prototype Pattern in Laravel"
permalink:  "prototype-pattern-in-laravel"
date:   2018-05-16 16:51:00 -0800
categories: general
---
I'm going to be talking about another creational pattern called the
**prototype pattern** today. This should be pretty straight forward to
implement in a lot of high-level languages and we'll take a look at a
common place we use it in Laravel.

## Prototype Pattern
The prototype pattern is one of the more simple ones to understand. In
this creational pattern, we create a new object by copying an existing
object. Wow that simple?! Yes. That simple.

PHP provides us a nice tool to easily copy an object too! The `clone`
keyword will call the magic function `__clone` on an object and return a
copy of the object.

{% highlight php %}
<?php
$obj = new Object;
$obj2 = $obj;
$obj3 = clone $obj;

/** returns true */
$obj === $obj2;
/** returns false */
$obj === $obj3;
{% endhighlight %}

Interestingly, we don't ever need to implement the `__clone` function if
we just want the default behavior. However, there are times where you
might want to implement custom behavior.

One of those times is if you want to deep clone an object. By default,
PHP will only create a copy of the top level object and if the object
has attributes referencing other objects, we'll just keep a reference to
them in our new object. That's commonly known as shallow copying.
In deep cloning, not only do we want a copy of the
original object, but we also want to get copies of all of its
attributes. In order to get the deep cloning behavior, we'll need to
override the default behavior like this:

{% highlight php %}
<?php
class TestObject
{
  public function __construct()
  {
    $this->objectAttribute = new OtherObject;
  }

  public function __clone()
  {
    $this->objectAttribute = clone $this->objectAttribute;
  }
}
{% endhighlight %}

Yes, we just used `clone` to implement deep cloning.

No, that's not cheating. I think... 😅

## Prototype Pattern in the Wild 🕵️
Since cloning objects is not a common interaction with the Laravel
framework, we'll need to dig a little big deeper in the code to see this
pattern in the wild.

One of the most common places where we see cloning in the Laravel
framework is around dispatching events and jobs in queues. It seems like
when we queue up a job or event, we need to process any arguments passed
to the job/event. Since those arguments could be reused further down in
the code execution, it's much safer for the framework to clone the
argument and mutate the cloned object rather than the original
(immutability anyone?).

Here's some example code from the
[`Illumiate\Events\Dispatcher`](https://github.com/laravel/framework/blob/9cf82b9fa7edf5a66a78e69c91abb685f13ed4a1/src/Illuminate/Events/Dispatcher.php){:target="_blank"}
class:

{% highlight php %}
<?php
  protected function createQueuedHandlerCallable($class, $method)
  {
      return function () use ($class, $method) {
          $arguments = array_map(function ($a) {
              return is_object($a) ? clone $a : $a;
          }, func_get_args());
          if ($this->handlerWantsToBeQueued($class, $arguments)) {
              $this->queueHandler($class, $method, $arguments);
          }
      };
  }
{% endhighlight %}

Wow do I see some object arguments being cloned before we pass it along to the
queue handler? 👀

### Another example in the wild
Another small example I can give is in the commonly used DateTime library
called [Carbon](https://carbon.nesbot.com/docs/). Carbon objects are
mutable so there are times when you do some calculation based on a date.
But in order to do the calculation, you add some date to the original.
Well now you've mutated the original date. To get around that, you need
to `clone` the Carbon object.  Carbon exposes an API called `copy()` to
create clones of the Carbon object. To see the implementation of `copy()` see
[this link](https://github.com/briannesbitt/Carbon/blob/b4d5ba6a2222cd80af1d51694015628bf0f7fed0/src/Carbon/Carbon.php#L937-L945).
This code example is straight from their docs:

{% highlight php %}
<?php
$dt = Carbon::now();
echo $dt->diffInYears($dt->copy()->addYear());

/** $dt was unchanged and still holds the value of Carbon:now() */
{% endhighlight %}

Well I went into a bit of a rabbit hole of learning how `clone` works in
PHP and digging around the Laravel framework code. Hopefully you learned
about the Prototype pattern and its usage. Cloning objects encourages
immutable programming and may lead to avoiding some head-scratching bugs
(speaking from personal experience... looking at you Moment.js 😑). Well
thanks for reading and as always, happy coding!
