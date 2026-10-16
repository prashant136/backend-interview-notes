
## 🚩 Window Function

Imagine you have employee salary data.

|id|name|department|salary|
|---|---|---|---|
|1|John|IT|80000|
|2|Alice|IT|90000|
|3|Bob|HR|60000|
|4|David|IT|70000|
|5|Emma|HR|65000|

Suppose you want

> Show every employee along with the average salary of their department.

Without window functions, you might think:

```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
```

| department | avg   |
| ---------- | ----- |
| IT         | 80000 |
| HR         | 62500 |

Problem:

Where is John?
Where is Alice?

GROUP BY collapsed all rows.
Sometimes we don't want to collapse rows.

We want

|Employee|Salary|Department Avg|
|---|---|---|
|John|80000|80000|
|Alice|90000|80000|
|David|70000|80000|
|Bob|60000|62500|
|Emma|65000|62500|

This is exactly why Window Functions exist.

They calculate values **while keeping every row.**

---

### Window Function = Aggregate without collapsing rows

SQL window functions allow performing calculations across a set of rows that are related to the current row, without collapsing the result into a single value.

### General Syntax

```sql
FUNCTION(...) OVER (
    PARTITION BY ...
    ORDER BY ...
)
```

Let's understand every component of window function.

Let's slow down and see **exactly what PostgreSQL is doing internally.**

---
#### Our Table

|id|name|department|salary|
|---|---|---|---|
|1|John|IT|80000|
|2|Alice|IT|90000|
|3|Bob|HR|60000|
|4|David|IT|70000|
|5|Emma|HR|65000|
|6|Tom|Sales|50000|
|7|Chris|Sales|55000|

Query:

```sql
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER(
        PARTITION BY department
        ORDER BY salary DESC
    ) AS rn
FROM employees;
```

---

### Step 1: PostgreSQL first reads all rows

Initially, nothing special happens.

```
John    IT      80000
Alice   IT      90000
Bob     HR      60000
David   IT      70000
Emma    HR      65000
Tom     Sales   50000
Chris   Sales   55000
```

---

### Step 2: PARTITION BY divides the rows

PARTITION BY: It divides the data into groups using PARTITION BY.

Think of `PARTITION BY` as saying:

> "Split the table into independent groups."

It is **NOT** sorting.

It is **NOT** grouping like `GROUP BY`.

It simply creates separate logical windows.

```
Entire Table
----------------------------

        ↓

+----------------------+
|      IT              |
|----------------------|
| John     80000       |
| Alice    90000       |
| David    70000       |
+----------------------+

+----------------------+
|      HR              |
|----------------------|
| Bob      60000       |
| Emma     65000       |
+----------------------+

+----------------------+
|      Sales           |
|----------------------|
| Tom      50000       |
| Chris    55000       |
+----------------------+
```

Notice:

The original rows still exist.

Nothing has been removed.

Nothing has been merged.

PostgreSQL simply says:

> "I now have 3 separate windows."

---

### Step 3: ORDER BY works INSIDE EACH partition

ORDER BY: It specifies the order of rows within each group using ORDER BY.

Now PostgreSQL goes to the first partition.

#### IT partition

Before sorting

```
John   80000
Alice  90000
David  70000
```

Apply

```sql
ORDER BY salary DESC
```

Now

```
Alice 90000
John  80000
David 70000
```

---

Now HR

Before

```
Bob   60000
Emma  65000
```

Sort

```
Emma 65000
Bob  60000
```

---

Now Sales

Before

```
Tom   50000
Chris 55000
```

Sort

```
Chris 55000
Tom   50000
```

---

Notice something important:

The IT rows do **NOT** get mixed with HR rows.

Each partition is sorted independently.

---

### Step 4: ROW_NUMBER starts counting

Assigns unique numbers.

Now PostgreSQL begins numbering.

#### IT

```
Alice 90000   → 1
John  80000   → 2
David 70000   → 3
```

---

#### HR

Starts again from 1

```
Emma 65000   → 1
Bob  60000   → 2
```

---

#### Sales

Starts again from 1

```
Chris 55000   → 1
Tom   50000   → 2
```

---

### Final Output

|name|department|salary|rn|
|---|---|---|---|
|Alice|IT|90000|1|
|John|IT|80000|2|
|David|IT|70000|3|
|Emma|HR|65000|1|
|Bob|HR|60000|2|
|Chris|Sales|55000|1|
|Tom|Sales|50000|2|

Notice how every department starts from **1**.

That happens because of `PARTITION BY`.

---

### Visual Flow

```
Entire Table
      │
      ▼
PARTITION BY department
      │
      ▼

IT               HR              Sales
│                │               │
▼                ▼               ▼

Sort             Sort            Sort
Salary DESC      Salary DESC     Salary DESC

│                │               │
▼                ▼               ▼

Assign           Assign          Assign
Row Numbers      Row Numbers     Row Numbers
```

---

### What if PARTITION BY is removed?

```sql
SELECT
    name,
    salary,
    ROW_NUMBER() OVER(
        ORDER BY salary DESC
    )
FROM employees;
```

Now there is **only one partition**.

Sorted table

```
Alice 90000
John 80000
David 70000
Emma 65000
Bob 60000
Chris 55000
Tom 50000
```

Numbering

```
Alice → 1
John → 2
David → 3
Emma → 4
Bob → 5
Chris → 6
Tom → 7
```

Output

|name|salary|rn|
|---|---|---|
|Alice|90000|1|
|John|80000|2|
|David|70000|3|
|Emma|65000|4|
|Bob|60000|5|
|Chris|55000|6|
|Tom|50000|7|

There is no reset because there is only one partition.

---

### Interview Tip

A common way interviewers phrase this is:

> **"What is the difference between `PARTITION BY` and `ORDER BY` inside `OVER()`?"**

A concise answer is:

- **`PARTITION BY`** divides the result set into independent windows (groups) without collapsing rows.
- **`ORDER BY`** defines the order of rows **within each partition**, which determines how functions like `ROW_NUMBER()`, `RANK()`, `LAG()`, `LEAD()`, and running totals are calculated.
- If `PARTITION BY` is omitted, the entire result set is treated as a single partition.


----------------
#### Question: Highest paid employee in every department ?

| id  | name  | department | salary |
| --- | ----- | ---------- | ------ |
| 1   | John  | IT         | 80000  |
| 2   | Alice | IT         | 90000  |
| 3   | David | IT         | 70000  |
| 4   | Bob   | HR         | 60000  |
| 5   | Emma  | HR         | 65000  |
| 6   | Tom   | Sales      | 50000  |
| 7   | Chris | Sales      | 55000  |

#### Solution

```sql
SELECT *
FROM (
		SELECT *,
		       ROW_NUMBER() OVER(
		           PARTITION BY department
		           ORDER BY salary DESC
		       ) AS rn
		FROM employees;
) t
WHERE rn = 1;
```

##### Why we need subquery ? 
PostgreSQL will complain because `rn` does not exist yet when `WHERE` is evaluated.

##### What is `FROM (...)` ?

Normally, you write:

```sql
SELECT * FROM employees;
```

But SQL also allows you to use the **result of another query as if it were a table**.

Example:

```sql
SELECT *
FROM (
    SELECT *
    FROM employees
) t;
```

The query inside the parentheses executes first.

Think of it like this:

```
employees table
        │
        ▼
SELECT * FROM employees
        │
        ▼
Temporary Table
        │
        ▼
Outer Query

// syntax
SELECT * FROM <ANY TABLE>
```

##### Why do we need `t`?

Because every table in the `FROM` clause must have a name.

--------------

## ROW_NUMBER(), RANK(), and DENSE_RANK()

Let's understand them using intuition first.

---

### Imagine a School Competition

There are 7 students.

|Student|Marks|
|---|---|
|Alice|100|
|Bob|95|
|Charlie|95|
|David|90|
|Emma|85|
|Frank|85|
|Grace|80|

Notice:

- Bob and Charlie have the same marks.
- Emma and Frank have the same marks.

Now let's see what happens.

---

### 1. ROW_NUMBER()

Think of it as assigning **serial numbers**.

Even if two students have the same marks, they still get different numbers.

```
1
2
3
4
5
6
7
```

SQL

```sql
SELECT
    student,
    marks,
    ROW_NUMBER() OVER(ORDER BY marks DESC) AS row_num
FROM students;
```

Result

|Student|Marks|ROW_NUMBER|
|---|---|---|
|Alice|100|1|
|Bob|95|2|
|Charlie|95|3|
|David|90|4|
|Emma|85|5|
|Frank|85|6|
|Grace|80|7|

Notice

Bob and Charlie have equal marks.

Still,

Bob → 2

Charlie → 3

because **ROW_NUMBER() never allows duplicates.**

Think:

> Every row gets a unique serial number.

---

### 2. RANK()

Now imagine a real competition.

If two students tie for first place...

Would the next student become second?

No.

The next student becomes third.

Example

```
100
100
95
90
```

Ranks become

```
1
1
3
4
```

Rank 2 disappeared.

Why?

Because two people already occupied Rank 1.

Let's use our table.

SQL

```sql
SELECT
    student,
    marks,
    RANK() OVER(ORDER BY marks DESC) AS rank
FROM students;
```

Result

|Student|Marks|Rank|
|---|---|---|
|Alice|100|1|
|Bob|95|2|
|Charlie|95|2|
|David|90|4|
|Emma|85|5|
|Frank|85|5|
|Grace|80|7|

Notice

```
1
2
2
4
5
5
7
```

Missing numbers

```
3
6
```

These are called **gaps**.

---

### 3. DENSE_RANK()

Dense Rank says

> Ignore the gaps.

Whenever score changes,

increase rank by exactly 1.

Example

```
100
95
95
90
85
85
80
```

Dense Rank

```
1
2
2
3
4
4
5
```

SQL

```sql
SELECT
    student,
    marks,
    DENSE_RANK() OVER(ORDER BY marks DESC) AS dense_rank
FROM students;
```

Result

|Student|Marks|Dense Rank|
|---|---|---|
|Alice|100|1|
|Bob|95|2|
|Charlie|95|2|
|David|90|3|
|Emma|85|4|
|Frank|85|4|
|Grace|80|5|

No missing numbers.

---

### Side-by-Side Comparison

|Student|Marks|ROW_NUMBER|RANK|DENSE_RANK|
|---|---|---|---|---|
|Alice|100|1|1|1|
|Bob|95|2|2|2|
|Charlie|95|3|2|2|
|David|90|4|4|3|
|Emma|85|5|5|4|
|Frank|85|6|5|4|
|Grace|80|7|7|5|

This is the table interviewers expect you to understand.

--------------------

|Name|Salary|ROW_NUMBER|RANK|DENSE_RANK|
|---|---|---|---|---|
|Alice|100000|1|1|1|
|Bob|90000|2|2|2|
|Charlie|90000|3|2|2|
|David|80000|4|4|3|
|Emma|70000|5|5|4|
|Frank|70000|6|5|4|
|Grace|60000|7|7|5|

#### Interview Question 1: Second Highest Salary

Suppose you use `ROW_NUMBER()`:

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(ORDER BY salary DESC) rn
    FROM employees
) t
WHERE rn = 2;
```

Result

```
Bob
```

But Charlie has the **same second-highest salary** and is missed.

Now use `DENSE_RANK()`:

```sql
SELECT *
FROM (
    SELECT *,
           DENSE_RANK() OVER(ORDER BY salary DESC) r
    FROM employees
) t
WHERE r = 2;
```

Result

```
Bob
Charlie
```

This is usually what interviewers expect when they ask for the **second-highest salary**.

---

#### Interview Question 2: Highest Paid Employee in Each Department

If you only want **one employee per department**, even when there are ties, use `ROW_NUMBER()`:

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(
               PARTITION BY department
               ORDER BY salary DESC
           ) rn
    FROM employees
) t
WHERE rn = 1;
```

If you want **all employees tied for the highest salary** in each department, use `RANK()` or `DENSE_RANK()`:

```sql
SELECT *
FROM (
    SELECT *,
           DENSE_RANK() OVER(
               PARTITION BY department
               ORDER BY salary DESC
           ) r
    FROM employees
) t
WHERE r = 1;
```


### LAG()

## What does it do?

LAG means

> "Look behind."

It returns data from a previous row.

Think while standing in a queue.

```
Tom → Chris → Bob → Emma → David
          ↑
       Who is behind me?
```

Chris looks behind.

Answer:

```
Tom
```

Bob looks behind.

Answer:

```
Chris
```

---

## Syntax

```
LAG(column)
OVER(
ORDER BY ...
)
```

---

## PostgreSQL Example

```
SELECT
name,
salary,
LAG(salary)
OVER(
ORDER BY salary
) previous_salary
FROM employees;
```

Result

|Name|Salary|Previous Salary|
|---|---|---|
|Tom|50000|NULL|
|Chris|55000|50000|
|Bob|60000|55000|
|Emma|65000|60000|
|David|70000|65000|
|John|80000|70000|
|Alice|90000|80000|

Tom has nobody before him.

Therefore

```
NULL
```

---

## Difference Between Current and Previous Salary

```
SELECT
name,
salary,
salary -
LAG(salary)
OVER(
ORDER BY salary
) difference
FROM employees;
```

Output

|Name|Salary|Difference|
|---|---|---|
|Tom|50000|NULL|
|Chris|55000|5000|
|Bob|60000|5000|
|Emma|65000|5000|
|David|70000|5000|
|John|80000|10000|
|Alice|90000|10000|

Very common interview question.

---

## Offset

Previous two rows

```
LAG(salary,2)
OVER(
ORDER BY salary
)
```

Result

|Salary|Previous 2|
|---|---|
|50000|NULL|
|55000|NULL|
|60000|50000|
|65000|55000|
|70000|60000|

---

## Default Value

Instead of NULL

```
LAG(salary,1,0)
```

Result

|Salary|Previous|
|---|---|
|50000|0|

---

# Real-world Use Cases of LAG()

- Month-over-month sales growth
- Compare current salary with previous salary
- Compare today's stock price with yesterday's
- Compare current order with previous order

---

# 2. LEAD()

Exactly opposite.

Instead of looking behind

It looks ahead.

Imagine

```
Tom → Chris → Bob → Emma
 ↑
Who comes after me?
```

Answer

```
Chris
```

---

## PostgreSQL Example

```
SELECT
name,
salary,
LEAD(salary)
OVER(
ORDER BY salary
)
next_salary
FROM employees;
```

Result

|Name|Salary|Next Salary|
|---|---|---|
|Tom|50000|55000|
|Chris|55000|60000|
|Bob|60000|65000|
|Emma|65000|70000|
|David|70000|80000|
|John|80000|90000|
|Alice|90000|NULL|

Alice has nobody after her.

---

Difference with next employee

```
SELECT
name,
salary,
LEAD(salary)
OVER(ORDER BY salary)-salary
AS difference
FROM employees;
```

---

Real-world

Next delivery

Next appointment

Next payment

Next stock price

---

# LAG vs LEAD

```
Tom → Chris → Bob → Emma

LAG

looks ←

LEAD

looks →
```

Simple memory trick

```
LAG = Left (back)

LEAD = Right (forward)
```

---

# 3. FIRST_VALUE()

This returns

> First value inside the window.

Suppose IT department

|Employee|Salary|
|---|---|
|David|70000|
|John|80000|
|Alice|90000|

Query

```
SELECT
name,
department,
salary,
FIRST_VALUE(name)
OVER(
PARTITION BY department
ORDER BY salary DESC
)
AS highest_paid
FROM employees;
```

Output

|Name|Salary|Highest Paid|
|---|---|---|
|Alice|90000|Alice|
|John|80000|Alice|
|David|70000|Alice|

Notice

For every IT employee

```
Highest Paid

Alice
```

Because Alice is first after sorting.

---

HR

|Employee|Salary|
|---|---|
|Emma|65000|
|Bob|60000|

Output

```
Emma

Emma
```

---

Think

```
FIRST_VALUE

always remembers

the first row.
```

---

# Real-world

Highest paid employee

First purchase

Earliest order

Top scorer






## 


## 

### 1. PRIMARY KEY

A **PRIMARY KEY** uniquely identifies each row in a table.

#### Rules

- Cannot contain NULL
- Must be unique
- Only one primary key per table (can be composite)

#### Example

```sql
CREATE TABLE employee (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    salary NUMERIC
);
```

Insert data

```sql
INSERT INTO employee VALUES
(1, 'John', 50000),
(2, 'Alice', 60000);
```

Trying duplicate

```sql
INSERT INTO employee VALUES
(1, 'Bob', 70000);
```

Output

```
ERROR:
duplicate key value violates primary key constraint
```

Trying NULL

```
INSERT INTO employee VALUES
(NULL, 'David', 40000);
```

Output

```
ERROR:
null value violates primary key constraint
```


---

### 2. FOREIGN KEY

A **FOREIGN KEY** creates a relationship between two tables.

It ensures the value exists in another table.

Example:

### Parent Table

```sql
CREATE TABLE department (
    id INT PRIMARY KEY,
    department_name VARCHAR(50)
);
```

Insert

```sql
INSERT INTO department VALUES
(1,'HR'),
(2,'IT');
```

---

### Child Table

```sql
CREATE TABLE employee (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY(department_id)
        REFERENCES department(id)
);
```

Valid

```sql
INSERT INTO employee VALUES
(101,'John',1);
```

Invalid

```sql
INSERT INTO employee VALUES
(102,'David',5);
```

Output

```
ERROR:
violates foreign key constraint
```

Because department 5 doesn't exist.


---

### 3. UNIQUE

Ensures duplicate values are not allowed.

Unlike Primary Key,

- NULL is allowed (PostgreSQL allows multiple NULLs)
- Multiple UNIQUE constraints can exist.

Example

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE
);
```

Insert

```sql
INSERT INTO users(email)
VALUES
('john@gmail.com');
```

Duplicate

```sql
INSERT INTO users(email)
VALUES
('john@gmail.com');
```

Output

```
duplicate key value violates unique constraint
```

NULL

```sql
INSERT INTO users(email)
VALUES
(NULL);

INSERT INTO users(email)
VALUES
(NULL);
```

This is allowed in PostgreSQL.

---

### 4. CHECK

CHECK validates data based on a condition.

Example

```sql
CREATE TABLE employee (
    id INT PRIMARY KEY,
    age INT CHECK(age >= 18),
    salary NUMERIC CHECK(salary > 0)
);
```

Valid

```sql
INSERT INTO employee VALUES
(1,25,50000);
```

Invalid

```sql
INSERT INTO employee VALUES
(2,16,40000);
```

Output

```
violates check constraint
```

Invalid salary

```sql
INSERT INTO employee VALUES
(3,30,-500);
```

Again,

```
violates check constraint
```


---

### 5. NOT NULL

Ensures a column must always contain a value.

Example

```sql
CREATE TABLE customer (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```

Valid

```sql
INSERT INTO customer(name)
VALUES ('John');
```

Invalid

```sql
INSERT INTO customer(name)
VALUES (NULL);
```

Output

```
null value violates not-null constraint
```


---

### 6. DEFAULT

Assigns a default value when none is provided.

Example

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    status VARCHAR(20) DEFAULT 'Pending'
);
```

Insert

```sql
INSERT INTO orders DEFAULT VALUES;
```

Result

```
id | status
---------------
1  | Pending
```

Or

```
INSERT INTO orders(id)
VALUES (2);
```

Result

```
2 | Pending
```

Override default

```
INSERT INTO orders(status)
VALUES ('Delivered');
```

Result

```
3 | Delivered
```


## 