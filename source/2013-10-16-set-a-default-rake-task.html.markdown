---
title: Set a default rake task
date: 2013-10-16 14:19 UTC
tags: rake
---

Ever found yourself with a large Rakefile and forgot the names of frequently
used tasks?  Well, you can set a default Rake task so when you run `rake` by
itself you'll run that default task.

```

task :default => [:test]

task :test do
  ruby "test/unittest.rb"
end

```
The above is pulled from the [docs](http://rake.rubyforge.org/).  Although it's
a small thing, having a default task set (such as your test suite) can be
helpful.
