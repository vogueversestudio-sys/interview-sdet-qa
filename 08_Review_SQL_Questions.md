# SQL Interview Questions

---

## Q1. WHERE vs HAVING clause?


The main difference is when the filtering happens.

    **WHERE** filters rows before grouping or aggregation.
    **HAVING** filters the aggregated result after the GROUP BY.
    
    **For example** if I want all employees from the QA department, I use WHERE Department = 'QA'.

    If I want departments having more than 10 employees, I use:
    
    SELECT Department, COUNT(*)
    FROM Employee
    GROUP BY Department
    HAVING COUNT(*) > 10;

**As an SDET,** I use WHERE  while validating API/database data, And HAVING is useful when validating reporting or analytics data.


---

## Q2. SQL Joins?


SQL Joins are used to combine data from two or more tables based on a common column or related key. They help retrieve meaningful information that is spread across multiple tables. There are several types of joins:

    **INNER** - Returns only the records that have matching values in both tables.
                Example: If you join an Employees table with a Departments table, it returns only employees who are assigned to a department.
    
    **LEFT** - Returns all records from the left table and the matching records from the right table. If there is no match, the right-side columns                   contain NULL
    
    **RIGHT** - Returns all records from the right table and the matching records from the left table. If there is no match, the left-side columns                    contain NULL
    
    **FULL** - Returns all records from both tables. Where there is no match, the missing side is filled with NULL.
    
    **CROSS** - Returns every possible combination of rows from the two tables, meaning every row from the first table is combined with every row                     from the second table.
    
    **SELF** - A table is joined with itself. It is commonly used to represent hierarchical relationships, such as finding an employee's manager from                the same Employees table.

In real-world projects, I most commonly use INNER JOIN to retrieve matching records and 
LEFT JOIN when I need all records from one table even if related data is missing. 
Self joins are useful for hierarchical data, 
while Cross and Full Joins are used in specific scenarios depending on the business requirement.


---

## Q3. What are Aggregate Functions?

Aggregate functions are SQL functions that perform calculations on a group of rows and return a single summarized value.
They are mainly used for reporting, data analysis, and validation. 
Aggregate functions can be used with or without the **GROUP BY** clause. 
When used with GROUP BY, they return one result for each group.


**In automation testing**, I use aggregate functions to validate data between the UI, APIs, and the database. For example, I use COUNT() to verify the number of records, SUM() to validate totals in reports, and AVG(), MAX(), or MIN() to ensure calculations are accurate.

**The most commonly used aggregate functions are:**

    COUNT() – Counts the number of rows.
    SUM() – Returns the total of a numeric column.
    AVG() – Calculates the average value.
    MAX() – Returns the highest value.
    MIN() – Returns the lowest value.

Q - Do aggregate functions always require GROUP BY? - 
    No. Aggregate functions can be used without GROUP BY, in which case they return a single result for the entire table. GROUP BY is only needed         when you want aggregate results for each group.

---

## Q4. What is GROUP BY?

GROUP BY is used to group rows that have the same values in one or more columns. It is commonly used with aggregate functions like COUNT(), SUM(), AVG(), MAX(), and MIN() to calculate summary results for each group


**In automation testing,** I frequently use GROUP BY for database validation.
For example, I compare the number of records displayed in the UI or returned by an API with the grouped results from the database. 
It's also useful for validating reports, dashboards, and business summaries

Q- Can we use GROUP BY without an aggregate function?- 
Yes, but it's generally equivalent to using SELECT DISTINCT on those columns. GROUP BY is most useful when it's combined with aggregate functions to summarize data.

---

## Q5. What is ORDER BY?

**Simple Answer:**
**ORDER BY** is used to sort the result set of a query based on one or more columns. 
By default, SQL sorts the data in ascending order **(ASC)**. To sort in descending order, we use the **DESC** keyword.

In automation testing, I use ORDER BY to retrieve data in a predictable order before comparing it with API responses or UI results. 
For example, if a report should display the highest-paid employees first, I validate it using **ORDER BY** salary **DESC**

Q: What is the execution order of GROUP BY and ORDER BY?- 
**GROUP BY** groups the data first, and then **ORDER BY** sorts the final result set.
That's why we can sort aggregated values, such as AVG(salary) or COUNT(*), after grouping

---

## Q6. How to handle NULL values?

**Simple Answer:**
NULL means "no value" or "unknown" — it is NOT zero, NOT empty string. You cannot check for NULL using `= NULL`. You must use `IS NULL` or `IS NOT NULL`.

-- IMPORTANT: NULL in aggregations is automatically ignored

**⚡ Key Points:**
- NULL ≠ 0 and NULL ≠ empty string. NULL means "no value"
- Always use `IS NULL` not `= NULL`

---

## Q7. How to remove duplicates?

There are two common ways to handle duplicates in SQL:

Use DISTINCT to remove duplicate values from the query result.
Use ROW_NUMBER() with a CTE to identify and permanently delete duplicate rows from a table. This is the preferred approach in real-world projects.

If I only need unique values in the output, I use DISTINCT. 
If duplicate records actually exist in the table, I use ROW_NUMBER() with a Common Table Expression (CTE) to identify and safely remove the duplicates while keeping one original record

---

## Q8. CASE Statement?

The CASE statement is used to implement conditional logic in SQL. It works like an if-else statement in programming languages. Based on one or more conditions, it returns different values. It is commonly used in SELECT, UPDATE, ORDER BY, and sometimes WHERE clauses.

In automation testing, I use CASE statements to validate business rules directly in SQL. 
For example, I categorize records, calculate expected values based on conditions, or verify that data falls into the correct business category. 
It helps reduce validation logic in the test code

---

## Q9. What is CTE (Common Table Expression)?

**Simple Answer:**
A CTE is a temporary, named result set that you define at the top of your query using the `WITH` keyword. It exists only for the duration of a single SQL statement and makes complex queries easier to read, write, and maintain. A CTE can also be referenced multiple times within the same query.

In automation testing, I use CTEs to simplify complex validation queries. 
Instead of writing deeply nested subqueries, I create CTEs to organize the logic, making the SQL easier to understand and maintain. 
This is especially useful when validating reports, dashboards, or comparing intermediate datasets

---

## Q10. RANK() vs DENSE_RANK() vs ROW_NUMBER()?

All three are SQL window functions used to assign sequential numbers to rows based on a specific order. The main difference is how they handle duplicate values or ties.

Salary: 100000, 100000, 90000, 80000

 **ROW_NUMBER()** - It assigns a unique number to every row, even if two rows have the same value. There are no duplicate rankings
                    ROW_NUMBER:1, 2, 3, 4 

 **RANK()** - It gives the same rank to rows with the same value, but it skips the next rank after a tie.
               RANK: 1, 1, 3, 4

**DENSE_RANK()** - It also gives the same rank for duplicate values, but it does not skip the next rank.
                  DENSE_RANK:1, 1, 2, 3

  
---

## Q11. DELETE vs TRUNCATE vs DROP?

**Simple Answer:**
- `DELETE` = remove specific rows (can undo it)
- `TRUNCATE` = remove ALL rows quickly (usually cannot undo)
- `DROP` = destroy the entire table including its structure (cannot undo)

**Detailed Comparison:**

| Aspect | DELETE | TRUNCATE | DROP |
|--------|--------|----------|------|
| What is removed | Specific rows (or all rows with no WHERE) | All rows | Entire table (structure + data) |
| WHERE clause | ✅ Yes | ❌ No | ❌ No |
| Can rollback? | ✅ Yes (it's DML) | ❌ Usually No | ❌ No |
| Speed | Slower (logs each row) | Faster (deallocates pages) | Fastest |
| Fires triggers? | ✅ Yes | ❌ No | ❌ N/A |
| Resets auto-increment? | ❌ No | ✅ Yes | ❌ N/A |
| Table still exists after? | ✅ Yes | ✅ Yes | ❌ No |
| SQL type | DML | DDL | DDL |

```sql
DELETE FROM employees WHERE dept_id = 5;   -- remove only QA department rows
TRUNCATE TABLE employees;                   -- wipe the whole table, reset IDs
DROP TABLE employees;                       -- the table is gone completely
```

**💬 How to say it in an interview:**
> "In testing, I use DELETE in my test teardown — to clean up the specific test data I inserted during a test. For example, after creating a test user, I DELETE that specific record by ID. I avoid TRUNCATE in test environments because it removes everything and resets auto-increment. DROP is only used in setup scripts to recreate tables. The key thing interviewers want to hear is: DELETE can be rolled back in a transaction, but TRUNCATE usually cannot."

**⚡ Key Points:**
- DELETE = surgical, row-by-row, can roll back, can use WHERE
- TRUNCATE = wipe all rows, reset IDs, faster than DELETE, usually cannot roll back
- DROP = kills the whole table, gone forever, can't roll back

---

## Q12. LIKE Operator and Wildcards?

**Simple Answer:**
LIKE is used for partial text matching. The `%` symbol means "any number of characters" and `_` means "exactly one character."

```sql
SELECT * FROM employees WHERE name LIKE 'V%';         -- starts with V (e.g., Vikrant)
SELECT * FROM employees WHERE name LIKE '%kumar';      -- ends with kumar
SELECT * FROM employees WHERE name LIKE '%vik%';       -- contains "vik" anywhere
SELECT * FROM employees WHERE name LIKE '_i%';         -- second character is 'i'
SELECT * FROM employees WHERE email LIKE '%@gmail%';   -- gmail users
SELECT * FROM employees WHERE name LIKE '___';         -- exactly 3 characters (3 underscores)
```

**⚡ Key Points:**
- `%` = zero or more characters (any text)
- `_` = exactly one character
- `LIKE '%abc'` (leading wildcard) cannot use an index — slow on large tables
- Use `ILIKE` for case-insensitive matching in PostgreSQL

---

## Q13. Find Nth Highest Salary?

**Simple Answer:**
This is the most common SQL interview question for SDETs. Use `DENSE_RANK()` — it is the cleanest and handles duplicate salaries correctly.

```sql
-- BEST METHOD: DENSE_RANK (handles ties, clean, works everywhere)
WITH ranked_salaries AS (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
)
SELECT DISTINCT salary
FROM ranked_salaries
WHERE rnk = 3;   -- change 3 to find any Nth highest

-- METHOD 2: LIMIT/OFFSET (MySQL — quick but doesn't handle ties)
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 2;   -- OFFSET 0=1st, 1=2nd, 2=3rd

-- METHOD 3: Subquery (works but less readable)
SELECT MAX(salary) FROM employees
WHERE salary < (
    SELECT MAX(salary) FROM employees   -- this gives 2nd highest
);

-- FOLLOW-UP: Nth highest salary PER DEPARTMENT (very common follow-up!)
SELECT name, department, salary FROM (
    SELECT name, department, salary,
           DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk = 2;   -- 2nd highest in each department
```

**💬 How to say it in an interview:**
> "For the Nth highest salary, I always use DENSE_RANK() in a CTE. Here's why: if two employees both earn 100,000, they both have rank 1. With DENSE_RANK, the next salary gets rank 2 — which correctly represents the 2nd highest unique salary. RANK() would skip rank 2 and give rank 3, which is wrong. I also know the department-wise Nth highest variation — that uses PARTITION BY department inside DENSE_RANK."

---

## Q14. Stored Procedure?

**Simple Answer:**
A stored procedure is a saved SQL program in the database that you can call by name. It's like a reusable function for SQL. You write it once and call it many times with different inputs.

```sql
-- Create a stored procedure that takes department name as input
DELIMITER //
CREATE PROCEDURE GetEmployeesByDept(IN dept_name VARCHAR(50))
BEGIN
    SELECT * FROM employees
    WHERE department = dept_name
    ORDER BY salary DESC;
END //
DELIMITER ;

-- Call it — like calling a function
CALL GetEmployeesByDept('Engineering');
CALL GetEmployeesByDept('QA');

-- Stored procedure with OUTPUT parameters
DELIMITER //
CREATE PROCEDURE GetDeptStats(
    IN  dept_name  VARCHAR(50),
    OUT emp_count  INT,
    OUT avg_salary DECIMAL(10,2)
)
BEGIN
    SELECT COUNT(*), AVG(salary)
    INTO emp_count, avg_salary
    FROM employees
    WHERE department = dept_name;
END //
DELIMITER ;
```

**Stored Procedure vs Function:**
| Aspect | Stored Procedure | Function |
|--------|-----------------|----------|
| Returns | 0 or more values via OUT parameters | Exactly 1 value (RETURN) |
| How to call | `CALL procedure_name()` | `SELECT function_name()` |
| Can modify data (INSERT/UPDATE/DELETE)? | ✅ Yes | ❌ Usually No |
| Can use inside SELECT? | ❌ No | ✅ Yes |

**💬 How to say it in an interview:**
> "In my API testing projects, I sometimes call stored procedures directly via JDBC or pyodbc to set up test data or verify backend state. For example, at Aflac, there was a stored procedure that calculated insurance premiums. Instead of going through the UI, I called it directly in my test to verify the calculation logic independently. This made my tests much faster and more focused."

---

## Q15. How to Optimise a SQL Query?

**Simple Answer:**
A slow SQL query usually means: too much data being scanned, missing indexes, or inefficient logic. These are the top fixes that will impress interviewers.

**Top 10 Optimisation Techniques:**

1. **Use Indexes** — add indexes on columns used in WHERE, JOIN ON, ORDER BY, GROUP BY
2. **SELECT only what you need** — never use `SELECT *` in production code
3. **Use EXPLAIN / EXPLAIN ANALYZE** — see how the database is executing your query
4. **Avoid functions on indexed columns** — `WHERE YEAR(created_at) = 2026` kills the index
5. **Use JOINs instead of correlated subqueries** — JOINs are almost always faster
6. **Avoid LIKE with a leading wildcard** — `LIKE '%abc'` cannot use an index
7. **Use EXISTS instead of IN** — faster for large subquery results
8. **Use LIMIT** — never fetch 10,000 rows if you only need 10
9. **Avoid DISTINCT if not needed** — it adds hidden sorting overhead
10. **Partition large tables** — split by date range or region for huge datasets

```sql
-- BAD: function on indexed column — cannot use the index
SELECT * FROM orders WHERE YEAR(order_date) = 2026;

-- GOOD: range query — uses the index on order_date
SELECT order_id, amount FROM orders
WHERE order_date >= '2026-01-01'
  AND order_date < '2027-01-01';

-- Check query plan
EXPLAIN SELECT * FROM employees WHERE department = 'QA';
```

**💬 How to say it in an interview:**
> "The first thing I do with a slow query is run EXPLAIN to see if it's doing a full table scan instead of using an index. At Office Depot, we had a product search query that was timing out. I ran EXPLAIN and saw it was scanning 2 million rows. I added a composite index on the search columns and the query went from 8 seconds to 200 milliseconds. The key rule I always follow: never apply a function to an indexed column in WHERE, because it makes the index useless."

---

## Q16. Indexing?

**Simple Answer:**
An index is like a book's index — instead of reading every page, you jump directly to the right page. It makes SELECT queries much faster, but slightly slows down INSERT/UPDATE/DELETE because the index also needs updating.

**Types of Indexes:**

| Type | Description | When to Use |
|------|-------------|------------|
| **Single-Column** | Index on one column | Frequently filtered/searched column |
| **Composite** | Index on multiple columns (order matters!) | Multiple columns always used together in WHERE |
| **Unique** | Enforces uniqueness + speeds up lookup | Email, username, employee ID |
| **Clustered** | Physically orders the table rows | Primary key (auto-created) |
| **Non-Clustered** | Separate structure with pointers to rows | Additional lookup columns |

```sql
CREATE INDEX idx_emp_name ON employees(name);
CREATE UNIQUE INDEX idx_emp_email ON employees(email);
CREATE INDEX idx_dept_salary ON employees(department, salary);  -- composite

SHOW INDEX FROM employees;         -- view all indexes on a table
DROP INDEX idx_emp_name ON employees;
```

**When NOT to create an index:**
- Very small tables (full scan is faster)
- Columns with many NULL values
- Columns that are rarely used in WHERE or JOIN
- Tables with very heavy INSERT/UPDATE/DELETE load (indexes slow down writes)

**💬 How to say it in an interview:**
> "I understand indexes from both a developer and tester perspective. When I write database validation queries in my tests, I make sure I'm not accidentally causing full table scans on production-size tables. I've added indexes to test databases to keep my validation queries fast. The key interview point is: indexes speed up reads but slow down writes — so you don't add an index on every column, only the ones that are frequently used in WHERE, JOIN, and ORDER BY."

---

## Q17. UNION vs UNION ALL?

**Simple Answer:**
Both combine results from two SELECT queries. `UNION` removes duplicates (slower). `UNION ALL` keeps everything including duplicates (faster).

| Aspect | UNION | UNION ALL |
|--------|-------|-----------|
| Duplicates | ❌ Removes duplicates (extra sort step) | ✅ Keeps all duplicates |
| Performance | Slower | **Faster** |
| Use when | You need unique results | You know there are no duplicates, or you want all rows |

```sql
-- UNION: removes duplicates — shows each name only once
SELECT name FROM employees_delhi
UNION
SELECT name FROM employees_pune;

-- UNION ALL: keeps duplicates — faster, no sorting
SELECT name FROM employees_delhi
UNION ALL
SELECT name FROM employees_pune;
```

**Important Rule:** Both SELECT statements must have the SAME number of columns with compatible data types.

**💬 How to say it in an interview:**
> "The rule is simple: UNION removes duplicates but is slower because it sorts the data. UNION ALL keeps everything and is faster. I use UNION ALL when I know the data sets don't overlap — like combining test results from two different test runs. I use UNION when I specifically need unique values, like getting a list of all unique email addresses from multiple tables."

---

## Q18. Subquery vs JOIN?

**Simple Answer:**
Both can get the same results, but JOINs are usually faster because the database can optimise them better. Subqueries are sometimes easier to read for simple cases. Use JOINs for large datasets and complex joins.

```sql
-- Subquery: find employees in the Engineering department
SELECT * FROM employees
WHERE dept_id IN (SELECT id FROM departments WHERE name = 'Engineering');

-- Same result with JOIN (usually faster on large tables)
SELECT e.* FROM employees e
JOIN departments d ON e.dept_id = d.id
WHERE d.name = 'Engineering';

-- Correlated subquery: re-executes for every row (SLOW — avoid on large tables)
SELECT name, salary FROM employees e1
WHERE salary > (
    SELECT AVG(salary) FROM employees e2
    WHERE e2.department = e1.department   -- references outer query
);

-- Better: use CTE or JOIN instead of correlated subquery
WITH dept_avg AS (
    SELECT department, AVG(salary) AS avg_sal FROM employees GROUP BY department
)
SELECT e.name, e.salary
FROM employees e
JOIN dept_avg d ON e.department = d.department
WHERE e.salary > d.avg_sal;
```

**💬 How to say it in an interview:**
> "I prefer JOINs over subqueries for performance, but I use subqueries when the logic is simpler to read. The one I always avoid is a correlated subquery — where the inner query references the outer query — because it executes once for every row in the outer query, which can be extremely slow on large tables. I replace those with CTEs or JOINs."

---

## Q19. Find Names Starting with a Vowel?

**Simple Answer:**
Use `REGEXP` if your database supports it, or use `LIKE` with multiple conditions. The cleanest modern approach is REGEXP.

```sql
-- Method 1: REGEXP (MySQL/PostgreSQL) — cleanest
SELECT name FROM employees
WHERE name REGEXP '^[AEIOUaeiou]';

-- Method 2: SUBSTRING — works everywhere
SELECT name FROM employees
WHERE LOWER(SUBSTRING(name, 1, 1)) IN ('a', 'e', 'i', 'o', 'u');

-- Method 3: Multiple LIKE — verbose but universally supported
SELECT name FROM employees
WHERE name LIKE 'A%' OR name LIKE 'E%' OR name LIKE 'I%'
   OR name LIKE 'O%' OR name LIKE 'U%';

-- Bonus: Names ending with a vowel
SELECT name FROM employees
WHERE LOWER(RIGHT(name, 1)) IN ('a', 'e', 'i', 'o', 'u');

-- Bonus: Names starting AND ending with a vowel
SELECT name FROM employees
WHERE LOWER(LEFT(name, 1))  IN ('a','e','i','o','u')
  AND LOWER(RIGHT(name, 1)) IN ('a','e','i','o','u');
```

---

## Q20. Find Duplicate Records?

**Simple Answer:**
Use GROUP BY + HAVING COUNT(*) > 1 to find which values are duplicated. To see the full duplicate rows with all their data, combine it with a subquery or JOIN.

```sql
-- Step 1: Which emails appear more than once?
SELECT email, COUNT(*) AS duplicate_count
FROM employees
GROUP BY email
HAVING COUNT(*) > 1;

-- Step 2: Show all the actual duplicate rows (with all columns)
SELECT * FROM employees
WHERE email IN (
    SELECT email FROM employees GROUP BY email HAVING COUNT(*) > 1
)
ORDER BY email;   -- group them together so duplicates appear side by side
```

**💬 How to say it in an interview:**
> "Finding duplicates is something I do regularly in database validation. After running a test that inserts data, I query for duplicates to make sure my test didn't accidentally create double records. GROUP BY + HAVING COUNT > 1 tells me which values are duplicated. The IN subquery then shows me all the actual rows so I can see the full details and decide which one to keep."

---

## Q21. Primary Key vs Foreign Key vs Unique Key?

**Simple Answer:**
- **Primary Key** = the unique identifier for each row — cannot be NULL, must be unique
- **Foreign Key** = a column that points to the Primary Key of another table — it links two tables together
- **Unique Key** = like Primary Key but allows one NULL value, and a table can have multiple Unique Keys

| Aspect | Primary Key | Foreign Key | Unique Key |
|--------|-------------|-------------|------------|
| Purpose | Uniquely identifies each row | Links two tables together | Ensures no duplicate values |
| NULL allowed? | ❌ Never | ✅ Yes | ✅ Yes (one NULL) |
| How many per table? | Only **1** | Multiple | Multiple |
| Duplicate values? | ❌ Not allowed | ✅ Allowed | ❌ Not allowed |
| Auto-creates index? | ✅ Yes (clustered) | Recommended | ✅ Yes (non-clustered) |

**💬 How to say it in an interview:**
> "Primary Key, Foreign Key, and Unique Key are the three main constraints. In my database validation, I often check referential integrity — meaning, if my test creates an order, the order's customer_id (Foreign Key) must point to a real customer_id in the customers table (Primary Key). If a test inserts an order with an invalid customer_id, the database should reject it. I verify this in negative test scenarios."

---

## Q22. Find Employees With No Department?

**Simple Answer:**
Use a LEFT JOIN between employees and departments, then filter where the department's ID is NULL. This finds employees who exist in the employees table but have no matching record in the departments table.

```sql
-- Find employees who don't belong to any department
SELECT e.*
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id
WHERE d.id IS NULL;    -- NULL means no match was found in departments table
```

**💬 How to say it in an interview:**
> "This is a classic 'find orphaned records' query. I use LEFT JOIN and then filter WHERE the joined table's column IS NULL. In testing, this is very useful — for example, finding orders that have no associated customer (data integrity violation), or finding test records that weren't cleaned up properly. It's one of the most practical SQL patterns in real testing work."

---

## Q23. Get Current Date and Time?

**Simple Answer:**
Different databases use different functions, but the concept is the same — get the current timestamp. In testing, you use this to validate that a record was created at the correct time.

```sql
SELECT NOW();                -- MySQL: returns 2026-07-21 11:22:00
SELECT CURRENT_DATE;         -- SQL standard: returns 2026-07-21 (date only)
SELECT CURRENT_TIMESTAMP;    -- SQL standard: returns date + time
SELECT GETDATE();            -- SQL Server
SELECT SYSDATE FROM DUAL;    -- Oracle
SELECT NOW()::DATE;          -- PostgreSQL: extract just the date part
```

**In Testing — How I Use This:**
```sql
-- Verify a record was created in the last 5 minutes (just now by my test)
SELECT * FROM orders
WHERE created_at >= NOW() - INTERVAL 5 MINUTE
ORDER BY created_at DESC;
```

---

## Q24. What is a View?

**Simple Answer:**
A View is a saved SQL query that you can use like a table. It doesn't store actual data — every time you query it, it runs the underlying SQL and gives you fresh results. Think of it as a shortcut to a complex query.

```sql
-- Create a view that shows only active employees and hides sensitive columns
CREATE VIEW active_employee_summary AS
SELECT id, name, department, job_title
FROM employees
WHERE status = 'Active';

-- Now use it like a normal table
SELECT * FROM active_employee_summary WHERE department = 'QA';
SELECT department, COUNT(*) FROM active_employee_summary GROUP BY department;

-- Drop a view
DROP VIEW active_employee_summary;
```

**Why Views are useful:**
- **Security:** Hide sensitive columns (like salary or SSN) from certain users
- **Simplicity:** Wrap a complex 5-table JOIN into one simple view name
- **Consistency:** All teams query the same view — no one writes the same complex query differently

**💬 How to say it in an interview:**
> "Views are useful for test data validation. At Aflac, I created a view that joined 4 tables — policy, customer, claims, and payments — into one clean view. All my test validation queries just selected from this view instead of writing the same complex JOIN every time. It made my SQL tests much shorter and easier to maintain."

---

## Q25. What is a Trigger?

**Simple Answer:**
A Trigger is an automatic action that runs when something happens to a table — like when a row is inserted, updated, or deleted. You don't call it manually; the database fires it automatically.

```sql
-- Create a trigger that automatically logs every salary change
CREATE TRIGGER log_salary_change
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    IF OLD.salary != NEW.salary THEN
        INSERT INTO salary_audit_log (
            emp_id, old_salary, new_salary, changed_by, changed_at
        )
        VALUES (
            NEW.id, OLD.salary, NEW.salary, CURRENT_USER(), NOW()
        );
    END IF;
END;
```

**Types of Triggers:**
- **BEFORE trigger** — runs before the INSERT/UPDATE/DELETE. Used to validate or modify data before it's saved.
- **AFTER trigger** — runs after the INSERT/UPDATE/DELETE. Used for audit logging, sending notifications.

**💬 How to say it in an interview:**
> "Triggers are important to understand from a testing perspective because they run automatically and can affect your test data in unexpected ways. At Aflac, there was a trigger on the payments table that automatically updated the policy status when a payment was inserted. When my test inserted a payment record, the trigger fired and changed the policy status — which then caused my policy status assertion to pass, even though I hadn't explicitly tested that flow. Understanding triggers helped me design better, more complete test scenarios."

**⚡ Key Points:**
- Trigger = automatic action on INSERT, UPDATE, or DELETE
- BEFORE = validate/modify before saving; AFTER = log/notify after saving
- As a tester, know that triggers exist — they can affect test data without you explicitly calling anything
