---
title: Array arithmetic in Ruby
date: 2013-10-21 19:24 UTC
tags: ruby, arrays, array, array arithmetic
---

I was digging around in some Rake source code this morning to learn about
navigating file structure and ran down a rabbit hole of array math.  I found some pretty neat things.

With Ruby#Array you can do something like the below:

```
ARRAY_METHODS = (Array.instance_methods - Object.instance_methods)
```

giving you the methods that are direct descendants of Array (and not inherited
from Object).  +1 for reading source code.

And back with the file traversal idea in mind I found a way to recursively list all the files in a directory without shipping out to the shell.

```
files_and_directories = Dir.glob("./test/**/*") # => Array
just_directories = Dir.glob("./test/**") # => Array
just_files = files_and_directories - just_directories # => Array of the
difference
```

These sorts of operations should have some pretty helpful applications when dealing with
[matrices](http://rubylearning.com/blog/2013/04/04/ruby-matrix-the-forgotten-library/),
which I look forward to exploring more in the future.
