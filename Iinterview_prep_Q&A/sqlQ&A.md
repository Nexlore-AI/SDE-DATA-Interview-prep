==============================
FILE: SQL
==============================

### HIGH PRIORITY

---

Q1. What is the difference between WHERE and HAVING clauses?

A1.
**WHERE** filters rows before grouping — it operates on individual rows from the table. **HAVING** filters groups after GROUP BY — it operates on aggregated results.

```sql
-- WHERE: filter rows before aggregation
SELECT department, COUNT(*) FROM employees
WHERE salary > 50000
GROUP BY department;

-- HAVING: filter after aggregation
SELECT department, COUNT(*) FROM employees
GROUP BY department
HAVING COUNT(*) > 10;
```

You can use both together: WHERE first filters individual rows, then GROUP BY creates groups, then HAVING filters the groups. A common mistake is trying to use WHERE with aggregate functions — `WHERE COUNT(*) > 5` won't work. That's what HAVING is for.

---

Q2. What are window functions? Explain ROW_NUMBER, RANK, and DENSE_RANK with examples.

A2.
Window functions perform calculations across a set of rows related to the current row — without collapsing rows like GROUP BY does. You keep all your rows and add computed columns.

```sql
SELECT name, department, salary,
  ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as rn,
  RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rnk,
  DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as drnk
FROM employees;
```

Suppose salaries in Engineering are: 150K, 120K, 120K, 100K.

- **ROW_NUMBER**: 1, 2, 3, 4 — always unique, tie-breaking is arbitrary.
- **RANK**: 1, 2, 2, 4 — ties get the same rank, next rank skips (no rank 3).
- **DENSE_RANK**: 1, 2, 2, 3 — ties get the same rank, no gaps.

Use ROW_NUMBER for pagination or deduplication (pick one row per group). Use RANK/DENSE_RANK when you need to handle ties meaningfully — like leaderboards.

---

Q3. How does GROUP BY work? Can you select non-aggregated columns that aren't in GROUP BY?

A3.
GROUP BY collapses rows that share the same values in the grouped columns into a single result row. Every column in SELECT must either be in the GROUP BY clause or wrapped in an aggregate function (COUNT, SUM, AVG, MAX, MIN).

```sql
-- Valid
SELECT department, COUNT(*), AVG(salary) FROM employees GROUP BY department;

-- Invalid in standard SQL (which 'name' is it for a group of rows?)
SELECT department, name, COUNT(*) FROM employees GROUP BY department;
```

MySQL in non-strict mode actually allows this — it picks an arbitrary value for `name` — which is dangerous and misleading. PostgreSQL rejects it outright, which is the correct behavior.

If you need a non-aggregated column, either add it to GROUP BY (creating more granular groups) or use a window function to compute the aggregate without collapsing rows.

---

Q4. What is a subquery vs. a CTE (Common Table Expression)? When do you prefer one over the other?

A4.
A **subquery** is a query nested inside another query:
```sql
SELECT * FROM employees WHERE department_id IN (
  SELECT id FROM departments WHERE location = 'NYC'
);
```

A **CTE** (WITH clause) is a named, temporary result set:
```sql
WITH nyc_depts AS (
  SELECT id FROM departments WHERE location = 'NYC'
)
SELECT * FROM employees WHERE department_id IN (SELECT id FROM nyc_depts);
```

Both produce the same result, but CTEs are more readable for complex queries. CTEs can be referenced multiple times in the main query — subqueries would need to be duplicated. CTEs can be recursive (tree traversal, hierarchical data). Performance-wise, most modern optimizers treat them identically.

Prefer CTEs when: the query is complex, you reference the result multiple times, or you need recursion. Use subqueries for simple, one-off filters. Avoid deeply nested subqueries — refactor to CTEs for readability.

---

Q5. Write a query to find the second highest salary. Explain multiple approaches.

A5.
**Approach 1 — Subquery**:
```sql
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Approach 2 — DENSE_RANK window function** (most flexible):
```sql
WITH ranked AS (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rnk
  FROM employees
)
SELECT DISTINCT salary FROM ranked WHERE rnk = 2;
```

**Approach 3 — LIMIT/OFFSET**:
```sql
SELECT DISTINCT salary FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;
```

DENSE_RANK is the best approach because it generalizes to Nth highest easily (just change `rnk = N`), handles ties properly, and works when you need it per department (`PARTITION BY department`). The LIMIT approach fails when there are duplicate top salaries.

---

Q6. What is the difference between UNION and UNION ALL?

A6.
**UNION** combines result sets and removes duplicates — it performs an implicit DISTINCT, which requires sorting or hashing the entire result.

**UNION ALL** combines result sets and keeps all rows including duplicates.

If you know there are no duplicates, or you want duplicates, always use UNION ALL — it's significantly faster because it skips the deduplication step. For large datasets, the performance difference is substantial.

```sql
-- Slow: deduplicates unnecessarily
SELECT city FROM customers UNION SELECT city FROM suppliers;

-- Fast: if you don't care about/expect duplicates
SELECT city FROM customers UNION ALL SELECT city FROM suppliers;
```

Rule: default to UNION ALL unless you specifically need deduplication.

---

Q7. Explain the different types of JOIN and write an example of a self-join.

A7.
INNER JOIN returns matching rows. LEFT JOIN returns all left rows plus matching right rows (NULLs where no match). FULL OUTER JOIN returns all rows from both. CROSS JOIN gives the Cartesian product.

**Self-join** — joining a table to itself. Classic use: finding employees and their managers from the same table:

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

This works because the employees table has both the employee's info and a `manager_id` that references another row in the same table. The LEFT JOIN ensures employees without managers (CEO) still appear.

Another use: finding duplicate records:
```sql
SELECT a.id, b.id FROM orders a
JOIN orders b ON a.customer_id = b.customer_id
  AND a.product_id = b.product_id
  AND a.id < b.id;
```

---

Q8. How do you optimize a slow SQL query? Walk through your approach.

A8.
Step-by-step process:

1. **EXPLAIN ANALYZE** the query — look at the execution plan. Check for sequential scans on large tables (should be index scans), nested loop joins on large datasets (consider hash or merge joins), and high row estimates vs. actual rows.

2. **Check indexes** — is the WHERE clause hitting indexed columns? Composite index column order matters. Functions on indexed columns (`WHERE YEAR(created_at) = 2024`) prevent index usage — rewrite as range: `WHERE created_at >= '2024-01-01'`.

3. **Reduce the result set early** — filter as much as possible in WHERE before joining. Don't SELECT * — select only columns you need.

4. **Avoid N+1 patterns** — if your ORM executes one query per row, batch the lookups with JOIN or IN clause.

5. **Check for implicit type conversions** — comparing a string column to an integer prevents index usage.

6. **Consider materialized views** for expensive aggregation queries that are read frequently.

7. **Partition large tables** — if queries always filter by date, range-partition by month.

---

Q9. What is indexing, and when should you NOT create an index?

A9.
An index speeds up reads at the cost of slower writes (every INSERT/UPDATE/DELETE must also update the index) and additional storage.

**Don't create indexes when**:
- The table is small (< few thousand rows) — full table scan is already fast.
- The column has low cardinality (e.g., a boolean or status field with 3 values) — the index doesn't narrow the search enough.
- The table is write-heavy with few reads — index maintenance overhead exceeds read benefits.
- You have too many indexes already — each index slows every write. 5-10 indexes per table is usually the practical limit.
- The column is rarely used in WHERE, JOIN, or ORDER BY.

**Do create indexes on**: primary keys (automatic), foreign keys (not always automatic — add them!), columns frequently in WHERE conditions, columns used in JOIN conditions, columns used in ORDER BY.

---

Q10. What is a NULL in SQL? How do comparisons and aggregations behave with NULLs?

A10.
NULL means unknown or missing — not zero, not empty string, not false. It represents the absence of a value.

**Comparison quirks**: NULL = NULL is NULL (not TRUE). You must use `IS NULL` or `IS NOT NULL`. Any arithmetic with NULL produces NULL: `5 + NULL = NULL`.

**Aggregation behavior**: Most aggregate functions ignore NULLs. `AVG(salary)` skips NULL salaries. `COUNT(salary)` counts non-NULL values. But `COUNT(*)` counts all rows including those with NULLs.

**Gotcha with NOT IN**: `WHERE id NOT IN (1, 2, NULL)` returns zero rows — because `id != NULL` evaluates to NULL, making the entire condition NULL. Use `NOT EXISTS` instead:
```sql
SELECT * FROM A WHERE NOT EXISTS (
  SELECT 1 FROM B WHERE A.id = B.id
);
```

---

### MEDIUM PRIORITY

---

Q11. Write a query to find duplicate records in a table.

A11.
```sql
-- Find duplicates based on email
SELECT email, COUNT(*) as cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Get the actual rows (with IDs)
SELECT * FROM users
WHERE email IN (
  SELECT email FROM users GROUP BY email HAVING COUNT(*) > 1
);

-- Delete duplicates, keeping the lowest ID
DELETE FROM users
WHERE id NOT IN (
  SELECT MIN(id) FROM users GROUP BY email
);
```

Alternative using window function (more flexible):
```sql
WITH ranked AS (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) as rn
  FROM users
)
SELECT * FROM ranked WHERE rn > 1;  -- duplicates
-- DELETE FROM users WHERE id IN (SELECT id FROM ranked WHERE rn > 1);
```

The window function approach is better when your deduplication logic is complex — like keeping the most recently updated row instead of the first one.

---

Q12. What is the difference between DELETE, TRUNCATE, and DROP?

A12.
**DELETE**: Removes rows one by one, logs each deletion, can have a WHERE clause, fires triggers, can be rolled back in a transaction. Slow for large tables because of per-row logging.

**TRUNCATE**: Removes ALL rows at once by deallocating data pages. Much faster than DELETE for full table clears. Doesn't fire row-level triggers. Resets auto-increment counters. In PostgreSQL, it's transactional (can be rolled back); in MySQL, it's not.

**DROP**: Removes the entire table — structure, data, indexes, constraints, everything. The table no longer exists.

Use DELETE when you need conditional removal or trigger execution. TRUNCATE for clearing a table you want to reuse. DROP when you want the table gone permanently. Always double-check before TRUNCATE or DROP in production — there's usually no going back.

---

Q13. What is a correlated subquery? How does it differ from a regular subquery?

A13.
A **regular subquery** runs once, independently of the outer query:
```sql
SELECT * FROM employees WHERE salary > (SELECT AVG(salary) FROM employees);
```

A **correlated subquery** references the outer query and runs once per row of the outer query:
```sql
SELECT * FROM employees e1
WHERE salary > (
  SELECT AVG(salary) FROM employees e2 WHERE e2.department_id = e1.department_id
);
```

This finds employees earning above their department's average. The subquery re-executes for each employee because it depends on `e1.department_id`.

Performance-wise, correlated subqueries can be slow — if the outer query has 10K rows, the subquery runs 10K times. Often, you can rewrite them as JOINs:
```sql
SELECT e.* FROM employees e
JOIN (SELECT department_id, AVG(salary) as avg_sal FROM employees GROUP BY department_id) d
  ON e.department_id = d.department_id
WHERE e.salary > d.avg_sal;
```

---

Q14. How do you use CASE WHEN for conditional logic in SQL?

A14.
CASE WHEN is SQL's if-else. It can go in SELECT, WHERE, ORDER BY, UPDATE, and inside aggregate functions.

```sql
-- Categorize salaries
SELECT name,
  CASE 
    WHEN salary > 150000 THEN 'Senior'
    WHEN salary > 100000 THEN 'Mid'
    ELSE 'Junior'
  END AS level
FROM employees;

-- Conditional aggregation (pivot-like)
SELECT department,
  COUNT(CASE WHEN status = 'active' THEN 1 END) AS active_count,
  COUNT(CASE WHEN status = 'inactive' THEN 1 END) AS inactive_count
FROM employees
GROUP BY department;

-- Conditional ordering
SELECT * FROM tickets
ORDER BY CASE WHEN priority = 'critical' THEN 1
              WHEN priority = 'high' THEN 2
              ELSE 3 END;
```

Conditional aggregation is incredibly useful for creating pivot tables without actually using PIVOT syntax (which varies across databases).

---

Q15. What is an execution plan, and how do you read it to identify bottlenecks?

A15.
An execution plan shows how the database engine will execute your query — which indexes it uses, join algorithms, estimated row counts, and costs.

Run `EXPLAIN ANALYZE` (PostgreSQL) or `EXPLAIN` (MySQL) before your query.

**Key things to look for**:
- **Seq Scan** on a large table → missing index. Should be Index Scan or Index Only Scan.
- **Nested Loop Join** with large tables → consider Hash Join or Merge Join (usually the optimizer's job, but missing indexes can force nested loops).
- **Rows: estimated vs. actual** (in EXPLAIN ANALYZE) → large discrepancies mean stale statistics. Run `ANALYZE` on the table.
- **Sort** operations with high cost → add an index to support the ORDER BY.
- **Filter** rows discarded after fetching → the index isn't selective enough, or you're fetching too many rows before filtering.

The cost numbers are relative — they're useful for comparing two versions of the same query, not as absolute metrics.

---

Q16. What is a recursive CTE? Write an example for hierarchical data.

A16.
A recursive CTE references itself to traverse hierarchical or graph-like data.

```sql
-- Find all managers above an employee (org chart traversal)
WITH RECURSIVE hierarchy AS (
  -- Base case: start with the employee
  SELECT id, name, manager_id, 1 AS level
  FROM employees WHERE id = 42
  
  UNION ALL
  
  -- Recursive case: join to get the manager
  SELECT e.id, e.name, e.manager_id, h.level + 1
  FROM employees e
  JOIN hierarchy h ON e.id = h.manager_id
)
SELECT * FROM hierarchy;
```

Other uses: bill of materials (parts → subparts → sub-subparts), category trees, graph traversal.

Be careful: always include a termination condition to avoid infinite loops. Most databases have a recursion limit (default 100 in PostgreSQL). Add `WHERE level < 10` or a cycle-detection column for safety.

---

Q17. What is the difference between EXISTS and IN? When is EXISTS more efficient?

A17.
```sql
-- IN: evaluates the full subquery, then checks membership
SELECT * FROM orders WHERE customer_id IN (SELECT id FROM customers WHERE country = 'US');

-- EXISTS: returns TRUE as soon as one matching row is found
SELECT * FROM orders o WHERE EXISTS (
  SELECT 1 FROM customers c WHERE c.id = o.customer_id AND c.country = 'US'
);
```

**EXISTS is better when**: the subquery returns a large result set — it short-circuits at the first match instead of materializing the full list. It also handles NULLs correctly (IN has the NOT IN + NULL trap).

**IN is better when**: the subquery returns a small set and is non-correlated — the optimizer can hash-join efficiently.

Modern optimizers (PostgreSQL 12+, MySQL 8+) often transform one into the other anyway. But for readability and NULL-safety, I default to EXISTS for correlated checks and IN for small static lists.

---

Q18. How do you handle pagination efficiently in SQL?

A18.
**Naive approach** — OFFSET/LIMIT:
```sql
SELECT * FROM products ORDER BY id LIMIT 20 OFFSET 10000;
```
The database still scans and discards 10,000 rows. Page 500 is much slower than page 1.

**Keyset pagination** (cursor-based) — much better:
```sql
SELECT * FROM products WHERE id > 10000 ORDER BY id LIMIT 20;
```
The client sends the last seen `id` instead of a page number. The database uses the index to jump directly to the right position — O(1) regardless of page depth.

Limitations of keyset: you can't jump to page N directly, only next/previous. It requires a unique, sequential ordering column. For most API endpoints and infinite-scroll UIs, keyset is the right answer. For admin panels with "go to page 50," consider caching total counts and living with OFFSET's performance cost.

---

### LOW PRIORITY

---

Q19. What is a pivot table in SQL, and how do you create one?

A19.
A pivot table rotates rows into columns — turning category values into column headers.

```sql
-- Using conditional aggregation (works everywhere)
SELECT product,
  SUM(CASE WHEN quarter = 'Q1' THEN revenue END) AS Q1,
  SUM(CASE WHEN quarter = 'Q2' THEN revenue END) AS Q2,
  SUM(CASE WHEN quarter = 'Q3' THEN revenue END) AS Q3,
  SUM(CASE WHEN quarter = 'Q4' THEN revenue END) AS Q4
FROM sales
GROUP BY product;

-- Using PIVOT (SQL Server / Oracle)
SELECT * FROM sales
PIVOT (SUM(revenue) FOR quarter IN ([Q1], [Q2], [Q3], [Q4])) AS pvt;
```

The conditional aggregation approach is more portable and flexible. It works in every SQL database. The PIVOT syntax is database-specific and limited to predefined column names.

---

Q20. What are the differences between CHAR, VARCHAR, and TEXT data types?

A20.
**CHAR(n)**: Fixed-length. Always stores exactly n characters, padded with spaces. CHAR(10) for "abc" stores "abc       ". Slightly faster for truly fixed-length data (state codes, country codes) because the database knows the exact byte offset.

**VARCHAR(n)**: Variable-length with a maximum. Stores only the actual characters plus 1-2 bytes of length overhead. VARCHAR(255) for "abc" stores "abc" (3 bytes + length). Most common choice for strings.

**TEXT**: Variable-length with no specified maximum (or database-defined max). Same storage as VARCHAR in PostgreSQL — there's literally no performance difference between VARCHAR(n) and TEXT in PostgreSQL. In MySQL, TEXT columns can't have default values and have some index limitations.

In PostgreSQL: prefer TEXT or VARCHAR without a length limit unless you have a business rule for maximum length. The length constraint on VARCHAR(n) is a data validation tool, not a performance tool.

---

Q21. What is a transaction savepoint, and how does it work?

A21.
A savepoint is a point within a transaction that you can roll back to without aborting the entire transaction.

```sql
BEGIN;
INSERT INTO orders (id, amount) VALUES (1, 100);
SAVEPOINT order_inserted;

INSERT INTO payments (order_id, amount) VALUES (1, 100);
-- Payment failed? Roll back just the payment, keep the order
ROLLBACK TO SAVEPOINT order_inserted;

-- Try a different payment approach
INSERT INTO payments (order_id, amount) VALUES (1, 100);
COMMIT;
```

Use cases: batch processing where some rows might fail but you want to continue with the rest. Nested operations where a sub-operation might fail frequently (like calling an external service) but the parent operation should continue.

In application code, ORMs often use savepoints to implement nested transaction blocks — Django's `atomic()` blocks create savepoints when nested.

---

Q22. How does COALESCE work? What are its common use cases?

A22.
COALESCE returns the first non-NULL value from its arguments.

```sql
-- Default value for NULLs
SELECT COALESCE(nickname, first_name, 'Anonymous') AS display_name FROM users;

-- Safe division
SELECT revenue / NULLIF(users_count, 0) AS arpu FROM metrics;
-- NULLIF returns NULL if users_count is 0, preventing division by zero

-- Merging columns from a LEFT JOIN
SELECT o.id, COALESCE(c.name, 'Guest') AS customer
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.id;
```

COALESCE is ANSI SQL and works everywhere. Some databases have `NVL` (Oracle) or `IFNULL` (MySQL) for the two-argument case, but COALESCE is portable and supports multiple fallback values.

---

Q23. What does EXPLAIN ANALYZE tell you that EXPLAIN alone does not?

A23.
**EXPLAIN** shows the planned execution — estimated costs and row counts based on table statistics. It doesn't actually run the query.

**EXPLAIN ANALYZE** actually executes the query and shows real numbers: actual execution time, actual rows processed, number of loops.

The key comparison: estimated rows vs. actual rows. If the planner estimates 100 rows but actually processes 100,000, the statistics are stale — run `ANALYZE tablename`. If the planner chooses a nested loop join expecting 10 rows but gets 10,000, performance tanks.

EXPLAIN ANALYZE also shows actual time per node — so you can identify which step is the bottleneck. A sequential scan taking 500ms on a step that follows an index scan taking 2ms tells you exactly where to optimize.

Caution: EXPLAIN ANALYZE executes the query, including mutations. For INSERT/UPDATE/DELETE, wrap it in a transaction and ROLLBACK: `BEGIN; EXPLAIN ANALYZE DELETE ...; ROLLBACK;`

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q24. How do LAG and LEAD window functions work? Give practical examples.

A24.
**LAG** looks at a previous row, **LEAD** looks at a future row — both within a window ordered by some column.

```sql
SELECT date, revenue,
  LAG(revenue, 1) OVER (ORDER BY date) AS prev_day_revenue,
  LEAD(revenue, 1) OVER (ORDER BY date) AS next_day_revenue,
  revenue - LAG(revenue, 1) OVER (ORDER BY date) AS day_over_day_change
FROM daily_sales;
```

LAG(column, n, default) — `n` is how many rows back (default 1), `default` is what to return when there's no previous row (defaults to NULL).

Practical uses: calculating day-over-day growth, detecting gaps in sequences, comparing current row to previous period. For example, finding sessions where response time spiked compared to the previous request:

```sql
SELECT request_id, response_time,
  LAG(response_time) OVER (PARTITION BY user_id ORDER BY timestamp) AS prev_response_time
FROM requests
WHERE response_time > 2 * LAG(response_time) OVER (PARTITION BY user_id ORDER BY timestamp);
```

These are much cleaner than self-joins for row-to-row comparisons.

---

Q25. How do you compute a running total or cumulative sum in SQL?

A25.
Use SUM as a window function with an ORDER BY clause:

```sql
SELECT date, amount,
  SUM(amount) OVER (ORDER BY date) AS running_total
FROM transactions;
```

The key is that `ORDER BY` inside `OVER()` creates a default frame of `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` — so it sums everything from the start up to the current row.

For a rolling 7-day sum:
```sql
SELECT date, amount,
  SUM(amount) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS rolling_7day
FROM daily_sales;
```

You can also PARTITION to reset the running total per group:
```sql
SELECT department, date, amount,
  SUM(amount) OVER (PARTITION BY department ORDER BY date) AS dept_running_total
FROM expenses;
```

This avoids correlated subqueries or self-joins, which would be much slower on large datasets.

---

Q26. How do you find the Nth highest salary without using LIMIT/OFFSET?

A26.
The classic approach uses a correlated subquery:

```sql
SELECT DISTINCT salary FROM employees e1
WHERE (N-1) = (SELECT COUNT(DISTINCT salary) FROM employees e2 WHERE e2.salary > e1.salary);
```

For the 3rd highest, set N=3. The subquery counts how many distinct salaries are greater than the current row's salary. For the highest salary, that count is 0; for the 2nd highest, it's 1; for the 3rd, it's 2.

A cleaner approach using DENSE_RANK:
```sql
SELECT salary FROM (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
  FROM employees
) ranked
WHERE rnk = N;
```

DENSE_RANK handles ties correctly — if two people share the 2nd highest salary, the next distinct salary is still 3rd, not 4th. Use DENSE_RANK over RANK here unless you specifically want gaps.

---

Q27. What are common date/time functions and how do you handle time zone issues in SQL?

A27.
Core functions vary by database, but the concepts are universal:

- **CURRENT_TIMESTAMP / NOW()**: Current datetime
- **DATE_TRUNC('month', timestamp)** (Postgres) / **DATE_FORMAT** (MySQL): Truncate to a period
- **EXTRACT(YEAR FROM date)** or **YEAR(date)**: Pull out components
- **DATE_ADD / DATE_SUB** or **INTERVAL**: Date arithmetic
- **DATEDIFF**: Days between two dates

```sql
-- Group by month
SELECT DATE_TRUNC('month', created_at) AS month, COUNT(*)
FROM orders GROUP BY 1 ORDER BY 1;

-- Events in the last 30 days
SELECT * FROM events WHERE created_at >= CURRENT_DATE - INTERVAL '30 days';
```

Time zone handling: Store everything in UTC. Convert for display using `AT TIME ZONE`:
```sql
SELECT created_at AT TIME ZONE 'UTC' AT TIME ZONE 'America/New_York' AS local_time
FROM orders;
```

Common pitfall: comparing `DATE` and `TIMESTAMP` — `WHERE date_col = '2024-01-15'` works, but `WHERE timestamp_col = '2024-01-15'` matches only midnight exactly. Use range: `WHERE timestamp_col >= '2024-01-15' AND timestamp_col < '2024-01-16'`.

---

### IMPORTANT

---

Q28. What's the difference between a temporary table, a CTE, and a subquery in terms of performance?

A28.
**Subquery**: Inline query — the optimizer can merge it into the outer query or materialize it. For simple subqueries, the optimizer usually inlines them. No physical storage.

**CTE (WITH clause)**: Named subquery. In PostgreSQL 12+, CTEs are inlined by default (optimized like subqueries). In earlier versions and some databases, CTEs are "optimization fences" — they materialize results, preventing the optimizer from pushing predicates into them.

**Temporary table**: Physical table that stores intermediate results on disk/memory. You can index it, analyze it, and reference it multiple times efficiently.

Performance guidelines:
- Use subqueries/CTEs for readability when the optimizer handles them well.
- Use temp tables when you need to reference the same intermediate result multiple times, or when you need indexes on intermediate results.
- Use `MATERIALIZED` / `NOT MATERIALIZED` hints on CTEs (Postgres 12+) to control materialization explicitly.

```sql
-- CTE for readability
WITH high_value AS (
  SELECT * FROM orders WHERE total > 1000
)
SELECT customer_id, COUNT(*) FROM high_value GROUP BY customer_id;

-- Temp table when referenced multiple times
CREATE TEMP TABLE high_value AS SELECT * FROM orders WHERE total > 1000;
CREATE INDEX ON high_value(customer_id);
SELECT ... FROM high_value JOIN ...; -- first use
SELECT ... FROM high_value JOIN ...; -- second use
```

---

Q29. How do you write an UPDATE with a JOIN?

A29.
Syntax varies by database:

**PostgreSQL**:
```sql
UPDATE orders o
SET status = 'vip'
FROM customers c
WHERE o.customer_id = c.id AND c.tier = 'gold';
```

**MySQL**:
```sql
UPDATE orders o
JOIN customers c ON o.customer_id = c.id
SET o.status = 'vip'
WHERE c.tier = 'gold';
```

**SQL Server**:
```sql
UPDATE o
SET o.status = 'vip'
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id
WHERE c.tier = 'gold';
```

The key difference: PostgreSQL uses a `FROM` clause and puts the join condition in `WHERE`. MySQL allows JOIN syntax directly. Always test with a SELECT first — replace `UPDATE … SET` with `SELECT *` to verify which rows will be affected before running the mutation.

---

Q30. What is a lateral join and when would you use it?

A30.
A **lateral join** (or `CROSS APPLY` in SQL Server) lets the right side of the join reference columns from the left side — like a correlated subquery that can return multiple rows.

```sql
-- Top 3 orders per customer
SELECT c.name, o.order_id, o.total
FROM customers c
CROSS JOIN LATERAL (
  SELECT order_id, total FROM orders
  WHERE customer_id = c.id
  ORDER BY total DESC
  LIMIT 3
) o;
```

Without LATERAL, you'd need awkward window functions:
```sql
SELECT name, order_id, total FROM (
  SELECT c.name, o.order_id, o.total,
    ROW_NUMBER() OVER (PARTITION BY c.id ORDER BY o.total DESC) AS rn
  FROM customers c JOIN orders o ON c.id = o.customer_id
) sub WHERE rn <= 3;
```

LATERAL is cleaner when you want "top-N per group" or when you need to call a set-returning function for each row. It's also useful with `UNNEST` — `FROM table t, LATERAL UNNEST(t.array_col) AS elem`.

---

Q31. How do you model and query hierarchical data in SQL?

A31.
Three main approaches:

**1. Recursive CTE** (most portable):
```sql
WITH RECURSIVE org_tree AS (
  SELECT id, name, manager_id, 1 AS depth
  FROM employees WHERE manager_id IS NULL  -- root
  UNION ALL
  SELECT e.id, e.name, e.manager_id, ot.depth + 1
  FROM employees e JOIN org_tree ot ON e.manager_id = ot.id
)
SELECT * FROM org_tree;
```

**2. Materialized Path**: Store the full path as a string — `"/1/5/12/"`. Easy ancestor queries with LIKE: `WHERE path LIKE '/1/5/%'`. Fast reads, slow moves.

**3. Nested Sets**: Each node stores `left` and `right` values. All descendants have values between parent's left and right. Very fast subtree queries, but insertions require updating many rows.

**4. Closure Table**: Separate table storing all ancestor-descendant pairs. Most flexible — fast reads and writes, but more storage. Best for deep/complex hierarchies.

For most cases, recursive CTEs are the safest starting point — they're standard SQL and handle moderate tree sizes well. For read-heavy workloads with rare writes, materialized paths are simple and efficient.

---

### GOOD-TO-HAVE

---

Q32. What's the difference between COUNT(*), COUNT(column), and COUNT(DISTINCT column)?

A32.
- **COUNT(*)**: Counts all rows, including those with NULLs. It counts the row's existence, not any specific value.
- **COUNT(column)**: Counts rows where that column is NOT NULL. If `email` has NULLs, `COUNT(email)` will be less than `COUNT(*)`.
- **COUNT(DISTINCT column)**: Counts unique non-NULL values in that column.

```sql
-- Table: users (5 rows)
-- name: Alice, Bob, NULL, Alice, Charlie
-- email: a@x.com, NULL, NULL, a@x.com, c@x.com

SELECT
  COUNT(*) AS total_rows,             -- 5
  COUNT(name) AS non_null_names,      -- 4
  COUNT(DISTINCT name) AS unique_names, -- 3 (Alice, Bob, Charlie)
  COUNT(email) AS non_null_emails,    -- 3
  COUNT(DISTINCT email) AS unique_emails -- 2 (a@x.com, c@x.com)
FROM users;
```

Performance-wise, `COUNT(*)` is usually the fastest — the engine just counts rows. `COUNT(DISTINCT)` is the most expensive because it needs to track unique values, often using a hash set internally. On large tables, consider `approx_count_distinct()` (available in some databases) if an estimate is acceptable.

---

Q33. What are common string functions and pattern matching techniques in SQL?

A33.
Core string functions:
- **CONCAT / ||**: Join strings. `CONCAT(first, ' ', last)` or `first || ' ' || last`
- **SUBSTRING / SUBSTR**: Extract part. `SUBSTRING(col FROM 1 FOR 3)`
- **UPPER / LOWER / INITCAP**: Case conversion
- **TRIM / LTRIM / RTRIM**: Remove whitespace
- **LENGTH / CHAR_LENGTH**: String length
- **REPLACE**: Substitute. `REPLACE(phone, '-', '')`
- **SPLIT_PART** (Postgres): `SPLIT_PART(email, '@', 2)` → domain
- **POSITION / STRPOS**: Find substring position

Pattern matching:
- **LIKE**: Simple patterns. `%` = any chars, `_` = one char. `WHERE name LIKE 'J%son'`
- **ILIKE** (Postgres): Case-insensitive LIKE
- **SIMILAR TO**: SQL-standard regex-ish
- **~ / REGEXP**: Full regex. `WHERE email ~ '^[a-z]+@gmail\.com$'`

Performance tip: `LIKE 'prefix%'` can use a B-tree index. `LIKE '%suffix'` cannot — it requires a full scan or a reverse index. For frequent text search, consider full-text search (tsvector in Postgres) or trigram indexes (`pg_trgm`).