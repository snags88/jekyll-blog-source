---
layout: post
title:  "The Adapater Pattern in Laravel"
permalink:  "adapter-pattern-in-laravel"
date:   2018-05-30 17:51:00 -0800
categories: general
---

Now that we've finished the creational patterns, we're moving on to
structural patterns. These patterns focus primarily on the composition
of classes and objects in order to create a scaleable structure for the
application. There are seven total patterns in this family and we'll
take a look one by one. First up: the Adapter pattern.

## Adapter Pattern
The Adapter pattern is often used when we have preexisting code that
needs to be used but doesn't have the expected API that the outside
world expects. You'll run into this type of situation a lot when you're
working with third-party libraries and you need to swap out an old
library for a new one (i.e. switching from Nexmo to Twilio for sending SMS).
The new library may not have the exact same API as the old, so in this
situation, you're left with two choices:

1. Rewrite everything in your app to conform to the library's API.
2. Write an adapter or a wrapper around the library to conform to the
API your app expects.

In general, even if it's your first time using a third-party library, it
helps to write a wrapper around the library in case you ever need to
swap out the library. For instance, you might name the Twilio library
`SmsClientAdapter` to keep it generic. Now, your app doesn't care if the
adapter uses the Twilio library or the Nexmo library.

## Adapter Pattern in the Wild 🕵️
In the world of Laravel, or any large framework for that matter, we are
able to choose the data storage that best fits are app's needs. One of
the storage types in Laravel is Redis, a fast storage solution that can
be used for caching or storing simple key-value pairs.

In PHP, there are two popular clients for Redis ([PhpRedis](https://github.com/phpredis/phpredis){:target="_blank"}
and [Predis](https://github.com/nrk/predis){:target="_blank"}) that
Laravel lets you choose from. After you configure the Redis driver,
Laravel will work its magic and allow you to send commands to your Redis
server.

I wanted to dig into this code a little bit to see if there was some
kind of adapter pattern. We can start at a `RedisManager` object in
`Illuminate\Redis\RedisManager.php`:


{% highlight php %}
<?php
  protected function connector()
  {
      switch ($this->driver) {
          case 'predis':
              return new Connectors\PredisConnector;
          case 'phpredis':
              return new Connectors\PhpRedisConnector;
      }
  }
}
{% endhighlight %}

Here, you can see that we return connector objects based on the driver
we've chosen. These connectors aren't too interesting, but they both
define a `connect` function that will return a connection object based
on which driver you're using:

{% highlight php %}
<?php
// in PhpRedisConnector

public function connect(array $config, array $options)
{
    return new PhpRedisConnection($this->createClient(array_merge(
        $config, $options, Arr::pull($config, 'options', [])
    )));
}

// in PredisConnector

public function connect(array $config, array $options)
{
    $formattedOptions = array_merge(
        ['timeout' => 10.0], $options, Arr::pull($config, 'options', [])
    );
    return new PredisConnection(new Client($config, $formattedOptions));
}
{% endhighlight %}

Next, we gotta look at each of these connections to see how their API
looks to the outside world. Low and behold, each of the connection
classes inherits an abstract class called `Connector` the binds the
connection to a particular API. The `PredisConnection` is not too
interesting to look at, since it mostly defaults to the abstract class
for its functionality. But if you take a look at `PhpRedisConnection`,
we see that a lot of the commonly used functions like `set` and makes
sure that it works propery with the PhpRedis client:

{% highlight php %}
<?php
public function set($key, $value, $expireResolution = null, $expireTTL = null, $flag = null)
{
    return $this->command('set', [
        $key,
        $value,
        $expireResolution ? [$flag, $expireResolution => $expireTTL] : null,
    ]);
}
{% endhighlight %}

We got into the weeds here a bit, but if a third Redis client came into
the PHP world, we could easily write a connector and connection for the
client and make sure that when we want to perform operations on the
Redis server, this adapter will tranform our exact same commands we've
been sending to be compatible with the new client.

We just went through an example with the Redis connection, but this
similar pattern applies to database connections as well. Laravel allows
us to use different relational SQL databases like MySQL, Postgres, and
SQLite. The adapter sits between the database connections and the main
app to allow us to write the same application code regardless of our
underlying SQL database.

Well, I hope that was educational and I encourage the usage of
adapters/wrappers whenever we're working with third-party code. Until
next time, happy coding!
