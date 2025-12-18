# Chapter 1 Introduction
Let's clarify some concepts first:
- Database (DB): a collection of **interrelated data**, stored as files.
- Database Management System (DBMS): a program to access DB. Responsible for **definition** of structures for storage of information, **data manipulation** mechanisms and **safety** mechanisms.

## Database Engine (DBMS)
**Definition**: A DBMS is partitioned into modules that deal with each responsibility.
- query processor
- storage manager
- transaction management

They will be introduced later.
# Chapter 2 Relational Model
## Structure of Relational Database (syntax)
**Definition**: a relational database consists of a **collection of tables**. Attributes as columns, tuples as rows.
The order of tuples is irrelevant, attribute orders are also irrelevant.
### Schema
**Definition**: the logical structure of database.
It looks like: `instructor (ID, name, dept_name, salary)`
The attributes are in parenthesis.
### Key
**Definition**: super key is a set of one or more attributes, that can be used to *identify uniquely* a tuple in the relation.
**Candidate Key** is the *minimal* super key.
**Primary key** is a candidate key as *principle means* to identify tuples, values on the primary key are not permitted to be **null**. e.g. we usually choose `id` as primary key.
Primary attributes are the attributes in candidate keys.
**Foreign Key**: describing the relationship among relations, e.g. a *instructor* has `dept_name` which is the **primary key** of another schema *department*, then `dept_name` is called a foreign key from instructor to department. It should be the primary key of the referenced schema!
## Integrity Constraints (semantic)
## Relational Algebra
Six basic operators:
- select: $\sigma$
- project: $\prod$
- Cartesian product: $\times$
- Union: $\cup$
- set difference: $-$
- rename: $\rho$

I am also going to list some corresponding SQL code too:
```sql
-- select
SELECT *
FROM instructor
WHERE dept_name = "Physics"

-- project
SELECT name, id
FROM instructor

-- Cartesian product
SELECT *
FROM instructor, teaches

-- Join, a combination of select and Cartesian product
SELECT *
FROM instructor NATURAL JOIN teaches
```

Different queries may have same results, but differs in performance, which leads to query optimization, discussed later.
# Chapter 3 Introduction to SQL
**Definition**: Structured Query Language, widely used query language, but not all examples work on particular systems.
SQL can do a lot of things with DB:
- query
- manipulation
- definition
- control
- transaction processing
- etc
## Definition
We create a new table with `create table`, google the syntax.
One thing needs to mention here is we can add integrity constraints in create table, like:
- Defining primary key
- Defining foreign key
- Set some attributes to be not null

To remove table, we just `drop` it.
To modify attributes, we can use `alter` to add or drop specific attributes.
Typically tuples are stored row by row (row major), which makes it very slow to modify attributes, because we are modifying whole column, which requires modification across all rows. Some new structure store data column major to tackle such case.
## Query
Use `select` clause to query, we can add some description to the attributes we want to select, like `all`, `distinct` (duplicate is allowed in SQL).
If we select from two table, then we are actually selecting from their Cartesian product.
Rather than Cartesian product (outer join), inner join (natural join) is more common, which only care about the tuples has something in common. You may use `join` `on` or add some restrictions in `where` clause, the previous one is recommended.
The running order of SQL queries is:
1. FROM + JOIN
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. ORDER BY
7. LIMIT

We can use `as` to rename relations and attributes. For string operations we can use `like` to do pattern matching:
- `%` to match any string
- `_` to match any character
- these two special char can combine with patterns like `%comp%___` to match string has `comp` as substring and has at least three chars after it.

The set operations mentioned in relational algebra can has keywords like `union`, `interset`, `except`.