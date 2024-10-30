<!-- layout: true -->

<!-- class: middle -->

---

<!-- class: center, middle, inverse -->

# Query Processing for Join Operations

.right[

  .invisible-slide-comment[See [^dot_right] about `.right`]

  Presented by Huanyi Chen

  huanyi.chen@uwaterloo.ca

]

---

## Review

Question: what the basic steps to process a query?

???

.invisible-slide-comment[See [^triple_question_marks] about `???`]

Comments:

  1. Parsing and translation
  2. Optimization
  3. Evaluation

---

## Cost

Cost is **estimated** in practice using statistical information recorded in the
system catalog, e.g.

- \# of tuples in a table, \# of entries in an index
- size of each tuple or index entry, height of a B+-tree index
- *cardinality* of an attribute: # distinct attribute values stored
- *selectivity* of an index: cardinality of indexed attribute(s) / total # of
  index entries

---

## Question: why don't we have everything in indexes?

???

.invisible-slide-comment[See [^triple_question_marks] about `???`]

Comments:

  Engineer's Dilemma

  Pros:
  - Adding indexes can speed up some queries drastically.


  Cons:
  - Indexing slows down insertions and updates.
  - Looking up an index repeatedly during a join is not necessarily faster than
    scanning the inner relation (see indexed nested loops joins vs block nested
    loops join).
  - The DBMS may create the most important indexes automatically (e.g., on primary
    keys and foreign keys), and it may not offer many choices of index structure
    (e.g., InnoDB in MySQL 5.6 only supports B+-tree indexes).
  - It is difficult to outsmart a good query optimizer, which has access to
    detailed statistics about tables and uses elaborate optimization algorithms.

  Therefore, designing a good physical schema may require the following:
  - Using a basic understanding of query evaluation, making an educated guess as
    to what index or indexes might benefit the most important queries.
  - Understanding how an index is used by the query optimizer by inspecting the
    evaluation plan for a given query.
  - Determining the impact of adding an index by measuring performance differences
    empirically.

  We will focus on the first two points.

---

## Nested Loop Join

Nested Loop Join for $R$ Join $R$:

```
for each tuple r in R do
  for each tuple s in S do
    if r and s satisfy the join condition then
      add tuple <r,s>
  end
end
```

--

- Question: what's the cost? (Assume a simplistic model where only disk read
  matters)
  - Best case: the buffer can hold both relation in memory
  - Worst case: the buffer can hold only one block of each relation in memory
  - $n_r$ = number of tuples in $R$, $n_s$ = number of tuples in $S$
  - $b_r$ = number of blocks in $R$, $b_s$ = number of blocks in $S$

???

.invisible-slide-comment[See [^triple_question_marks] about `???`]

Comments:

  - Best-case: $b_r + b_s$ block transfers and $2$ seeks
  - Worst-case: $n_r \ast b_s + b_r$ block transfers and $b_r + b_s$ seeks

---

## Block Nested Loop Join

Block Nested Loop Join for $R$ Join $S$:

```
for each block b_r in R do
  for each block b_s in S do
    for each tuple r in b_r do
      for each tuple s in b_s do
        if r and s satisfy the join condition then
          add tuple <r,s>
      end
    end
  end
end
```

--

- Question: what's the cost? (Assume a simplistic model where only disk read
  matters)
  - Best case: the buffer can hold both relation in memory
  - Worst case: the buffer can hold only one block of each relation in memory
  - $n_r$ = number of tuples in $R$, $n_s$ = number of tuples in $S$
  - $b_r$ = number of blocks in $R$, $b_s$ = number of blocks in $S$

???

.invisible-slide-comment[See [^triple_question_marks] about `???`]

Comments:

  - Best-case: same as nested loop join
  - Worst-case: $b_r \ast b_s + b_r$ block transfers and $2 \ast b_r$ seeks

---

## Indexed Nested Loop Join

Indexed Nested Loop Join for $R$ Join $S$:

```
for each tuple r in R do
  for each tuple s in S in the index lookup do
      add tuple <r,s>
  end
end
```

The index can be:
- an existing index; or
- a temporary index created for the sole purpose of evaluating the join.

---

## Indexed Nested Loop Join

- Worst case: the buffer can hold only one block of R and one block of the index
  for S in memory

$$b_r \ast (t_T + t_S) + n_r \ast C$$

- Since the disk head may have moved in between each I/O, so each I/O requires a
  seek, which takes time $t_S$, and a block transfer, which takes time $t_T$.
- $C$ is the cost of a single selection on $S$ using the join condition.

Table 15.3 in the textbook (Database System Concepts) has summarized the costs
for different cases.

> e.g., for an InnoDB primary key (a B+-tree index) and equality on key as the
> condition, the cost is $(h_i + 1) \ast (t_T + t_S)$, $h_i$ denotes the height
> of the index. Index lookup traverses the height of the tree plus one I/O to
> fetch the record; each of these I/O operations requires a seek and a block
> transfer.

---

## Exercise: Employees DB

![:centerwidth 45%](https://dev.mysql.com/doc/employee/en/images/employees-schema.png)

???

.invisible-slide-comment[See [^triple_question_marks] about `???`]

Comments:

  See <https://dev.mysql.com/doc/employee/en/sakila-structure.html>

  .invisible-slide-comment[Why the hell is `sakila-structure` in the url?]

---

## Exercise: Employees DB

Find out employees who was born after year 1960 and became a manager at some
point.

Query

```sql
select
  first_name,
  last_name,
  birth_date,
  title
from
  employees
  inner join titles using (emp_no)
where
  birth_date > "1960-01-01"
  and title = "Manager";
```

---

```sql
+-----------+--------------------------------------------------------------------+
| Table     | Create Table                                                       |
+-----------+--------------------------------------------------------------------+
| employees | CREATE TABLE `employees` (                                         |
|           |   `emp_no` int NOT NULL,                                           |
|           |   `birth_date` date NOT NULL,                                      |
|           |   `first_name` varchar(14) NOT NULL,                               |
|           |   `last_name` varchar(16) NOT NULL,                                |
|           |   `gender` enum('M','F') NOT NULL,                                 |
|           |   `hire_date` date NOT NULL,                                       |
|           |   PRIMARY KEY (`emp_no`)                                           |
|           | ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci |
+-----------+--------------------------------------------------------------------+
```

---

```sql
+--------+---------------------------------------------------------------------------------------------------------+
| Table  | Create Table                                                                                            |
+--------+---------------------------------------------------------------------------------------------------------+
| titles | CREATE TABLE `titles` (                                                                                 |
|        |   `emp_no` int NOT NULL,                                                                                |
|        |   `title` varchar(50) NOT NULL,                                                                         |
|        |   `from_date` date NOT NULL,                                                                            |
|        |   `to_date` date DEFAULT NULL,                                                                          |
|        |   PRIMARY KEY (`emp_no`,`title`,`from_date`),                                                           |
|        |   CONSTRAINT `titles_ibfk_1` FOREIGN KEY (`emp_no`) REFERENCES `employees` (`emp_no`) ON DELETE CASCADE |
|        | ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci                                      |
+--------+---------------------------------------------------------------------------------------------------------+
```

---

First, in addition to `explain`, let's also use `STRAIGHT_JOIN` to force the
 optimizer to join the tables in the order in which they are listed in the
 `FROM` clause.

```sql
explain
select straight_join
  first_name,
  last_name,
  birth_date,
  title
from
  employees
  inner join titles using (emp_no)
where
  birth_date > "1960-01-01"
  and title = "Manager";
```

```sh
+----+-------------+-----------+------------+------+---------------+---------+---------+----------------------------------+--------+----------+-------------+
| id | select_type | table     | partitions | type | possible_keys | key     | key_len | ref                              | rows   | filtered | Extra       |
+----+-------------+-----------+------------+------+---------------+---------+---------+----------------------------------+--------+----------+-------------+
| 1  | SIMPLE      | employees | <null>     | ALL  | PRIMARY       | <null>  | <null>  | <null>                           | 299335 |  33.33   | Using where |
| 1  | SIMPLE      | titles    | <null>     | ref  | PRIMARY       | PRIMARY | 206     | employees.employees.emp_no,const | 1      | 100.0    | Using index |
+----+-------------+-----------+------------+------+---------------+---------+---------+----------------------------------+--------+----------+-------------+
```

---

Now try normal `select` and let the optimizer do its job.

```sql
explain
select
  first_name,
  last_name,
  birth_date,
  title
from
  employees
  inner join titles using (emp_no)
where
  birth_date > "1960-01-01"
  and title = "Manager";
```

```sh
+----+-------------+-----------+------------+--------+---------------+---------+---------+-------------------------+--------+----------+--------------------------+
| id | select_type | table     | partitions | type   | possible_keys | key     | key_len | ref                     | rows   | filtered | Extra                    |
+----+-------------+-----------+------------+--------+---------------+---------+---------+-------------------------+--------+----------+--------------------------+
| 1  | SIMPLE      | titles    | <null>     | index  | PRIMARY       | PRIMARY | 209     | <null>                  | 442545 | 10.0     | Using where; Using index |
| 1  | SIMPLE      | employees | <null>     | eq_ref | PRIMARY       | PRIMARY | 4       | employees.titles.emp_no | 1      | 33.33    | Using where              |
+----+-------------+-----------+------------+--------+---------------+---------+---------+-------------------------+--------+----------+--------------------------+
```

???

.invisible-slide-comment[See [^triple_question_marks] about `???`]

Comments:

  See <https://dev.mysql.com/doc/refman/8.0/en/explain-output.html> to interpret
  the `EXPLAIN` output.

---

### Try adding an index

```sql
create index emp_birth on employees (emp_no, birth_date, first_name, last_name);

-- to drop it later
-- alter table employees drop index emp_birth;
```

- Question: does it work?

???

.invisible-slide-comment[See [^triple_question_marks] about `???`]

Comments:

  It actually doesn't work. See the following EXPLAIN output. Under `key`, it still makes use the PRIMARY instead of the index.

  ```sh
  +----+-------------+-----------+------------+--------+-------------------+---------+---------+-------------------------+--------+----------+--------------------------+
  | id | select_type | table     | partitions | type   | possible_keys     | key     | key_len | ref                     | rows   | filtered | Extra                    |
  +----+-------------+-----------+------------+--------+-------------------+---------+---------+-------------------------+--------+----------+--------------------------+
  | 1  | SIMPLE      | titles    | <null>     | index  | PRIMARY           | PRIMARY | 209     | <null>                  | 442545 | 10.0     | Using where; Using index |
  | 1  | SIMPLE      | employees | <null>     | eq_ref | PRIMARY,emp_birth | PRIMARY | 4       | employees.titles.emp_no | 1      | 33.33    | Using where              |
  +----+-------------+-----------+------------+--------+-------------------+---------+---------+-------------------------+--------+----------+--------------------------+
  ```

---

- Content of the join operations is based on Chapter 15.5 of the textbook Database System Concepts.

- You might also want to take a look at
  <https://dev.mysql.com/doc/refman/8.0/en/optimization-indexes.html> and
  <https://dev.mysql.com/doc/refman/8.0/en/execution-plan-information.html>


.invisible-slide-comment[

See all footnotes below.

[^dot_right]: `.right` is a syntax to align slide content to the right and can
be ignored when reading the .md source file.

[^triple_question_marks]: `???` is a syntax used for slide show and can be
ignored when reading the .md source file. It states that the content after `???`
is only visible in the presenter mode.

]
