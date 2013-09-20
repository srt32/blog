---
title: Tests, tmux, and t-shirts
date: 2013-09-20 15:02 UTC
tags: testing, minitest, git, tmux, vim
---

This week was full of learning new ideas, philosophies, and dev tools.  The
week's project was Event Reporter, a command line app that loads, parses, and
queries a CSV file containing attendee data.  

The project's spec can be found
[here](http://tutorials.jumpstartlab.com/projects/event_reporter.html).  This
project was our first open ended assignment and to make it more fun we started
testing our code using
[MiniTest](http://tutorials.jumpstartlab.com/academy/workshops/testing_event_manager.html)
as well as using [git](http://try.github.io/levels/1/challenges/8) for versioning.  I am loosely familiar with both testing and git but had a bit of a rough start to the week.  To add to the fun I got myself set up with tmux and vim (both new to me).  The week started out slow but I feel like I've made an evolving but solid dev workflow for myself.  

###Tests
The results of my labor this week are
[here](https://github.com/srt32/event_reporter).  There are some more classes to
be extracted and as of this writing the queue class (which I'm calling Result)
is partially extracted.  I did choose to truly test drive this piece of
extraction and it's been really fun to see how powerful a well tested class can
be.  In addition, the MiniTest core team has a good sense of humor and got
a deep laugh out of me when I read through the documentation:

<blockquote class="twitter-tweet"><p>My new favorite piece of <a
href="https://twitter.com/search?q=%23ruby&amp;src=hash">#ruby</a> documentation
minitest&#39;s &quot;i_suck_and_my_tests_are_order_dependent!()&quot; method. <a
href="http://t.co/ikyw3hZQSg">http://t.co/ikyw3hZQSg</a></p>&mdash; Simon
Taranto (@SimonTaranto) <a
href="https://twitter.com/SimonTaranto/statuses/380832537481732096">September
19, 2013</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

###Tmux
All I can say is whoa (and productivity).  A classmate of mine helped me get set
up with [iterm2](http://www.iterm2.com/#/section/home),
[tmux](http://tmux.sourceforge.net/), and
[vim](http://www.vim.org/).  I was previously using a terminal and Sublime but
have found that, while there is a non-trivial learning curve for tmux/vim, the
ability to have your entire workspace at your fingertips without distraction is
liberating and exciting.  I look forward to learning more arcane keyboard
shortcuts.


###T-shirts
Galvanize hosted more than a dozen events this week as part of [Denver Startup Week](http://www.denverstartupweek.com/).  We had to dodge events all week to find classroom space but the energy in the building and city were palpabale and contagious.  As an added benefit my start-up t-shirt collection has ballooned.

####Quotation of the week
```
Your love of ternary operators must be broken.
```

This gem of a quotation came out of a super helpful 3-hour code review marathon
where we went through a bunch of implementations of Event Reporter.  I learned
a tremendous amount about good patterns (and bad ones).

