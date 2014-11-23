---
title: Speed Up JavaScript Capybara Specs by Blacklisting URLs
date: 2014-11-23 03:40 UTC
tags: performance, testing
---

This post was originally published on [thoughtbot's blog](http://robots.thoughtbot.com/speed-up-javascript-capybara-specs-by-blacklisting-urls).

On a project recently, our full test suite began to crawl (taking ~9 minutes instead of less than 1) on our local machines running OS X but ran normally on CI. This slowdown took our productivity to near zero. We discovered our Capybara specs with js: true set were the culprit but we couldn’t figure out why.

[read more...](http://robots.thoughtbot.com/speed-up-javascript-capybara-specs-by-blacklisting-urls)
