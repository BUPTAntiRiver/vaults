# Chapter 1 Introduction

Let's clarify some concepts first:

- Database (DB): a collection of **interrelated data**, stored as files.
- Database management system (DBMS): a program to access db. responsible for **definition** of structures for storage of information, **data manipulation** mechanisms and **safety** mechanisms.

## Database Engine (DBMS)

**Definition**: a DBMS is partitioned into modules that deal with each responsibility.

- query processor
- storage manager
- transaction management

they will be introduced later.

# Chapter 2 Relational Model

## Structure of Relational Database (syntax)

**Definition**: a relational database consists of a **collection of tables**. attributes as columns, tuples as rows.
The order of tuples is irrelevant, attribute orders are also irrelevant.

### Schema

**Definition**: the logical structure of database.
It looks like: `instructor (id, name, dept_name, salary)`
The attributes are in parenthesis.

### Key

**Definition**: super key is a set of one or more attributes, that can be used to _identify uniquely_ a tuple in the relation.
**Candidate key** is the _minimal_ super key.
**Primary key** is a candidate key as _principle means_ to identify tuples, values on the primary key are not permitted to be **null**. e.g. we usually choose `id` as primary key.
primary attributes are the attributes in candidate keys.
**Foreign key**: describing the relationship among relations, e.g. a _instructor_ has `dept_name` which is the **primary key** of another schema _department_, then `dept_name` is called a foreign key from instructor to department. it should be the primary key of the referenced schema!

## Integrity Constraints (semantic)

## Relational Algebra

Six basic operators:

- Select: $\sigma$
- Project: $\prod$
- Cartesian product: $\times$
- Union: $\cup$
- Set difference: $-$
- Rename: $\rho$

I am also going to list some corresponding SQL code too:

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

Different queries may have same results, but differs in performance, which leads to query optimization, discussed later.

# Chapter 3 Introduction to SQL

**Definition**: structured query language, widely used query language, but not all examples work on particular systems.
sql can do a lot of things with db:

- query
- manipulation
- definition
- control
- transaction processing
- etc

## Definition

We create a new table with `create table`, google the syntax.
one thing needs to mention here is we can add integrity constraints in create table, like:

- defining primary key
- defining foreign key
- set some attributes to be not null

To remove table, we just `drop` it.
To modify attributes, we can use `alter` to add or drop specific attributes.
Typically tuples are stored row by row (row major), which makes it very slow to modify attributes, because we are modifying whole column, which requires modification across all rows. some new structure store data column major to tackle such case.

## Query

Use `select` clause to query, we can add some description to the attributes we want to select, like `all`, `distinct` (duplicate is allowed in SQL).
If we select from two table, then we are actually selecting from their Cartesian product.
Rather than Cartesian product (outer join), inner join (natural join) is more common, which only care about the tuples has something in common. you may use `join` `on` or add some restrictions in `where` clause, the previous one is recommended.
The running order of SQL queries is:

1. from + join
2. where
3. group by
4. having
5. select
6. order by
7. limit

We can use `as` to rename relations and attributes. for string operations we can use `like` to do pattern matching:

- `%` to match any string
- `_` to match any character
- these two special char can combine with patterns like `%comp%___` to match string has `comp` as substring and has at least three chars after it.

The set operations mentioned in relational algebra can has keywords like `union`, `interset`, `except`.
There are also aggregate functions that operate on values of **column**, but return **a value**, like `avg,min,max,sum,count`. we can combine aggregate functions and `group by` to perform finer-grained operations. It is also okay to just use it over the whole table. one note when using `group by`, attributes in select clause **outside** of aggregate functions _must appear in group by list_.
We also have keywords like `some,all,exist,unique`.
We can use sub queries in `from` clause to perform more complex operations. But this is not recommended, `with` clause can create table ahead of time, reduce the cost of running sub query again and again.

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
You can materialize views, but after that, if we update the original tables, the view will not change, so we will spend extra effort to maintain it by ourselves. But this enables use to update view solely too.

## Transactions

**Definition**: a transaction consists of a _sequence_ of query and/or update statements and it is a _unit_ of work. (similar to what we learned in OS)
**Properties**: Atomicity, Consistency, Isolation, Durability.

```sql
DECLARE @transfer_name varchar(10)
	SET @transfer_name = "demo"
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
There is one more thing called `cascade`, we can set `UPDATE cascade` and `DELETE cascade`, so that when update or delete on the referenced table, it will also affect the table that references it. e.g. we have department table and course table, course references department on `dept_name`, when we change it in department table, it will also change `dept_name` of courses have same `dept_name` previously when `UPDATE cascade` is set.
When initializing information in two table with foreign references, we should always insert the referenced data before inserting the table references it. Or we can initialize the data with null first, then update it afterwards.

# Chapter 5 Advanced SQL

Use SQL with programming language.

# Chapter 6 E-R Model

**Definition**: the entity-relationship model, which facilitates database design. Represents the overall logical structure of DB, useful in mapping the meanings and interactions of _real world enterprises_ onto a _conceptual schema_.

## Concepts

ER model employs three concepts:

- entity sets
- relationship sets
- attributes

**Entities** are specific people or objects represented by attributes. e.g. an instructor entity can be represented by an instructor id and an instructor name.
**Relationships** represents associations between the entities in real world enterprise. e.g. we may have advisor relationship between an instructor and a student.
**Attributes** can also be associated with a relationship set. e.g. the advisor relationship may have a data attribute to tell when the relationship starts.
**Roles** can be used to label entity sets in relationship sets, which makes thing clearer.

## Constraints

### Cardinalities Constraints

The mapping in relationship may be constrained like in database. So we may have one-to-one, one-to-many, many-to-one, many-to-many.
When drawing the ER diagram, we use _directed_ line to signify "one" and _undirected_ line to signify "many" between the relationship set and entity sets.

### Participation Constraints

For cases like every entity must have at least one relationship.
We just write it in the ER diagram on the line between entity set and relationship set like: 0..\*. In the instructor case above, we write it on the line between instructor and advisor, it means an instructor has at least 0 and at most many students. So we have lowest bound on left side and highest bound on right side of the "..". This is called _Cardinality limits_.

### Primary Key

We also have things like primary keys in ER diagram, they have same definition as we talked before.

## Reduce Into Relation Schema

How to reduce ER diagram into relation schema?

### Entity Set

For **strong entity sets**, they can reduce to relation schema with the same attributes.
For **weak entity sets** (do not have their own primary keys, depend on other entities, like section to course), they need to _include a column for the primary key_ of the identifying strong entity set. e.g. `section(sec_id, semester, year)` becomes `section(course_id, sec_id, semester, year)`.
For composite attributes, we flatten it with separate ones.

### Relationship Set

The reduction of relationship sets is strongly dependent on the mapping cardinality constraint and partial constraint.

#### Many-to-Many

A many-to-many relationship set is represented as a table with columns for the primary keys of two participating entity sets, and any descriptive attributes of the relationship set.
Only in this case, relationship set can **have attributes**.

#### Many-to-One, One-to-Many

Many-to-one and one-to-many relationship sets that **total on the many-side** can be represented by adding an extra attribute to the many side, containing the primary key of the one side. e.g. all instructors have a department, so we can just add a new column `dept_id` to instructor.
For **partial on the many-side**, we treat it like many-to-many, build a new relationship table. Or we will have null values.

#### One-to-One

For one-to-one relationship sets, either side can be chosen to act as the many side. An extra attribute can be added to either of the tables corresponding to the two entity sets.

# Chapter 7 Relational DB Design

This chapter is about how to make our tables in the DB more robust and easy to use.

## Normal Forms

### First Normal Form and Atomic Domains

**Definition**: a relational schema is in **first normal form** if the domains of all attributes in it are _atomic_.
A domain is atomic if its elements are _indivisible_ units. e.g. names can be decomposed into last name, middle name and first name.

### Second Normal Form

**Definition**: the schema is in first normal form and each attributes $A$ meets one of the criteria:

- if it appears in a **candidate key**, it is okay
- if it does not appear in a **candidate key**, it must be completely dependent on a **candidate key**, not _partially_, which means it is dependent on a subset of the candidate key

If $A$ breaks the second rule, this reminds us we can separate the schema into two. e.g. `demo(a_id, a_item, b_id, b_item)` where we have `a_id -> a_item, b_id -> b_item` and candidate key is `(a_id, b_id)`, so `a_item, b_item` both partially depend on candidate key, which means this schema is not 2NF.

### Boyce-Codd Normal Form

**Definition**: a relation schema $R$ is in **BCNF** with respect to a function dependency set if for all function dependencies in it of the form $\alpha\to\beta$ at least one of the following statement holds:

- $\alpha\to\beta$ is trivial, which means $\beta\subseteq\alpha$
- $\alpha$ is a super key

e.g. if we mix instructor and department into a `in_dep` like `(ID, name, dept_name, building)`, then the super key is `(ID, dept_name)`, but we have function dependency like `dept_name -> building`, since `dept_name` is not super key solely, it is not BCNF, it can be decomposed into two distinct table.

### Third Normal Form

**Definition**: 3NF is a weaker form of BCNF, it is 2NF plus for all $\alpha\to\beta$ at least one of the following holds:

- $\alpha\to\beta$ is trivial
- $\alpha$ is a super key
- each attributes in $\beta-\alpha$ is contained in a candidate key

# Chapter 13 Storage Structures

We have two main issues in physical DB:

- data organization, e.g. physical storage structure of data.
- data access, e.g. how to index.

## File Organization

Each relational table is a set of **tuples**.
Each file is a sequences of **records**. So it is natural to store each tuple as a record in DB. But we may have fixed length record and variable length record for things like `varchar`.

### Fixed-length Records

A simple approach is we can just store the records in a contiguous region. When deleting old records, we can delete and compact, or move the bottom one to this position and we can also do nothing but use a free list to track, thought free list allow record to be stored in non-contiguous space, kind of breaking the rule.

### Variable length Records

Variable length attributes can be represented by **fixed** size things like offset and length with actual data stored **after** all fixed length attributes. Null value can be represented by a null-value bitmap.
Another method is we store the size and location info in the header of the block, then store the record from the end of the block to the start. So the free spaces are in the middle. When the records meet the header, the block is full.

## Organization of Records

DB file can be viewed as a set of records at logical level. These records are logically organized as: **heap, sequential, hash, clustering**.

### Heap

Any record can be placed anywhere in the file where there is space for the record:

- There is **no ordering** of records
- Records usually do not move once allocated
- There is **one single file for a relation**

We use a free-space map, it is an array with 1 entry per-block, each entry records fraction of block that is free. We can extend it into second level free-space map to control more blocks.

### Sequential

Records are logically ordered by **search keys**. So the records are stored physically in search key order but may not be exactly the same due to deletion and insertion, we try to make it as close as possible.
The records are chained by pointers, so its actually a link list, but has approximate ordered. Pointers enabled records to be stored in non-contiguous space and to be **reordered from time to time** to maintain sequential order.

### Clustering

Better for search and IO with tables has relationship with each other, but worse for single table operations, due to overheads on other tables.

### Hashing

The file records are stored in _buckets_, the bucket is the address of the record. We have a hash function **on some attributes** (search key), determines the addresses of the records.

## Data Dictionary Storage

Stores the catalog, metadata about data. Supports advanced optimization towards data storage. It can store statistical and descriptive data and so on.

## Data Buffer

Data are stored in disk, which is slow to access, we need to keep them in memory for efficient manipulation.
**Definition**: **portion of main memory** available to store _copies_ of disk blocks.
It is usually called buffer pool, and we have a buffer pool manager to do the allocation scheduling, eviction, etc.

### Replacement Strategy

Least Recently Used, LRU.
Toss-immediate: **free** the space occupied by block as soon as the final tuple of that block has been processed.
Most Recently Used: pin the currently being processed block, after done with it, remove the pin.
Buffer manage can have more advanced strategies with respect to statistical information like data dictionary.

## Column Oriented Storage

Store each attribute of a relation separately.
Pros: reduce IO if only access some of attributes, improve CPU cache performance and compression.
Cons: need to reconstruct tuple and has higher cost when update or delete tuple.

# Chapter 14 Indexing

In this chapter, we are going to talk about how to increase accessing speed to DB with indexing. And the effect on select/update/insert/delete.

## Basic Concepts

The problem is how to locate tuples/records quickly. And **indexing** mechanisms speed up access to desired data, e.g. if we let dept_name be index, it might just equal to the **physical address of record** in DB file instructor.

### Search Key

**Definition** of search key: an attribute or set of attributes used to look up records.
The file might be logically sequential, but physically non-contiguous to be compatible with search key.

### Index File

We can also use a **index file** to do this.
**Definition** of index file: a file consists of records (index entries) of the form, search key and pointer to real data.
And **indexing** is just: mapping from search key to storage locations of the records in disk. This introduces other methods like hash indices. Instead of building a index file that takes extra space cost, we can use a smart hash function that hashes search key to storage locations.

#### Dense and Sparse Index

Dense means we have index record for every search key in the table file, and each value of search key corresponds to an index entry in the index file.
Sparse means index files contains index entries for **only some** search key values.

#### Multi-level Index

If the index file is very large (the table is even larger! So we need index!), we can create multi-level index, so that each time we only put part of the index file into memory, then check the inner index file to locate the record. If the outer index file is still too big to put in, we can create more levels.

## B+ Tree Index

**Definition**: B tree is self-balancing tree structure that allows sequential accesses, insertions, deletions in **logarithmic time**.
B+ tree derives from B tree, its indices are an alternative to the design of indexed-sequential file, it uses multi-level index with advantages of automatically reorganizing itself for insertions and deletions. The indices are stored at the leaf node, the value in intermediate nodes are used for structure maintenance.

### Properties

B+ tree is a rooted tree, with the parameter degree factor n (at most n branches), it is also a balanced tree, which means all paths from root to leaf are of the same length. And:

- Each **internal** node (not root or leaf) has between $\lceil \frac{n}{2} \rceil$ and $n$ children
- Each **leaf** node has between $\left\lceil  \frac{n-1}{2}  \right\rceil$ and $n-1$ search key values.
- Special cases for the root, if it is not a leaf, it has at least 2 children, if it is a leaf, it has between 0 and $n-1$ values.

### Node Structure

The search keys in node are ordered, between the search keys, there are pointers to children for non-leaf nodes, and pointers to records or buckets of records for leaf nodes.
In **LEAF NODE**:

- $P_{i}$ points to a record with search key value $K_{i}$, $i<n-1$
- $P_{n}$ points to the _next leaf node_
- If $L_{i}, L_{j}$ are leaf nodes and $i<j$, then all search key values in $L_{i}$ will be less than or equal to $L_{j}$

In **NON-LEAF NODE**: it is quite similar, the search key values in the node that pointers points to are in between the search key values around it.

## Hash Indices

The records are stored in buckets. **Bucket** is a _unit_ of storage containing records, such as a disk block. Bucket can be obtained from search key using hash function.
**Hash function** is a function from _search key_ $K$ to _bucket address_. Entries with different search key may map to the same bucket, and the entire bucket is sequentially searched to locate an entry.

### Bucket Overflow

There might not be enough buckets or non-uniform distribution of records due to bad hash function or non-uniform distribution of search key, so some of the buckets may be full, we can add overflow buckets to it like link list.

### Comparing

Hashing is better at retrieving records having a specified value of the key. If range queries are more common, the ordered indices are preferred.

## Multiple Indices

We can take one as primary index and sort in this way. Or we can create two ordered version and take intersection (not recommended). If the primary index is **not used** in a query, then even secondary index is used, the query won't be sped up.

## Create Indices

The attributes after `where` `group by` `join` are better to be set as indices.
With indexing, `update` will be much faster, but `insert` and `delete` will be slower because after modification, these two operations might change the structure of B+ tree, so the system will run some transform. But `update` does not have such concern.

# Chapter 15 Query Processing

**Definition**: Query Processing refers to activities _extracting_ data from DB, including:

1. translation of SQL into _physical_ level of the system
2. query optimization
3. actual evaluation of queries

## Overview

**Step 1: Parsing and translation**. Translate each query into a parser tree, the parser will check _syntax_ and _replaces_ the view with relations on which built. Then it is translated into _relational algebra_.
**Step 2: Optimization**. Choose the lowest cost plan with among _equivalent query evaluation plans_.
**Step 3: Evaluation plan execution**. The execution engine runs the query and returns.

## Measure of Query Costs

**Definition**: cost is measured as _the total elapsed_ time for answering query. Factors contribute to time cost are: disk access and network communication.
**Disk access** is the _predominant_ cost, measured by:

1. number of seeks
2. number of block _read_
3. number of block _written_, the cost of write is greater than read

Time cost can be computed with: $b\times t_{T} +s\times t_{S}$, $b$ refers to the number of block transferred from disk, $t_{T}$ refers to time to transfer one block, $s$ refers to the number of seeks, $t_{S}$ refers to time for one seek. When analyzing cost, we can just pay attention to transfer number and seek number.

## Selection Operation

Types of query conditions are different: equality, range, comparison.
We also have different scan algorithms, so will have different cost estimations.

### Linear Scan

Just scan all the blocks and test all records, on average cost $\frac{b_{r}}{2}$ transfer and 1 seek.
It is **suitable** for:

- **heap file** organized relational table
- seeking on **non-index** attributes

### Selections using Indices

Search algorithms with an index, but the selection condition can be on search key or not.
We have two kinds of index: primary and secondary.
The logic of secondary index and primary index is the same, maybe their index attributes are different. The key difference between secondary index and primary index is that, we can _only have one_ physical storage of the table, or that would be much waste of space, so we can only make _primary index layout compatible with the physical layout, and secondary index lose the contiguous property_.
We also has two cases for the predication: it is a key (unique), it is non-key (maybe multiple result).

#### Primary Index, Equality On Key

Such cases need to retrieve a single record.
The cost will be $(h_{i} + 1)\times(t_{T}+t_{S})$, $h_{i}$ refers to the height of the tree.

#### Primary Index, Equality On Non-Key

This case will be slower than previous one, because we may have multiple results.
cost = $h_{i}\times(t_{T}+t_{S})+t_{S}+t_{T}\times b$, $b$ refers to the number of blocks containing matching records.

#### Secondary Index

If equality on key, then it's just like primary index, we only need to find one record, cost is $(h_{i} + 1)\times(t_{T}+t_{S})$.
But if on non-key, it becomes different, because same non-key value may not resides in contiguous space (we are using secondary index!), so the cost becomes $(h_{i} + n)\times(t_{T}+t_{S})$, each time fetching the tuple needs to do random access. $n$ refers to the number of matching records.

### Selections Involving Comparison - Range Search

#### Primary Index

The comparing attribute is already sorted, so we can use **index** to find the **first** tuple that greater than lower bound, and start linear scanning until we find a tuple greater than upper bound.
The cost is $h_{i}\times(t_{T}+t_{S})+t_{S}+t_{T}\times b$, since they are in contiguous region.

#### Secondary Index

The access to data cost also changes from contiguous blocks to physically separated nodes.
cost = $(h_{i} + n)\times(t_{T}+t_{S})$. Linear scan maybe cheaper in this case.

### Complex Selections - Conjunction

#### Using One Index

We have a lot of conditions, we pick one as index, _seek_ the tuples with previous introduced algorithms and _test_ whether or not other conditions are satisfied.

#### Using Composite Index

Try to use multiple-key index if available.

#### Set Operation

Selection each condition to get sets, then do intersection or union over them to get the final results.

## Sorting

Sorting data is needed in `order by` and `join` to improve efficiency.
Records are ordered physically, like the primary key index makes the tuples stored in **ascending order** of primary key.

## Evaluation of Expressions

### Materialization

Generate results of an expression whose inputs are relations or are already computed, materialize (store) it on disk. Start from the lowest level of the parser tree, evaluated one operation at a time. The results of each evaluation is stored to _temporal relations_ on **disk**. The cost is high.

### Pipelining

Pass on tuples to parent operations even as the operation is being executed. This enables parallel execution and no need to store temporary relations to disk.

# Chapter 16 Query Optimization

## Procedure

An evaluation plan is: **what algorithm** is used for each operation, **how execution** of the operations is coordinated.
The optimization procedure is:

1. Generating the _equivalent_ query trees/expressions by transforming relational algebra expressions using _equivalence rules_.
2. Generating the alternative _evaluation plans_ by annotating the resultant equivalent expressions with algorithms for each operations.
3. Choosing the optimal or near optimal plan based on _estimated cost_ by **heuristic optimization** or **cost-based optimization**.

## Equivalence Rules

**Rule 1**. Conjunctive selection operations can be deconstructed into a sequence of individual selections.
**Rule 2**. Selection operations are commutative.
**Rule 3**. Only the final projection is needed, in final projection is subset of inner projection.
**Rule 4**. Theta Join on condition can replace condition over Cartesian product.
**Rule 5**. Theta Join operations are commutative. We should use relation of smaller size as the outer relation to reduce cost (better locality on larger relation).
Why? For each tuple on left hand side, we scan through it and do search on right hand side. So the complexity will be $O(m+m\log_{d}(n))$. $m$ for left, $n$ for right, this clearly shows why we should left $m$ be smaller.
**Rule 6**. Natural join and theta joins are associative.
**Rule 7**. Selection operation distributes over theta join, if a selection only involve the attributes of one of the expressions, it can be done before join.
**Rule 8**. Projection operation distributes over theta join. The core rule is we can do the projections that **retain the attributes going to be used** ahead of time. For example we will use $a$ in theta join and $b,c$ in projection but the relations actually has $a$ to $z$, then, we can drop $d$ to $z$ (by projecting $a,b,c$) since they are not used.
**Rule 9**. The set operations union and intersection are **commutative and associative**.
**Rule 10**. The selection operation distributes over union, intersection and difference.
**Rule 11**. The projection operation distributes over union.

## Join Ordering

**Principle**: use relation with smaller size as outer relation.

## Heuristic Optimization

Practically we use two methods: cost-based and heuristic optimization. We mainly introduce heuristic optimization here.
**Definition**: we choose better plan following heuristics, sub-optimal plan with lower costs may be obtained.
**Principles**: transforms query tree by heuristics to reduce cost.

- perform _selection_ early to reduce the number of tuples
- perform _projection_ early to reduce the number of attributes
- perform _join_ to substitute Cartesian product and selection
- perform _restrictive selection and join_ before other operations, since the most restrictive operations generates the smallest relations

There are many examples in the slides, worth taking a look.

# Chapter 17 Transactions

Transaction is the basic logical unit of DBAS and basic unit of DBMS managing DB. Access of single user to DB appears to be the execution of 1 or multiple transactions. It also represents the concurrent access of multiple users.

## Concepts

**Definition**: transaction is a _unit_ of DBAS. Executing _accesses and updates_ various data items in DB.
A transaction consists of operations delimited by statements like begin and end.

## Transaction Model

Each data access, such as **select, update** in SQL is translated/decomposed into **read** and **write** operations.
DBMS executes read and write operations to fulfill data access in transactions. It allocates **local buffer** in main memory as working area. The DB permanently resides on disk, but some portion is temporarily residing in buffer in main memory.
The _write_ operation does not result in the _immediate update_ of data on disk. It may be _temporarily stored_ in disk buffer.

### Consistency

Execution of a transaction in isolation preserves **consistency** of DB. DB is in a correct DB state (like the total currency keeps the same), before and after the transaction execution.

### Atomicity

Either all operations of transaction are reflected/persisted or none. It is all or zero.

### Isolation

Even though multiple transactions execute concurrently, all transactions seem to execute serially, so the consistence is preserved. Each transaction is unaware of other transactions executing concurrently.

### Durability

Once the transaction has completed, the updates to the DB by the transaction must **persist** even if there are hardware or software failures.

## State Model for Atomicity and Durability

**Aborted**: a transaction may not complete its execution successfully.
**Committed**: it completes its execution successfully.
**Rollback**: the changes by an **aborted** transaction has been **undone**.
**Terminated**: either committed or aborted.

## Transaction Isolation

Schedule on a set of concurrent transactions, specifies the order of the executed operations of concurrent transactions. It consists of _all_ operations of transactions, preserves _the order_ of operations in each individual transaction.
A success transaction will have a **commit** as last statement, a failed transaction will have an **abort** as last statement.

## Serializability

If we process all transactions sequentially, it ensures ACID properties, but lose efficiency, but if we process them concurrently, there might be errors. So we should obey the principle that: do conflict operations sequentially, do non-conflict operations concurrently.

### So what is conflict?

**Definition**: for transaction $T_{i},T_{j}$, there are instructions $I_{i},I_{j}$ that conflict with each other, if and only if for some item $Q$ accessed by both $I_{i},I_{j}$ and at least one of these two instructions is `WRITE(Q)`.
Two schedules are _conflict equivalent_, if they can be transformed into each other by a series of swaps of _non-conflicting_ instructions.
Note that the swap must happen between two transactions, not inside one transaction! Which means swapping should **not change the orders of instruction** in each transaction.

### Identification

Whether a concurrent schedule $S$ is conflict?

1. starting from the concurrent $S$, do the conflict equivalent swap and see whether a serial $S'$ can be obtained
2. precedence graph

The first method is quite easy to understand, let's dive into precedence graph here.
**Definition**: it is a direct graph with transactions as vertices. Arc from $T_{i}$ to $T_{j}$ exists for which one of the three conditions holds:

- $T_{i}$ executes write $Q$ before $T_{j}$ executes read $Q$
- $T_{i}$ executes read $Q$ before $T_{j}$ executes write $Q$
- $T_{i}$ executes write $Q$ before $T_{j}$ executes write $Q$

Then we can infer that a schedule is **conflict serializable** if and only if its precedence graph is **acyclic**.

## Isolation and Atomicity

Two schedules:

- Recoverable scheduling
- Cascadeless scheduling

The two schedule methods will be introduced later.
The requirements are:

- when one transaction failed, all operations that depends on it already executed should rollback or aborted
- ensure **atomicity**

### Recoverable

**Definition**: recoverable schedule is a schedule for each pair of $T_{i}$ and $T_{j}$, if $T_{j}$ reads data item $Q$ previously **written** by $T_{i}$, commit of $T_{i}$ should appear before commit of $T_{j}$.
WHY? This is because $T_{j}$ reads data from $T_{i}$ written data, so if we encounter a failure, we are sure that $T_{j}$ definitely hasn't committed (it is the last to commit). While, if $T_{j}$ commits before $T_{i}$, then it could be possible that $T_{j}$ committed but $T_{i}$ aborted, when it is redone, the value it write to $Q$ maybe different, or it is just canceled then $Q$ never exists. $T_{j}$ will be interacting with **ghost** value, that will be a problem.

### Cascadeless

Before talking about cascadeless, let's take a look at cascading rollback.
**Definition**: a _single_ transaction failure leads to series of transactions rollbacks.
It describes scenarios like $T_{1}$ rollbacks due to failure, so that $T_{2}$ should rollback (read from $T_{1}$ written data), so that $T_{3}$ should rollback (read from $T_{2}$ written data).
Cascading rollback is undesirable, because we need to undo a lot of previous works. This leads to _cascadeless schedules_.
**Definition**: for each pair of $T_{i},T_{j}$, where $T_{j}$ reads $Q$ written by $T_{i}$, and the _commit_ of $T_{i}$ happens before the read. It's just $T_{j}$ only read data from _committed_ $T_{i}$. So that we can avoid a lot of rollback.
If a transaction is not recoverable, then it mustn't be cascadeless.

# Chapter 18 Concurrent Control

When we have multiple concurrent transactions, we apply concurrency control through **transaction scheduling**. We have principles like _serializability_.
Now the question is, how to implement it?

## Locks

Use lock to control concurrent transactions' access to shared item. Acquire, process, release.
We have two kinds of locks:

- Exclusive (X) mode, data item can be _both_ read as well as written.
- Shared (S) mode, data item can _only be read_.

We use lock-S and lock-X to differ. S-lock is compatible with S-lock, all other cases are incompatible.
There are also problems like dead lock and writer starvation due to too many S-lock holders. We should design a smart concurrency control manager to prevent such cases.

## Locking Protocol

**Motivation**: the _orders_ of granting locks and revoking locks must be limited to avoid transaction _deadlock_.
**Locking protocols** restrict the possible schedules **leading to** the conflict serializable schedules.

### Two Phase Locking Protocol (2PL)

**Definition**: $T_{i}$ issues lock and unlock requests in two phases,

- **Growing Phase**: $T_{i}$ may obtain locks but may _not_ release any locks, when $T_{i}$ begins, it enters growing phase
- **Shrinking Phase**: $T_{i}$ may release locks, but may _not_ obtain any locks, when $T_{i}$ releases _the first lock_, it enters shrinking phase

It is quite easy to understand, we can imagine a graph of two phase lock number looks like a mountain.
![[2PL.png]]
The red line denotes when the transaction has acquired to obtain its _final lock_, we call it a _lock point_. The motivation of naming a lock point is that transactions can be serialized in the order of their lock points.

#### Properties

2PL guarantees _conflict serializability_, but does _not_ ensure _deadlock_ and _cascading rollback prevention_. The equivalent serial schedule has the same transaction order as the lock point order in 2PL schedule.

#### How to Construct?

First we are going to add lock and release to each transaction, and make sure they obey the rules of 2PL, which is separate the transaction into growing and shrinking phase.
According to the lock point and exclusion, arrange the concurrent execution of each transaction. The growing phase and shrinking phase of each transaction should be as disjoint as possible.
The **key point** is, we do not allow two _conflict transactions_ (accessing same item) being at growing phase at the same time.

### Enhanced 2PL Protocol

#### Strict 2PL

**Definition**: all _X-locks_ taken by a transaction are held until that transaction _commits/aborts_, all X-locks can only be unlocked _at the end_ of the transaction.
Leading to _cascadeless_, recoverable and _serializable_ schedules.

#### Rigorous 2PL

**Definition**: all _X-locks and S-locks_ are held until commits or aborts.
Ensure recoverability and avoids cascading roll-backs.

## Multiple Granularity

**Definition**: granularity means the size of the data items Q, multiple granularity means data items Q can be of various sizes such as logical units like tuple, relation, index, even DB, or physical units like data page, index page, data block and file.
With multiple granularity, we can represent the hierarchy as a tree. When locking coarsely (high level), we have low locking overhead but low concurrency. When locking finely (low level), we have high locking overhead but high concurrency. The lowest granularity level is usually tuple.

### Intention Lock Modes

**Definition**: if a node is locked in an _intention_ mode, explicit locking is being done at a lower level of the tree. Intention locks are put on _all ancestors_ of a node before that node is locked explicitly. With intention lock, we can avoid cases like A locked relation but B locked DB to access same item, then neither can access the item (locked by each other).
There are three modes:

- Intention-Shared (IS): means only shared is done in lower level
- Intention-Exclusive (IX): means exclusive or shared is done in lower level
- Shared and Intention-Exclusive: means the sub-tree rooted by that node (its children) is locked in shared mode and exists exclusive mode lock at lower level of that(those) sub-tree(s).

### Locking Scheme

Acquired from root to leaf, release from leaf to root.
