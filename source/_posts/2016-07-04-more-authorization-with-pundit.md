---
layout: post
title: "More Authorization with Pundit"
permalink: "more-authorization-with-pundit"
date: 2016-07-04 07:30:00 -0500
categories: web-development
---

Ok the whole reason why I wrote my [previous post on authorization with Pundit]({% post_url 2016-07-02-authorization-with-pundit %}) was so that I could talk about more advanced usages of Pundit in our application.

The Learn app started as a student facing curriculum management system
with very simple admin interfaces. The admin tasks included adding
students to certain batches and deploying curriculum out to them.
However as our use cases grew, the complexity of the admin tasks grew
as well. Until finally, we ended up with two large admin apps, the Organizations
app and the Curriculum app (among other small admin UIs).

We now have a third admin app in development and, though it's tied to
the students, it no longer has the concept of student's batches. This
poses a new kind of challenge since we haven't been doing
authorization by apps. We've been inferring the authorization of an app
by the combination of the user's role and the object they have the role
for.

With that in mind, let's talk about a solution to handle
app specific authorization.

## Creating the concept of sub-apps
Since our current implementation requires us to associate a user + role
+ object, we need to create the concept of sub-apps. The way that I
addressed this was to create a new table in our Rails app called
`sub_apps` and an associated model.

{% highlight ruby %}
# Migration
class AddSubAppTable < ActiveRecord::Migration
  def change
    create_table :sub_apps do |t|
      t.string :name
      t.text :description

      t.timestamps
    end
  end
end

# Model
class LearnApp < ActiveRecord::Base
end
{% endhighlight %}

## Create user roles for the sub-apps
We now have all three parts of user + role + object (sub-app), so we can
go forward with creating some user roles. For brevity, we'll only focus
on one of the apps. I just threw this into my `rakefile`:

{% highlight ruby %}
namespace :sub_app_roles do
  task seed_curriculum_app: :environment do
    curriculum_app = SubApp.find_or_create_by(name: 'Curriculum App', description: 'Helps deploy stuff to students')
    admin_role = Role.admin
    deployer_role = Role.deployer

    UserRole.find_or_create_by(user: User.find_by(username: 'AdminUser'), roleable: curriculum_app, role: admin_role)
    UserRole.find_or_create_by(user: User.find_by(username: 'DeployerUser'), roleable: curriculum_app, role: deployer_role)
  end
end
{% endhighlight %}

Running the above task should build an admin user role for `AdminUser`
and a deployer role for `AdminUser`, both for the curriculum app.

## Configuring our policy object
Next, we'll want to configure a policy object for our `SubApp` objects
called `SubAppPolicy`. In `app/policies/` create a file called
`sub_app_policy.rb` and create some general authorization rules for the
sub apps:

{% highlight ruby %}
# app/policies/sub_app_policy.rb

class SubAppPolicy < ApplicationPolicy

  def general_access?
    !!user.user_role_for(record)
  end

  def admin_view?
    user.has_role?(record, :admin)
  end

  def send_invitations?
    user.has_role?(record, :admin)
  end
end
{% endhighlight %}

Remember from the previous post about `Pundit` that the policy objects take in
a user and a record object. Each of the instance methods defined can be
used in the controller to authorize a particular action:

{% highlight ruby %}
# app/controllers/curriculum_controller.rb

class CurriculumController < ApplicationController
  def index
    curriculum_app = SubApp.find_by(name: 'Curriculum App')
    authorize curriculum_app, :general_access?
  end
end
{% endhighlight %}

Great, now we can use the `SubAppPolicy` to authorize users. So another
example might be authorizing a user to deploy curriculum out to students:

{% highlight ruby %}
# app/policies/sub_app_policy.rb

class SubAppPolicy < ApplicationPolicy
  # ...

  def deploy_access?
    user.has_role?(record, :deployer)
  end

  # ...
end

# app/controllers/deployment_controller.rb

class DeploymentController < ApplicationController
  def create
    curriculum_app = SubApp.find_by(name: 'Curriculum App')
    authorize curriculum_app, :deploy_access?
  end
end
{% endhighlight %}

Hmm... that particular implementation feels weird though. All `SubApps`
should have an authorization method called `deploy_access?`. It would be
confusing for anyone working with this code in the future to figure out
where I'm authorizing the user to deploy content.

## Method delegation to other policy objects
Remember that at the core of it, the `authorize` helper method is doing
the following things:

- Get the class of the object and instantiate the policy object for
that class. For our particular case:
{% highlight ruby %}
policy_class = "#{curriculum_app.class.to_s}Policy".classify
policy_object = policy_class.new(current_user, curriculum_app)
{% endhighlight %}
- Send the method passed in as the second argument:
{% highlight ruby %}
policy_object.deploy_access?
{% endhighlight %}
- If it returns a truthy value, then allow, otherwise raise an
authorization error.

Armed with this knowledge, we can delegate missing methods out to more sub app
specific policy objects.

When implementing something like this, I usually like to whitelist the
objects that the method calls will be delegated out to. In order to do
so, let's just create a mapping:

{% highlight ruby %}
# app/policies/sub_app_policy.rb

class SubAppPolicy < ApplicationPolicy

  # Mapping of app name to policy object
  APP_POLICY_MAPPING = {
    'Curriculum App': SubApp::CurriculumPolicy
  }

  # ...
end
{% endhighlight %}

Next, we'll override the `method_missing` method for the `SubAppPolicy`
objects to try to delegate out to the specific app policy objects if
allowed.

{% highlight ruby %}
# app/policies/sub_app_policy.rb

class SubAppPolicy < ApplicationPolicy
  # ...

  def method_missing(method, *args, &block)
    if policy_class = APP_POLICY_MAPPING[record.name.to_sym]
      policy_object = policy_class.new(user, record)

      policy_object.send(method, *args, &block)
    else
      super
    end
  end
end
{% endhighlight %}

Cool, now all we have to do is set up the `deploy_access?` method in the
`SubApp::CurriculumPolicy`.

{% highlight ruby %}
# app/policies/sub_app/curriculum_policy.rb

class SubApp::CurriculumPolicy < ApplicationPolicy

  def deployer_access?
    user.has_role?(record, :admin, :deployer)
  end
end
{% endhighlight %}

## Friendly errors
Almost done! The last thing we want to do is that if the specific app
policy object also doesn't know how to respond to the method, we want to
give a more descriptive error message. We can rescue a `NoMethodError`
and add our own messaging to it:

{% highlight ruby %}
# app/policies/sub_app_policy.rb

class SubAppPolicy < ApplicationPolicy
  # ...

  def method_missing(method, *args, &block)
    if policy_class = APP_POLICY_MAPPING[record.name.to_sym]
      policy_object = policy_class.new(user, record)

      begin
        policy_object.send(method, *args, &block)
      rescue NoMethodError
        raise MethodDelegationError.new "undefined method \"#{method}\" for <SubAppPolicy> and <#{policy_object.class.to_s}>"
      end
    else
      super
    end
  end

  class MethodDelegationError < StandardError
  end
end
{% endhighlight %}

With the above custom error, our message will look something like
`SubAppPolicy::MethodDelegationError: undefined method
"some_crazy_method_name" for <SubAppPolicy> and <SubApp::CurriculumPolicy>`

## Conclusion
Now that we have this in place, it should be more easy to find app
specific authorization calls as well as general authorization across the
apps.

Note that there are definitely other ways to go about solving this
issue, like starting to create separate apps as opposed to a monolithic
Rails app, but given the constraints and requirements, this was the
best way to tackle the issue in this case.
