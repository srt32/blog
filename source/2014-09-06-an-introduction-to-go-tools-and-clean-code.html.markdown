---
title: An Introduction to Go Tools and Clean Code
date: 2014-09-06 21:46 UTC
tags: golang, tooling
---

I've begun learning [Go](http://golang.org/) and while the language has been
fun to learn and use, figuring out all the tooling has been less than simple.
Here's a quick overview of a sample tool chain to get you started with
community acceptable code.

### Step 1: the compiler

Coming from a dynamically typed background, types and
compile errors have been enjoyable (I hope the feeling lasts).

### Step 2: go fmt

This tool automatically formats Go source code and as the
[docs](http://blog.golang.org/race-detector) explain, it:

  * removes minor formatting concerns
  * makes code consistent across the community
  * allows diffs to show only real changes
  * removes the ability to squabble over spaces or tabs

It's important to realize that this tool manipulates your code. You really
shouldn't commit any code without running `go fmt` first.

### Step 3: go vet

This tool uses heuristics to catch potential issues not possible to be caught
by the compiler and is the next step in clean code. This tool does not edit
your code but rather prints out potential issues that should be inspected. An
example issue is passing the wrong format type to a `Printf` statement. The
[docs](https://godoc.org/code.google.com/p/go.tools/cmd/vet) list the possible
cases handled and describe the various flags available for your use.

### Step 4: golint

This [tool](https://github.com/golang/lint) does not edit your code either but
prints out likely style violations such as returning inside an else block
instead of using a guard clause (as explained by [in this
post](http://engineeredweb.com/blog/2014/golint/)).

## Want to learn more?

  * Many of the non-compiler enforced 'rules' are enumerated in [Go Code Review
    Comments](https://code.google.com/p/go-wiki/wiki/CodeReviewComments)
  * Experiment with [pre-commit hooks](http://golang.org/misc/git/pre-commit)
    that force you to use the tools.
  * For next steps, explore the [race detector](http://blog.golang.org/race-detector)
