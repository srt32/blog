---
title: Workflows for Writing Migrations With Rollbacks in Mind
date: 2014-06-02 01:14 UTC
tags: rails, rake
---

This post was originally published on [thoughtbot's blog](http://robots.thoughtbot.com/workflows-for-writing-migrations-with-rollbacks-in-mind)


thoughtbot's [dotfiles](https://github.com/thoughtbot/dotfiles) include an [alias]
(https://github.com/thoughtbot/dotfiles/blob/master/aliases#L27)
mapping `migrate` to

`rake db:migrate db:rollback && rake db:migrate`

It is also a good practice in our workflow to use the `migrate` alias in place
of `rake db:migrate`. Let's explore what this command does and why we recommend
it.

## Reversibility

`rake db:migrate db:rollback` will run any pending migrations and then attempt
to roll back those migrations. If the migration and rollback are successful
(signified by `&&`), `rake db:migrate` will then run the
migration again. If the command is not successful, an error will be thrown. In
this way we can be confident our migrations' `down` commands are syntactically
valid (no typos) and logically valid (no invalid castings of values, for example).

Beyond having a short alias in place of a multi-word rake command, ensuring
that each migration we write is reversible is very handy for iterative
development locally and for dreaded production rollbacks.

## An Exception

One time when we break this guideline is when we explicitly require an
irreversible migration because the transformation is destructive.
In the special circumstances when this behavior is required, the
`down` command should raise an `ActiveRecord::IrreversibleMigration` exception
as stated by the
[docs](http://api.rubyonrails.org/classes/ActiveRecord/Migration.html). As a
result, a rollback is impossible and we'll run `rake db:migrate` instead of `migrate`.

## What's next

We have found having a frictionless way of running migrations in both
directions has saved us lots of time and headaches.

If you'd like to learn more about handling irreversible migrations check out
the following resources:

  * ActiveRecord::Migration [documention](http://api.rubyonrails.org/classes/ActiveRecord/Migration.html)
  * [Rails Guides on Migrations](http://guides.rubyonrails.org/migrations.html)

Happy migrating!

