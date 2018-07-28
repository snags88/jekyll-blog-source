---
layout: post
title:  "Backoff Strategy for Laravel Jobs"
permalink:  "backoff-strategy-for-laravel-jobs"
date:   2018-07-27 17:51:00 -0800
categories: general
---

## What are jobs?
Background jobs are key to any seemingly fast application. They're
really useful for running tasks asynchronously so that your user isn't
waiting for some code to run that doesn't directly impact their
experience in the app.

One small example of using a background job is when a user logs in to
our app, we want to update their `last_logged_in_at` timestamp in our
database. From the user's perspective, do they really care about this
timestamp? Nope, not really. This would be a perfect time to use a
background job to do a database write operation.

## How do jobs work?
When these jobs are initially fired off, they are put onto the
application's queue. The queue could be anything as long as it tracks
the jobs' information. Some are backed by the application's database,
Redis, RabbitMQ, etc.

Once the jobs are on the queue, there is a worker process that will grab
jobs off of the queue and execute the job's code.

## Configuring jobs in Laravel
Laravel provides background jobs for us out of the box. All you need to
do is create a plain old class, add some traits, implement an interface,
and dispatch it from your main app code.

For more information about managing jobs in Laravel, take a look at
their [detailed documentation](https://laravel.com/docs/5.6/queues#creating-jobs){:target="_blank"}.

## Backoff strategy?
In modern applications when an issue occurs, there is an attempt to
retry the block of code that caused the issue. For example, if you are
on GMail and you lose internet connection, the website will attempt to
reconnect to the Google servers. What you might notice is that on the
first try, it will attempt to reconnect immediately. Then it'll wait a
few seconds and try again. Then wait 10-15 second and try again. Then
another 30-40 second wait, etc. What you experienced is called a backoff
strategy for reattempting a failed task.

As you can imagine, with queued jobs, it would be great to adopt this
strategy especially for jobs that deal with third-party APIs. If the
third-party is down, then it may take some time for it to come back up
so it's beneficial to have a backoff strategy.

Sadly, Laravel does not have a backoff strategy built into their jobs.
However, with the failed job handling it exposes to us, we can write our
own retriable job easily. The particular backoff strategy we'll
implement is called an exponential backoff strategy because we increase
our backoff exponentially (i.e. 2, 4, 8, 16, 32, etc.).

{% highlight php %}
<?php
namespace App\Jobs;

abstract class RetriableJob
{
    public $tries = 1; // override default and make sure we don't requeue automatically

    public $currentRetryCount = 1;
    public $maxRetries = 7; // 3, 9, 27, 81, 243, 729, 2187 seconds

    public $backoffFactor = 3;

    public function failed(\Exception $e)
    {
        if ($this->currentRetryCount <= $this->maxRetries) {
            $this->delay(now()->addSeconds($this->backoffFactor ** $this->currentRetryCount));
            $this->currentRetryCount += 1;

            return dispatch($this);
        }
    }
}
{% endhighlight %}

The implementation above uses the `failed` hook to update the job's
traits `delay` and `currentRetryCount`. Afterwards, it redispatches the
job onto the queue.

To use this, you just need to inherit the abstract class to your job:

{% highlight php %}
<?php
namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;

class MyJob extends RetriableJob implements ShouldQueue
{
    public function handle() {
        // Do my job

    }
}
{% endhighlight %}

## Future Improvements
This isn't the cleanest implementation since we try a job once, and then
we just requeue it. To do this correctly, we actually need to dive into the
framework itself and update the code. Ideally the code would be changed
so that when the worker processes the job and it has multiple `tries`,
it would allow use to specify a `delay` time before we release it back
into the queue.

In any case, the implementation is easy to understand and shouldn't have
any performance impact on the worker queue. I couldn't really find a
good example of this online when I needed to implement it, so hopefully
this helps someone in the future.
