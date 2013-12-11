---
title: "It all comes together: Ruby, JS, and functional programming"
date: 2013-12-11 15:10 UTC
tags: ruby, javascript, clojure, functional programming
---

Whenever you learn a new word, you inevitably see that word in the newspaper
a day later.  You've been seeing that word all along you just didn't know it was
there.  I had a similar experience with programming yesterday when Ruby,
Javascript, and [functional
programming](http://en.wikipedia.org/wiki/Functional_programming) concepts (by way of Clojure and Ruby
lambdas) all came together.  It was an exciting day...

###Javascript and Ajax
I took my first deep dive into javascript a couple weeks ago doing exercises on
[exercism.io](http://exercism.io/) and [Javascript
Patterns](http://www.amazon.com/JavaScript-Patterns-Stoyan-Stefanov/dp/0596806752).
I am proud to say the curly braces no longer scare me.  Javascript logically led
me to Ajax in Rails.  We spent some time writing simple Ajax calls using JQuery
similar to those below:

```
$.ajax({
  type: 'POST',
  url: '/articles',
  success: function(data){
    alert("ok");
    $("#title").html(data);
  }
});
```

The concept of the success callback function was new but seemed reasonable
enough.

###Functional Programming and Clojure
Also in the past few weeks I started working my way through
[Functional Programming for the Object-Oriented Programmer](https://leanpub.com/fp-oo),
which is the [Ruby Rogues](http://rubyrogues.com/) next book club book.  This
book is based in Clojure and I considered reading it to be a type of
procrastination from doing my Rails project work.

###Refactoring Rails Controllers using lambdas
Refactoring post: 
http://www.simontaranto.com/2013/11/15/a-refactoring-epiphany.html
