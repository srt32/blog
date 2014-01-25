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
docs](https://www.digitalocean.com/community) are stupendous, I highly recommend
reading through [this article](https://www.digitalocean.com/community/articles/how-to-set-up-automatic-deployment-with-git-with-a-vps).
We will follow the convention covered in this article of placing bare repos that
receive pushes in `/var/repo/site.git` and the actual source code used by
the server in `/var/www/site.com`.  The commands to run are:

```
cd /var
mkdir repo && cd repo
mkdir site.git && cd site.git
git init --bare
```

An important piece to understand about this system is the idea of a bare repo.
A bare repo is a git repo (conventionally in a directory named *.git) that
contains no working tree (no source code) just the version control files.
Here's another [concise
explanation](http://stackoverflow.com/a/7632497/1949363).

There are some details that the Digital Ocean article covers that we won't go
over here but the meat of the logic happens within `site.git/hooks/post-receive`
where we define where to deploy the code:

```
#!/bin/sh
git --work-tree=/var/www/domain.com --git-dir=/var/repo/site.git checkout -f
```

With that we can set up a git remote on our local machine and we're good to
push:

```
git remote add live ssh://user@<your IP or domain>/var/repo/site.git
git push live master
```

All done?  Well, not quite.  The code is now on the server but if you are
running Passenger, like we were, you're likely to face the error page because it
can't find gems.  Whoops, we forgot to bundle.  Let's ssh into the project
folder and run

```
bundle install --deployment
```

The `--deployment` flag tells bundler to read straight from the Gemfile.lock
file instead of building out the dependency tree dynamically.  This deployment
bundle process is faster and safer because you can bundle locally, commit the .lock
file, and know exactly which gems are going to be bundle in production.

Ok, things should be working now.  But, we just had to go run commans on the
server.  That's not what we want.

### Bundler, rake, and passenger

The post-receive hook can do much more than just deploy the code.  Let's set up
the script to run bundler as well.  I found [this gist](https://gist.github.com/yortz/1047083)
to be particularly helpful in providing a template to build out the hook.
Definitely check it out.  For a slimmed down example, check out [the hook](https://github.com/FooFoBerry/post-receive-hooks/blob/master/api.git/hooks/post-receive) 
my team ended up using.

To have our hook bundle for us we basically want to run `bundle install` like we
discussed earlier.  But, the hook is run in a different context then when you
ssh in as a user (root or otherwise) so without further instruction, our hook
won't be able to find bundler and will fail out.  To load up bundler we
re-source the script to call in RVM:

```
[[ -s "/usr/local/rvm/bin/rvm" ]] && . "/usr/local/rvm/bin/rvm"
```

and then call bundle with the full path to the bin script:

```
/usr/local/rvm/gems/ruby-2.0.0-p353@global/bin/bundle install --deployment
```

In this way bundle will be on our $PATH and will be runnable.  One other thing
to note is that we ended up using bash instead of shell so we changed the first
line of the script to be `#!/bin/bash` instead of `#!/bin/sh`.  If anyone has
a good example of a similar script to this one written in shell please let me
know.

With those updates, the next time we push to the server, bundler will run
automatically allowing us to stay off the server.

Here's an example we ended up with:

```
#!/bin/bash

APP_PATH=/var/www/foofoberry/api
GIT_DIR=/var/repo/foofoberry/api.git

[[ -s "/usr/local/rvm/bin/rvm" ]] && . "/usr/local/rvm/bin/rvm"

git --work-tree=${APP_PATH} --git-dir=${GIT_DIR} checkout -f
echo "==========  CODE DEPLOYED =========="

cd ${APP_PATH}
/usr/local/rvm/gems/ruby-2.0.0-p353@global/bin/bundle install --deployment
echo "==========  BUNDLED  =========="

echo "==========  RESTARTING APP  =========="
mkdir -p tmp
touch tmp/restart.txt
echo "==========  APP RESTARTED  =========="
```

The part about restart.txt is a convenience Passenger provides allowing us to
restart just this Passenger app and allow other running Passenger apps to be
uninterrupted.  Before we learned this trick we were wholesale restarting Nginx
(not as fun).

### Additional resources

There is a lot more we could add to this script such as:

- asset compilation (using the Rails Asset Pipeline)
- database migrations (using rake)
- error checking

The below links should be helpful if you would like to learn more about how to
make your deploy process more enjoyable.

- [http://git-scm.com/book/en/Customizing-Git-Git-Hooks](http://git-scm.com/book/en/Customizing-Git-Git-Hooks)
- [http://githooks.com](http://githooks.com)
