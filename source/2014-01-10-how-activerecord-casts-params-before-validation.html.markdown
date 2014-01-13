---
title: How ActiveRecord casts params before validation
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

and the table looks like this:

```
 create_table "events", force: true do |t|
    t.integer  "user_id"
    t.string   "title"
    t.text     "description"
    t.string   "venue"
    t.string   "address"
    t.datetime "created_at"
    t.datetime "updated_at"
    t.tsrange  "when"
  end
```

We're interested in 1) taking the start\_date, start\_hour, and end\_hour parameters
 and 2) validating the start\_hour and end\_hour params are valid (between 0 and
23 in this case) so we can produce a single `when` value. Notice that we want to validate the start\_hour and end\_hour as
integers and not strings but the parameters come from the request as strings.
One solution is to just validate the hours as strings, which would produce the following
type of validation

```
validates :start_hour, presence: true, inclusion: { in: ('0'..'23') }
```

While this validation gets the job done it doesn't feel right to validate numbers
as strings.  So, we'll need to convert the parameters first.  At first, this conversion
didn't make sense to me because normally no conversion is necessary before validations.
The reason we need to do this conversion by hand is because the hours attributes are virtual
attributes and as such are not defined in the schema.  As a result, Rails has no knowledge
about the data type.

When an attribute is defined in the schema (and has a data type defined
in the database) this attribute's data type is cached by Rails upon initialization and
the parameters from the request are converted to this data type before validation.  In
this way, if you have an integer column in the database you can safely run model validations
using integers without having to first manually cast the data.  Rails' knowledge of
the database types comes from the db adapter in ActiveRecord.  In our case, we're using
postgres so that code can be found in 'activerecord-4.0.2/lib/active\_record/connection\_adapters/postgresql_adapter.rb'.
Thanks to [these slides](http://www.slideshare.net/thehoagie/active-records-beforetypecast)
for pointing this out.

Back to our world, where we have virtual attributes, one way to solve the casting
issue is to use a before\_validation call.

```
class Event < ActiveRecord::Base

  attr_accessor :start_hour,
                :end_hour

  before_validation :cast_hours

  validates :start_hour, presence: true, inclusion: { in: (0..23) }
  validates :end_hour, presence: true, inclusion: { in: (0..23) }

  # other code...

  private

  def cast_hours
    self.start_hour = start_hour.to_i
    self.end_hour = end_hour.to_i
  end

  # other code...
end
```

Using this code our validations can now validate on integers instead of strings
so we end up with the following:


```
validates :start_hour, presence: true, inclusion: { in: (0..23) }
```

It's always nice to run into cases where you realize another thing that Rails
has been doing for you all along.  Please [let me know](https://twitter.com/SimonTaranto)
if you have any comments or tips on better ways to handle this sort of situation.
