---
layout: post
title:  "The Builder Pattern in Laravel"
permalink:  "builder-pattern-in-laravel"
date:   2018-05-14 16:51:00 -0800
categories: general
---
Wow I should have waited another 10 days so I could have written a new
blog post on the 1 year anniversary of not writing in my blog... jk. I
also like how my last post was about trying to learn Python and now I'm
talking about PHP. I guess a quick catch up of what happened over the
last year:

- I left Flatiron School last May.
- I started at Curology in July where I work in the Laravel framework (PHP) and
  React + MobX on the front-end.
- Also just attended a conference last week and realized I've been
  slacking off since I started at Curology hence the sudden revival of
the blog.
- Between July and last week, I've been on a quest to find the holy
grail and fight off vicious dragons... where holy grail = PHP/Laravel
knowledge and vicious dragons = random bugs that I probably caused
myself.

In any case, felt like I should get back into the good ol' technical
writing. And since I already know that I'm going to have trouble
writing consistently, I decided to choose a theme to provide me
blogging topics for the next few weeks (months? 😱). My
writing topic will be...

**DRUMROLL**

to find each of the [Design Patterns from the Gang of Four](https://en.wikipedia.org/wiki/Design_Patterns){:target="_blank"} in the Laravel ecosystem.

So without further ado, here we gooooooo!

## Builder Pattern
The first pattern I'll talk about is probably one of my favorites and
it's called the builder pattern.
By definition, the builder pattern is a creational
pattern, where the whole idea is that we're going to be creating a new
object that fits our needs. In our code, we'll just call methods on the
builder to end up with the exact object we want. To see more of the
technical definition, you can visit the [wikipedia page
here](https://en.wikipedia.org/wiki/Builder_pattern){:target="_blank"}.

As pseudo code, it might look something like this:
{% highlight php %}
<?php
$train = (new TrainBuilder)
    ->addEngines(3)
    ->addPassengers(2)
    ->addLuggageCars(1)
    ->getTrain();
{% endhighlight %}

We have a builder where we can mutate what we're
building by using its public API.

LOOK AT HOW FLUENT THE CODE IS. LOOK AT IT 🤩

So simple to read and abstracts away what's happening so we don't care
how the passangers are being added. Just that they are added to the
train! The implementation of the builder might look something like this:

{% highlight php %}
<?php
class TrainBuilder
{
    private $train;

    public function __construct()
    {
        $this->train = new Train
    }

    public function addEngines(int $engines)
    {
        $this->train->engines = $engines;
        return $this;
    }

    public function addPassengers(int $passengers)
    {
        for($i = 0; $i < $passengers; $i++) {
            $this->train->addPassenger(new Passenger);
        }
        return $this;
    }

    public function addLuggageCars(int $cars)
    {
        $this->train->luggageCars = $cars;
        return $this;
    }

    public function getTrain()
    {
        return $this->train;
    }
}
{% endhighlight %}

Notice the process of adding passangers is different, but as I mentioned
above, its abstracted away from the rest of the world and the builder object
has a nice API that's uniform and predictable. Another thing to note is
that we're returning `$this` on the methods so that it's chainable.
I'm pretty sure there's a fancy term for it, but I'll just call it chainable for now.

## Builder Pattern in the Wild 🕵️
Alright, this is hella cheating because we don't have to look much
further than Laravel to find the builder pattern in action. Laravel's
ORM, _Eloquent_, uses the builder pattern in... yup you guessed it, the
query builder.

The query builder is the object that gets returned anytime you're doing
a query on an Eloquent Model like this:

{% highlight php %}
<?php
$users = User::where('id', '<', 100)
    ->orderBy('created_at', 'desc')
    ->limit(20);
{% endhighlight %}

And as the name suggests, you can continue to build queries like
complex joins or conditional wheres. Once you're done building your
query, you can call any of the fetching methods like `get()` or
`first()` to run the query against the database and get a collection or
single record.

It's actually a pretty common pattern for ORMs to have chainable methods
to build a query. You can see ActiveRecord in Rails also use this pattern
in their [documentation](http://guides.rubyonrails.org/active_record_querying.html#understanding-the-method-chaining){:target="_blank"}.

If we take a look at the implementation of the `QueryBuilder`methods used
in the example code, we can see that it is similar to our `TrainBuilder`
example. It returns `$this` and mutates its own properties:

{% highlight php %}
<?php
// code from https://github.com/laravel/framework/blob/f91e049f9c7b97e0e85d56961d1eeae82a1815a2/src/Illuminate/Database/Query/Builder.php

class Builder
{
    // A bunch of other Builder function implementations

    public function where($column, $operator = null, $value = null, $boolean = 'and')
    {
        // Some code handling specific conditions and

        // delegating to helper functions


        $type = 'Basic';
        $this->wheres[] = compact(
            'type', 'column', 'operator', 'value', 'boolean'
        );
        if (! $value instanceof Expression) {
            $this->addBinding($value, 'where');
        }
        return $this;
    }

    public function orderBy($column, $direction = 'asc')
    {
        $this->{$this->unions ? 'unionOrders' : 'orders'}[] = [
            'column' => $column,
            'direction' => strtolower($direction) == 'asc' ? 'asc' : 'desc',
        ];
        return $this;
    }

    public function limit($value)
    {
        $property = $this->unions ? 'unionLimit' : 'limit';
        if ($value >= 0) {
            $this->$property = $value;
        }
        return $this;
    }
}
{% endhighlight %}

Alright did I blow your mind yet? I hope so because I have nothing else
left to show you 😂. I hope this example of a builder pattern in the wild
shows you the usage and implementation. The builder pattern isn't for
every occasion, but when you have something that needs to be built
differently based on the context of the code, it's a pretty good sign
to reach for this pattern.

At Curology, we recently implemented the builder pattern to handle
searching for our bottle shipments from the front end. Based on
different queries passed in by the admin, we built the shipment query
properly. That builder is now reusable across multiple locations in our
app.

Anyway, thanks for reading and happy coding!
