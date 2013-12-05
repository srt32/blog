---
title: "rails_12factor logging problems"
date: 2013-12-05 14:51 UTC
tags: rails, heroku, ninefold
---

Our most recent
[project](http://tutorials.jumpstartlab.com/projects/fourth_meal.html) is to extend our previous project using
another team's codebase.  One of the first things we did was run the tests.  We
were greeted with a mess of ActiveRecord and view rendering logs.  Oh man, this
last team did some strange stuff, we thought.

We were expecting to see some pretty green (and red) dots but instead the terminal output looked like this:

```
$ rake test_all
  ActiveRecord::SchemaMigration Load (1.2ms)  SELECT "schema_migrations".* FROM
  "schema_migrations"
  Run options: --seed 56236

# Running tests:

   (0.3ms)  BEGIN
   -----------------------------------------------------------
   CanLogInAndOutTest: test_cannot_sign_up_with_invalid_params
   -----------------------------------------------------------
   Started GET "/" for 127.0.0.1 at 2013-12-05 07:53:45 -0700
   Processing by ItemsController#index as HTML
  Category Load (0.6ms)  SELECT "categories".* FROM "categories"
Rendered items/_header.html.erb (2.6ms)
  Item Load (0.8ms)  SELECT "items".* FROM "items" WHERE
  "items"."retired" = 'f'
  Rendered items/_item.html.erb (0.0ms)
  Rendered items/index.html.erb within layouts/application (20.1ms)
  Rendered users/_user_nav.html.erb (27.2ms)
  Rendered layouts/_header.html.erb (29.5ms)
Rendered layouts/_main_nav.html.erb (0.9ms)
  Order Load (8.5ms)  SELECT "orders".* FROM "orders" WHERE
  "orders"."id" IS NULL LIMIT 1
  CACHE (0.0ms)  SELECT "orders".* FROM "orders" WHERE
  "orders"."id" IS NULL LIMIT 1
Rendered layouts/_footer.html.erb (0.4ms)
  Completed 200 OK in 178ms (Views: 157.2ms
      | ActiveRecord: 16.1ms)
  Started GET "/log_in" for 127.0.0.1 at 2013-12-05
  07:53:45 -0700
  Processing by SessionsController#new as HTML
  Rendered shared/_errors.html.erb (3.6ms)
Rendered users/_form.html.erb (505.8ms)
  Rendered sessions/new.html.erb within
  layouts/application (508.7ms)
  Rendered users/_user_nav.html.erb (0.3ms)
  Rendered layouts/_header.html.erb (0.6ms)
Rendered layouts/_main_nav.html.erb (0.2ms)
  Order Load (0.3ms)  SELECT "orders".*
  FROM "orders" WHERE "orders"."id" IS NULL
  LIMIT 1
  CACHE (0.0ms)  SELECT "orders".* FROM
  "orders" WHERE "orders"."id" IS NULL
  LIMIT 1
```

We started investigating and with the help of team members from the original
team we found that the Gemfile had some suprises in it.  Namely, rails_12factor, used to deploy to Heroku, was in the production and test groups.

```
group :production, :test do
  gem 'rails_12factor'
end
```

Having been bitten on Heroku before by not including this gem I knew that this
gem injects two other gems into your dependencies: rails\_serve\_static\_assets and
rails\_stdout\_logging.  If the names of the gems didn't give away the punchline,
we then excluded rails\_12factor and included only rails\_stdout\_logging.  Boom, we
still had the problem.  So, the rails\_12factor gem was to blame.  To fix the
problem we just removed the :test group so this part of the Gemfile became:

```
group :production do
  gem 'rails_12factor'
end
```

Problem solved.  This gem is an interesting one.  I recently had another issue
when moving a different project from Heroku to [Ninefold](ninefold.com).
I accidentally left the rails\_12factor gem in my Gemfile when deploying and
after successfully deploying I received no application logs.  I could see system
deploy logs (bundler running, asset compilation, processes starting) but none
of the web traffic to the site was being logged (I was using Ninefold's web
interface to tail my logs).  The culprit again was rails\_12factor and more
specifically, rails\_stdout\_logging. After removing the gem all was good.

So, what does rails\_12factor do?  As mentioned earlier in this post and
explained on the gem's [Readme](https://github.com/heroku/rails_12factor) the
gem simply adds two other gems, which are required for Rails 4 apps to work on
Heroku.  In Rails 2.x and Rails 3.x apps, Heroku was able to inject these
dependencies into the vendor/plugins directory. This directory though, was
deprecated in Rails 4, so Heroku found another way.

It's nice that Ninefold found a way to deploy apps without requiring specific additional gems
or injected plugins.  I look forward to learning more about how their system
works.
