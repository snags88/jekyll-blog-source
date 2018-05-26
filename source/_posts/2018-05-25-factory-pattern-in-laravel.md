---
layout: post
title:  "The Factory Pattern in Laravel"
permalink:  "factory-pattern-in-laravel"
date:   2018-05-25 17:51:00 -0800
categories: general
---

I took a little break this past week, but back on it for the last
creational pattern. Well, there's technically two more, but they're very
similar so I'm going to combine them into one post.

## Factory Pattern
The factory pattern and the abstract factory patterns are similar in
that they create objects like a factory by exposing some kind of
interface to the outside world like `createXYZ`.

Here is a short example:

{% highlight php %}
<?php
class PhoneFactory
{
  public createPhone()
  {
    return new \Phone();
  }
}

$phoneFactory = new PhoneFactory();
$phoneA = $phoneFactory->createPhone();
$phoneB = $phoneFactory->createPhone();
{% endhighlight %}

## Abstract Factory Pattern
The main difference between the factory and abstract factory  is that
the abstract factory pattern is used as a contract to produce objects
of a similar group. So for instance:

{% highlight php %}
<?php
interface PhoneFactory
{
  public createPhone();
}

class AndroidFactory implements PhoneFactory
{
  public createPhone()
  {
    return new \AndroidPhone();
  }
}

class IPhoneFactory implements PhoneFactory
{
  public createPhone()
  {
    return new \IPhone();
  }
}

if ($user->wantsIPhone()) {
  $factory = new IPhoneFactory;
} else {
  $factory = new AndroidFactory;
}

$phone = factory->createPhone();
{% endhighlight %}

The key with the abstract factory pattern is that the outside world
doesn't care which factory is being used as long as it inherits or
implements the interface of the abstract factory.

## Factory Pattern in the Wild 🕵️
In the world of Laravel, we use the `factory` helper function to create
Eloquent models in our tests. The factory helper is used like this `$user =
factory(\App\Models\User::class)->create();`.

I dug into the source code a little bit and found that under the hood,
it uses an object called `EloquentFactory`.

{% highlight php %}
<?php
function factory()
{
  $factory = app(EloquentFactory::class);
  $arguments = func_get_args();

  if (isset($arguments[1]) && is_string($arguments[1])) {
    return $factory->of($arguments[0], $arguments[1])->times($arguments[2] ?? null);
  } elseif (isset($arguments[1])) {
    return $factory->of($arguments[0])->times($arguments[1]);
  }

  return $factory->of($arguments[0]);
}
{% endhighlight %}

Ok, next question: What is this `of` function called on the `EloquentFactory`?
Well, it uses another class called `FactoryBuilder` where we pass along the name of
the class we want to create from the factory.

{% highlight php %}
<?php
public function of($class, $name = 'default')
{
  return new FactoryBuilder(
    $class, $name, $this->definitions, $this->states,
    $this->afterMaking, $this->afterCreating, $this->faker
  );
}
{% endhighlight %}

Digging more into the `FactoryBuilder` code, I traced the commonly used
`create` and `make` functions to a function called `makeInstance`.
Here's what `makeInstance` looks like:

{% highlight php %}
<?php
protected function makeInstance(array $attributes = [])
{
  return Model::unguarded(function () use ($attributes) {
    $instance = new $this->class(
      $this->getRawAttributes($attributes)
    );
    if (isset($this->connection)) {
      $instance->setConnection($this->connection);
    }
    return $instance;
  });
}
{% endhighlight %}

Ah, finally we find the factory pattern with some meta-programming
involved. In the contructor of the object, we passed in the class name
and stored it to `$this->class`. PHP allows you to hold the names of
class, function, and variables in another variable to be used later. In
the example here, we see that the string held in `$this->class` is used
to instantiate a new object by simply doing `new $this->class()`. Whoa.

Well, hopefully that gave you some insight into what the factory pattern
is and how it's used in Laravel's factories to help you create instances
of Eloquent models. Until next time, happy coding!
