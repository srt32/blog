---
title: A primer on Bundler basics
date: 2013-10-17 14:15 UTC
tags: ruby, bundler, testing, dependencies
---

Ruby's <strong>[Bundler](http://bundler.io/)</strong> performs all sorts of magic for your apps'
dependencies.  All sorts of magic I don't fully understand but after reviewing some open source
projects this week I gained some new insight on some of the knobs that Bundler makes
available to you. Below is a summary of some of the more commmon Gemfile configs
and associated Bundler commands you'll see in the wild.

Here's an example Gemfile we'll use to illustrate our points:

```

source "https://rubygems.org"

ruby "1.9.3"

gem 'pry', require: false
gem 'pony'
gem 'airbrake'

group :test do
  gem 'minitest', require: false
  gem 'rack-test', require: false
end

```
#####A couple things to note:

1. pry is set to require: false
2. there are gems in the test group (also require: false)

#####Your command options:

1. Run just `bundle` --> You'll install all the gems in your Gemfile regardless
   of environment or require declarations.  Using our example, running `bundle`
   will install all 5 gems onto our machine.

2. Run `bundle --without test` -->  You'll make available to your application
   all the gems in the Gemfile not in the test group.  In our case, you'll end
   up with three active gems.

3. Including the following in your app

  ```
  require 'bundler'
  Bundler.require
  ```

will make available to your app only those gems without a false flag. In our
case, we'll end up with two gems (pony and airbrake).

Using a mixture of these commands you can minimize the installation of
unncessary external dependencies and make your app smaller.  Happy bundling!
