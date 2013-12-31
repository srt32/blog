---
title: Using Postgresql's tsrange Range Type with Rails
date: 2013-12-31 04:47 UTC
tags: rails, postgresql
---


####The Range Data Type

Postgres 9.2 and later supports [range data types](http://www.postgresql.org/docs/devel/static/rangetypes.html)
including:

* int4range — Range of integer
* int8range — Range of bigint
* numrange — Range of numeric
* tsrange — Range of timestamp without time zone
* tstzrange — Range of timestamp with time zone
* daterange — Range of date

and as of Rails 4.0.0 in [pull request 7345](https://github.com/rails/rails/pull/7345)
so does Rails.  Some of the commentary on this PR about bounds is interesting if you have
some time to kill. The [release
notes](https://github.com/slbug/rails/blob/af1ef85ad2b227c8d1fb4c90f814e5a8414faa92/activerecord/CHANGELOG.md)
have a simple example of how to use the range, which holds a start and end date
(or datetime) in a single field.  Here's the given example:

```
create_table :Room do |t|
  t.daterange :availability
end

Room.create(availability: (Date.today..Float::INFINITY))
Room.first.availability # => Wed, 19 Sep 2012..Infinity
```

Instead of having multiple columns named something like start\_date and end_date
you can have a single attribute.  The daterange object provides `#begin` and
`#end` methods that can be used to retrieve the starting and ending date
objects, respectively.


####Gotchas:

I did run into a couple interesting things while implementing the range data
type though.  There is at least one additional [PR](https://github.com/rails/rails/pull/12694)
related to this functionality so check your Rails version before diving in.

There have been [some limitations](https://github.com/sferik/rails_admin/issues/1734)
reported with RailsAdmin so test thoroughly if this limitation could impact you.

Casting range data types via Rails migrations gets interesting.  For example, if
you started with a daterange type and eventually wanted a tsrange so that the
field includes time as well as date you can't get away with a simple migration
such as the one below (assuming the column name is 'when'):

```
def self.up
  change_column :events, :when, :tsrange
end
```

because there is no default casting value from daterange to tsrange.  You can
[get around the issue](http://stackoverflow.com/questions/20820997/rails-4-migration-to-change-datatype-of-column-from-daterange-to-tsrange-causin) with something like the below:

```
connection.execute(%q{
    alter table events
    alter column "when"
    type tsrange using tsrange(lower("when"), upper("when"))
})
```

The app where I used this range type is very young so time will tell if this
decision was a good one but it's always fun to experiment with new Rails
features on top of Postgres.
