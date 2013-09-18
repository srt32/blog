---
title: Ruby Enumerable is amazing
date: 2013-09-18 03:27 UTC
tags: refactoring, ruby, enumerable, collections
---

[Katrina](https://twitter.com/kytrinyx) unleashed this gem of a document on us today

###[Focus on Collections](http://tutorials.jumpstartlab.com/topics/collections.html)

Now, documentation doesn't always get me amped up but I enjoyed reading this document so much I tweeted about it.

<blockquote class="twitter-tweet"><p>The most exciting technical doc I&#39;ve ever seen by <a href="https://twitter.com/jumpstartlab">@jumpstartlab</a> <a href="https://twitter.com/search?q=%23ruby&amp;src=hash">#ruby</a> <a href="https://twitter.com/search?q=%23enumerable&amp;src=hash">#enumerable</a> <a href="http://t.co/4fXgquJG7F">http://t.co/4fXgquJG7F</a></p>&mdash; Simon Taranto (@SimonTaranto) <a href="https://twitter.com/SimonTaranto/statuses/380093012476309504">September 17, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

In my previous ruby work I've implemented, by hand, many of these methods (map, any?, find) simply because I didn't know Enumerable was so powerful. 

####Map: like each but with fewer lines

Map (or collect) takes you from 

```
numbers = [1,2,3,4]
doubles = []
numbers.each do |number|
  doubles.push(number*2)
end

doubles => [2, 4, 6, 8] 
```

to

```
numbers = [1,2,3,4]
doubles = numbers.map {|number| number*2}

doubles => [2, 4, 6, 8] 
```


####The take away?  

It's nothing all that crazy but reading [the docs](http://ruby-doc.org/core-2.0.0/Enumerable.html) can really payoff.

