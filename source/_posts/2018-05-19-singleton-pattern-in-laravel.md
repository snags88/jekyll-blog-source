---
layout: post
title:  "The Singleton Pattern in Laravel"
permalink:  "singleton-pattern-in-laravel"
date:   2018-05-19 9:51:00 -0800
categories: general
---

Alright, next up in the creational design patterns is the singleton
pattern. Its pretty easy to remember this one. The singleton pattern is
when a class has a single instance of itself. Meaning that whenver I
want to use an instance of the class, I'm using the same instance.

## Singleton Pattern
The implementation of the singleton pattern varies based on the
language. I remember for Ruby, there's a built in `Singleton` module
that you can just mixin to a class and it'll become a singleton.

For PHP, it's a little bit more complicated. Digging around, I found
that the typical way to implement the singleton pattern is to create a
singleton trait (basically a Ruby module) that can be used by any class
that wants to be a singleton.

The singleton trait might look something like this:

{% highlight php %}
<?php
trait Singleton {
   protected static $instance = null;

  /** call this method to get instance */
   public static function instance(){
      if (static::$instance === null){
         static::$instance = new static();
      }
      return static::$instance;
  }

  /** protected to prevent cloning */
  protected function __clone(){
  }

  /** protected to prevent instantiation from outside of the class */
  protected function __construct(){
  }
}
{% endhighlight %}

Most notably, the singleton trait implements the constructor and clone
function as protected. It also uses a static variable to hold the
single instance of the class. The only way to get an instance of this
object is by using the static `instance` method: `MyClass::instance()`.

So why all the fuss about singletons? Its useful for when a class has
some configuration that it registers and never changes for the duration
of its existance. For example, if I have a class that talks to GitHub's
API, I probably designed it so that you can pass in configurations to
its constructor like my API key and version. The rest of the interface
on the client object would just accept paramters to make API calls and
return the response. Thus, it's unnecessary to instantiate multiple copies
of this class.

One downside to keep in mind is that these instances are held in memory so
they will never get garbage collected. If you have a ton of singletons,
you might be holding on to a lot of unnecessary memory space.

## Singleton Pattern in the Wild 🕵️
In the world of Laravel, there is a concept of Service Containers. You
can read more about them in their [official documentation](https://laravel.com/docs/5.6/container){:target="_blank"}.

The tl;dr of Service Containers is that at the application's boot, you
can register and bind various classes to the service container (via
service providers) to be resolved elsewhere in your app. For example,
one of the things you can do is register a class and have it injected
into other places like your controller's constructor or actions. You
just type hint the class you want and BAM! the registered class is
injected into your function. Dependency injection FTW:

{% highlight php %}
<?php
// binding a class to the service container

$this->app->bind(MyAwesomeClass::class, function ($app) {
    return new MyAwesomeClass();
});


// in a controller somewhere in your app...

public function show(MyAwesomeClass $awesomeThing)
{
  // ...

}
{% endhighlight %}

One thing to note about the implementation of the service container in
Laravel is that it uses a `Container` singleton object to keep track of
all the bindings/instances that have been registered/resolved for the app.
This actually makes a lot of sense because we have 1 app so we should
have 1 container for the app. In fact, the `Container` singleton object
implements the singleton pattern we wrote out in the previous section.
You can see it in the source code [here](https://github.com/laravel/framework/blob/b8e2cd03663bad9cf91342b641ca96573b06dd0f/src/Illuminate/Container/Container.php#L1171-L1178){:target="_blank"}

Funny thing is that I actually set out write about how you can bind
singletons to Laravel's service container, but while I was digging
through the `Container` object in the framework's source code, I
found that it uses the singleton pattern. That's pretty neat that I was able
to recognize the pattern now that I've seen it written in PHP. Well,
that's all I have for now. Happy coding!
