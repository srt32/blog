---
title: Why Postgres Won't Always Use an Index
date: 2014-08-11 02:03 UTC
tags: postgres, performance
---

This post was originally published on [thoughtbot's blog](http://robots.thoughtbot.com/why-postgres-wont-always-use-an-index).

We were analyzing slow responses on a project recently and found ourselves
questioning the performance of some PostgreSQL queries. As is typical when
digging into queries, we had some questions around what the query was doing.
So, we used our handy `EXPLAIN` command to shed some light on the database's behavior.
Upon inspecting the query it turned out an index we had created was not being used.
We wanted to know why.

[read more...](http://robots.thoughtbot.com/why-postgres-wont-always-use-an-index)
