---
title: How ActiveRecord casts data types before validation
date: 2014-01-10 16:06 UTC
tags: Rails
---

I was building a model recently to support a form where users can create events
with a given day and time.  I was experimenting with using [Postgres' range data
types](http://www.simontaranto.com/2013/12/31/using-postgresql-s-tsrange-range-type-with-rails.html)
and needed to parse the user input from many fields into a single tsrange
(timestamp range) field.  Using attr\_accessors got me most of the way there but
I learned some interesting things about how ActiveRecord casts data types before
validation.

The data being passed from the controller looks like this:

```
{ :user_id => 1,
  :title => "party!",
  :description => "new years",
  :venue => "my house",
  :address => "down the street 4",
  :start_date => '2014-01-20',
  :start_hour => '18',
  :end_hour => '23' }
```

We're interested in 1) taking the start_date, start_hour, and end_hour parameters
 and 2) validating the start_hour and end_hour params are valid (between 0 and
23 in this case). Notice that we want to validate the start_hour and end_hour as
integers and not strings.  The parameters come from the request as strings so we'll
need to convert those.  
