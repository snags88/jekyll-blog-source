---
layout: post
title: "Hello, my name is..."
permalink: "hello-my-name-is"
date: 2015-05-20 03:51:00 -0500
categories: web-development
---
If you've ever worked on a project with me, you know that I love aliasing and renaming things so that code is more legible. One of the latest discoveries I had was that you can alias join tables so your code makes more sense when you read it out loud.

### Scenario

Let's say that I'm creating an app where a user can own a playlist and also contribute to other user's playlists. Our schema might look something like below:

<img style="margin: 0 auto; display: block;" src="https://dl.dropboxusercontent.com/u/16107869/playlistuser.png" alt="playlistuser" width="400" height="auto" />

A __user__ has many __playlists__ through the __owner_id__ foreign key, but they also make many __contributions__ to __playlists__ through the __playlist_user __ join table. Vice versa, a __playlist__ has an __owner__ through their __owner_id__ foreign key, but they also have many __contributors__ through the __playlist_user__ join table.

### Implementation

There's actually two steps in implementing the relationship I previously described. The first is to create a direct relationship between the users and playlists through the owner_id foreign key. This can be done by setting up __has_many__ and __belongs_to__ relation:

{% highlight ruby %}
class Playlist < ActiveRecord::Base
  belongs_to :owner, class_name: "User"
end

class User < ActiveRecord::Base
  has_many :playlists, :foreign_key => 'owner_id'
end
{% endhighlight %}

Now we can call something like:

{% highlight ruby %}
playlist.owner #=> returns the current playlist's owner
user.playlists #=> returns the user's playlists that they own/created
{% endhighlight %}

The next step is to alias the __has_many through__ relation with the join table. It turns out it's pretty straight forward. You just have to declare the __has_many__ with whatever alias you want to give the association, __through__ the join table, and then specify the __source__, which is the table you want to join to. In this specific case, our code will look as follows:

{% highlight ruby %}
class Playlist < ActiveRecord::Base
  has_many :playlist_users
  has_many :contributors, through: :playlist_users, source: :user
end

class User < ActiveRecord::Base
  has_many :playlist_users
  has_many :contributions, through: :playlist_users, source: :playlist
end
{% endhighlight %}

This association will now allow us to call things like:

{% highlight ruby %}
playlist.contributors #=> returns the list of users that are allowed to contribute to the playlist
user.contributions #=> returns the list of playlists the the user does not own, but can contribute to
{% endhighlight %}

Cool! Now our code is a little bit more legible thanks to a few extra configurations. Until next time, happy coding!
