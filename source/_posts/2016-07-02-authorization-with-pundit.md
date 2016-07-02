---
layout: post
title: "Authorization with Pundit"
permalink: "authorization-with-pundit"
date: 2016-07-02 07:30:00 -0500
categories: web-development
---
In a Rails app, authorization can be implemented in many ways. You can
roll out your own simple authorization by assigning a `role` number
where `role = 1` means regular user, `role = 3` means admin, etc.

The above implementation was how the Learn app originally had
authorization, but since then, our product has grown a lot and we
switched to using a gem called [**Pundit**](https://github.com/elabs/pundit).

A few nice things about Pundit is that it's very light weight and
uses plain old Ruby objects (PORO). Beyond that, how you implement the
authorization is pretty flexible. Pundit also comes with built in helper methods so you can authorize your users easily.

## Configuring Pundit
The first step is to include the `pundit` gem in your `Gemfile`.
Afterwards, you need to include the `Pundit` module in your
`ApplicationController` so all that authorization goodness is available
in your app.

{% highlight ruby %}
class ApplicationController < ActionController::Base
  include Pundit
  ...
end
{% endhighlight %}

Next, we need to setup a "policies" folder so go ahead and create one in
`app/policies`. It really doesn't matter where we put the policies, but
this is the place that's suggested. More importantly, what are policies...

## Policy Objects
`Pundit` uses what is called "Policy Objects" to determine a particular
user's authorization on the specific object. For instance, let's say we
have a resource in our app called `Organization`. In order to check a
user's authorization for an organization, pundit will look for an
`OrganizationPolicy`.

So you can imagine the `OrganizationPolicy` looks something like this:

{% highlight ruby %}
class OrganizationPolicy
  attr_reader :user, :organization

  def initialize(user, organization)
    @user = user
    @organization = organization
  end

  def show?
    #=> some code here to determine if a user can see an organization.
  end
end
{% endhighlight %}

Like I mentioned earlier, this is a PORO and doesn't depend on any
inheritance. But you know what might be good is if we create an
`ApplicationPolicy` that takes care of some of the boilerplate stuff
like the `initialize` method. That way all of our policy objects can
inherit from it:

{% highlight ruby %}
class ApplicationPolicy
  attr_reader :user, :record

  def initialize(user, record)
    @user = user
    @record = record
  end
end

class OrganizationPolicy < ApplicationPolicy
  def show?
    #=> some code here to determine if a user can see an organization.
  end
end
{% endhighlight %}

## Determining Authorization
I also mentioned earlier that you can implement your authorization logic
how you see fit. So there's no hard fast rule on the implementation, but
to give you an idea, there might be a polymorphic table in our database
that determines the authorization level. For instance you might have
something like:

{% highlight ruby %}
org = Organization.create(name: 'My Organization')
user = User.create(username: 'TestUser')
role = Role.create(name: 'admin')

UserRole.create(roleable: org, user: user, role: role)
{% endhighlight %}

Don't worry too much about where all those `ActiveRecord` models came from.
What we really care about is that there is now a `UserRole` that ties
the user, organization, and a role. Now for our `OrganizationPolicy` we
can do something like:

{% highlight ruby %}
class OrganizationPolicy < ApplicationPolicy
  def show?
    has_role_for_record?(:admin)
  end

  private

  def has_role_for_record?(role_name)
    !!UserRole.find_by(user: user, roleable: record, role:
    Role.find_by(name: role_name))
  end
end
{% endhighlight %}

Now, the `#show?` method returns a boolean value based on if the user
has the proper record in the `UserRole` model.

## Authorizing a User
Ok we set up all the policy objects and the proper authorization logic.
Now how do we authorize a particular user?

At the core of it, you actually don't even need the helper methods
provided by `Pundit`. In your controller, you can easily do:

{% highlight ruby %}
class OrganizationController < ApplicationController

  def show
    # current_user is logged in user
    organization = Organization.find_by(name: 'My Organization')

    unless OrganizationPolicy.new(current_user, organization).show?
      redirect_to root_path
    end

    # show the organization page.
  end
end
{% endhighlight %}

But that's meh. You can use a helper method called 'authorize' provided by `Pundit` to make
it more sexy. `authorize` takes in the record object you want to
authorize and an optional method name that should be called on the
policy object:

{% highlight ruby %}
class OrganizationController < ApplicationController

  def show
    # current_user is logged in user
    organization = Organization.find_by(name: 'My Organization')

    authorize organization, :show?

    # show the organization page.
  end
end
{% endhighlight %}

If the user is not authorized, `Pundit` will raise `Pundit::NotAuthorizedError` that you can rescue in your `ApplicationController`.
Another thing that the `authorize` method does is that if you don't pass
in the method name, it infers it by the controller action name and
tacking on a question mark on it.


{% highlight ruby %}
class OrganizationController < ApplicationController

  def show
    # current_user is logged in user
    organization = Organization.find_by(name: 'My Organization')

    authorize organization
    # this is running OrganizationPolicy.new(current_user, organization).show?
    # because we're in the `show` controller action.

    # show the organization page.
  end
end
{% endhighlight %}

What's cool is that we can use the policy objects without the helper
methods so they can be used to authorize users in our API endpoints, our
serialized objects, and so on.

There's much more you can do with Pundit like change the scope for a
resource based on the user's authorization, custom error messages,
headless policy objects, etc. I highly recommend checking out [this blog
post](https://www.varvet.com/blog/simple-authorization-in-ruby-on-rails-apps/),
written by one of team members behind `Pundit`.
