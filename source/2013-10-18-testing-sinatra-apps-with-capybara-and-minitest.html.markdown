---
title: Testing Sinatra apps with Capybara and Minitest
date: 2013-10-18 03:42 UTC
tags: testing, capybara, minitest, sinatra
---

I went through some headaches recently getting a Sinatra app configured to use Capybara with
Minitest (plain vanilla Minitest as opposed to Unit or Spec).  There's a lot of
nuiance I skip over but below I walk through setting up Capybara with Minitest
for a Sinatra app.  Please [chime in](http://twitter.com/simontaranto) if you
  have any tips.

My test folders look like this

```
test  
|  
|\  
| \acceptance  
|  
|\  
| \helpers  
|  
|\  
| \integration  
|  
|\  
| \unit  
```

All that we're interested in are acceptance and helpers.  The integration and
unit tests will use their own helpers so they don't have to worry about extra
dependencies.  In other words, only the acceptance tests will require Capybara.
Let's take a look at the acceptance_helper found under /helpers.

```
ENV['RACK_ENV'] = 'test'

require_relative '../../lib/<app_name>' # app is the name of your app file

# These two lines are optional but make your other files cleaner
require 'bundler'
Bundler.require

require 'rack/test'
require 'minitest/autorun' # optional but makes life easier
require 'minitest/pride' # optional but why would you not
require 'capybara'
require 'capybara/dsl'

Capybara.app = MyApp # the name of your app class
Capybara.register_driver :rack_test do |app|
  Capybara::RackTest::Driver.new(app, :headers =>  { 'User-Agent' => 'Capybara' })
end
```

And the GEMFILE:

```
gem 'sinatra', require: 'sinatra/base' # you may have more than just base
gem 'minitest'
gem 'rack-test'
gem 'capybara'
gem 'minitest-capybara'
```
With all those pieces in place you can set up your acceptance test class.  Mine
looks something like this:

```
require_relative '../helpers/acceptance_helper' # require the helper

class UserAcceptanceTest < Minitest::Test
  include Capybara::DSL # gives you access to visit, etc.

  def test_it_doesnt_blow_up
    visit '/'
    assert page.has_content?("It's alive")
  end

  # write your tests here

end
```

That about sums it up.  There are likely cleaner ways to get everything set up
but the above should get you started.
