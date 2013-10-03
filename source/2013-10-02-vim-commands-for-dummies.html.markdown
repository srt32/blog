---
title: Vim commands for dummies
date: 2013-10-02 23:17 UTC
tags: vim, Vim, editors, productivity, learning
---

I've been using Vim over the past couple weeks and between spurts of deleting
dozens of lines of code by accident, I've actually learned the basics.  The
learning curve is steep but the productivity gains are noticeable.

My approach to learning is know the minimal amount of commands and force
yourself to use Vim.  Then, when you find youself doing something over and over
again (or just a particularly annoying operation) go learn the command to fix that
particular problem.  Reviewing a massive commands cheatsheet is just
overwhelming and not helpful.  Learn (and commit to memory) one command at
a time.

Below are the few commands you'll need to get started.  Get these commands under
your belt and start branching out to more advanced (and fun) commands when you're
ready.

###The basics
1. :E

  opens the directory you're in.  Basically the equivalent of command line 'cd'
2. :e <directory_path>

  opens the given directory
3. v <make a selection> d p

  pressing 'v' puts you into visual mode where you can navigate around using the
  hjkl keys and make a selection.  Then pressing 'd' (for delete) will remove
  the code you highlighted and place it in Vim's clipboard.  Then pressing
  p will paste the code wherever your pointer is.  This operation is
  basically the equivalent of copy, cut, paste.
4. v <make a selection> y p

  This set of commands is very similar to vdp but instead of deleting the code
  it leaves it in place and yanks it to the clipboard.  This operation is
  basically the equivalent of copy, paste (without the cut).
5. :q

  This command quits your Vim session.  If you have unsaved changes (see :w)
  you'll be prompted about the unsaved work and if you want to force the quit
  you can run q!.
6. :w

  This operation writes the changes to the file (also know as save). A frequent
  command you'll run is :wq, which saves changes and quits the file all in one
  swoop.
7. hjkl keys

  These keys (which are home keys) move your cursor left,down,up, and right
  (respectively).
  
###Extra credit
1. :%s/foo/bar/gc

  This command is your basic find and replace.  There are all sorts of nuances
  that you can read more about
  [here](http://vim.wikia.com/wiki/Search_and_replace) but to get started this
  command will look for 'foo' and replace it with 'bar' globally (that's what
  the g is for).  Importantly, it will confirm (that's what the c is for) before
  each operation so you can check that you really want to replace the current
  selection.

Have fun.
