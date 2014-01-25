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

We want to be able to deploy code with a single command.  This command should
deploy the code and run any necessary additional logic to make the new code go
live.  The command could look like:

```
git push live master
```

A hacky solution that does not meet our requirement is to just ssh into the
server and `git clone` the code we want grom GitHub into the right folder on the
server, run any required bundler or rake commands, and then restart the app.
This sort of workflow sounds hideous.  So, how do we get to `git push live
master`?

### How git hooks work

This sort of task is exactly what git hooks are designed to do.  As listed out at [githooks.com](http://githooks.com/) we have the below options at our disposal:

- applypatch-msg
- pre-applypatch
- post-applypatch
- pre-commit
- prepare-commit-msg
- commit-msg
- post-commit
- pre-rebase
- post-checkout
- post-merge
- pre-receive
- update
- post-receive
- post-update
- pre-auto-gc
- post-rewrite

Whoa, that's a lot of hooks!  You can think of these hooks as an analog to
[ActiveRecord callbacks](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html)
in Rails such as before\_create or after\_save, for example.  The code
held within each of these hooks is executed at the given point of the git life
cycle.  We're going to focus on the post-receive hook to solve our problem.

### The post-receive hook

Because we are working with Digital Ocean and [their
docs](https://www.digitalocean.com/community) are stupendous I highly recommend
reading through [this article](https://www.digitalocean.com/community/articles/how-to-set-up-automatic-deployment-with-git-with-a-vps).

bare repos
ssh keys
users

### Bundler, rake, and passenger
Some help:
https://gist.github.com/yortz/1047083

The type of hook we ended up using: 
https://github.com/FooFoBerry/post-receive-hooks/blob/master/api.git/hooks/post-receive

Ideas:

- which user is running the code and in which context (no .bash\_profile loaded)
- bundle location
- can't find rake
- assets compiling
- probably want to force RAILS\_ENV to be production
- restarting passenger app

### Additional resources

- http://git-scm.com/book/en/Customizing-Git-Git-Hooks
- http://githooks.com/
