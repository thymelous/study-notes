---
layout: post
title: "Databases: Course Summary"
date: 2018-06-07
---
A summary of the Databases course I had this term (Spring 2018), made in preparation for the exams.
Information is compiled from various websites, Wikipedia in particular.

## The Relational Model

In the relational model, all data is represented in terms of _tuples_, grouped into _relations_.
A _tuple_ is an ordered set of attribute values, belonging to a particular _domain_ (data type) --
a set of atomic (indivisible) values. A _relation_ consists of a _heading_, a set of
_(attribute name, type name)_ pairs, and a _body_, a **set** of _tuples_ (meaning there can't be
two fully identical _tuples_).

_Cardinality_ in the context of a data model refers to the number of _tuples_ in a relation,
_power of relation_ -- to the number of _attributes_.

In database systems that adopt the relational model,
_tuples_ are similar in concept to _rows_, while _relations_ are called _tables_, _attributes_ -- _columns_.

## SQL

SQL is a domain-specific language for managing data in a relational database.
It is _declarative_, meaning that you specify what data to fetch, not how (e.g. using an index or not).

SQL's statements are divided into several groups:

__Data Definition Language (DDL)__: `CREATE`, `DROP`, `ALTER`
__Data Manipulation Language (DML)__: `SELECT`, `INSERT`, `UPDATE`, `DELETE`
__Data Control Language (DCL)__: `GRANT`, `REVOKE`, `DENY`
__Transaction Control Language (TCL)__: `COMMIT`, `ROLLBACK`, `SAVEPOINT`

SQL has procedural extensions (e.g. _pl/pgSQL_), which introduce the ability to use loops and other control structures.

## Architecture

Three layers of the ANSI-SPARC architecture:
* **external**: user views of data, independent and customized, with irrelevant (or unaccessible by policy) data hidden
* **conceptual**: the exact kind of data stored (entities) and its interrelationship, independent of hardware/software
* **internal**: physical representation of the database (hardware/software)

## Views and Cursors

A _view_ encapsulates a query and appears to `SELECT`s as an ordinary relation, which can be used to e.g. represent joined tables.
By default, it is not physically materialized (the query is run every time the view is referenced), however, PostgreSQL
supports materialized views, which are populated at creation and may be updated with a special command, `REFRESH MATERIALIZED VIEW`.

A _cursor_ encapsulates a query and then reads its result a few rows at a time. It is mostly used to return a large dataset
from a function (the function opens the cursor and returns a reference to it). Cursors are also used in `FOR` loops:

```plpgsql
FOR row IN cursor LOOP
  ...
END LOOP;
```

## Normalization

Normalization helps to avoid undesirable effects caused by data modification, namely _update, insertion, and deletion anomalies_.
Let's use the following table as an example:

`teacher_name` | `department` | `course_name`

**Update anomaly** happens when multiple rows contain the same information.
In this example, assuming a teacher can have multiple courses, there can be more than one row connecting a teacher with their department.
Should the latter be changed, the data could be left in an inconsistent state, where some records have the old department and some have the new one.

**Insertion anomaly** is the inability to record facts in certain circumstances.
In this example, information about a newly hired teacher that doesn't have courses yet cannot be stored.

**Deletion anomaly** is when deleting certain facts leads to a loss of other, unrelated information.
In this example, if a teacher stops having courses for a while, their name and department are lost.

The concept of normalization was introduced along with the _first normal form_ (1NF).
Later, higher forms (2NF, 3NF, ...) were defined as well.

The three normal forms have a mnemonic:
"Non-keys must provide a fact about the key [1NF], the whole key [2NF], and nothing but the key [3NF]."

### First Normal Form

The domain of each attribute must contain indivisible values,
and the value of each attibute must contain a single value from that domain.

<u>Example violations from Wikipedia</u>:

`name` | `phone_numbers`

_where `phone_numbers` is a comma-separated list of numbers (not indivisible)_

`name` | `phone_number1` | `phone_number2`

_where `phone_numberNs` form a repeating group of attributes_

<u>Example solution</u>:

`name` | `phone_number`

_with a separate row for each phone number_

### Second Normal Form

To satisfy 2NF, the relation should already be in 1NF, plus
its attibutes that are not a part of a candidate key should not depend on a subset of a candidate key.

<u>Example from Wikipedia</u>:

`manufacturer` | `model` | `model_full_name` | `manufacturer_country`

Both `(model, manufacturer)` and `model_full_name` uniquely identify rows.
Even if the latter is the primary key, the former is still a candidate key,
and `manufacturer_country` depends on a part of it (`manufacturer`).
To satisfy 2NF, an additional table with `manufacturer, manufacturer_country` needs to be created.

### Third Normal Form

A relation is in 3NF if it satisfies both 1NF and 2NF, and none of its non-prime attributes
(i.e. not belonging to any candidate keys) depend on other non-prime attributes.

<u>Example from Wikipedia</u>:

`tournament` | `year` | `winner` | `winner_date_of_birth`

The candidate key is `(tournament, year)`, but `winner_date_of_birth` is dependent on `winner`.
The problem is, there's nothing that would not allow the same person to have different dates of birth.

### Boyce–Codd Normal Form

BCNF is slightly stronger than 3NF, and is only different from 3NF when there are multiple overlapping
candidate keys. Assuming that candidate keys of `ABC` are `A,B` and `A,C`, if `B` depends on `C`,
the relation does not satisfy BCNF.

### Fourth Normal Form

4NF only applies to relations that have more than three attributes (`ABC`), and for a single value of `A`
multiple values of `B` exist that are not dependent on `C`.
