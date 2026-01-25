---
title: "SQL Mastery - Supplement 1: SQL Quick Reference Cheat Sheet"
date: 2024-02-10 00:00:00 +0530
categories: [SQL, SQL Mastery]
tags: [SQL, Database, Reference, Cheat-sheet, Quick-reference, Syntax]
---

# SQL Quick Reference Cheat Sheet

One-page comprehensive SQL syntax reference for quick lookups.

---

## Query Structure & Execution Order

```sql
SELECT [DISTINCT] columns
FROM table1
[JOIN table2 ON condition]
[WHERE condition]
[GROUP BY columns]
[HAVING condition]
[ORDER BY columns [ASC|DESC]]
[LIMIT n [OFFSET m]]
```

**Execution Order:** FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT

---

## SELECT Basics

```sql
-- Basic selection
SELECT column1, column2 FROM table;
SELECT * FROM table;                    -- Avoid in production!

-- Aliases
SELECT column AS alias_name FROM table;
SELECT CONCAT(first, ' ', last) AS full_name FROM users;

-- DISTINCT
SELECT DISTINCT category FROM products;

-- LIMIT
SELECT * FROM users LIMIT 10;           -- First 10
SELECT * FROM users LIMIT 10 OFFSET 20; -- Rows 21-30
```

---

## WHERE Clause

```sql
-- Comparison
WHERE price > 100
WHERE status = 'active'
WHERE created_at BETWEEN '2024-01-01' AND '2024-12-31'

-- Logical operators
WHERE price > 100 AND category = 'Electronics'
WHERE status = 'active' OR status = 'pending'
WHERE NOT deleted

-- Pattern matching
WHERE email LIKE '%@gmail.com'          -- Ends with
WHERE name LIKE 'John%'                  -- Starts with
WHERE code LIKE 'A_C'                    -- A, any char, C

-- NULL checks
WHERE email IS NULL
WHERE email IS NOT NULL
WHERE COALESCE(discount, 0) > 0

-- IN / NOT IN
WHERE status IN ('active', 'pending')
WHERE id NOT IN (SELECT customer_id FROM blacklist)

-- EXISTS
WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = c.id)
```

---

## JOINS

```sql
-- INNER JOIN (only matching rows)
FROM orders o
INNER JOIN customers c ON o.customer_id = c.id

-- LEFT JOIN (all left rows + matches)
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id

-- RIGHT JOIN (all right rows + matches)
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.id

-- FULL OUTER JOIN (all rows from both) - PostgreSQL only
FROM customers c
FULL OUTER JOIN orders o ON c.id = o.customer_id

-- CROSS JOIN (Cartesian product)
FROM colors CROSS JOIN sizes

-- SELF JOIN
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id
```

---

## Aggregate Functions

```sql
COUNT(*)              -- Count all rows
COUNT(column)         -- Count non-NULL values
COUNT(DISTINCT col)   -- Count unique values

SUM(amount)           -- Total
AVG(price)            -- Average (ignores NULL)
MIN(price)            -- Minimum
MAX(price)            -- Maximum

-- String aggregation
GROUP_CONCAT(name)              -- MySQL
STRING_AGG(name, ', ')          -- PostgreSQL
```

---

## GROUP BY & HAVING

```sql
-- Basic grouping
SELECT category, COUNT(*) as count
FROM products
GROUP BY category;

-- Multiple columns
SELECT category, brand, AVG(price)
FROM products
GROUP BY category, brand;

-- HAVING (filter groups)
SELECT category, AVG(price) as avg_price
FROM products
GROUP BY category
HAVING AVG(price) > 100;

-- WHERE vs HAVING
WHERE status = 'active'    -- Filter rows BEFORE grouping
HAVING COUNT(*) > 10       -- Filter groups AFTER aggregation
```

---

## CASE Expressions

```sql
-- Simple CASE
CASE status
    WHEN 'active' THEN 'Active User'
    WHEN 'inactive' THEN 'Inactive User'
    ELSE 'Unknown'
END

-- Searched CASE
CASE
    WHEN price < 100 THEN 'Budget'
    WHEN price < 500 THEN 'Standard'
    ELSE 'Premium'
END

-- Conditional aggregation
COUNT(CASE WHEN status = 'active' THEN 1 END) as active_count
```

---

## Window Functions

```sql
-- Row numbering
ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC)
RANK() OVER (ORDER BY score DESC)              -- 1,1,3,4
DENSE_RANK() OVER (ORDER BY score DESC)        -- 1,1,2,3

-- Offset functions
LAG(price) OVER (ORDER BY date)                -- Previous row
LEAD(price) OVER (ORDER BY date)               -- Next row
FIRST_VALUE(price) OVER (...)
LAST_VALUE(price) OVER (... ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)

-- Aggregates as windows
SUM(amount) OVER (ORDER BY date)               -- Running total
AVG(price) OVER (PARTITION BY category)       -- Average per group
AVG(sales) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) -- 7-day avg

-- NTILE (percentiles)
NTILE(4) OVER (ORDER BY price)                 -- Quartiles
```

---

## Subqueries

```sql
-- Scalar subquery (single value)
WHERE price > (SELECT AVG(price) FROM products)

-- IN subquery
WHERE customer_id IN (SELECT id FROM premium_customers)

-- EXISTS (faster for large tables)
WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = c.id)

-- Correlated subquery
WHERE price > (SELECT AVG(price) FROM products p2 WHERE p2.category = p1.category)

-- Derived table
FROM (SELECT category, AVG(price) as avg_price FROM products GROUP BY category) sub
```

---

## Common Table Expressions (CTEs)

```sql
-- Basic CTE
WITH active_users AS (
    SELECT * FROM users WHERE status = 'active'
)
SELECT * FROM active_users WHERE created_at > '2024-01-01';

-- Multiple CTEs
WITH 
cte1 AS (SELECT ...),
cte2 AS (SELECT ... FROM cte1)
SELECT * FROM cte2;

-- Recursive CTE
WITH RECURSIVE numbers AS (
    SELECT 1 as n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 10
)
SELECT * FROM numbers;
```

---

## INSERT

```sql
-- Single row
INSERT INTO users (name, email) VALUES ('John', 'john@example.com');

-- Multiple rows
INSERT INTO users (name, email) VALUES
    ('Alice', 'alice@example.com'),
    ('Bob', 'bob@example.com');

-- From SELECT
INSERT INTO archive SELECT * FROM users WHERE inactive = TRUE;

-- UPSERT (MySQL)
INSERT INTO users (id, name, email) VALUES (1, 'John', 'john@example.com')
ON DUPLICATE KEY UPDATE name = VALUES(name), email = VALUES(email);

-- UPSERT (PostgreSQL)
INSERT INTO users (id, name, email) VALUES (1, 'John', 'john@example.com')
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name, email = EXCLUDED.email;
```

---

## UPDATE

```sql
-- Basic update
UPDATE users SET status = 'inactive' WHERE last_login < '2023-01-01';

-- Update multiple columns
UPDATE products SET price = price * 1.1, updated_at = NOW() WHERE category = 'Electronics';

-- Update with JOIN (MySQL)
UPDATE orders o
JOIN customers c ON o.customer_id = c.id
SET o.customer_name = c.name;

-- Update with JOIN (PostgreSQL)
UPDATE orders o
SET customer_name = c.name
FROM customers c
WHERE o.customer_id = c.id;
```

---

## DELETE

```sql
-- Basic delete
DELETE FROM users WHERE inactive_days > 365;

-- Delete all rows (use TRUNCATE instead)
DELETE FROM temp_table;

-- Delete with JOIN (MySQL)
DELETE o FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE c.status = 'deleted';

-- Delete with JOIN (PostgreSQL)
DELETE FROM orders o
USING customers c
WHERE o.customer_id = c.id AND c.status = 'deleted';
```

---

## Indexes

```sql
-- Single column index
CREATE INDEX idx_email ON users(email);

-- Composite index
CREATE INDEX idx_category_price ON products(category, price);

-- Unique index
CREATE UNIQUE INDEX idx_unique_email ON users(email);

-- Partial index (PostgreSQL)
CREATE INDEX idx_active_users ON users(email) WHERE status = 'active';

-- Drop index
DROP INDEX idx_email;  -- PostgreSQL
DROP INDEX idx_email ON users;  -- MySQL
```

---

## Transactions

```sql
-- Basic transaction
BEGIN;  -- or START TRANSACTION
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Rollback on error
BEGIN;
-- ... queries
ROLLBACK;  -- Undo changes

-- Savepoint
BEGIN;
INSERT INTO users VALUES (...);
SAVEPOINT sp1;
DELETE FROM users WHERE id = 100;
ROLLBACK TO SAVEPOINT sp1;  -- Undo delete, keep insert
COMMIT;
```

---

## Date/Time Functions

```sql
-- Current date/time
NOW()                          -- Current timestamp
CURRENT_DATE                   -- Current date
CURRENT_TIME                   -- Current time

-- Extract parts (MySQL)
YEAR(date), MONTH(date), DAY(date)
DATE(timestamp)                -- Extract date from timestamp
TIME(timestamp)                -- Extract time

-- Extract parts (PostgreSQL)
EXTRACT(YEAR FROM date)
EXTRACT(MONTH FROM date)
DATE_TRUNC('month', timestamp) -- Truncate to month

-- Date arithmetic (MySQL)
DATE_ADD(date, INTERVAL 1 DAY)
DATE_SUB(date, INTERVAL 1 MONTH)
DATEDIFF(date1, date2)         -- Days between

-- Date arithmetic (PostgreSQL)
date + INTERVAL '1 day'
date - INTERVAL '1 month'
date2 - date1                  -- Interval between
```

---

## String Functions

```sql
-- Case conversion
UPPER('hello') → 'HELLO'
LOWER('HELLO') → 'hello'

-- Concatenation
CONCAT('Hello', ' ', 'World')           -- MySQL & PostgreSQL
'Hello' || ' ' || 'World'               -- PostgreSQL

-- Substring
SUBSTRING('Hello', 1, 3) → 'Hel'        -- Both
SUBSTR('Hello', 1, 3) → 'Hel'           -- MySQL

-- Trim
TRIM('  text  ') → 'text'
LTRIM('  text') → 'text'
RTRIM('text  ') → 'text'

-- Replace
REPLACE('Hello World', 'World', 'SQL') → 'Hello SQL'

-- Length
LENGTH('Hello') → 5                     -- PostgreSQL (bytes)
CHAR_LENGTH('Hello') → 5                -- MySQL (characters)
```

---

## Numeric Functions

```sql
ROUND(123.456, 2) → 123.46
FLOOR(123.456) → 123
CEIL(123.456) → 124
ABS(-5) → 5
POWER(2, 3) → 8
SQRT(16) → 4
MOD(10, 3) → 1                          -- Modulo
```

---

## NULL Handling

```sql
-- COALESCE (return first non-NULL)
COALESCE(discount_price, regular_price, 0)

-- NULLIF (return NULL if equal)
NULLIF(value, 0)  -- Returns NULL if value = 0

-- IS NULL / IS NOT NULL
WHERE email IS NULL
WHERE email IS NOT NULL

-- NULL in expressions
NULL + 5 → NULL
NULL = NULL → NULL (not TRUE!)
NULL AND TRUE → NULL
NULL OR TRUE → TRUE
```

---

## Set Operations

```sql
-- UNION (removes duplicates)
SELECT id FROM table1
UNION
SELECT id FROM table2;

-- UNION ALL (keeps duplicates, faster)
SELECT id FROM table1
UNION ALL
SELECT id FROM table2;

-- INTERSECT (common rows) - PostgreSQL
SELECT id FROM table1
INTERSECT
SELECT id FROM table2;

-- EXCEPT (rows in first but not second) - PostgreSQL
SELECT id FROM table1
EXCEPT
SELECT id FROM table2;
```

---

## Performance Tips

```sql
-- ✅ DO
SELECT id, name FROM users;              -- Specific columns
WHERE id = 123;                          -- Direct comparison
WHERE date >= '2024-01-01' AND date < '2025-01-01';  -- Sargable
CREATE INDEX idx_user_id ON orders(customer_id);     -- Index FKs

-- ❌ DON'T
SELECT * FROM users;                     -- Wasteful
WHERE YEAR(date) = 2024;                 -- Function prevents index
WHERE user_id = '123';                   -- Type mismatch
LIMIT 10 OFFSET 1000000;                 -- Slow for large offset
```

---

## Common Patterns

```sql
-- Top N per group
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as rn
    FROM products
)
SELECT * FROM ranked WHERE rn <= 3;

-- Running total
SELECT date, amount,
    SUM(amount) OVER (ORDER BY date) as running_total
FROM sales;

-- Find duplicates
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;

-- Gaps in sequence
SELECT id + 1 as gap_start
FROM records r1
WHERE NOT EXISTS (SELECT 1 FROM records r2 WHERE r2.id = r1.id + 1);

-- Pagination (keyset)
SELECT * FROM products WHERE id > 12345 ORDER BY id LIMIT 10;
```

---

## MySQL vs PostgreSQL Key Differences

| Feature | MySQL | PostgreSQL |
|---------|-------|------------|
| String concat | `CONCAT()` | `||` or `CONCAT()` |
| String agg | `GROUP_CONCAT()` | `STRING_AGG()` |
| LIMIT syntax | `LIMIT n, m` | `LIMIT n OFFSET m` |
| FULL OUTER JOIN | ❌ Use UNION | ✅ Native |
| BOOLEAN | `TINYINT(1)` | True `BOOLEAN` |
| Auto-increment | `AUTO_INCREMENT` | `SERIAL` / `IDENTITY` |
| UPSERT | `ON DUPLICATE KEY` | `ON CONFLICT` |
| NULLS ordering | No native support | `NULLS FIRST/LAST` |
| DISTINCT ON | ❌ | ✅ |
| Partial indexes | ❌ | ✅ |
| RETURNING | ❌ | ✅ |

---

## Quick Troubleshooting

| Problem | Solution |
|---------|----------|
| "Unknown column in WHERE" | Column alias used in WHERE → Use actual column or subquery |
| "Can't use aggregate in WHERE" | Use HAVING instead of WHERE for aggregates |
| Query slow with OR | Replace with UNION or rewrite |
| NOT IN returns no rows | Check for NULLs → Use NOT EXISTS |
| "Column must appear in GROUP BY" | Add to GROUP BY or use aggregate function |
| Large OFFSET slow | Use keyset pagination instead |

---

## EXPLAIN Output (Quick Check)

```sql
EXPLAIN SELECT ...;

-- MySQL: Look for
-- type: ref/eq_ref (good), ALL (bad - full scan)
-- key: index name (using index), NULL (no index)
-- Extra: Using index (best), Using filesort (slower)

-- PostgreSQL: Look for
-- Seq Scan (full table scan - bad for large tables)
-- Index Scan / Index Only Scan (good)
-- Hash Join / Merge Join / Nested Loop
```

---

## Essential Keyboard Shortcuts (Client-Dependent)

Most SQL clients:
- `Ctrl+Enter` / `Cmd+Enter` → Execute query
- `F5` → Refresh
- `Ctrl+/` → Comment/uncomment
- `Ctrl+Space` → Auto-complete

---

## Remember

1. **Always use parameterized queries** to prevent SQL injection
2. **Index foreign keys** for join performance
3. **EXPLAIN before deploying** to production
4. **SELECT specific columns**, never `*` in production
5. **Handle NULLs explicitly** with IS NULL/COALESCE
6. **Use transactions** for data consistency
7. **Keep statistics updated** (ANALYZE tables regularly)
8. **Monitor slow queries** and optimize proactively

---

**Print this cheat sheet and keep it handy for quick reference!**
