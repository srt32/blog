---
title: Run a single Minitest test
date: 2013-10-21 14:17 UTC
tags: testing, minitest, ruby
---

Being able to single out a particular minitest test from a full suite can be
a huge time saver when you're in the middle of debugging a problem and all you
care about is one test.  Similar to RSpec, Minitest allows you to run just one
test with the following command:

```
ruby /path/to/file/name.rb --name name_of_test_method
```

Or for an example:

```
ruby test/spec/user_test.rb --name test_it_can_sign_in
```

I've found narrowing down the tests running to be particularly helpful when
you're seeing odd behavior in application code from only a single test but
others tests are running fine.  I frequently will throw in a `binding.pry` into
the troublesome part of the code but then need to dodge around the other tests
that also exercise that code.  Give it a try!
