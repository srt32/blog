---
title: "The power of hashes: why thinking about your problem matters"
date: 2013-10-04 14:25 UTC
tags: hash, enumerable, performance,Sales Engine, metaprogramming
---

We wrapped up the SalesEngine project this week and during our code review we
discussed ways to improve the performance of our code.

We were given a [spec harness] (https://github.com/gSchool/sales_engine_spec_harness) to run against
our code to validate all the features worked.  Most students' first round
implementations took 10-15 minutes to run the specs.  That amount of time is
painful when trying to debug problems.  But, most of us just dealt with it and
moved on. This approach was the wrong one.

####Some major takeaways:

* Hashes are great for looking up data
* Modules and clever inheritance can really tidy things up
* Think about your problem (aka, there is always a better way)

The code for the project discussed in this post can be found
[here](https://github.com/srt32/sales_engine). Feedback is welcome.
A description of the project is
[here](http://tutorials.jumpstartlab.com/projects/sales_engine.html).

#####Hashes

The requirements of the project are to load up half a dozen thousand-line CSV
files, build relationships (basically building ActiveRecord sans database from
scratch) across the tables in memory, and create 'business intelligence' logic across this data.  For example,
a user could ask for the top 3 merchants by revenue.  This question requires
taking a look at merchants, the merchant's transactions, and those transaction's
invoice_items.

The first implementation used a host of dynamically generated (using Ruby's
[define_method](http://www.trottercashion.com/2011/02/08/rubys-define_method-method_missing-and-instance_eval.html)) find_by methods
that scanned the whole dataset using Enumerable#select.  These operations took
a long time. During our review we drilled down and found that these large select
calls were the performance culprit and we really needed an index.  But, how to
do that in Ruby we thought.  Ruby
[Hash}(http://www.ruby-doc.org/core-2.0.0/Hash.html) to the rescue!  We
updated our dynamically generated finders to reference a hash holding the
pre-grouped values we were looking for. As a result, the spec harness, which
previsouly took 10-15 minutes, now runs in under 3 seconds.

#####Modules

We extracted the dynamic finder methods and hash creation methods into its own
module and etended it into each class that needed it instead of duplicating code
across classes.  While there was some overhead on learning how to use the
Modeuls the dozens of lines of code and duplication saved certainly paid off.
Here's a [gist](https://gist.github.com/burtlo/6817579) including a simple
example.

#####A better way

The biggest win to come out of this project beyond some nice new tricks is
a reminder that if you're confronted with a problem (a slow test suite in this
case) there is always a way to do things better.  We should have taken a step
back a week ago and thought through the problem before blindly pushing forward
with the single solution we knew worked.  If you're newish to Ruby and have
a couple weekends to burn I highly recommend working through the
[project](http://tutorials.jumpstartlab.com/projects/sales_engine.html).
