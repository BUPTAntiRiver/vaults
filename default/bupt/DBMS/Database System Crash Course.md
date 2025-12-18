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

I am also going to list their corresponding SQL code too:
```sql
-- select
SELECT *
FROM instructor
WHERE dept_name =
```

```
```