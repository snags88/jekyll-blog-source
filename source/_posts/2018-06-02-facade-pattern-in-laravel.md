---
layout: post
title:  "The Facade Pattern in Laravel"
permalink:  "facade-pattern-in-laravel"
date:   2018-06-02 12:51:00 -0800
categories: general
---

Next up on the structural pattern is the _Facade_ pattern. Supposedly,
it derives its name from the word facade, which (according to Google) means:

1. the face of a building, especially the principal front that looks onto a street or open space.
2. an outward appearance that is maintained to conceal a less pleasant or creditable reality.

I'm a fan of definition #2 there because it really describes the purpose
of the facade pattern in layman's terms.

## Facade Pattern
Understandably, the facade pattern is often confused with the adapter
pattern because you're using a class to present a less complex API to
the outside word. However, I think one of the biggest differences is
that the facade pattern can actually encapsulate many classes that are
coordinating together to accomplish a complex task.

For example, we could start with an interface that enforces the
facade:

{% highlight php %}
<?php
interface FastFoodFacade
{
    public function orderComboOne();
    public function orderComboTwo();
}
{% endhighlight %}

Then we might have classes that implements the interface that
coordinates various classes:

{% highlight php %}
<?php
class McDonalds implements FastFoodFacade
{
    public function orderComboOne()
    {
        $food = (new McDouble())->withCheese();
        $drink = (new Drink())->size('large');
        $fries = (new Fries())->size('large');

        return [
            'food' => $food,
            'drink' => $drink,
            'extra' => [$fries]
        ];
    }

    public function orderComboTwo()
    {
        $food = (new McNuggets())->count(10);
        $drink = (new Drink())->size('medium');
        $sauce = (new Sauce('Bbq'));
        $sauce2 = (new Sauce('honeyMustard'));

        return [
            'food' => $food,
            'drink' => $drink,
            'extra' => [$sauce, $sauce2]
        ];
    }
}
{% endhighlight %}

We could extend this pattern to another class like `Arbys` and implement
the same type of facade. When the outside world is interacting with any
fastfood class, we know that we can order combo meal #1 or combo meal
#2.

## Facade Pattern in the Wild 🕵️
If you've been using Laravel for sometime, you probably noticed things
called Facades exist in Laravel. They are in
`Illuminate\Support\Facades\..` and many of the common tools in the
framework like `Queue`, `Job`, and `Event` are called through these
facades.

I'v been using these facades, but didn't really dig into them until I
started writing this and here's how they work.

All facades inherit from the abstract facade class in
[`Illuminate\Support\Facades\Facade`](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Support/Facades/Facade.php){:target="_blank"}.
That particular abstract class implements a lot of test helpers that
allow you to mock out the facade like `shouldReceive` or `spy`. The
other important part is that it also defines a delegation function for
any missing static functions called on the class:

{% highlight php %}
<?php
public static function __callStatic($method, $args)
{
    $instance = static::getFacadeRoot();
    if (! $instance) {
        throw new RuntimeException('A facade root has not been set.');
    }
    return $instance->$method(...$args);
}
{% endhighlight %}

For any missing static functions, the above function will try to
forward the method to the instance to whatever is being facaded (is that
even a word? 😂).

Finally, the concrete facade classes implements the abstract class and
needs to define one method at the minimum called `getFacadeAccessor`.
That function should return the string name of the object as it has been
registered in the service provider.

Lets take a look at the `Queue` facade for example. This is the implementation
of `getFacadeAccessor`:

{% highlight php %}
<?php
protected static function getFacadeAccessor()
{
    return 'queue';
}
{% endhighlight %}

And in `QueueServiceProvider` we see that that the `QueueManager` gets
registered to the app container as `queue`:

{% highlight php %}
<?php
protected function registerManager()
{
    $this->app->singleton('queue', function ($app) {
        return tap(new QueueManager($app), function ($manager) {
            $this->registerConnectors($manager);
        });
    });
}
{% endhighlight %}

What's really cool is that you can build onto this pattern and create
your own facades in your app or as a library. One of the libraries we
use called
[Bugsnag](https://docs.bugsnag.com/platforms/php/laravel/){:target="_blank"}
implements its own facade so we can easily use its functionality without
having to reach into the app container all the time.

Apparently some people don't consider this to be a real facade pattern,
but I feel like it does coordinate complex interactions behind the
scenes so that we can easily use the tools that Laravel provides for us.
There no need to resolve the queue manager every time we want to use it.

Well hopefully that gave some insight to the facade pattern in OOP and
the facade concept in Laravel. Until next time, happy coding! 😎
