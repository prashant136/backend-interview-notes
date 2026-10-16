

## Why do we need an Index?

Imagine you have a table with **10 million products**.

```
products
------------------------------------
id | name       | price | brand
------------------------------------
1  | iPhone     | 80000 | Apple
2  | Galaxy     | 70000 | Samsung
3  | Pixel      | 65000 | Google
...
10,000,000 rows
```

Now suppose a user searches:

```sql
SELECT * FROM products
WHERE id = 9876543;
```

How does PostgreSQL find that row?

There are two possibilities:

1. Without an Index
2. With an Index

---

### Without Index → Sequential Scan (Full Table Scan)

If no index exists, PostgreSQL has **no shortcut**.

It starts reading rows one by one.

```
Row 1
↓

Row 2
↓

Row 3
↓

Row 4
↓

...

↓

Row 9,876,543 ← Found
```

This is called a **Sequential Scan** (also known as a Full Table Scan).
The database literally scans every row until it finds the required one.

**Sequential Scan Example**

```sql
SELECT *
FROM products
WHERE name='iPhone';
```


---

### With an Index

Now create an index:

```sql
CREATE INDEX idx_products_id
ON products(id);
```

PostgreSQL creates a separate data structure (by default, a B-tree).

Conceptually:

```
Index

10 → Page A

50 → Page C

100 → Page F

150 → Page K

200 → Page R
```

The index stores sorted key values and pointers to the corresponding table rows.

When searching:

```sql
WHERE id=150
```

PostgreSQL doesn't scan every row. Instead, it navigates the tree and directly reaches the page containing the row.

#### Time Complexity

Without Index

```
Sequential Scan

O(n)
```

With a B-tree Index

```
Tree Search

O(log n)
```

---

#### Visual Comparison

#### Without Index

```
Need ID = 90

10

20

30

40

50

60

70

80

90 ✓
```

Every row is checked.

---

#### With Index

```
          50
        /    \
      20      80
     / \     / \
   10 30   60 90 ✓
```

The search quickly narrows down to the correct value.

---

### Interview Answer

**Q: Why do we need indexes in a database?**

> An index is a special data structure that helps the database locate rows quickly without scanning the entire table. Without an index, the database performs a sequential scan, checking rows one by one, which has **O(n)** time complexity. With a B-tree index, the database can locate data in approximately **O(log n)** time by traversing the tree. This significantly reduces disk I/O, improves query performance, and makes applications scalable, especially for large tables.


## How Does an Index Work?

An **index** is a **separate data structure** that helps the database find rows quickly without scanning the entire table.


#### What Does the Index Store?

An index **does not store the complete row**.

Instead, it stores:

```
Indexed Column Value
        +
Pointer (reference) to the actual table row
```

Example:

```
IT  ------> Row #2
IT  ------> Row #4
HR  ------> Row #1
```

The database uses the pointer to retrieve the full row from the table.

---

## B-Tree Index ⭐⭐⭐⭐⭐

The **B-Tree (Balanced Tree)** is the **default index type in PostgreSQL**.

If you don't specify an index type, PostgreSQL creates a **B-Tree index**.

```sql
CREATE INDEX idx_name
ON employees(name);
```

is equivalent to:

```sql
CREATE INDEX idx_name
ON employees USING BTREE(name);
```

---

### What is a B-Tree?

A **B-Tree (Balanced Tree)** is a tree-like data structure where:

- Data is stored in **sorted order**
- Every path from the root to a leaf has the **same height**
- Searching requires traversing only a few levels
- The tree stays balanced automatically after inserts and deletes

Example:

```
                [50]
              /      \
          [20]        [80]
         /   \       /    \
     [10] [30]   [60]   [90]
```

Instead of checking every value one by one, PostgreSQL follows the correct branch.

---

#### How Search Works

Suppose we search for **30**.

```
                [50]
               /
            [20]
               \
               [30]
```

Steps:

```
30 < 50
↓

Go Left

30 > 20
↓

Go Right

Found 30
```

Only a few comparisons are needed.

Instead of searching:

```
10
20
30
40
50
60
70
80
90
```

the tree allows PostgreSQL to eliminate half of the remaining values at each level.

Time Complexity:

```
Search = O(log n)
```

---

#### How Insert Works

Insert **25**.

Current tree:

```
                [50]
              /
           [20]
          /   \
      [10]   [30]
```

25 belongs between 20 and 30.

```
                [50]
              /
           [20]
          /   \
      [10]   [30]
             /
          [25]
```

If a node becomes full, PostgreSQL **splits the node** and keeps the tree balanced automatically.

```
Before Split

[10 | 20 | 25 | 30 | 40]

     ↓

After Split

        [25]
       /    \
 [10 |20] [30|40]
```

Time Complexity:

```
Insert = O(log n)
```

---

#### How Delete Works

Suppose we delete **30**.

```
Before

       [25]
      /    \
 [10|20] [30|40]

    ↓

After

       [25]
      /    \
 [10|20] [40]
```

If deleting causes a node to become too empty, PostgreSQL automatically merges or redistributes nodes to keep the tree balanced.

Time Complexity:

```
Delete = O(log n)
```

---

#### Why "Balanced" Matters

Imagine an unbalanced tree:

```
10
 \
 20
   \
   30
     \
     40
       \
       50
```

Searching for 50 requires visiting every node.

```
10
↓

20
↓

30
↓

40
↓

50
```

Time Complexity:

```
O(n)
```

Balanced B-Tree:

```
          30
        /    \
      20      40
     /          \
   10            50
```

Search:

```
30

↓

40

↓

50
```

Time Complexity:

```
O(log n)
```

That is why PostgreSQL keeps the tree balanced automatically.

---

### When Does PostgreSQL Use a B-Tree Index?

B-Tree works best for:

#### Equality

```sql
SELECT * FROM employees
WHERE id = 100;
```

---

#### Greater Than / Less Than

```sql
WHERE salary > 50000;
```

```sql
WHERE age < 30;
```

---

#### BETWEEN

```sql
WHERE salary BETWEEN 50000 AND 80000;
```

---

#### ORDER BY

```sql
SELECT *
FROM employees
ORDER BY salary;
```

The index is already sorted, so PostgreSQL can often avoid a separate sort.

---

#### MIN() and MAX()

```sql
SELECT MIN(price) FROM products;

SELECT MAX(price) FROM products;
```

The smallest and largest values are at the ends of the B-Tree.

---

#### Prefix Search

```sql
WHERE name LIKE 'Pra%';
```

This can use a B-Tree index because the prefix defines a contiguous range.

---

### W hen B-Tree Is NOT Useful

#### Contains Search

```sql
WHERE name LIKE '%abc%';
```

The leading wildcard prevents efficient B-Tree usage.

---

#### Full-Text Search

```sql
WHERE description @@ to_tsquery('phone');
```

Use a **GIN** index instead.

---

#### Arrays / JSONB Containment

```sql
WHERE tags @> ARRAY['java'];
```

Use **GIN**.

---

#### Spatial Data

```sql
WHERE ST_Contains(...);
```

Use **GiST** or **SP-GiST**.

---

### Why Is B-Tree the Default Index? ⭐ Interview Question


> PostgreSQL uses B-Tree as the default index because it supports the most common query patterns efficiently. It provides **O(log n)** search, insert, and delete operations while keeping data sorted. Since many applications frequently use equality (`=`), range (`>`, `<`, `BETWEEN`), sorting (`ORDER BY`), and aggregate operations like `MIN()` and `MAX()`, B-Tree is the best general-purpose index. It automatically stays balanced, ensuring consistent performance even as the table grows.



## EXPLAIN and EXPLAIN ANALYZE ⭐⭐⭐⭐⭐

One of the most important PostgreSQL interview topics.

---

### EXPLAIN ?

`EXPLAIN` shows the **execution plan** that PostgreSQL intends to use.

It **does not execute** the query.

Example:

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department = 'IT';
```

Example output:

```
Index Scan using idx_department on employees
```

This tells you PostgreSQL plans to use the `idx_department` index.

---

### EXPLAIN ANALYZE ?

`EXPLAIN ANALYZE` actually **executes the query** and then shows:

- Execution plan
- Actual execution time
- Number of rows processed
- Actual vs estimated rows

Example:

```sql
EXPLAIN ANALYZE
SELECT *
FROM employees
WHERE department = 'IT';
```

Example output:

```
Index Scan using idx_department on employees
(actual time=0.12..0.20 rows=2 loops=1)
```

This is what DBAs and backend engineers use to analyze slow queries.

---

#### Seq Scan (Sequential Scan)

This means PostgreSQL reads **every row** in the table.

**When does PostgreSQL choose Seq Scan?**

- No index exists
- Table is very small
- Most rows match the condition (using an index would not save much work)

---

#### Index Scan

Suppose an index exists:

```sql
CREATE INDEX idx_department
ON employees(department);
```

Instead of reading every row, PostgreSQL jumps directly to matching rows.

**When is Index Scan used?**

- Few rows match
- Index exists
- Random row access is cheaper than scanning the whole table

---

### Bitmap Index Scan

### Bitmap Heap Scan

### Bitmap Index Scan + Bitmap Heap Scan Together


### Interview Answer (2 Minutes)

> `EXPLAIN` shows the execution plan that PostgreSQL intends to use without running the query. `EXPLAIN ANALYZE` executes the query and shows the actual execution plan, timing, and row counts. A **Sequential Scan** reads every row in the table. An **Index Scan** uses an index to directly fetch a small number of matching rows. When a query returns many matching rows, PostgreSQL may use a **Bitmap Index Scan** to collect matching row locations and then a **Bitmap Heap Scan** to read the required table pages efficiently, reducing random disk access. The optimizer chooses the plan with the lowest estimated cost.



## Composite, Covering and Partial Index ?

These are **very common PostgreSQL interview topics**. Many candidates know what they are but struggle to explain **when to use them**. Let's use one realistic e-commerce application throughout.

Suppose we have an `orders` table.

```
orders

order_id
customer_id
status
order_date
total_amount
city
payment_method
```

Sample data:

|order_id|customer_id|status|order_date|total_amount|city|
|---|---|---|---|---|---|
|101|1|DELIVERED|2026-01-10|1200|Delhi|
|102|2|PENDING|2026-01-12|800|Mumbai|
|103|1|DELIVERED|2026-02-01|650|Delhi|
|104|3|CANCELLED|2026-02-05|400|Pune|

---

### Composite Index ⭐⭐⭐⭐⭐


A **Composite Index** is an index built on **multiple columns**.

```sql
CREATE INDEX idx_customer_status
ON orders(customer_id, status);
```

Notice the order:

```
(customer_id, status)
```

Order matters!

---

#### Real World Scenario

Imagine Amazon.

A customer opens:

> "Show me all my delivered orders."

Query:

```sql
SELECT *
FROM orders
WHERE customer_id = 101
AND status = 'DELIVERED';
```

Without an index:

```
Read Order 1

Read Order 2

Read Order 3

...

Read 10 million rows
```

With a composite index:

```
(customer_id, status)

(101, CANCELLED)

(101, DELIVERED)

(101, DELIVERED)

(101, PENDING)

(102, DELIVERED)
```

PostgreSQL directly jumps to:

```
(101, DELIVERED)
```

Very few rows are read.

---

#### Why Not Two Separate Indexes?

Suppose we create:

```sql
CREATE INDEX idx_customer
ON orders(customer_id);

CREATE INDEX idx_status
ON orders(status);
```

Now PostgreSQL may need to:

```
Search customer index

+

Search status index

+

Combine results
```

A composite index is often more efficient because both conditions are stored together.

---

#### Left Prefix Rule 

This is the most important interview point.

Index:

```
(customer_id, status)
```

Works for:

```sql
WHERE customer_id = 101
```

✅ Uses index

---

Works for:

```sql
WHERE customer_id = 101
AND status='DELIVERED'
```

✅ Uses index

---

Works for:

```sql
WHERE customer_id=101
ORDER BY status;
```

✅ Uses index

---

Does NOT work efficiently for:

```sql
WHERE status='DELIVERED'
```

❌ Because the first indexed column (`customer_id`) is missing.

This is called the **Left Prefix Rule**.

---

#### Interview Question

**Why is column order important?**

Because PostgreSQL stores values in sorted order.

```
(customer_id, status)

101 DELIVERED

101 PENDING

102 DELIVERED

102 PENDING
```

It cannot efficiently jump directly to:

```
status = DELIVERED
```

without first organizing by `customer_id`.

---

#### When should you create a Composite Index?

When multiple columns are **frequently queried together**.

Examples:

```
(customer_id, status)

(user_id, created_at)

(category_id, price)

(country, city)
```

---

### Covering Index ⭐⭐⭐⭐⭐

One of the most misunderstood topics.

A **Covering Index** is simply **an index that contains everything a query needs**, so PostgreSQL doesn't have to go back to the table.

---


```sql
SELECT customer_id, status
FROM orders
WHERE customer_id=101;
```

Index:

```
(customer_id)
```

PostgreSQL:

```
Search index

↓

Find row pointer

↓

Read table

↓

Return customer_id,status
```

Even though `customer_id` is in the index, PostgreSQL still needs to visit the table to get `status`.

---

**Solution**

Create an index that also stores `status`.

In PostgreSQL:

```sql
CREATE INDEX idx_customer_cover
ON orders(customer_id)
INCLUDE(status);
```

Now the index stores:

```
customer_id

status
```

Query:

```
SELECT customer_id,status
FROM orders
WHERE customer_id=101;
```

Everything required is already inside the index.

```
Search Index

↓

Return Result

↓

No Table Lookup
```

This is called an **Index Only Scan**.

---

### Partial Index ⭐⭐⭐⭐⭐

A Partial Index indexes **only some rows**, not the whole table.

---

Suppose:

```
orders

10 million rows
```

Status distribution:

```
DELIVERED = 9.7 million

PENDING = 200k

CANCELLED =100k
```

Your customer support team always searches only pending orders.

```sql
SELECT *
FROM orders
WHERE status='PENDING';
```

Creating an index on every row:

```sql
CREATE INDEX idx_status
ON orders(status);
```

This indexes all 10 million rows.

Wasteful.

---

*Instead:*

```sql
CREATE INDEX idx_pending
ON orders(status)
WHERE status='PENDING';
```

Only pending rows are indexed.

```
Index

PENDING

PENDING

PENDING

PENDING
```

No delivered rows.

---

#### Real World Scenario

Suppose Flipkart has:

```
50 million orders
```

Customer support always opens:

```
Pending Orders
```

Never searches delivered orders.

Instead of indexing everything:

```
Index only pending orders.
```

Massive memory savings.

---

### Another Real Example

Users table

```
id

email

is_active
```

99% users are active.

Admin checks only inactive users.

```sql
SELECT *
FROM users
WHERE is_active=false;
```

Create:

```sql
CREATE INDEX idx_inactive
ON users(is_active)
WHERE is_active=false;
```

Perfect use case.

---

### When to Use Which?

#### Composite Index

```
WHERE customer_id=?

AND status=?
```

Use:

```
(customer_id,status)
```

---

#### Covering Index

```
WHERE customer_id=?

SELECT customer_id,status
```

Use:

```
(customer_id)
INCLUDE(status)
```

---

#### Partial Index

```
WHERE status='PENDING'
```

Use:

```
WHERE status='PENDING'
```

---

### Interview Answer (2 Minutes)

> A **Composite Index** is created on multiple columns and is useful when those columns are frequently used together in filtering or sorting. The order of columns is important because PostgreSQL follows the left-prefix rule. A **Covering Index** includes additional columns using the `INCLUDE` clause so PostgreSQL can satisfy the query directly from the index, often resulting in an **Index Only Scan** without accessing the table. A **Partial Index** indexes only rows that satisfy a condition, such as `status = 'PENDING'`. This reduces index size, improves write performance, and is ideal when queries repeatedly target a small subset of the data.



## Hash Index 

A **Hash Index** is a special index type optimized **only for equality (`=`) searches**.

Unlike a B-Tree, a Hash Index **does not store values in sorted order**. Instead, it stores a **hash value** (a computed code) for each indexed value, making exact lookups very fast.

---
#### Example Table

```
users

id
email
name
city
```

Suppose we frequently search by email.

```sql
SELECT *
FROM users
WHERE email = 'john@gmail.com';
```

Create a hash index:

```sql
CREATE INDEX idx_email_hash
ON users USING HASH(email);
```

---

#### How It Works

Suppose the emails are:

|Email|
|---|
|alice@gmail.com|
|bob@gmail.com|
|john@gmail.com|

The hash function computes something like:

```
alice@gmail.com → Bucket 12

bob@gmail.com   → Bucket 7

john@gmail.com  → Bucket 25
```

Internally:

```
Bucket 7
↓

bob@gmail.com

----------------

Bucket 12
↓

alice@gmail.com

----------------

Bucket 25
↓

john@gmail.com
```

Searching for:

```sql
WHERE email='john@gmail.com'
```

becomes:

```sql
Hash(email)

↓

Bucket 25

↓

Found
```

Very fast.

---

### Supported Queries

#### Equality (`=`) ✅

```sql
SELECT *
FROM users
WHERE email='john@gmail.com';
```

Excellent.

---

#### IN

```sql
WHERE email IN ('a@gmail.com','b@gmail.com');
```

Also works well because each value is an equality lookup.

---

#### Unsupported Queries 
- Greater Than
- BETWEEN
- ORDER BY
- Prefix Search

#### Real World Scenario

Suppose you're building a login service.

Table:

```
users

id

email

password
```

Every login does:

```sql
SELECT *
FROM users
WHERE email='abc@gmail.com';
```

Always equality.

A Hash Index is a good conceptual fit because you're always looking for an exact email match.

---

### But Why Don't We Use Hash Indexes Everywhere?

Because B-Tree already handles equality very efficiently **and** supports many more operations.













































# Database Indexing: Complete Guide

### What is an Index?

A database index is a special data structure that helps the database find rows faster without scanning the entire table.

Think of a book:

- Without an index → You read every page to find a topic.
- With an index → You go directly to the page number.

Similarly, a database index helps locate records quickly.

---

### Why is Indexing Needed?

Imagine an `employees` table with 10 million records.

```
SELECT * FROM employees
WHERE email = 'john@gmail.com';
```

Without an index:

- Database performs a **Full Table Scan**
- Checks every row
- Time Complexity ≈ O(n)

With an index on email:

```
CREATE INDEX idx_employee_email
ON employees(email);
```

Database:

- Uses B-Tree structure
- Navigates directly to the record
- Time Complexity ≈ O(log n)

Instead of reading 10 million rows, it may read only a few pages.

---

### How Index Works Internally

Most databases use **B-Tree Indexes**.

Example:

```
                    M
                  /   \
              G-L      N-Z
             /            \
       Adam...        Steve...
```

Searching for "Steve":

1. Start at root
2. Compare values
3. Move to correct branch
4. Find record quickly

---

### Without Index

Table:

|id|name|
|---|---|
|1|Adam|
|2|John|
|3|Steve|
|4|Mike|

Query:

```
SELECT * FROM users
WHERE name = 'Steve';
```

Database checks:

```
Adam? No
John? No
Steve? Yes
```

This is called:

**Full Table Scan**

---

### With Index

```
CREATE INDEX idx_name
ON users(name);
```

Database directly jumps to:

```
Steve -> Row #3
```

Much faster.

---

## Types of Indexes

### 1. Single Column Index

Most common.

```
CREATE INDEX idx_email
ON users(email);
```

Used for:

```
SELECT *
FROM users
WHERE email='abc@gmail.com';
```

---

### 2. Composite Index (Multi-Column)

```
CREATE INDEX idx_first_last
ON users(first_name, last_name);
```

Useful for:

```
SELECT *
FROM users
WHERE first_name='John'
AND last_name='Doe';
```

#### Leftmost Prefix Rule

Index:

```
(first_name, last_name)
```

Works for:

```
WHERE first_name='John'
```

Works for:

```
WHERE first_name='John'
AND last_name='Doe'
```

Does NOT efficiently work for:

```
WHERE last_name='Doe'
```

because first column is skipped.

---

### 3. Unique Index

Ensures uniqueness.

```
CREATE UNIQUE INDEX idx_email
ON users(email);
```

or

```
ALTER TABLE users
ADD CONSTRAINT unique_email UNIQUE(email);
```

Prevents:

```
abc@gmail.com
abc@gmail.com
```

duplicate entries.

---

### 4. Primary Key Index

```
CREATE TABLE users(
   id BIGINT PRIMARY KEY
);
```

Database automatically creates an index.

No need to create manually.

---

### 5. Clustered Index

Determines physical storage order of rows.

Example:

```
PRIMARY KEY(id)
```

Rows stored:

```
1
2
3
4
5
```

Benefits:

- Fast range queries

Usually only one clustered index per table.

#### In InnoDB (MySQL)

Primary Key = Clustered Index.

---

### 6. Non-Clustered Index

Stores:

```
Indexed Value -> Pointer to Row
```

Example:

```
CREATE INDEX idx_salary
ON employees(salary);
```

Table order remains unchanged.

---

## When Should You Create Indexes?

### 1. Frequently Used WHERE Columns

Good:

```
SELECT *
FROM employees
WHERE email = ?;
```

Create:

```
CREATE INDEX idx_email
ON employees(email);
```

---

### 2. JOIN Columns

Good:

```
SELECT *
FROM orders o
JOIN users u
ON o.user_id = u.id;
```

Create:

```
CREATE INDEX idx_user_id
ON orders(user_id);
```

---

### 3. ORDER BY Columns

```
SELECT *
FROM employees
ORDER BY created_at DESC;
```

Index:

```
CREATE INDEX idx_created
ON employees(created_at);
```

---

### 4. GROUP BY Columns

```
SELECT department,
COUNT(*)
FROM employees
GROUP BY department;
```

Index:

```
CREATE INDEX idx_department
ON employees(department);
```

---

### 5. Range Queries

```
SELECT *
FROM orders
WHERE created_at
BETWEEN '2025-01-01'
AND '2025-01-31';
```

Index:

```
CREATE INDEX idx_order_date
ON orders(created_at);
```

---

## When NOT to Create Indexes?

### 1. Small Tables

Table with 100 rows:

```
SELECT * FROM users;
```

Index gives almost no benefit.

---

### 2. Frequently Updated Columns

Bad:

```
CREATE INDEX idx_status
ON orders(status);
```

if status changes frequently.

Example:

```
PENDING -> SHIPPED -> DELIVERED
```

Every update requires index updates.

---

### 3. Low Cardinality Columns

Bad candidate:

```
gender
```

Values:

```
Male
Female
```

Only 2 values.

Database often prefers full scan.

---

### 4. Too Many Indexes

Table:

```
users
```

20 columns.

Creating indexes on all 20 columns is bad.

---

## Advantages of Indexing

### 1. Faster SELECT Queries

Huge performance improvement.

Example:

```
100 ms -> 2 ms
```

---

### 2. Faster JOIN Operations

```
users
orders
```

joined much faster.

---

### 3. Faster Sorting

```
ORDER BY
```

can use index.

---

### 4. Faster Aggregations

```
GROUP BY
COUNT
MIN
MAX
```

often benefit.

---

## Disadvantages of Indexing

### 1. More Storage

Table:

```
5 GB
```

Indexes:

```
2 GB
```

Total:

```
7 GB
```

---

### 2. Slower INSERT

```
INSERT INTO users ...
```

Database must:

1. Insert row
2. Update all indexes

More indexes = slower inserts.

---

### 3. Slower UPDATE

```
UPDATE users
SET email='new@gmail.com';
```

Index on email must be updated.

---

### 4. Slower DELETE

Deleted row must be removed from all indexes.

---

### Example: Instagram Clone

Suppose you have:

```
users
posts
comments
likes
followers
```

#### Users Table

```
CREATE INDEX idx_user_email
ON users(email);

CREATE INDEX idx_user_username
ON users(username);
```

Because login/search uses them frequently.

---

#### Posts Table

```
CREATE INDEX idx_post_user
ON posts(user_id);

CREATE INDEX idx_post_created
ON posts(created_at);
```

Queries:

```
SELECT *
FROM posts
WHERE user_id = 10
ORDER BY created_at DESC;
```

---

#### Followers Table

```
CREATE INDEX idx_following
ON followers(following_id);

CREATE INDEX idx_follower
ON followers(follower_id);
```

Queries:

```
Get all followers of user
Get all following users
```

become efficient.

---

## How to Check If Query Uses an Index?

Use `EXPLAIN`.

### MySQL

```
EXPLAIN
SELECT *
FROM users
WHERE email='abc@gmail.com';
```

Output contains:

```
type: ref
key: idx_email
```

Meaning index is used.

Bad output:

```
type: ALL
```

Meaning full table scan.

---

## Interview Questions

### Q1: Why is indexing required?

To avoid full table scans and improve query performance.

---

### Q2: Does indexing improve INSERT?

No.

INSERT, UPDATE, DELETE become slower because indexes must be maintained.

---

### Q3: What columns should be indexed?

- WHERE columns
- JOIN columns
- ORDER BY columns
- GROUP BY columns
- High-cardinality columns (many unique values)

---

### Q4: What is cardinality?

Number of unique values in a column.

Example:

```
email      -> High Cardinality
user_id    -> High Cardinality
gender     -> Low Cardinality
status     -> Low Cardinality
```

Higher cardinality columns are generally better index candidates.

---

### Practical Rule for Backend Engineers

For applications like Spring Boot + PostgreSQL/MySQL:

Create indexes on:

```
PRIMARY KEY
FOREIGN KEY
email
username
user_id
created_at
```

and any column that:

- Appears frequently in `WHERE`
- Appears frequently in `JOIN`
- Appears frequently in `ORDER BY`
- Appears frequently in `GROUP BY`

Then use:

```
EXPLAIN ANALYZE
```

(PostgreSQL)

or

```
EXPLAIN
```

(MySQL)

to verify that the database is actually using the index. Avoid creating indexes blindly; measure query performance first.

-----

# How do you implement pagination in SQL ? (offset vs cursor) based pagination ?


Pagination is used when you don't want to load millions of rows at once.

Example:

Instagram Feed

```
Post 1
Post 2
Post 3
...
```

Instead of loading 1 million posts:

```
SELECT * FROM posts;
```

you load only a small chunk (10, 20, 50 records).

---

### Why Pagination?

Without Pagination

```
SELECT * FROM posts;
```

Problems:

- High memory usage
- Slow response time
- Network overhead
- Bad user experience

With Pagination

```
SELECT * FROM posts
LIMIT 20;
```

Only 20 records returned.

---

## Types of Pagination

There are mainly 2 types:

1. Offset Pagination
2. Cursor (Keyset) Pagination

---

## 1. Offset Pagination

Most commonly used.

#### Page 1

```
SELECT *
FROM posts
ORDER BY created_at DESC
LIMIT 10 OFFSET 0;
```

Returns:

```
Rows 1 - 10
```

---

#### Page 2

```
SELECT *
FROM posts
ORDER BY created_at DESC
LIMIT 10 OFFSET 10;
```

Returns:

```
Rows 11 - 20
```

---

#### Page 3

```
SELECT *
FROM posts
ORDER BY created_at DESC
LIMIT 10 OFFSET 20;
```

Returns:

```
Rows 21 - 30
```

---

#### Formula

```
OFFSET = (PageNumber - 1) * PageSize
```

Example:

```
Page Number = 5
Page Size = 20

OFFSET = (5 - 1) * 20
OFFSET = 80
```

Query:

```
SELECT *
FROM posts
ORDER BY created_at DESC
LIMIT 20 OFFSET 80;
```

---

#### Java Spring Example

```
int page = 3;
int size = 10;

int offset = (page - 1) * size;
```

SQL:

```
SELECT *
FROM posts
ORDER BY created_at DESC
LIMIT :size OFFSET :offset;
```

---

### [ Problem With Offset Pagination ]

### The Problem is - pagination drift ?

Pagination Drift happens when data changes (INSERT/DELETE) between page requests, causing:

- Duplicate records
- Missing records
- Inconsistent results

because OFFSET is based on **position**, not on a fixed record.

---

#### Example 1: Duplicate Records Due to INSERT

Suppose posts are ordered by newest first.

Table:

|id|
|---|
|100|
|99|
|98|
|97|
|96|
|95|
|94|
|93|
|92|
|91|

Query for Page 1:

```
SELECT *
FROM posts
ORDER BY id DESC
LIMIT 5 OFFSET 0;
```

Result:

|id|
|---|
|100|
|99|
|98|
|97|
|96|

User sees:

```
100 99 98 97 96
```

---

##### Now a New Post Arrives

Someone creates a new post.

```
101
100
99
98
97
96
95
94
93
92
91
```

Notice every old row shifted down by 1 position.

---

##### User Requests Page 2

Page 2:

```
SELECT *
FROM posts
ORDER BY id DESC
LIMIT 5 OFFSET 5;
```

Database skips first 5 rows:

```
101
100
99
98
97
```

Then returns:

```
96
95
94
93
92
```

User now sees:

```
Page 1:
100 99 98 97 96

Page 2:
96 95 94 93 92
```

###### Problem

Record `96` appeared twice.

This is Pagination Drift.

---

##### Visual Representation

Before insert:

```
Position   Record
1          100
2          99
3          98
4          97
5          96
6          95
7          94
8          93
```

Page 1 = positions 1-5

```
100 99 98 97 96
```

---

After insert:

```
Position   Record
1          101
2          100
3          99
4          98
5          97
6          96
7          95
8          94
```

Page 2 asks:

```
OFFSET 5
```

Database starts from position 6:

```
96 95 94 ...
```

Thus `96` appears again.

---

#### Example 2: Missing Records Due to DELETE

Original data:

```
100
99
98
97
96
95
94
93
92
91
```

Page 1:

```
LIMIT 5 OFFSET 0
```

Result:

```
100
99
98
97
96
```

---

Now record `99` gets deleted.

New table:

```
100
98
97
96
95
94
93
92
91
```

---

User requests Page 2:

```
LIMIT 5 OFFSET 5
```

Database skips:

```
100
98
97
96
95
```

Returns:

```
94
93
92
91
```

---

Notice what happened?

User saw:

```
Page 1:
100
99
98
97
96

Page 2:
94
93
92
91
```

Record `95` was never shown.

It disappeared from pagination.

This is another form of Pagination Drift.

---

#### Why Does This Happen?

Because OFFSET works on:

```
Row Position
```

not

```
Record Identity
```

When rows are inserted or deleted:

```
Position changes
```

But OFFSET still says:

```
Skip first 5 rows
```

###### The database doesn't know what the user saw previously.
---

### Performance Issue

Suppose:

```
10 Million Records
```

Query:

```
LIMIT 20 OFFSET 5000000
```

Database still needs to skip:

```
5 Million Rows
```

before returning data.

Very expensive.

---

#### Offset Pagination Advantages

#### Good For

- Admin dashboards
- Reporting systems
- Small datasets

---

#### Offset Pagination Disadvantages

#### Slow For Large Data

```
OFFSET 1000000
```

can become expensive.

##### Duplicate Records

New inserts can shift pages.

##### Missing Records

Deletes can shift pages too.

## 2. Cursor Pagination (Keyset Pagination)

Used by:

- Instagram
- Facebook
- Twitter/X
- LinkedIn
- YouTube Feed

Most large-scale systems prefer this.

---

### Idea

Instead of saying:

```
Skip 100 rows
```

Say:

```
Give rows after ID 95
```

---

### Table

```
100
99
98
97
96
95
94
93
92
91
```

---

#### First Request

```
SELECT *
FROM posts
ORDER BY id DESC
LIMIT 5;
```

Result:

```
100
99
98
97
96
```

Last id:

```
96
```

Return cursor:

```
{
  "nextCursor": 96
}
```

---

#### Next Request

```
SELECT *
FROM posts
WHERE id < 96
ORDER BY id DESC
LIMIT 5;
```

Result:

```
95
94
93
92
91
```

---

#### Next Cursor

```
91
```

---

### Why Faster?

Because index can directly jump.

```
WHERE id < 96
```

uses Primary Key index.

Database doesn't scan millions of rows.

---

### Typical API Design

Request

```
GET /posts?cursor=96&limit=20
```

SQL

```
SELECT *
FROM posts
WHERE id < 96
ORDER BY id DESC
LIMIT 20;
```

Response

```
{
  "data": [...],
  "nextCursor": 76
}
```

---

### Cursor Using Created At

Instead of ID:

```
SELECT *
FROM posts
WHERE created_at < '2026-06-20 10:00:00'
ORDER BY created_at DESC
LIMIT 20;
```

Very common for feeds.

---

#### [ Composite Cursor ]

**Composite cursor pagination** is ==a database fetching technique that tracks the last-seen item using multiple columns== (e.g., a timestamp and a unique ID). It solves the duplicates and performance issues of offset pagination by executing deterministic "index seeks" instead of scanning the entire dataset.

Sometimes timestamps can be equal.

Bad:

```
10:00:00
10:00:00
10:00:00
```

Use:

```
(created_at, id)
```

Query:

```
SELECT *
FROM posts
WHERE
(created_at, id)
<
('2026-06-20 10:00:00', 500)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

This guarantees stable ordering.

---

#### Required Index

For cursor pagination:

```
CREATE INDEX idx_posts_created_id
ON posts(created_at DESC, id DESC);
```

Very important.

Without index, cursor pagination loses much of its advantage.

---

#### Offset vs Cursor

|Feature|Offset|Cursor|
|---|---|---|
|Easy Implementation|✅|Slightly harder|
|Page Numbers|✅|❌|
|Large Dataset Performance|❌|✅|
|Duplicate Records|❌|✅|
|Missing Records|❌|✅|
|Infinite Scroll|❌|✅|
|Social Media Feed|❌|✅|
|Admin Dashboard|✅|✅|

---

### Real Interview Answer

**Question:** How would you implement pagination?

> There are two common approaches: Offset Pagination and Cursor Pagination.
> 
> Offset pagination uses LIMIT and OFFSET:
> 
> 
	SELECT *
	FROM posts
	ORDER BY created_at DESC
	LIMIT 20 OFFSET 40;
> 
> 
> It is easy to implement and supports page numbers, but becomes slow for large offsets because the database still scans and skips rows.
> 
> Cursor pagination uses the last retrieved record as a cursor:
> 
> 
	SELECT *
	FROM posts
	WHERE id < :lastId
	ORDER BY id DESC
	LIMIT 20;
> 
> 
> This leverages indexes efficiently, avoids duplicate/missing records caused by concurrent inserts/deletes, and scales much better for large datasets. That's why social media feeds and large-scale systems typically use cursor pagination, while admin dashboards often use offset pagination.

### FAANG-Level Follow-up

If asked:

**"Which pagination would you use for Instagram?"**

Answer:

> I would use Cursor Pagination based on `(created_at, id)` with a composite index:
> 
> 
	CREATE INDEX idx_posts_created_id
	ON posts(created_at DESC, id DESC);
> 
> 
> Cursor pagination provides O(log n) index lookup, supports infinite scrolling efficiently, and avoids pagination drift caused by new posts being inserted while the user is scrolling.




# Views over tables? Why use views? View vs Materialized Views? Advantage /Disadvantage ?

## What is a View in SQL?

A **View** is a virtual table created from a SQL query.

Instead of storing data itself, it stores only the query definition.

Example:

Suppose we have:

### employees

|id|name|salary|department|
|---|---|---|---|
|1|John|100000|IT|
|2|Mike|80000|HR|
|3|David|120000|IT|

---

Create a View:

```
CREATE VIEW it_employees AS
SELECT id, name, salary
FROM employees
WHERE department = 'IT';
```

Now:

```
SELECT *
FROM it_employees;
```

Output:

|id|name|salary|
|---|---|---|
|1|John|100000|
|3|David|120000|

---

### Intuition: Why Were Views Invented?

Imagine every developer writes:

```
SELECT e.id,
       e.name,
       d.department_name,
       m.manager_name
FROM employees e
JOIN departments d
ON e.department_id = d.id
JOIN managers m
ON e.manager_id = m.id;
```

Again and again.

Problems:

- Query duplication
- Hard maintenance
- High chance of mistakes

Instead:

```
CREATE VIEW employee_details AS
SELECT ...
```

Then everyone uses:

```
SELECT *
FROM employee_details;
```

One place to maintain.

---

### Think of a View Like a Saved Query

A View is essentially:

```
View = Stored SELECT statement
```

When you query a View:

```
SELECT *
FROM employee_details;
```

Database internally executes:

```
SELECT ...
FROM employees
JOIN departments
JOIN managers
```

every time.

---

## Why Use Views ? (advantage)

### 1. Simplify Complex Queries ~ cleaner SQL

Without View:

```
SELECT e.name,
       d.name,
       m.name
FROM employees e
JOIN departments d
ON e.department_id=d.id
JOIN managers m
ON e.manager_id=m.id;
```

With View:

```
SELECT *
FROM employee_details;
```

Much cleaner.

---

### 2. Reusability

Used by:

- Backend APIs
- Reports
- BI dashboards
- Analysts

Everyone shares same logic.

---

### 3. Security

Suppose employees table:

|id|name|salary|
|---|---|---|

Don't want everyone to see salary.

Create View:

```
CREATE VIEW employee_public AS
SELECT id, name
FROM employees;
```

Give access only to View.

```
GRANT SELECT
ON employee_public
TO analyst;
```

Salary hidden.

---

### 4. Business Logic Centralization

Instead of:

```
WHERE status='ACTIVE'
```

written in 50 places.

Create:

```
CREATE VIEW active_users AS
SELECT *
FROM users
WHERE status='ACTIVE';
```

Everyone uses:

```
SELECT *
FROM active_users;
```

---

## View Disadvantages

### No Performance Improvement

Many beginners think:

```
View = Faster Query
```

Wrong.

Normal views don't store results.

Every execution reruns query.

---

### Complex Views Become Slow

View:

```
10 joins
5 aggregations
```

Every call executes all of it.

---

### Nested Views

Bad design:

```
View A
uses View B

View B
uses View C

View C
uses View D
```

Hard to debug.

---

## [ Materialized View ]

Now comes the important part.

---

### Problem with Normal View

Suppose:

```
SELECT department,
       COUNT(*)
FROM employees
GROUP BY department;
```

Table:

```
100 million rows
```

Running every minute is expensive.

---

Materialized View solves this.

```
CREATE MATERIALIZED VIEW department_stats AS
SELECT department,
       COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

Database stores:

```
department | employee_count
IT         | 5000000
HR         | 2000000
```

physically on disk.

---

### Intuition

Normal View:

```
Recipe
```

Materialized View:

```
Cooked Food
```

---

Normal View:

```
Every request:
Cook again
```

Materialized View:

```
Food already prepared
Serve immediately
```

---

### Normal View vs Materialized View

#### Normal View

```
Stores Query
```

Example:

```
CREATE VIEW v AS
SELECT ...
```

Data stored?

```
NO
```

---

### Materialized View

```
Stores Query Result
```

Example:

```
CREATE MATERIALIZED VIEW mv AS
SELECT ...
```

Data stored?

```
YES
```

---

### Example

### Orders

10 Crore Rows

Need dashboard:

```
SELECT DATE(order_date),
       SUM(amount)
FROM orders
GROUP BY DATE(order_date);
```

Running every page load:

```
Very expensive
```

Create Materialized View:

```
CREATE MATERIALIZED VIEW daily_sales AS
SELECT DATE(order_date) day,
       SUM(amount) total_sales
FROM orders
GROUP BY DATE(order_date);
```

Dashboard:

```
SELECT *
FROM daily_sales;
```

Super fast.

---

### Materialized View Refresh

Problem:

New data arrives.

Materialized View becomes stale.

Need refresh.

PostgreSQL:

```
REFRESH MATERIALIZED VIEW daily_sales;
```

After refresh:

```
Latest data appears
```

---
## Real-World Examples

### View

Instagram User Profile

```
CREATE VIEW user_profile AS
SELECT ...
FROM users
JOIN posts
JOIN followers;
```

Used by APIs.

Purpose:

```
Abstraction
Security
Reuse
```

---

### Materialized View

Analytics Dashboard

```
Daily Active Users
Monthly Revenue
Top Products
```

Huge tables.

Need fast reads.

Purpose:

```
Performance
Pre-computation
```

---

### Interview Answer

### What is a View?

> A View is a virtual table based on a stored SQL query. It doesn't store data itself; it stores only the query definition and executes that query whenever the view is accessed.

### Why use Views?

> To simplify complex queries, centralize business logic, improve reusability, and restrict access to sensitive columns.

### What is a Materialized View?

> A Materialized View stores the result of a query physically on disk. Since results are precomputed, reads are much faster, but the data can become stale and requires refreshes.

### Difference?

> A View stores only the query and always returns fresh data. A Materialized View stores the query result, offers much faster reads, but consumes storage and requires refresh operations to stay up to date.

---

## FAANG-Level Rule

Use a **View** when the goal is:

- Abstraction
- Security
- Reusability
- Cleaner SQL

Use a **Materialized View** when the goal is:

- Faster reporting
- Analytics dashboards
- Aggregations over millions of rows
- Read-heavy workloads where slight staleness is acceptable.

-----
----


---
----

# Self join? Why it is required ?

A **Self Join** is a join where a table is joined **with itself**.

Even though it is the same table, SQL treats each occurrence as a different table using **aliases**.

```
SELECT *
FROM employees e1
JOIN employees e2
ON e1.manager_id = e2.id;
```

Here:

- `e1` → Employee
- `e2` → Manager

Both refer to the same `employees` table.

### Why is Self Join Needed?

Sometimes **related data exists within the same table**.

For example, consider an `employees` table.

|id|name|manager_id|
|---|---|---|
|1|Alice|NULL|
|2|Bob|1|
|3|Charlie|1|
|4|David|2|
|5|Emma|2|

Notice:

- Bob is an employee.
- Alice is also an employee.
- Alice is Bob's manager.

There is **no separate `managers` table**.

The relationship is stored using `manager_id`.

So, to retrieve an employee along with their manager's name, the table must be joined with itself.

---

### Problem Without Self Join

Suppose you want:

```
Employee      Manager
Bob           Alice
Charlie       Alice
David         Bob
Emma          Bob
```

If you simply query the table:

```
SELECT * FROM employees;
```

Result:

|id|name|manager_id|
|---|---|---|
|2|Bob|1|

You know Bob's manager has ID = 1.

But:

**Who is ID 1?**

The database doesn't automatically replace the ID with the manager's name.

---

### Solution: Self Join

```
SELECT
    e.name AS employee_name,
    m.name AS manager_name
FROM employees e
JOIN employees m
ON e.manager_id = m.id;
```

Result

|Employee|Manager|
|---|---|
|Bob|Alice|
|Charlie|Alice|
|David|Bob|
|Emma|Bob|
### Other Common Uses of Self Join

#### 1. Find Customers Referred by Other Customers

Table

|id|name|referred_by|
|---|---|---|
|1|John|NULL|
|2|Mike|1|
|3|Steve|2|

Query

```
SELECT
    c.name AS customer,
    r.name AS referred_by
FROM customers c
LEFT JOIN customers r
ON c.referred_by = r.id;
```

Result

|Customer|Referred By|
|---|---|
|Mike|John|
|Steve|Mike|

---

#### 2. Category Hierarchy

Table

|id|category|parent_id|
|---|---|---|
|1|Electronics|NULL|
|2|Mobiles|1|
|3|Laptops|1|
|4|Android|2|

Query

```
SELECT
    c.category,
    p.category AS parent_category
FROM categories c
LEFT JOIN categories p
ON c.parent_id = p.id;
```

Result

|Category|Parent|
|---|---|
|Mobiles|Electronics|
|Android|Mobiles|
### Interview Answer (2-Minute Version)

> A self join is a join where a table is joined with itself using aliases. It's required when related data exists within the same table, such as employees and their managers, categories and their parent categories, or customers and their referrers.

----
-----

# what is Partitioning ? Different types of partitioning? (Range, List, Hash) ?


### Database Partitioning (One of the Most Important Scalability Concepts)

Partitioning is a technique of **splitting one large table into multiple smaller physical pieces (partitions)** while the application still sees it as **one logical table**.

Think of it as dividing a huge book into multiple volumes.

Instead of one giant table:

```
Orders
----------------------------------
1
2
3
4
...
500 Million rows
```

Database internally stores it like:

```
Orders (Logical Table)

        Orders
           |
    -------------------
    |    |    |    |
 P1    P2   P3   P4
```

Your SQL is still

```
SELECT * FROM orders;
```

You don't write

```
SELECT * FROM orders_2022;
SELECT * FROM orders_2023;
```

Database automatically decides which partition to read.

---

### First Understand WHY Partitioning is Needed (Intuition)

Imagine Amazon.

Every order ever placed is stored in one table.

```
Orders

ID      Customer    Date
---------------------------------
1
2
3
...
2 Billion rows
```

Now suppose a customer asks

> Show my orders from 2026.

Without partitioning:

```
Database

-----------------------------------
2020
2021
2022
2023
2024
2025
2026
-----------------------------------
```

Database may have to scan a massive amount of data (unless indexes fully solve the query).

Even with indexes:

- Index becomes huge.
- Cache efficiency decreases.
- Maintenance becomes slower.
- Backups become slower.

---

### What Does Database Actually Do?

Suppose:

```
Orders Table

500 Million rows
```

Partition by Year.

Internally:

```
Orders

Partition 1
2022 rows

Partition 2
2023 rows

Partition 3
2024 rows

Partition 4
2025 rows
```

Your application still queries

```
SELECT *
FROM orders
WHERE order_date='2025-04-15';
```

Database says

```
Date = 2025

↓

Only check 2025 partition

↓

Done
```

It ignores the other partitions.

This is called **Partition Pruning**.

This is the biggest advantage of partitioning.

---

### Partition Pruning

**Partition pruning** is a performance optimization in databases and distributed query engines (like [Oracle Database], [Apache Spark], and [BigQuery] that ignores unneeded partitions of data during a query. By reading only the relevant partitions, it dramatically reduces disk I/O and speeds up processing time.

Without partitioning

```
Orders

-----------------------------------
2022
2023
2024
2025
-----------------------------------

Need 2025

↓

Scan everything
```

With partitioning

```
Orders

2022
2023
2024
2025

Need 2025

↓

Directly open only 2025 partition
```

Huge performance improvement.

---

## Types of Partitioning

Most interviewers expect these three:

1. Range Partitioning
2. List Partitioning
3. Hash Partitioning

Let's understand each.

---

### 1. Range Partitioning

Most common.

Data is divided according to a **range of values**. (dates, timestamps, or numerical IDs)

Example:

Orders table.

```
OrderID    OrderDate

1          2023-01-01
2          2023-06-20
3          2024-02-10
4          2025-01-01
```

Partition:

```
P1

2023 data

------------------

P2

2024 data

------------------

P3

2025 data
```

SQL (example)

```
PARTITION BY RANGE(order_date)
```

---

Suppose query:

```
SELECT *
FROM orders
WHERE order_date='2025-01-10';
```

Database

```
Need 2025

↓

Open only Partition 2025
```

Instead of

```
Scan 3 years
```

---

#### Best Use Cases

- Sales
- Orders
- Transactions
- Logs
- Events
- Audit tables

Basically anything based on date.

---

#### Visualization

```
Orders

2022
2022
2022
2023
2023
2024
2024
2025
2025

↓

Partition

P1
2022

P2
2023

P3
2024

P4
2025
```

---

#### Interview Intuition

"I split the data based on a range like dates, IDs, or salary ranges. When queries filter on that range, the database reads only the relevant partition instead of the entire table."

---

### 2. List Partitioning

List partitioning generally refers to splitting a collection or dataset into smaller, discrete chunks based on a specific size or criteria.

Instead of ranges,

Split based on **specific values**.

Example:

Country.

```
Customers

ID      Country

1       India

2       USA

3       UK

4       India

5       Japan
```

Partition

```
P1

India

----------------

P2

USA

----------------

P3

UK

----------------

P4

Others
```

Query

```
SELECT *
FROM customers
WHERE country='India';
```

Database

```
Open India partition only.
```

---

#### Best Use Cases

Country

Region

Department

Category

State

Branch

Status

Example:

```
Status

Pending

Completed

Cancelled
```

Each status can have its own partition.

---

#### Visualization

```
Customers

India
USA
India
Japan
UK
India

↓

India Partition

India
India
India

----------------

USA Partition

USA

----------------

UK Partition

UK
```

---

#### Interview Explanation

"List partitioning is used when data belongs to a predefined set of values, like country or department. The database places each row into the partition matching that value."

---

### 3. Hash Partitioning

Hash-based partitioning is a data distribution technique that assigns rows to specific partitions by applying a hash function to a chosen column (the partition key).

[ In a distributed database, a **hotspot** ] refers to an overworked node or a specific piece of data experiencing a disproportionately high volume of traffic compared to the rest of the system.

Suppose there is **no natural range**.

Example:

Users.

```
UserID

1
2
3
4
5
6
7
8
```

We want equal distribution.

Database computes

```
Hash(UserID)

↓

Modulo(Number of partitions)
```

Example:

```
UserID % 4
```

Result

```
1 → Partition1

2 → Partition2

3 → Partition3

4 → Partition0

5 → Partition1

6 → Partition2
```

Rows become

```
Partition1

1
5
9
13

----------------

Partition2

2
6
10
14

----------------

Partition3

3
7
11
15

----------------

Partition4

4
8
12
16
```

Notice:

No order.

Just equal distribution.

---

#### Why Hash?

Suppose:

```
100 Million users
```

You want

```
25 Million

25 Million

25 Million

25 Million
```

Hash partitioning automatically balances the data.

---

#### Best Use Cases

User table

Employee table

Session table

Large lookup tables

Whenever balanced distribution is more important than grouping by a meaningful value.

---

#### Interview Explanation

"Hash partitioning distributes rows uniformly using a hash function. It helps avoid hotspots and keeps partitions approximately equal in size."

---

## Comparison

|Feature|Range|List|Hash|
|---|---|---|---|
|Based On|Value ranges|Specific values|Hash function|
|Example|Dates|Country|User ID|
|Best For|Time-series data|Categories|Uniform distribution|
|Query Performance|Excellent for range queries|Excellent for equality queries|Good load balancing|
|Easy to Archive|✅ Yes|Sometimes|❌ Difficult|
|Balanced Data|Not always|Not always|✅ Yes|

---

### Advantages of Partitioning

#### 1. Faster Queries

Instead of

```
500 Million rows
```

Database reads

```
20 Million rows
```

because of partition pruning.

---

#### 2. Easier Maintenance

Instead of rebuilding indexes on a massive table:

```
Orders

500 Million rows
```

You rebuild only one partition.

Much faster.

---

#### 3. Faster Backups

Backup only one partition.

Example:

```
Backup only 2026 partition.
```

---

#### 4. Easy Data Archiving

Need to delete old data?

Without partitioning:

```
DELETE FROM orders
WHERE order_date<'2020-01-01';
```

Millions of deletes.

Slow.

With partitioning:

```
Drop partition 2019
```

Done in seconds.

---

#### 5. Better Parallelism

Database can process multiple partitions in parallel.

```
P1

P2

P3

P4
```

Multiple CPU cores can work simultaneously.

---

### Disadvantages

#### 1. More Complex Design

Choosing the wrong partition key can negate the benefits.

---

#### 2. Not Every Query Benefits

Suppose you partition by year:

```
2022
2023
2024
2025
```

Query

```
SELECT *
FROM orders
WHERE customer_id=100;
```

The database may need to search every partition because the filter isn't on the partition key.

---

#### 3. Poor Partition Key Causes Problems

Suppose you partition by year:

```
2022 → 10 Million

2023 → 15 Million

2024 → 20 Million

2025 → 500 Million
```

The 2025 partition becomes a hotspot while older partitions remain small.

---

#### 4. Joins Can Become More Expensive

If related rows are spread across many partitions (or different servers in distributed systems), joins may need to access multiple partitions, increasing overhead.

---

### Partitioning vs Indexing (A Common Interview Question)

|Indexing|Partitioning|
|---|---|
|Makes row lookup faster within a table|Splits a large table into smaller physical pieces|
|Improves search performance|Reduces the amount of data scanned|
|Uses structures like B-trees or hash indexes|Uses partition boundaries or hash rules|
|One table remains one storage structure|One logical table maps to multiple physical partitions|

Think of it this way:

- **Indexing** helps you **find rows faster** within a table.
- **Partitioning** helps you **avoid looking at irrelevant parts of the table** in the first place.

In practice, they are often used together. For example, a table might be partitioned by `order_date`, and each partition can have its own indexes on columns like `customer_id` or `product_id`.

---

## Interview Answer (2-Minute Version)

> **Partitioning** is the process of dividing a large table into smaller physical partitions while presenting it as a single logical table to the application. It improves scalability by allowing the database to access only the relevant partition instead of scanning the entire table, a technique known as **partition pruning**.
> 
> There are three common types:
> 
> - **Range partitioning** splits data by ranges (for example, orders by year or month). It's ideal for time-based data and archival.
> - **List partitioning** groups rows by specific values (for example, country, department, or status). It's useful when queries filter on a fixed set of categories.
> - **Hash partitioning** uses a hash function to distribute rows evenly across partitions. It's best when you need balanced data distribution and there is no natural range.
> 
> Partitioning improves query performance, maintenance, backups, and data archival, but it requires choosing the right partition key. Queries that don't filter on the partition key may not benefit because multiple partitions may still need to be scanned.

-----
----

# What is the difference between horizontal partitioning and vertical partitioning ?

### Imagine an Employee Table

```
Employee
------------------------------------------------------------
EmpID | Name | Age | Department | Salary | Address | Photo
------------------------------------------------------------
1     | John | 30  | HR         | 50000  | NY      | ...
2     | Mike | 28  | IT         | 70000  | CA      | ...
3     | Alex | 35  | Sales      | 65000  | TX      | ...
```

Now let's see two different ways to split it.

---

### Horizontal Partitioning (Split by Rows)

#### Intuition

Imagine a classroom with **1,000 students**.

One teacher cannot manage everyone.

So the school creates sections:

```
Class 10

Section A
Students 1-250

Section B
Students 251-500

Section C
Students 501-750

Section D
Students 751-1000
```

Notice:

Every section has the **same columns**

```
Roll No
Name
Age
Address
```

Only the **students (rows)** are divided.

This is Horizontal Partitioning.

---

### Database Example

Original Table

```
Employee

------------------------------------------------------------
EmpID | Name | Age | Department | Salary
------------------------------------------------------------
1
2
3
4
5
6
7
8
```

After Horizontal Partitioning

```
Partition 1

---------------------------------------------------
EmpID | Name | Age | Department | Salary
---------------------------------------------------
1
2
3
```

```
Partition 2

---------------------------------------------------
EmpID | Name | Age | Department | Salary
---------------------------------------------------
4
5
6
```

```
Partition 3

---------------------------------------------------
EmpID | Name | Age | Department | Salary
---------------------------------------------------
7
8
```

Notice:

✅ Same columns

Different rows

---

#### Real Use Case 1: Amazon Orders

Suppose Amazon has

```
5 Billion Orders
```

Instead of keeping everything together

```
Orders

5 Billion Rows
```

Amazon partitions by year.

```
Orders_2023

Orders_2024

Orders_2025
```

Every partition has

```
OrderID
CustomerID
Amount
Status
Date
```

Only rows are different.

If customer asks

```
Show my 2025 orders
```

Database directly reads

```
Orders_2025
```

instead of scanning all 5 billion orders.

---

#### Real Use Case 2: Banking

Transactions

```
20 Years

2004
2005
...
2025
```

Partition

```
Transactions_2024

Transactions_2025

Transactions_2026
```

Monthly statements only read the relevant partition.

---

#### Why Use Horizontal Partitioning?

Because the **number of rows becomes very large**.

Problems solved:

- Faster queries (partition pruning)
- Smaller indexes
- Easier backups
- Easy archival
- Better parallel processing

---

### Vertical Partitioning (Split by Columns)

Now imagine something different.

Employee table

```
------------------------------------------------------------
EmpID | Name | Age | Salary | Address | Resume | Photo
------------------------------------------------------------
```

Notice

```
Photo
Resume
```

are huge.

But HR software mostly needs

```
Name
Age
Salary
```

Loading photo every time is wasteful.

So we split the columns.

---

Original

```
Employee

------------------------------------------------------------
EmpID | Name | Age | Salary | Address | Resume | Photo
------------------------------------------------------------
```

After Vertical Partitioning

#### Table 1

```
Employee_Basic

---------------------------------------
EmpID | Name | Age | Salary
---------------------------------------
```

#### Table 2

```
Employee_Details

------------------------------------------
EmpID | Address | Resume | Photo
------------------------------------------
```

Notice

Every employee still exists.

Only columns are separated.

---

#### Real Use Case 1: Social Media

Suppose Instagram stores

```
User

ID
Name
Username
Email
Bio
Profile Picture
Cover Photo
Video Intro
```

Every login only needs

```
ID
Username
Name
```

Not

```
Video Intro
Cover Photo
```

So database becomes

```
User_BasicIDNameUsername
```

and

```
User_Basic

ID
Name
Username
```

Login becomes much faster.

---

#### Real Use Case 2: E-commerce

Product Table

```
ID
Name
Price
Description
Manual PDF
Images
Videos
```

Homepage only needs

```
Name
Price
Thumbnail
```

Large PDFs and videos aren't needed.

So split them.

```
Product

ID
Name
Price
```

```
ProductMedia

ID
Description
PDF
Images
Videos
```

Homepage loads much faster.

---

#### Why Use Vertical Partitioning?

Because the **number of columns (or their size)** becomes the problem.

Benefits:

- Smaller rows
- Less disk I/O
- Better cache utilization
- Faster queries that only need common columns

---

### Which One Do Big Companies Use?

The answer is often **both**, depending on the problem.

### Amazon

**Horizontal Partitioning**

```
Orders

2023
2024
2025
```

Billions of rows are split by date to improve query performance and make old data easier to archive.

**Vertical Partitioning**

```
Customer

ID
Name
Email
```

```
Customer_Profile

ID
Address
Preferences
Profile Photo
```

Frequently accessed customer information stays in one table, while less frequently used or larger columns are stored separately.

---

### Interview Answer (2 Minutes)

> **Horizontal partitioning** divides a table by **rows**. Each partition has the same schema, but stores a subset of the data. It's mainly used when tables become very large, such as order or transaction tables. For example, an `Orders` table can be partitioned by year so queries for 2025 only scan the 2025 partition.
> 
> **Vertical partitioning** divides a table by **columns**. Each partition contains different attributes but is linked using the same primary key. It's used when some columns are large or rarely accessed. For example, a `Product` table may keep `id`, `name`, and `price` in one table, while large fields like images, PDFs, and videos are moved to another table. This reduces I/O and speeds up common queries.
> 
> In short, **horizontal partitioning scales data volume**, while **vertical partitioning optimizes data access by separating frequently used columns from infrequently used or large columns**.


# How do you manage indexes on partitioned tables ?

This is an excellent interview question because it checks whether you understand **how partitioning and indexing work together**.

A common misconception is:

> "If I partition a table, I don't need indexes."

**Wrong.** Partitioning reduces the amount of data to scan, while indexes help find rows quickly **within each partition**.

---

### First Understand the Relationship

Suppose we have an `Orders` table with **1 billion rows**.

Without partitioning:

```
Orders (1 Billion Rows)Index on customer_id
```

The index itself is huge.

Now partition the table by year.

```
Orders

├── Partition 2023
├── Partition 2024
└── Partition 2025
```

Now the question is:

> **Where should the indexes live?**

There are two approaches.

---

## 1. Local Index (Most Common)

A **local index** means **each partition has its own separate index**.

Imagine:

```
Orders

├── 2023
│     └── Index on customer_id
│
├── 2024
│     └── Index on customer_id
│
└── 2025
      └── Index on customer_id
```

Each partition manages its own index.

---

### Example

Partition by year.

```
Orders

2023
------------------
1
2
3
```

Index

```
CustomerID

100
105
120
```

Another partition

```
2024

4
5
6
```

Its own index

```
CustomerID

101
130
150
```

Each partition has an independent B-Tree.

---

#### Query

```
SELECT *
FROM Orders
WHERE order_date = '2025-03-10'
  AND customer_id = 105;
```

Database works like this:

#### Step 1

Partition pruning

```
Need 2025

↓

Ignore 2023
Ignore 2024
```

#### Step 2

Use the index inside the **2025 partition**.

Only one small index is searched.

---

#### Advantages

- Smaller indexes
- Faster index rebuilds
- Easier maintenance
- Better partition pruning
- Dropping a partition automatically removes its index

---

### Real-world Example

Amazon partitions orders by month.

```
Orders

Jan2026
Feb2026
Mar2026
```

Each partition has

```
Index(customer_id)

Index(order_status)

Index(product_id)
```

Searching March orders only touches:

- March partition
- March indexes

Everything else is ignored.

---

## 2. Global Index

Instead of one index per partition,

Create **one index for the entire table**.

```
Orders

2023
2024
2025

↓

One Global Index
```

The index points to rows in every partition.

Example

```
CustomerID

100 → 2023

101 → 2025

102 → 2024

103 → 2023
```

One index covers everything.

---

#### Query

```
SELECT *
FROM Orders
WHERE customer_id = 105;
```

No date is specified.

Database cannot prune partitions based on `order_date`.

Instead,

Global index directly tells the database:

```
Customer 105

↓

Partition 2025

↓

Go there
```

---

### Advantages

Good when queries **don't use the partition key**.

Example:

```
SELECT *
FROM Orders
WHERE customer_id = 100;
```

No need to search every partition.

---

### Disadvantages

Suppose you drop the 2023 partition.

```
DROP PARTITION 2023;
```

Now the global index contains references to deleted rows.

It must be rebuilt or maintained.

Maintenance becomes more expensive.

---

### Local vs Global Index

|Feature|Local Index|Global Index|
|---|---|---|
|One index per partition?|✅ Yes|❌ No|
|Partition pruning friendly?|✅ Excellent|Sometimes|
|Easier maintenance?|✅ Yes|❌ No|
|Drop partition impact|Only that partition's index is removed|Global index may need rebuilding or maintenance|
|Best for|Queries using the partition key|Queries that don't use the partition key|

---

## Example Scenario

### Orders Table

Partitioned by:

```
order_date
```

#### Query 1

```
SELECT *
FROM Orders
WHERE order_date = '2025-06-01';
```

Partition pruning immediately selects the 2025 partition.

An index inside that partition makes the lookup even faster.

This is where **local indexes** shine.

---

#### Query 2

```
SELECT *
FROM Orders
WHERE customer_id = 101;
```

The partition key (`order_date`) isn't used.

Without a global index:

```
Search 2023

Search 2024

Search 2025
```

Potentially every partition is checked.

With a global index:

```
Customer 101

↓

Located in 2025 partition

↓

Fetch row
```

Much faster.

---

### Which One Do Companies Use?

For large time-series tables (orders, logs, transactions), companies typically:

- Partition by **date**
- Create **local indexes** on frequently searched columns like `customer_id`, `product_id`, or `status`

This combination provides:

- Efficient partition pruning
- Smaller indexes
- Easier maintenance
- Fast archival (dropping old partitions)

Global indexes are used more selectively because they add maintenance overhead when partitions are added, split, merged, or dropped.

---

### Best Practices

If you partition by `order_date`:

```
SELECT *
FROM Orders
WHERE order_date = '2025-06-01'
  AND customer_id = 100;
```

Use:

- Partition key: `order_date`
- Local index: `customer_id` (inside each partition)

This gives the best performance:

1. Partition pruning reduces the search to one partition.
2. The local index quickly finds the matching rows within that partition.

---

### Interview Answer (2 Minutes)

> On a partitioned table, indexes are usually managed as **local indexes** or **global indexes**.
> 
> A **local index** is partitioned the same way as the table, so each table partition has its own index. When a query filters on the partition key, the database first performs partition pruning and then searches only the relevant partition's index. Local indexes are smaller, easier to rebuild, and simplify partition maintenance.
> 
> A **global index** spans all partitions and is useful when queries don't include the partition key. For example, if an `Orders` table is partitioned by `order_date` but many queries search only by `customer_id`, a global index on `customer_id` can avoid scanning multiple partitions. However, global indexes are more expensive to maintain because operations like dropping or reorganizing partitions may require updating or rebuilding the index.
> 
> In practice, for large partitioned tables, local indexes are generally preferred unless there's a strong workload requirement for global indexes.


-----
-----

# What is sub-partitioning and when would you use it ?

This is a **good follow-up interview question** because it tests whether you know that partitioning can happen at **multiple levels**.

---

### What is Sub-Partitioning?

**Sub-partitioning** means **partitioning a partition again**.

Think of it like organizing a library:

- **First level:** Books are arranged by **year**.
- **Second level:** Within each year's section, books are arranged by **genre**.

You first divide into large groups, then further divide each group into smaller groups.

---

### Database Example

Suppose Amazon has an `Orders` table.

Without partitioning:

```
Orders

1 Billion Rows
```

Partition by **Year**.

```
Orders

2024
2025
2026
```

Now suppose the 2025 partition alone has **300 million rows**.

It's still very large.

So we partition **inside** the 2025 partition.

For example, by **Region**.

```
Orders

2025
│
├── North
├── South
├── East
└── West
```

This is **sub-partitioning**.

The complete structure becomes:

```
Orders

├── 2024
│     ├── North
│     ├── South
│     ├── East
│     └── West
│
├── 2025
│     ├── North
│     ├── South
│     ├── East
│     └── West
│
└── 2026
      ├── North
      ├── South
      ├── East
      └── West
```

---

### How Does It Work?

Suppose the table is:

```
OrderID   OrderDate   Region
--------------------------------
1         2025-01-10  North
2         2025-02-15  South
3         2024-12-01  East
4         2025-03-20  West
```

Primary partition:

```
Range Partition

2024

2025

2026
```

Inside 2025:

```
List Partition

North

South

East

West
```

So row

```
2025
North
```

goes to

```
2025 Partition

↓

North Sub-partition
```

---

### Example Query

```
SELECT *
FROM Orders
WHERE order_date = '2025-05-10'
  AND region = 'North';
```

Database processing:

#### Step 1

Partition pruning

```
Need Year = 2025

↓

Open only 2025 partition
```

#### Step 2

Sub-partition pruning

```
Need Region = North

↓

Open only North sub-partition
```

Instead of searching:

```
2024

2025

2026

All Regions
```

It searches only:

```
2025

↓

North
```

This further reduces the amount of data read.

---

### Common Types of Sub-Partitioning

Suppose your main partition is **Range**.

Inside each range you can use:

#### Range → Hash

```
Orders

2024
   Hash

2025
   Hash

2026
   Hash
```

Useful when a year's data is still too large and you want an even distribution.

---

#### Range → List

```
Orders

2024
   North
   South

2025
   North
   South
```

Useful when queries often filter by both year and region.

---

#### Range → Range

```
2025

Jan-Mar

Apr-Jun

Jul-Sep

Oct-Dec
```

A year is first partitioned, then divided into quarters or months.

---

### Real-World Use Cases

#### 1. E-commerce (Amazon)

Orders table

Primary partition:

```
Year
```

Sub-partition:

```
Warehouse
```

```
2025

Warehouse Delhi

Warehouse Mumbai

Warehouse Bengaluru
```

Query:

```
SELECT *
FROM Orders
WHERE order_date BETWEEN '2025-01-01' AND '2025-01-31'
  AND warehouse = 'Delhi';
```

The database only scans:

```
2025

↓

Delhi warehouse
```

---

#### 2. Banking

Transactions

Primary partition

```
Year
```

Sub-partition

```
Branch
```

```
2025

Delhi

Mumbai

Pune

Lucknow
```

Monthly reports for a specific branch become much faster.

---

### Difference Between Partitioning and Sub-Partitioning

| Partitioning              | Sub-Partitioning              |
| ------------------------- | ----------------------------- |
| Splits the original table | Splits each partition further |
| One level                 | Two or more levels            |
| Example: By year          | Inside each year, by region   |
| Reduces large tables      | Reduces large partitions      |

---

### When Should You Use It?

Use sub-partitioning when:

- A single partition is still too large to manage efficiently.
- Queries commonly filter on **two dimensions**, such as:
    - `year` and `region`
    - `month` and `customer_type`
    - `date` and `warehouse`
- You want more granular maintenance or parallelism.

Avoid it when:

- The table isn't large enough to justify the added complexity.
- Queries rarely use the second partitioning key.
- It creates too many tiny partitions, which can increase metadata and maintenance overhead.

---

### Interview Answer (2 Minutes)

> **Sub-partitioning** (also known as composite partitioning) is a database technique that further divides individual partitions of a large table into smaller, more granular sub-tables. It enables faster data retrieval, optimized storage management, and improved query parallelization for massive datasets. 
>  For example, an `Orders` table can be range-partitioned by `order_date` (year), and then each yearly partition can be list-partitioned by `region`.
> 
> This is useful when a single partition is still very large or when queries frequently filter on both the primary partition key and another column.


-----
-----

# Composite keys?Primary keys? Candidate Keys? Surrogate Keys? Is it possible to have a table without a primary key in a relational database? If yes, what are the implications for data integrity and query performance? How would you uniquely identify rows in such a table?


