before learning JDBC, Spring DATA JPA. learn SQL first.

## DDL and DML

### DDL (Data Definition Language)

Used to **define or modify database structure**.

Examples:

```sql
CREATE TABLE employee (
    id INT,
    name VARCHAR(50)
);

ALTER TABLE employee
ADD salary DECIMAL(10,2);

DROP TABLE employee;

TRUNCATE TABLE employee;
```

---

### DML (Data Manipulation Language)

Used to **insert, update, delete actual data** inside tables.

Examples:

```sql
INSERT INTO employee VALUES(1,'Prashant');

UPDATE employee
SET name='Rahul'
WHERE id=1;

DELETE FROM employee
WHERE id=1;
```

#### What DML affects?

Only table records (rows).

---

#### Interview Answer

> DDL is used to define or modify database objects like tables, columns, indexes, and constraints. Examples are CREATE, ALTER, DROP, and TRUNCATE.
> 
> DML is used to manipulate data stored inside tables. Examples are INSERT, UPDATE, and DELETE.
> 
> DDL changes the schema structure, while DML changes the actual data.


## DELETE vs TRUNCATE

Suppose Employee table contains:

| id  | name     |
| --- | -------- |
| 1   | Prashant |
| 2   | Rahul    |
| 3   | Amit     |

### DELETE

```sql
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

#### Remove all rows

```sql
DELETE FROM employee;
```

Table remains:

```
employee
```

but empty.

---

#### How DELETE works internally

Database:

1. Finds matching rows.
2. Removes row by row.
3. Writes log for every row.

Because it processes row-by-row, it is slower.

---

### TRUNCATE

```sql
TRUNCATE TABLE employee;
```

Result:

All rows removed instantly.

Table structure remains.

---

#### How TRUNCATE works internally

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

### Interview Answer

> DELETE removes rows one by one and supports a WHERE clause, so we can selectively delete records.
> 
> TRUNCATE removes all rows at once by deallocating data pages. It is much faster, does not support WHERE, and is generally used when we want to clear an entire table.

------

## WHERE vs HAVING

Many developers memorize:

```
WHERE -> before GROUP BY
HAVING -> after GROUP BY
```

But don't understand why.

|id|department|salary|
|---|---|---|
|1|IT|50000|
|2|IT|60000|
|3|HR|30000|
|4|HR|40000|

---

### WHERE

Filters rows BEFORE grouping.

Query:

```sql
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

### HAVING

Filters groups AFTER grouping.

Query:

```sql
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

Remember:

**WHERE filters rows.**

**HAVING filters groups.**

---

### Interview Answer (Best)

> WHERE filters individual rows before grouping happens. It cannot use aggregate functions like COUNT(), SUM(), or AVG().
> 
> HAVING filters grouped data after GROUP BY is executed and is commonly used with aggregate functions.

---

## Joins ☯️





## Subquery ☯️ 

A subquery is simply:

> A query inside another query.

Example:

```sql
SELECT *
FROM employees
WHERE department_id = (
    SELECT department_id
    FROM departments
    WHERE department_name = 'Engineering'
);
```

There are two queries.

Outer query

```sql
SELECT *
FROM employees
WHERE department_id = (...)
```

Inner query

```sql
SELECT department_id
FROM departments
WHERE department_name='Engineering';
```

---

### Types of Subqueries

There are mainly four types.

```
Subquery

├── Scalar (returns one value)
├── Row (returns one row)
├── Column (returns one column)
├── Table (returns multiple rows & columns)


SELECT * 
FROM employees
WHERE salary > ( 
	SELECT AVG(salary)
	FROM employees
);
```
### [[ SQL Execution Order ☯️ ]]

Very important for interviews.

```java
1. FROM
2. JOIN
3. WHERE
4. GROUP BY
5. Aggregate Functions (SUM, COUNT, AVG...)
6. HAVING
7. WINDOW FUNCTIONS (OVER(...))
8. SELECT
9. DISTINCT
10. ORDER BY
11. LIMIT / OFFSET
```


### Non-Correlated Subquery -

These are the easiest.

```sql
SELECT *
FROM employees
WHERE department_id =
(
    SELECT department_id
    FROM departments
    WHERE department_name='Engineering'
);
```

Suppose

Departments

|id|name|
|---|---|
|1|HR|
|2|Engineering|
|3|Finance|

Subquery runs

```sql
SELECT department_id
FROM departments
WHERE department_name='Engineering';
```

Returns

```
2
```

Outer query becomes

```sql
SELECT *
FROM employees
WHERE department_id = 2;
```

Notice

The inner query executes **only once**.

This is called

```
Non-Correlated Subquery
```


### Correlated Subquery

This is the interview favourite.

```
A correlated subquery depends on the current row of the outer query.

It cannot execute independently.
```

Employees

|id|name|dept|salary|
|---|---|---|---|
|1|John|HR|50000|
|2|Alice|HR|70000|
|3|Bob|IT|80000|
|4|Tom|IT|90000|

Question

> Find employees earning above their department average.

```sql
SELECT *
FROM employees e
WHERE salary >
(
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

Notice

```
e.department_id
```

comes from the outer query.

The subquery cannot execute first.

#### How PostgreSQL Executes It

Outer query starts.

##### Row 1

```
John              HR           50000
```

Subquery becomes

```sql
SELECT AVG(salary)
FROM employees
WHERE department_id=HR;
```

Result

```
60000
```

Compare

```
50000 > 60000

False
```

Discard.

---

Row 2

Alice

Subquery

```
AVG(HR)=60000
```

Compare

```
70000 > 60000

True
```

Return Alice.

---

Row 3

Bob

Subquery

```
AVG(IT)=85000
```

```
80000 > 85000

False
```

---

Row 4

Tom

```
90000 > 85000

True
```

Result

```
Alice
Tom
```

### IN Operator

Departments

|id|name|
|---|---|
|1|HR|
|2|IT|
|3|Finance|

Question:  Find employees in HR or IT ?

```sql
SELECT *
FROM employees
WHERE department_id IN
(
    SELECT department_id
    FROM departments
    WHERE department_name IN ('HR','IT')
);
```

Subquery returns

```
1
2
```

Outer query becomes

```
department_id IN (1,2)
```

Equivalent to

```
department_id=1
OR
department_id=2
```

### EXISTS  ☯️ 

`EXISTS` is probably the **most misunderstood SQL operator**.

Many people think:

> "It returns rows from the subquery."

**Wrong.**

`EXISTS` only asks one question:

> **"Does the subquery return at least one row?"**

It doesn't care about the data in those rows.

---

### Real-world analogy

Imagine you're a teacher.

You have two tables.

#### Students

|student_id|name|
|---|---|
|1|John|
|2|Alice|
|3|Bob|
|4|Emma|
#### Exams

|exam_id|student_id|subject|
|---|---|---|
|101|1|Math|
|102|1|Science|
|103|3|English|

Question

> Find students who have taken **at least one exam**.

This is exactly what `EXISTS` is made for.

---

```sql
SELECT *
FROM students s
WHERE EXISTS
(
    SELECT 1
    FROM exams e
    WHERE e.student_id = s.student_id
);
```

Notice this line

```sql
e.student_id = s.student_id
```

This is why it's called a **correlated subquery**.

The subquery depends on the current student.

---

### How PostgreSQL Executes It

Let's execute it manually.

#### Step 1

Outer query starts.

Current row

|student|
|---|
|John|

Now PostgreSQL substitutes

```
s.student_id = 1
```

Subquery becomes

```sql
SELECT 1
FROM exams
WHERE student_id = 1;
```

Result

|1|
|---|
|1|
|1|

Two rows found.

Now EXISTS asks

```
Did I find at least one row?
```

Answer

```
YES
```

Return John.

---

#### Step 2

Next student

Alice

Subquery becomes

```sql
SELECT 1
FROM exams
WHERE student_id = 2;
```

Result

Empty

```
(no rows)
```

EXISTS asks

```
Any row?
```

Answer

```
NO
```

Discard Alice.

---

#### Step 3

Bob

Subquery

```sql
SELECT 1
FROM exams
WHERE student_id = 3;
```

Result

|1|
|---|
|1|

Found.

Return Bob.

---

#### Step 4

Emma

```sql
SELECT 1
FROM exams
WHERE student_id = 4;
```

No rows.

Discard.

---

Final result

|Student|
|---|
|John|
|Bob|

#### Why `SELECT 1`?

Many beginners ask this.

Why not

```
SELECT *
```

or

```
SELECT exam_id
```

Because **EXISTS ignores the selected columns.**

These are identical.

```sql
SELECT *
FROM students s
WHERE EXISTS
(
    SELECT *
    FROM exams e
    WHERE e.student_id = s.student_id
);
```

or

```sql
SELECT *
FROM students s
WHERE EXISTS
(
    SELECT exam_id
    FROM exams e
    WHERE e.student_id = s.student_id
);
```

or

```sql
SELECT *
FROM students s
WHERE EXISTS
(
    SELECT 'hello'
    FROM exams e
    WHERE e.student_id = s.student_id
);
```

or

```sql
SELECT *
FROM students s
WHERE EXISTS
(
    SELECT 999
    FROM exams e
    WHERE e.student_id = s.student_id
);
```

All behave exactly the same.

`SELECT 1` is simply the convention because it makes it obvious that **the returned value isn't used**.

---

### Important optimisation

Suppose John has **10,000 exams**.

Without optimisation PostgreSQL would scan

```
Exam 1

Exam 2

Exam 3

...

Exam 10000
```

It doesn't.

As soon as PostgreSQL finds the **first matching row**, `EXISTS` becomes `TRUE`.

```
Exam 1 found

↓

TRUE

↓

STOP
```

It doesn't need the remaining 9,999 rows.

This is why `EXISTS` is often efficient for existence checks.

---

### Another practical example (E-commerce)

#### Customers

| id  | name  |
| --- | ----- |
| 1   | John  |
| 2   | Alice |
| 3   | Bob   |

#### Orders

|order_id|customer_id|
|---|---|
|101|1|
|102|1|
|103|3|

Question

> Find customers who have placed at least one order.

```sql
SELECT c.*
FROM customers c
WHERE EXISTS
(
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
);
```


## ANY

| Emp   | Department | Salary |
| ----- | ---------- | ------ |
| John  | HR         | 50000  |
| Alice | HR         | 70000  |
| Bob   | IT         | 60000  |
| Tom   | IT         | 90000  |
| Emma  | Finance    | 80000  |
| David | Finance    | 100000 |


Think of `ANY` as

> **Compare with at least one value.**

or

> **One comparison must be TRUE.**

General syntax

```
value operator ANY(subquery)
```

Suppose

```sql
SELECT salary
FROM employees
WHERE department='HR';
```

returns

```
50000
70000
```

Now execute

```sql
SELECT *
FROM employees
WHERE salary > ANY
(
    SELECT salary
    FROM employees
    WHERE department='HR'
);
```

---

### Step-by-step execution

#### Step 1

Subquery executes first.

```
50000
70000
```

Outer query becomes

```
salary > ANY(50000,70000)
```

Now PostgreSQL checks each employee.

---

#### John

Salary

```
50000
```

Comparison

```
50000 > 50000 → False

50000 > 70000 → False
```

At least one TRUE?

```
No
```

Not returned.

---

#### Bob

Salary

```
60000
```

Comparison

```
60000 > 50000 → True

60000 > 70000 → False
```

One TRUE?

Yes.

Return Bob.

---

#### Emma

Salary

```
80000
```

Comparison

```
80000 > 50000 → True

80000 > 70000 → True
```

Returned.

---

#### David

```
100000 >50000 True

100000 >70000 True
```

Returned.

---

Result

|Employee|Salary|
|---|---|
|Bob|60000|
|Alice|70000|
|Emma|80000|
|Tom|90000|
|David|100000|

---

### Visual understanding

Imagine PostgreSQL internally doing

```
salary > ANY(50000,70000)

↓

salary >50000
OR
salary >70000
```

Notice

`ANY` behaves like **OR**.

Only one comparison has to succeed.

-------------

## ALL

Now use

```sql
SELECT *
FROM employees
WHERE salary > ALL
(
    SELECT salary
    FROM employees
    WHERE department='HR'
);
```

Subquery

```
50000
70000
```

Outer query becomes

```
salary > ALL(50000,70000)
```

---

### Check every employee

#### John

```
50000 >50000 False

50000 >70000 False
```

Returned?

No.

---

#### Bob

```
60000 >50000 True

60000 >70000 False
```

Returned?

No.

---

#### Emma

```
80000 >50000 True

80000 >70000 True
```

Returned.

---

#### David

```
100000 >50000 True

100000 >70000 True
```

Returned.

---

#### Tom

```
90000 >50000 True

90000 >70000 True
```

Returned.

---

Result

|Employee|Salary|
|---|---|
|Emma|80000|
|Tom|90000|
|David|100000|

---

### Visual understanding

```
salary >ALL(50000,70000)

↓

salary >50000

AND

salary >70000
```

`ALL` behaves like **AND**.

Every comparison must be TRUE.

--------------------------------------------









##

-------

## Window Functions -






-----
-----
-----





