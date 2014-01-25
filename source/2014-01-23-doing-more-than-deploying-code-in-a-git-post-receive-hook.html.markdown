---
title: Doing more than deploying code in a git post-receive hook
date: 2014-01-23 00:00 UTC
tags: git, devops
---

### Editing on the server
You know that terrible feeling you get when you're editing code on the
production server?  Wouldn't it be nice if we could easily deploy code from our
command line to our VPS?  Our most recent project was deployed to a VPS on
Digital Ocean and having come from a lot of experience deploying to Heroku, we
had to script out our deploy pipeline by hand.  It was a fun learning experience
ripe with seemingly undebuggable pitfalls.  Here's what I learned.

### What we want

```
git push live master
```

### How git hooks work

How do they work

### The post-receive hook
base article:
https://www.digitalocean.com/community/articles/how-to-set-up-automatic-deployment-with-git-with-a-vps

### Bundler, rake, and passenger
Some help:
https://gist.github.com/yortz/1047083

Ideas:

- which user is running the code and in which context (no .bash\_profile loaded)
- bundle location
- can't find rake
- assets compiling
- probably want to force RAILS\_ENV to be production
- restarting passenger app

### Additional resources
PUT SOME THINGS HERE

