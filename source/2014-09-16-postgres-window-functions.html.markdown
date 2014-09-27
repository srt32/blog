---
title: Postgres Window Functions
date: 2014-09-16 17:17 UTC
tags: performance, postgres, sql
---

This post was originally published on [thoughtbot's blog](http://robots.thoughtbot.com/postgres-window-functions).

We recently ran into a case where a join was getting out of hand and we were
struggling with how to rein in the size of it. Then we found [window
functions](http://www.postgresql.org/docs/9.3/static/tutorial-window.html).
Window functions (Oracle calls them [analytic
functions](http://docs.oracle.com/cd/E11882_01/server.112/e26088/functions004.htm#SQLRF06174))
are a part of the SQL standard and this post will explore how to use them in
Postgres. Let's see how they work and what kind of problems they can help us
solve.

[read more...](http://robots.thoughtbot.com/postgres-window-functions)
