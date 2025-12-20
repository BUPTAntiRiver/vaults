# chapter 1 introduction
let's clarify some concepts first:
- database (db): a collection of **interrelated data**, stored as files.
- database management system (dbms): a program to access db. responsible for **definition** of structures for storage of information, **data manipulation** mechanisms and **safety** mechanisms.

## database engine (dbms)
**definition**: a dbms is partitioned into modules that deal with each responsibility.
- query processor
- storage manager
- transaction management

they will be introduced later.
# chapter 2 relational model
## structure of relational database (syntax)
**definition**: a relational database consists of a **collection of tables**. attributes as columns, tuples as rows.
the order of tuples is irrelevant, attribute orders are also irrelevant.
### schema
**definition**: the logical structure of database.
it looks like: `instructor (id, name, dept_name, salary)`
the attributes are in parenthesis.
### key
**definition**: super key is a set of one or more attributes, that can be used to *identify uniquely* a tuple in the relation.
**candidate key** is the *minimal* super key.
**primary key** is a candidate key as *principle means* to identify tuples, values on the primary key are not permitted to be **null**. e.g. we usually choose `id` as primary key.
primary attributes are the attributes in candidate keys.
**foreign key**: describing the relationship among relations, e.g. a *instructor* has `dept_name` which is the **primary key** of another schema *department*, then `dept_name` is called a foreign key from instructor to department. it should be the primary key of the referenced schema!
## integrity constraints (semantic)
## relational algebra
six basic operators:
- select: $\sigma$
- project: $\prod$
- cartesian product: $\times$
- union: $\cup$
- set difference: $-$
- rename: $\rho$

i am also going to list some corresponding sql code too:
```sql
-- select
select *
from instructor
where dept_name = "physics"

-- project
select name, id
from instructor

-- cartesian product
select *
from instructor, teaches

-- join, a combination of select and cartesian product
select *
from instructor natural join teaches
```

different queries may have same results, but differs in performance, which leads to query optimization, discussed later.
# chapter 3 introduction to sql
**definition**: structured query language, widely used query language, but not all examples work on particular systems.
sql can do a lot of things with db:
- query
- manipulation
- definition
- control
- transaction processing
- etc
## Definition
we create a new table with `create table`, google the syntax.
one thing needs to mention here is we can add integrity constraints in create table, like:
- defining primary key
- defining foreign key
- set some attributes to be not null

to remove table, we just `drop` it.
to modify attributes, we can use `alter` to add or drop specific attributes.
typically tuples are stored row by row (row major), which makes it very slow to modify attributes, because we are modifying whole column, which requires modification across all rows. some new structure store data column major to tackle such case.
## Query
use `select` clause to query, we can add some description to the attributes we want to select, like `all`, `distinct` (duplicate is allowed in sql).
if we select from two table, then we are actually selecting from their cartesian product.
rather than cartesian product (outer join), inner join (natural join) is more common, which only care about the tuples has something in common. you may use `join` `on` or add some restrictions in `where` clause, the previous one is recommended.
the running order of sql queries is:
1. from + join
2. where
3. group by
4. having
5. select
6. order by
7. limit

we can use `as` to rename relations and attributes. for string operations we can use `like` to do pattern matching:
- `%` to match any string
- `_` to match any character
- these two special char can combine with patterns like `%comp%___` to match string has `comp` as substring and has at least three chars after it.

the set operations mentioned in relational algebra can has keywords like `union`, `interset`, `except`.
there are also aggregate functions that operate on values of **column**, but return **a value**, like `avg,min,max,sum,count`. we can combine aggregate functions and `group by` to perform finer-grained operations. it is also okay to just use it over the whole table. one note when using `group by`, attributes in select clause **outside** of aggregate functions *must appear in group by list*.
we also have keywords like `some,all,exist,unique`.
we can use sub queries in `from` clause to perform more complex operations. But this is not recommended, `with` clause can create table ahead of time, reduce the cost of running sub query again and again.
## Manipulation
### DELETE
Similar to select, use `from` clause.
### INSERT
`insert into`, can use `value` to insert single tuple or use `select` to insert multiple tuples.
### UPDATE
`update` to determine which table to update, and `set` to define how to update the value. We can also use `from` clause to use data from other table.
# Chapter 4 Intermediate SQL
## Join
There are many join methods. We can use `on` to utilized predicates just like `where` clause.
We also have **outer join** which extends join operation avoids loss of information. It will keep the not matched tuples and pad it with null. Since then, we have left outer, right outer and full outer.
## View
Usually we don't want client to see the full table but only the data they want. So we may select some of the attributes, or we can use `create view` to do this explicitly.
In previous chapters, I was using table to call these selected views, actually they are virtual tables.
You can materialize views, but after that, if we update dependent tables, the view will not change, so we will spend extra effort to maintain it by ourselves. But this enables use to update view solely too.
## Transactions
**Definition**: a transaction consists of a *sequence* of query and/or update statements and it is a *unit* of work. (similar to what we learned in OS)
**Properties**: Atomicity, Consistency, Isolation, Durability.
```sql
DECLARE @transfer_name varchar(10)
	SET @transfer_name = "I-transfer-from-A-to-B"
	BEGIN TRANSACTION @transfer_name
	USE ACCOUNT
	GO
	UPDATE A
		SET balance = balance – 50
		WHERE branch_name = ‘Brooklyn’
	UPDATE B
		SET balance = balance + 50
		WHERE branch_name = ‘Brooklyn’
	GO
	COMMIT TRANSACTION @transfer_name
	GO
```
## Integrity Constraints
**Definition**: used to guard DB from accidental damage.
The constraints on single DB can be:
- primary key
- not null
- unique
- check(P), where P is a predicate

Reference also shows integrity constraints. The foreign key must be the subset of referenced table's primary key. Primary key, foreign key are also part of `create table` syntax.
There is one more thing called `cascade`, we can set `UPDATE cascade` and `DELETE cascade`, so that when update or delete on the referenced table, it will also affect the table that references it. e.g. we have department table and course table, course references department on `dept_name`, when we change it in department table, it will also change course when `UPDATE cascade` is set.
When initializing information in two table with foreign references, we should always insert the referenced data before inserting the table references it. Or we can initialize the data with null first, then update it again.
# Chapter 5 Advanced SQL
Use SQL with programming language.
# Chapter 6 E-R Model
**Definition**: the entity-relationship model, which facilitates database design. Represents the overall logical structure of DB, useful in mapping the meanings and interactions of *real world enterprises* onto a *conceptual schema*.
## Concepts
ER model employs three concepts:
- entity sets
- relationship sets
- attributes

**Entities** are specific people or objects represented by attributes. e.g. an instructor entity can be represented by a instructor id and instructor name.
**Relationships** represents associations between the entities in real world enterprise. e.g. we may have advisor relationship between an instructor and a student.
**Attributes** can also be associated with a relationship set. e.g. the advisor relationship may have a data attribute to tell when the relationship starts.
**Roles** can be used to label entity sets in relationship sets, which makes thing clearer.
## Constraints
### Cardinalities Constraints
The mapping in relationship may be constrained like in database. So we may have one-to-one, one-to-many, many-to-one, many-to-many.
When drawing the ER diagram, we use *directed* line to signify "one" and *undirected* line to signify "many" between the relationship set and entity sets.
### Participation Constraints
For cases like every entity must have at least one relationship.
We just write it in the ER diagram on the line between entity set and relationship set like: 0..\*. In the instructor case above, we write it on the line between instructor and advisor, it means an instructor has at least 0 and at most many students. So we have lowest bound on left side and highest bound on right side of the "..".
### Primary Key
We also have things like primary keys in ER diagram, they have same definition as we talked before.
## Reduce Into Relation Schema
How to reduce ER diagram into relation schema?
### Entity Set
For **strong entity sets**, they can reduce to relation schema with the same attributes.
For **weak entity sets** (depends on other entities, like section to course), they need to *include a column for the primary key* of the identifying strong entity set.
For composite attributes, we flatten it with separate ones.
### Relationship Set
The reduction of relationship sets is strongly dependent on the mapping cardinality constraint and partial constraint.
#### Many-to-Many
A many-to-many relationship set is represented as a table with columns for the primary keys of two participating entity sets, and any descriptive attributes of the relationship set.
Only in this case, relationship set can **have attributes**.
#### Many-to-One, One-to-Many
Many-to-one and one-to-many relationship sets that **total on the many-side** can be represented by adding an extra attribute to the many side, containing the primary key of the one side.
For **partial on the many-side**, we treat it like many-to-many, build a new relationship table.
#### One-to-One
For one-to-one relationship sets, either side can be chosen to act as the many side. An extra attribute can be added to either of the tables corresponding to the two entity sets.
# Chapter 7 Relational DB Design
