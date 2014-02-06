---
title: Writing and deploying multiple Rails apps with Nginx
date: 2014-02-06 05:01 UTC
tags: Rails, Nginx, VPS
---

For our last project, my team built a real time dashboard for development teams
so managers and team managers can have an at a glance look at what's happening
in a project.  A user can create a project and link up Github repos, Pivotal
Tracker Projects, Travis CI builds, and Code Climate scores.  We built the app
using Services Oriented Architecture and learned quite a bit along the way.  We
built 3 Rails apps, a Sinatra app, and gem.  We used Nginx and Passenger on
a Digital Ocean VPS to make it live.

### Architecture
Below is a summary of the architecture.  A user comes to the site and hits the
authentication app where they can log in via Github.  Upon successful
authentication, a signed cookie is set and the user is seamlessly directed to the dashboard
app where they can add projects and link accounts, which sends JSON requests to
the API backend via a custom gem.  When new events (commits, completed stories,
failing builds, etc) occur the dashboard updates in real time by parsing webhook
payloads from the various external services via the Receiver Sinatra app.

```

          |------------|
          |Github OAuth|
          |------------|
             |
          |------|         /  \
          | Auth |  --->  | DB |
          |------|         \  /
|----|  /         \
|User|             |-cookie-|
|----|  \         /
          |-------- |         |-------|       /  \
          |Dashboard|  -gem-> |  API  | ---> | DB |
          |-------- |         |-------|       \  /
                                 ^
                                 |
                                gem
                                 |
                              |--------|      |------------------|
                              |Receiver| <--- | External Services|
                              |--------|      |------------------|

```


#### The Code:

All of our code can be found on our
[organization's](https://github.com/foofoberry) page.  Each particular app and
process had it's own repo, which are summarized below:

- Authentication: [https://github.com/FooFoBerry/feed\_engine\_auth](https://github.com/FooFoBerry/feed_engine_auth)
- Frontend / Dashboard: [https://github.com/FooFoBerry/feed\_engine\_front\_end](https://github.com/FooFoBerry/feed_engine_front_end)
- API: [https://github.com/FooFoBerry/feed\_engine\_api](https://github.com/FooFoBerry/feed_engine_api)
- Callback Receiver: [https://github.com/FooFoBerry/costner\_goes\_postal](https://github.com/FooFoBerry/costner_goes_postal)
- Gem: [https://github.com/FooFoBerry/foofoberry](https://github.com/FooFoBerry/foofoberry)
- Demo Bot: [https://github.com/FooFoBerry/api\_bot](https://github.com/FooFoBerry/api_bot)
- Helper Scripts: [https://github.com/FooFoBerry/processes](https://github.com/FooFoBerry/processes)

### Databases
Our app has two databases.  There is one Postgres database tied to the Authentication
app that holds User log on info (email, github token, and a user id).  Then, the
API has another Postgres database that manages projects and associated accounts.

XXX MORE INFO ON THE DATABASES

### Signed Cookies
The user gets passed from the Authentication app to the Dashboard app.  The way
these two apps communicate with each other is through a signed cookie.

XXX SHARED SECRET

XXX Example of the cookie.

Here's a great post that goes into more details on how signed cookies work:
http://blog.bigbinary.com/2013/03/19/cookies-on-rails.html

### Namespacing

To make all these apps work together we took the approach of using a single
domain (with no subdomains) that utilized sub directories.  So, each app has
its own routing namespace.  Here's how the app's namespaces broke down:

- Authentication: /
- API:            /api
- Frontend:       /dashboard
- Receiver:       /notifications

In the Rails apps, we achieved this namespacing by wrapping all the routes in
a namespace block.  For example, the API's routes look like this:

```
FoofoberryApi::Application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :projects,           only: [:index, :show, :create] do
        resources :commits,          only: [:index, :show]
      end
      ...
    end
  end
end
```

Notice that all the routes are under `namespace :api`.  In this way, the apps
should be able to redirect to each other without having to know about how any of
the other apps work.  At this point we realized we needed some way to actually
tie all the apps together.

### Rack Proxy -> fail

We basically needed a reverse proxy so that an app could do its thing within
its own routes but also be able to transfer the over to another app.  For example,
when a user logs into the website they are on the authentication app (at '/' route)
but after they log in they ger redirected to '/dashboard', which is the dashboard /
frontend app.  The auth app needs to be able to do this:

```
redirect_to '/dashboard'
```

Our first attempt was to set up an entirely different app that solely runs a 
RackProxy and routes every request to the proper app depending on the routes.
We got something going with the below server:

```
require 'rack-proxy'

class AppProxy < Rack::Proxy
  def rewrite_env(env)
    request = Rack::Request.new(env)
    if request.path =~ %r{^/api}
      env["HTTP_HOST"] = "localhost:3001"
    elsif request.path =~ %r{^/dashboard}
      env["HTTP_HOST"] = "localhost:3002"
    else
      env["HTTP_HOST"] = "localhost:3000"
    end
    env
  end
end

run AppProxy.new
```

While this approach worked it was a pain to deal with because we needed to start
an additional process locally to develop across multiple apps but more importantly,
every request in any of the apps had to be explicitly routed out back through
this proxy.  To achieve this requirement we added a before_action to the application
controller that hit the proxy.  You can see that implementation [here](https://github.com/FooFoBerry/feed_engine_front_end/blob/0b0460262d09eaa30daecbfcc4e0579cb9bb0df4/app/controllers/application_controller.rb).
The problem with this approach is each app had to know about how to talk to the
other apps.  This coupling seemed like a nightmare to maintain and unlikely to work
on production so we abandonded this concept.  Onto Nginx.

### VPS and Nginx

Digital Ocean's docs on setting up a server are superb.  Check out [this article](https://www.digitalocean.com/community/articles/how-to-install-rails-and-nginx-with-passenger-on-ubuntu) on how to get Rails up and running.

XXX NGINX CONFIG FILE

### Easy deployments

See [my recent post](http://www.simontaranto.com/2014/01/23/doing-more-than-deploying-code-in-a-git-post-receive-hook.html)
for some more details on how to make git post-receive hooks do deploymeny chores
for you.
