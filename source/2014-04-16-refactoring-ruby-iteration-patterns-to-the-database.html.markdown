---
title: Refactoring Ruby Iteration Patterns to the Database
date: 2014-04-16 05:21 UTC
tags: thoughtbot, sql, rails
---

Check out my [recent
article](http://robots.thoughtbot.com/refactoring-ruby-iteration-patterns-to-the-database)
on [thoughtbot's](http://thoughtbot.com/) blog.

Frequently on projects we need to run a calculation through an ActiveRecord
association. For example, we might want to get a user's all time purchases,
a company's total amount of products sold, or in our case, the total amount of
loans made to a campaign. This sort of calculation is ripe for using a map and
an inject but frequently the solution is more elegant and faster if we can have
SQL do the work instead.

[read
more...](http://robots.thoughtbot.com/refactoring-ruby-iteration-patterns-to-the-database)
