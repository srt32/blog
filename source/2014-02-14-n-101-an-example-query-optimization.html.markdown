---
title: "N + 101: an example query optimization"
date: 2014-02-14 19:10 UTC
tags: SQL, query optimization, Rails
---

Meet [FreshFinder](http://freshfinder.us/).  FreshFinder is a location aware website that allows
you to explore farmers' markets across the US.  Data is sourced from the
[USDA](http://search.ams.usda.gov/farmersmarkets/) and presented in a beautiful
map-based interface.  The full application architecture can be seen [here](https://docs.google.com/drawings/d/1XqxsDhP2Msip2JOPe-BSOQpqvQ9HOLtri-xeEdK-kG4).
The code is available under the [FreshFinder
organization](https://github.com/FreshFinder) on GitHub.  The site is pretty zippy
but we had to do some work to get it that way. Here's the story.

### TL;DR
We had some serious pageload issues and we resolved them with some simple eager
loading.  Don't let rogue queries ruin your user experience.

### The problem
The site was really slow.  When loading up the map or performing a search query the map
rendered in 1-2 seconds and the market pins dropped in 3-4 seconds later.  The load
time was noticeably slow and not acceptable.  What did we do?

### The search endpoint
Of particular interest to this post is the Market API /search endpoint that
allows API users and the frontend app to perform geocoded searches across the
market data.  Sending a GET to

```
/api/v1/search/markets?zipcode=<search_query>
```

will return markets within 30 miles of the search\_query using the Google Maps
API to geocode the query.

### The culprit
The first thing we did to figure out what was slowing us down was tail the server
logs for the Market API and see what was going on during a search request.
With a quick look at the logs we could see that hundreds of SQL queries were
flying by for each search.  That's not good.

It turned out that for each search we did we were performing a whole bunch of N+1 queries.
The process looked like this:

1.  In one big query, geocode the search param and return all the addresses near that search (a single query)
2.  Perform one query per nearby address (potentially 100's of queries) to get the associated market for that adddress
3.  Finally, go fetch again each associated address for each market

What?  We're doing a lot of work to begin with but then we're duplicating our
work by finding addresses again.  Something is not right here.  Let's take
a look at the code.

The basic underlying model structure is as follows:

```
class Market < ActiveRecord::Base
  has_one :address
  ... # other stuff
end

class Address < ActiveRecord::Base
  belongs_to :market
  # geocoded by lat / long
  ... # other stuff
end
```

And the code responsible for handling the search query:

##### The old code

```
class Api::V1::SearchesController < ApplicationController

  def markets
    markets = Market.with_addresses.by_zipcode(params[:zipcode]).as_json(:include => :address)
    render json: markets
  end

end

class MarketSearchService < ActiveRecord::Base

  def self.by_zipcode(target_zipcode, radius = 30)
    Address.near(target_zipcode.to_s, radius).map(&:market)
  end

end
```

The problem is, when we call the markets method in the SearchesController we're
loading markets with address, then in the by\_zipcode method we're narrowing
down our addresses, followed by mapping out the markets (at the Ruby layer).
Essentially, after all this, we end up with a bunch of markets and then when we
finally call as_json we ask it to include the addresses again. We have to go get the addresses again because we've thrown away that information.  Silly! Eager loading to the rescue.

### Eager loading with Rails

We fortunately had some help from [an issue](https://github.com/alexreisner/geocoder/issues/294)
on the geocoder's tracker where it was pointed out there is no direct way to
query for associated models with a near query.  We'll have to first get the id's
of the markets we want (a single query) and then use those market id's to filter
down the second query that gets markets eager loaded with addresses.  The [RailsGuides](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations)
have a nice explanation of eager loading that I would recommend reading.

After rejiggering things to not duplicate our work, combined with eager loading
at the right place, we end up with the below code that runs only 2 queries and
produces the same data we previously got with 100's of queries.

##### The new code

```
class Api::V1::SearchesController < ApplicationController

  def markets
    markets = MarketSearchService.by_zipcode(params[:zipcode]).as_json(:include => :address)
    render json: markets
  end

end

class MarketSearchService < ActiveRecord::Base

  def self.by_zipcode(target_zipcode, radius = 30)
    market_ids = Address.near(target_zipcode.to_s, radius).select("id").map(&:market_id)
    markets = Market.includes(:address).where(:id => market_ids)
  end

end
```
The important pieces to note are that we are now eager loading at the point when
we actually call on markets (so the addresses get loaded only once) and that we map
out id's to be used in a following query instead of returning mapped markets (which can't
have the addresses loaded with them).  As a result of these small updates, load times dropped drastically and load on
the server came way down.

### Make it even better

There is some additional work we could do in the future if we wanted to speed
things up even more.  Some options are:

* Query caching so we don't even hit the database if the query has been performed
previously
* View caching so we don't have to render views as often

#### Some more reading
* Jumpstart Lab's [tutorial on queries](http://tutorials.jumpstartlab.com/topics/performance/queries.html).
