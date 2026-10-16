
### 1. Second Highest Salary

https://leetcode.com/problems/second-highest-salary/

+-------------+------+
| Column Name | Type |
+-------------+------+
| id                    | int       |
| salary             | int       |
+-------------+------+
id is the primary key (column with unique values) for this table.
Each row of this table contains information about the salary of an employee.

Write a solution to find the second highest **distinct** salary from the `Employee` table. If there is no second highest salary, return `null (return None in Pandas)`.

The result format is in the following example.

```
	

Input:
Employee table:
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+

Output:
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+



Input: -
Employee table:
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
+----+--------+

Output: 
+---------------------+
| SecondHighestSalary |
+---------------------+
| null                |
+---------------------+


Input: -
| id | salary |
| -- | ------ |
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |

Output: -
| secondhighestsalary |
| ------------------- |
| 200                 |

```


```sql

select (
    select distinct salary
    from (
        select *,
        dense_rank() over(order by salary desc) as rank
        from employee
    ) t
    where rank = 2
) as SecondHighestSalary;

```

```sql


```


### 2. Find Employee having Nth Highest Salary

```sql

	SELECT id, name, salary
	FROM (
	    SELECT *,
	           DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
	    FROM Employee
	) e
	WHERE salary_rank = 3;
	
```

### 3.  Employees earning more than average salary of their department and rank them


```sql
	
SELECT
    emp_id,
    name,
    dept,
    salary,
    avg_salary,
    salary_rank
FROM (
    SELECT
        emp_id,
        name,
        dept,
        salary,
        AVG(salary) OVER(PARTITION BY dept) AS avg_salary,
        DENSE_RANK() OVER(
            PARTITION BY dept
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employee
) t
WHERE salary > avg_salary;

```


### 4. Top 3 Highest Paid Employees in each Department

```sql

Step 1: Create Table -

CREATE TABLE employee (
    emp_id INT PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(30),
    salary INT
);
```

---


```sql

Step 2: Insert Sample Data -

INSERT INTO employee (emp_id, name, department, salary) VALUES
(1, 'Alice',   'IT',      12000),
(2, 'Bob',     'IT',      10000),
(3, 'Charlie', 'IT',      10000),
(4, 'David',   'IT',       8000),
(5, 'Eva',     'IT',       7000),

(6, 'Frank',   'HR',      11000),
(7, 'Grace',   'HR',       9000),
(8, 'Henry',   'HR',       9000),
(9, 'Ivy',     'HR',       7000),
(10,'Jack',    'HR',       6000),

(11,'Kevin',   'Finance', 15000),
(12,'Laura',   'Finance', 13000),
(13,'Mike',    'Finance', 13000),
(14,'Nancy',   'Finance', 11000),
(15,'Oscar',   'Finance', 10000);
```

---

**table** -

|emp_id|name|department|salary|
|--:|---|---|--:|
|15|Kevin|Finance|15000|
|12|Laura|Finance|13000|
|13|Mike|Finance|13000|
|14|Nancy|Finance|11000|
|15|Oscar|Finance|10000|
|6|Frank|HR|11000|
|7|Grace|HR|9000|
|8|Henry|HR|9000|
|9|Ivy|HR|7000|
|10|Jack|HR|6000|
|1|Alice|IT|12000|
|2|Bob|IT|10000|
|3|Charlie|IT|10000|
|4|David|IT|8000|
|5|Eva|IT|7000|

---

Output -

```sql
SELECT
    emp_id,
    name,
    department,
    salary
FROM (
    SELECT
        emp_id,
        name,
        department,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employee
) t
WHERE salary_rank <= 3
```


### 5. Employees earning more than average salary of their department

```sql

SELECT
    emp_id,
    name,
    dept,
    salary
FROM (
    SELECT *,
           AVG(salary) OVER(PARTITION BY dept) AS avg_salary
    FROM employee
) t
WHERE salary > avg_salary;

```


### 6. 

