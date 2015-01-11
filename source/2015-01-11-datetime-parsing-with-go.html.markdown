---
title: Datetime Parsing with Go
date: 2015-01-11 03:09 UTC
tags: golang
---

While working on a side project recently
([https://github.com/srt32/sfpd_crime](https://github.com/srt32/sfpd_crime)) I
wrote a CSV parsing script in Go and I realized I did not understand how to
parse a non-standard string into a `time.Time` type. Here's what I learned.

My date data looked like the following: `12/18/2014 07:00:00` but the Go
reference time is in the following format: `Mon Jan 2 15:04:05 -0700 MST 2006`.
So, how do we go from my format to the required format?

It seemed that [`time.Parse`](http://golang.org/pkg/time/#Parse) should have
done the trick but I kept getting errors during parsing. After mucking around
in a [playground session](http://play.golang.org/p/F2vGyc3QAz) for awhile I
realized the reference format is not only a format but also data. The important
data there is January 2nd at 3:04:05 in the afternoon Mountain Time. When
formatted a different way we get: `01/02 03:04:05PM '06 -0700`, which is an
increasing series of integers. More info
[here](http://stackoverflow.com/questions/20530327/origin-of-mon-jan-2-150405-mst-2006-in-golang).

Now that we have the reference time our solution becomes clear:

`time.Parse("01/02/2006 15:04:05", "12/18/2014 07:00:00")`

Go's parsing seems quite clever, actually, in that by passing in the formatted
string with the correct date and time (and optionally time zone) you have a
human readable example of what the input datetime should be. In
[Ruby](http://apidock.com/ruby/DateTime/strftime), for example, this kind of
parsing would look something like the following (assuming we didn't want to let
Ruby 'guess' how to parse our string), which feels harder to grok without
looking up the various character options.

`Time.strptime("12/18/2014 07:00:00", "%m/%d/%Y %H:%M:%S")`
