---
title: "N + 101: an introduction to query optimization"
date: 2014-02-14 19:10 UTC
tags: SQL, query optimization, Rails
---

### Introduction
Meet [FreshFinder](http://freshfinder.us/).  FreshFinder is a location aware website that allows
you to explore farmers' markets across the US.  Data is sourced from the
[USDA](http://search.ams.usda.gov/farmersmarkets/) and presented in a beautiful
map-based interface.  The full application architecture can be seen [here](https://docs.google.com/drawings/d/1XqxsDhP2Msip2JOPe-BSOQpqvQ9HOLtri-xeEdK-kG4).
The code is available under the [FreshFinder
organization](https://github.com/FreshFinder) on GitHub.  The site pretty zippy
but we had to do some work to get it that way. Here's the story.

### The problem
The site was really slow.  When loading up the map or performing a search query the map
rendered in 1-2 seconds and the market pins dropped in 3-4 seconds later.  The load
time was noticeably slow and not acceptable.  What did we do?

### The search endpoint
Of particular interest to this post is the Market API /search endpoint that
allows API users and the frontend app to perform geocoded searches across the
market data.  Sending a GET to `/api/v1/search/markets?zipcode=<search_query>`
will return markets within 30 miles of the search\_query using the Google Maps
API to geocode the query.

### The culprit
The first thing we did to figure out what was slowing us down was tail the server
logs for the Market API and see what was going on during a search request.
With a quick look at the logs we could see that hundreds of SQL queries we're
flying by for each search.  That's not good.

It turned out that for each search we did we were performing a whole bunch of N+1 queries.
The process looked like this:

1.  In one big query, geocode the search param and return all the addresses,
which belong to a market, near that search (a single query)
2.  Perform one query per nearby address (potentially 100's of queries) another query to get the associated
market for that adddress
4.  Finally, go fetch again each associated address for each market

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
  ... # other stuff
end
```

And the code responsible for handling the search query:

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

An issue on the Geocoder issues list helped me out:
https://github.com/alexreisner/geocoder/issues/294


### Eager loading with Rails
Eager loading to the rescue.

Here is how [RailsGuides explains it](http://guides.rubyonrails.org/active_record_querying.html#eager-loading-associations).

what gets returned at each step
AS_JSON

Here is the updated code:

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

### Make it even better
* Query caching, RailsCast on Rails model caching
* View caching
* Caching at both apps
