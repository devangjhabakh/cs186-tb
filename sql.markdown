---
layout: page
title: SQL Part 1 - Basic Queries
permalink: /basicsqlqueries/
---

## 1. Database Lingo

Relational databases are made up of **tables** (aka relations). A table has a name (we’ll call this one
`Person`) and looks like this:

| name | age | num_dogs |
|:---- |:----|:---------|
| Ace  | 20  | 4        |
| Ada  | 18  | 3        |
| Ben  | 7   | 2        |
| Cho  | 27  | 3        |

Tables have rows (aka tuples) and columns (aka attributes). In this example, the columns are `name`,
`age`, `num dogs`.

## 2. Querying a Table

The most fundamental SQL query looks like this:

```bash
SELECT <columns>
FROM <tb1>;
```

The `FROM` clause tells SQL which table you’re interested in, and the `SELECT` clause tells SQL which
columns of that table you want to see. For example, consider a table Person(`name`, `age`, `num_dogs`)
containing the data below:

| name | age | num_dogs |
|:---- |:----|:---------|
| Ace  | 20  | 4        |
| Ada  | 18  | 3        |
| Ben  | 7   | 2        |
| Cho  | 27  | 3        |

If we executed this SQL query:

```bash
SELECT name , num dogs
FROM Person ;
```

then we could get the following output.

| name | age | num_dogs |
|:---- |:----|:---------|
| Ace  | 20  | 4        |
| Ada  | 18  | 3        |
| Ben  | 7   | 2        |
| Cho  | 27  | 3        |

Optionally, we can also add a `DISTINCT` keyword to the `SELECT` statement which removes
duplicate rows before output. If we execute the following query to the previous example, the
output does not change because all rows are already unique:

```bash
SELECT DISTINCT name , num dogs
FROM Person ;
```

| name | age | num_dogs |
|:---- |:----|:---------|
| Ace  | 20  | 4        |
| Ada  | 18  | 3        |
| Ben  | 7   | 2        |
| Cho  | 27  | 3        |

In SQL, however, the order of the rows is nondeterministic unless the query contains an `ORDER BY`
(we’ll get to this later). So the following output is equally valid:

| name | age | num_dogs |
|:---- |:----|:---------|
| Ben  | 7   | 2        |
| Cho  | 27  | 3        |
| Ace  | 20  | 4        |
| Ada  | 18  | 3        |

In fact, any ordering of those 4 rows is correct – so unless your query contains an `ORDER BY` clause,
don’t make any assumptions about the order of your results.

## 3. Filtering out uninteresting rows

Frequently we are interested in only a subset of the data available to us. That is, even though we
might have data about many people or things, we often only want to see the data that we have
about very specific people or things. This is where the `WHERE` clause comes in handy; it lets us
specify which specific rows of our table we’re interested in. Here’s the syntax:

```bash
SELECT <columns>
FROM <tbl>
WHERE <predicate> ;
```

Once again, let’s consider our table Person(`name`, `age`, `num_dogs`). Suppose we want to see how
many dogs each person owns — same as before — but this time we only care about the dog-owners
who are adults. Let’s walk through this SQL query:

```bash
SELECT name, num_dogs
FROM Person
WHERE age >= 18 ;
```

When reasoning about query execution you can use the following rule: each clause in a SQL query
happens in the order it’s written, except for `SELECT` which happens last. This is not necessarily
the order the database actually does these operations (more on this later in the semester), but it
is easy to think about and will always give us the correct answer. The `FROM` clause tells us we’re
interested in the Person table, so this is the table we start with:

| name | age | num_dogs |
|:---- |:----|:---------|
| Ace  | 20  | 4        |
| Ada  | 18  | 3        |
| Ben  | 7   | 2        |
| Cho  | 27  | 3        |

Next we move on to the `WHERE` clause. It tells us that we only want to keep the rows satisfying the
predicate age >= 18, so we remove the row with Ben, and are left with the following table:

| name | age | num_dogs |
|:---- |:----|:---------|
| Ace  | 20  | 4        |
| Ada  | 18  | 3        |
| Cho  | 27  | 3        |

And finally, we `SELECT` the columns `name` and `num_dogs` to obtain our final result. (Again, any
permutation of this result is equally valid so you shouldn’t make any assumptions about the order
of the rows.) This gives us our final table:

| name | num_dogs |
|:---- |:---------|
| Ace  | 4        |
| Ada  | 3        |
| Cho  | 3        |

## 4. Boolean operators

If you want to filter on more complicated predicates, you can use the boolean operators `NOT`,
`AND`, and `OR`. For instance, if we only cared about dog-owners who are not only adults, but also
own more than 3 dogs, then we would write the following query:

```bash
SELECT name, num_dogs
FROM Person
WHERE age >= 18
AND num dogs > 3;
```

As in Python, this is the order of evaluation for boolean operators:
1. `NOT`
1. `AND`
1. `OR`

That said, it is good practice to avoid ambiguity by adding parentheses even when they are not
strictly necessary.

## 5. Filtering Null Values

In SQL, there is a special value called NULL, which can be used as a value for any data type, and
represents an “unknown” or “missing” value.

Bear in mind that some values in your database may be `NULL` whether you like it or not, so
it’s good to know how SQL handles them. It pretty much boils down to the following rules:

- If you do anything with `NULL`, you’ll just get `NULL`. For instance if x is `NULL`, then x > 3, 1
= x, and x + 4 all evaluate to `NULL`. Even x = `NULL` would evaluate to `NULL`; if you want to
check whether x is `NULL`, then write x IS `NULL` or x IS NOT `NULL` instead.
- `NULL` is falsey, meaning that `WHERE NULL` is just like `WHERE FALSE`. The row in question does
not get included.
- `NULL` short-circuits with boolean operators. That means a boolean expression involving `NULL`
will evaluate to:
    - `TRUE`, if it’d evaluate to `TRUE` regardless of whether the NULL value is really `TRUE` or
`FALSE`.
    - `FALSE`, if it’d evaluate to `FALSE` regardless of whether the `NULL` value is really `TRUE` or
`FALSE`.
    - Or `NULL`, if it depends on the `NULL` value.

Let's walk through this query as an example:

```bash
SELECT name, num_dogs
FROM Person
WHERE age <= 20
OR num dogs = 3;
```

Let’s assume we change some values to NULL, so after evaluating the FROM clause we are left with:

| name | age   | num_dogs |
|:---- |:------|:---------|
| Ace  | 20    | 4        |
| Ada  | NULL  | 3        |
| Ben  | NULL  | NULL     |
| Cho  | 27    | NULL     |

Next we move on to the `WHERE` clause. It tells us that we only want to keep the rows satisfying the
predicate `age` <= 20 OR `num_dogs` = 3. Let’s consider each row one at a time:

- For Ace, `age` <= 20 evaluates to `TRUE` so the claim is satisfied.
- For Ada, `age` <= 20 evaluates to `NULL` but `num_dogs` = 3 evaluates to `TRUE` so the claim is
satisfied.
- For Ben, `age` <= 20 evaluates to `NULL` and `num_dogs` = 3 evaluates to `NULL` so the overall
expression is `NULL` which has a falsey value.
- For Cho, `age` <= 20 evaluates to `FALSE` and `num_dogs` = 3 evaluates to `NULL` so the overall
expression evaluates to `NULL` (because it depends on the value of the `NULL`). Because `NULL`
is falsey, this row will be excluded.

Thus we keep only Ace and Ada.