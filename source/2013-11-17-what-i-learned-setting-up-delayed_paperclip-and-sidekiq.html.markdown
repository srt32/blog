---
title: "What I learned setting up delayed_paperclip and Sidekiq"
date: 2013-11-17 20:11 UTC
tags: ruby, sidekiq, delayed\_paperclip
---

A recent project required an admin user to be able to upload images.  Using [paperclip](https://github.com/thoughtbot/paperclip) we got the uploading going in no time.  But, having the uploads going through the app server was very slow (~1.5s requests) so we moved the
uploading into a background job.  Having never messed with background jobs before
I chose to use [delayed_paperclip](https://github.com/jrgifford/delayed\_paperclip/)
as an easy way to wade into the topic.

Using delayed\_paperclip is appealing because it will solve our problem and teach
us about background jobs all without having to write custom code.  We have to
just figure out how to get everything installed.

Railscasts has a great [walkthrough](http://railscasts.com/episodes/366-sidekiq) of setting up vanilla Sidekiq and Sidekiq's [docs](http://sidekiq.org/) are pretty helpful if you have specific
questions.

So, jumping into the [code...](https://github.com/JonahMoses/dinner_dash)

###The model
We're working with the Item model, which allows the admin to upload images.
Adding

```
process_in_background :image

```

is all we need to get things going from the model's perspective.


```
# app/models/image.rb
class Item < ActiveRecord::Base
  # other code goes here (validations, relationships, etc)

  has_attached_file :image, style: {
    thumb: '100x100>',
    square: '200x200#'
    }

  process_in_background :image

  # other code goes here
end
```

###Additions to Gemfile
We'll need these gems.

```
# Gemfile
# other gems...
gem 'sidekiq'
gem 'delayed_paperclip'
gem 'sinatra', require: false
gem 'slim'
```

###Testing for user access

We want only admins to be able to upload images so using RSpec, you can test that non admin users receive a RoutingError upon trying to access the Sidekiq page.


```
expect{visit '/sidekiq'}.to raise_error(ActionController::RoutingError)
```

Some more specifics can be found on the [Sidekiq monitoring wiki](https://github.com/mperham/sidekiq/wiki/Monitoring)

###Updated routes

To implement the Sidekiq monitoring page and limit access to it we do the following:

```
# config/routes.rb
require 'sidekiq/web'
require 'admin_constraint'

YourApp::Application.routes.draw do
  # other routes go here

  mount Sidekiq::Web, at: '/sidekiq', :constraints => AdminConstraint.new

end

```

###Limiting access

We define the below class to act as the constraint on the '/sidekiq' route.
In this way the server responds with a routing error (like we tested for) instead
of a not authorized redirect, which stops the server from divulging that the
/sidekiq routes exists to those that should not be able to access it.

```
# lib/admin_constraint.rb
class AdminConstraint

  def mathces?(request)
    return false unless request.session[:user_id]
    user = User.find request.session[:user_id]
    user && user.admin?
  end

end
```

###Getting it all to work
If your prod app is going to run off a Procfile it's best to develop using a
Procfile as well.  Heroku Toolbelt makes this task really simple.  You can
run your app locally and have it boot up with a Procfile by running 

```
foreman start
```

Learn more about Heroku and foreman [here](https://devcenter.heroku.com/articles/procfile#developing-locally-with-foreman).
More on the Procfile settings in just a bit.

Locally, I still had to fire up a redis server manually.  So, if you see

  ```
Error connecting to Redis on 127.0.0.1:6379 (ECONNREFUSED)
  ```

Check to make sure redis is running (by default, on port 6379).

At this point you should be good to go with image uploading happening
in a background job locally.


###Deployment checklist
We're going to deploy to Heroku as they have some nicities that make getting
our background jobs off the ground easier so we can see our work in action
faster.

1. Create or update your Procfile to fire up Sidekiq upon launch.
  Mine looks like:

  ```
  web: bundle exec rails server -p $PORT
  worker: bundle exec sidekiq -c 5 -v -q paperclip
  ```

  Notice that I'm taking a shortcut and creating a new queue (`-q paperclip`) for
  Sidekiq instead of using the default queue, which is named 'default'.  I'm sure
  there are headaches coming down the pipe later on managing multiple queues but
  in the name of getting things going, I'd recommend naming your queue 'paperclip'.

2. I had some issues getting Heroku to automatically start my worker so I had
to manually start it using `heroku ps:scale worker+1`. After a couple dyno
restarts, the worker started picking up automatically.  So, if your worker isn't
running (you can check by running `heroku ps`) you can manually start it up.

3. Heroku provides a plug and play Redis instance that you can install by
running

  ```
  Heroku addson:add redistogo
  ```

###Next level resources
I look forward to learning more about advanced background job setups and
how best to test these sorts of systems.  For when you write your own code to
utilize threads check out another great [Railscast](http://railscasts.com/episodes/365-thread-safety) on the topic.
