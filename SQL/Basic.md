before learning JDBC, Spring DATA JPA. learn SQL first.

# ==1. DDL and DML ==

## DDL (Data Definition Language)

Used to **define or modify database structure**.

Examples:

```
CREATE TABLE employee (
    id INT,
    name VARCHAR(50)
);

ALTER TABLE employee
ADD salary DECIMAL(10,2);

DROP TABLE employee;

TRUNCATE TABLE employee;
```

### What DDL affects?

- Tables
- Database schema
- Columns
- Constraints
- Indexes

### Auto Commit

DDL statements generally auto-commit.

```
CREATE TABLE test(id INT);

ROLLBACK; -- won't remove table
```

---

## DML (Data Manipulation Language)

Used to **insert, update, delete actual data** inside tables.

Examples:

```
INSERT INTO employee VALUES(1,'Prashant');

UPDATE employee
SET name='Rahul'
WHERE id=1;

DELETE FROM employee
WHERE id=1;
```

### What DML affects?

Only table records (rows).

### Transaction Support

```
BEGIN;

DELETE FROM employee;

ROLLBACK;
```

Rows come back after rollback.

---

### Interview Answer

> DDL is used to define or modify database objects like tables, columns, indexes, and constraints. Examples are CREATE, ALTER, DROP, and TRUNCATE.
> 
> DML is used to manipulate data stored inside tables. Examples are INSERT, UPDATE, and DELETE.
> 
> DDL changes the schema structure, while DML changes the actual data.

---


-------------
# 
-----------------

# ==2. DELETE vs TRUNCATE==

Suppose Employee table contains:

| id  | name     |
| --- | -------- |
| 1   | Prashant |
| 2   | Rahul    |
| 3   | Amit     |

## DELETE

```
DELETE FROM employee
WHERE id = 2;
```

Result:

|id|name|
|---|---|
|1|Prashant|
|3|Amit|

Only one row removed.

---

### Remove all rows

```
DELETE FROM employee;
```

Table remains:

```
employee
```

but empty.

---

### How DELETE works internally

Database:

1. Finds matching rows.
2. Removes row by row.
3. Writes log for every row.

Because it processes row-by-row, it is slower.

---

## TRUNCATE

```
TRUNCATE TABLE employee;
```

Result:

All rows removed instantly.

Table structure remains.

---

### How TRUNCATE works internally

Instead of deleting rows one by one:

```
DELETE:
Row1 -> remove
Row2 -> remove
Row3 -> remove
...
```

TRUNCATE:

```
Remove entire data pages
Reset storage metadata
```

Much faster.

---

## Important Difference

|Feature|DELETE|TRUNCATE|
|---|---|---|
|Type|DML|DDL|
|WHERE allowed|Yes|No|
|Removes selected rows|Yes|No|
|Removes all rows|Yes|Yes|
|Speed|Slow|Fast|
|Row-by-row logging|Yes|No (minimal logging)|
|Rollback|Usually Yes|DB dependent|
|Identity reset|No|Yes|

---

## ==Identity Example - { counter reset in truncate }==

```
CREATE TABLE employee(
    id SERIAL PRIMARY KEY,
    name VARCHAR(50)
);
```

Data:

```
1 Prashant
2 Rahul
3 Amit
```

### DELETE

```
DELETE FROM employee;
```

Insert again:

```
INSERT INTO employee(name)
VALUES('John');
```

Output:

```
4 John
```

Identity continues.

---

### TRUNCATE

```
TRUNCATE TABLE employee RESTART IDENTITY;
```

Insert:

```
INSERT INTO employee(name)
VALUES('John');
```

Output:

```
1 John
```

Counter reset.

---

## Interview Answer

> DELETE removes rows one by one and supports a WHERE clause, so we can selectively delete records.
> 
> TRUNCATE removes all rows at once by deallocating data pages. It is much faster, does not support WHERE, and is generally used when we want to clear an entire table.

---
#

------

# ==3. WHERE vs HAVING==

Many developers memorize:

```
WHERE -> before GROUP BY
HAVING -> after GROUP BY
```

But don't understand why.

---

## Sample Data

|id|department|salary|
|---|---|---|
|1|IT|50000|
|2|IT|60000|
|3|HR|30000|
|4|HR|40000|

---

## WHERE

Filters rows BEFORE grouping.

Query:

```
SELECT *
FROM employee
WHERE salary > 40000;
```

Output:

|id|department|salary|
|---|---|---|
|1|IT|50000|
|2|IT|60000|

---

## HAVING

Filters groups AFTER grouping.

Query:

```
SELECT department,
       AVG(salary)
FROM employee
GROUP BY department
HAVING AVG(salary) > 45000;
```


Output:

|department|avg_salary|
|---|---|
|IT|55000|

---

## 
## [SQL Execution Order]

Very important for interviews.

```
1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. SELECT
6. ORDER BY
7. LIMIT
```

Remember:

**WHERE filters rows.**

**HAVING filters groups.**

---

## Real Production Example

Find departments having more than 100 employees.

```
SELECT department,
       COUNT(*) AS total_emp
FROM employee
GROUP BY department
HAVING COUNT(*) > 100;
```


---

## Interview Answer (Best)

> WHERE filters individual rows before grouping happens. It cannot use aggregate functions like COUNT(), SUM(), or AVG().
> 
> HAVING filters grouped data after GROUP BY is executed and is commonly used with aggregate functions.
> 
> For example, WHERE salary > 50000 filters employees, while HAVING AVG(salary) > 50000 filters departments based on average salary.

---

## 30-Second Interview Summary

### DDL vs DML

> DDL modifies database structure (CREATE, ALTER, DROP, TRUNCATE), while DML manipulates data inside tables (INSERT, UPDATE, DELETE).

### DELETE vs TRUNCATE

> DELETE removes rows one by one and supports WHERE. TRUNCATE removes all rows quickly by deallocating storage pages and doesn't support WHERE.

### WHERE vs HAVING

> WHERE filters rows before grouping, while HAVING filters groups after GROUP BY. Aggregate functions can be used in HAVING but not in WHERE.


#

------
-----


# ==Explain INNER JOIN vs LEFT JOIN with examples ?==


# ==ACID = Atomicity + Consistency + Isolation + Durability==

### Atomicity -
> Atomicity ensures that a transaction is treated as a single unit of work. Either all operations succeed or all are rolled back.

### Consistency -

> Consistency ensures that all database constraints, rules, and relationships remain valid before and after a transaction.

### Isolation -

> Concurrent transactions running simultaneously should not interfere with each other.

### Durability -

> Durability guarantees that once a transaction is committed, its changes are permanently stored and will survive system failures.


#