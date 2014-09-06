---
title: "Writing Functional SQL in PostreSQL: Don't do it"
date: 2014-09-06 20:29 UTC
tags: sql, postgresql
---

During some recent PostgreSQL [rabbit hole
digging](http://ledgersmbdev.blogspot.com/2014/08/math-and-sql-part-1-introduction.html)
I discovered the concept of 'functional query notation' and it mildly blew my
mind.

## The Question

Assuming we have two tables:

  1. `contacts` table having columns: `id` and `name`
  2. `conversations` table having columns: `id` and `contact_id` (FK to `contacts.id`)

The following two queries return the same data:

### dot notation (pretty typical):

```
  SELECT contacts.name, contacts.id, conversations.id
  FROM contacts
  INNER JOIN conversations
  ON contacts.id = conversations.contact_id;
```

and

### functional notation (pretty atypical):

```
  SELECT contacts.name, contacts.id, conversations.id
  FROM contacts
  INNER JOIN conversations
  ON id(contacts) = contact_id(conversations);
```

This second approach raised some questions:

  * Why and how does this second approach work?
  * Is this syntax in the SQL standard or in only PostgreSQL?
  * Is it performant? If so, why is it not used more widely?

I asked these questions on
[StackOverflow](http://stackoverflow.com/questions/25396772/using-functional-notation-in-postgresql-queries-instead-of-dot-notation)
and I will summarize my findings here.

## Why and how?

First of all, it works. That point is interesting in itself. The next idea to
wrap your head around is that using the functional notation approach considers
the column names (`id` and `contact_id` in the example above) to be functions.
Why does it make sense that we can essentially alias column names to be
functions?

We can think of functions as an operation (or query) that given an
input X is guaranteed to return Y (assuming the underlying data is unchanged).
Think of a SUM(). Adding 1 plus 2 is always 3. It's the same way with columns
(again assuming constant data). If I ask for all the conversations with a
contact_id of 1, I will always get the same set of records back. Bam, columns
are functions. Check out [this article from the
docs](http://www.postgresql.org/docs/current/interactive/xfunc-sql.html#XFUNC-SQL-COMPOSITE-FUNCTIONS)
for an example of how to write a custom function.

## Is this syntax in the SQL standard or in only PostgreSQL?

Neither StackOverflow community members nor I could find any reference to this
functional notation in the SQL standard so the answer seems to be no.
Functional notation of this type is an extension of the standard and
implemented only in PostgreSQL.

## Is it performant? If so, why is it not used more widely?

In short, yes, but this notation can be confusing to developers and there are
potential name collisions with column names and function names so it is not
widely used.

## Wrapping up

Do you use functional notation in your day to day? If so feel
free to jump into the
[thread](http://stackoverflow.com/questions/25396772/using-functional-notation-in-postgresql-queries-instead-of-dot-notation)
to add more detail or understanding.
