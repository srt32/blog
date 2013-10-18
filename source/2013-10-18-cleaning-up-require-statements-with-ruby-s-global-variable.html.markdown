---
title: "Cleaning up require statements with Ruby's $: global variable"
date: 2013-10-18 15:18 UTC
tags: ruby, global vars, $:, require, testing
---

Ever run across something like the below line in helper files?

```
$:.unshift File.expand_path("../../lib", __FILE__)
```

This little trick allows you to be lazy with your require statements further
down the road.  This line will load onto your path your lib library so that your
downstream require statments can look like

```
require '/class_name'
```

instead of

```
require './lib/class_name'
```

Doesn't seem like much but it does make file dependencies easier to sort out if
you have a big project.

But, how does that code work?  Let's break it down.

* `$:` is the equivalent of `$LOADPATH` and holds an array of paths that your
  files will traverse through when you call `require`.
* `.unshift` says add the following element to the front of the loadpath
* `File.expand_path` will turn the following relative URL into an absolute URL
* `"../../lib"` is the directory path you want to add to the loadpath
* `__FILE__` is the current file.

In other words, this statment ways, from the current file (__FILE__) go up two
directories (../..) and into the lib folder.  Then, make this path absolute
(expand_path) and push it into the loadpath (.unshift).

To learn more, check out this [helpful
post](http://jimneath.org/2010/01/04/cryptic-ruby-global-variables-and-their-meanings.html).
