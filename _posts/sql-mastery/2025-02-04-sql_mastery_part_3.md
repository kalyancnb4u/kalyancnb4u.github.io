---
title: "Complete SQL Mastery Part 3: Advanced SQL Techniques"
date: 2024-02-04 00:00:00 +0530
categories: [SQL, Mastery]
tags: [SQL, Database, PostgreSQL, MySQL, Performance, Optimization, Joins, Window-functions, CTE, Sub-queries, Analytics]
---

# Complete SQL Mastery Part 3: Advanced SQL Techniques

## Introduction

Welcome to Part 3 of the Complete SQL Mastery series. In Parts 1 and 2, we covered SQL fundamentals, command categories, and database internals. Now we tackle advanced SQL techniques that separate intermediate developers from expert database engineers.

This part focuses on complex query patterns that solve real-world business problems:
* **Complex joins** for combining data from multiple sources
* **Subqueries and CTEs** for breaking down complex logic
* **Window functions** for analytics and ranking
* **Advanced aggregation** for reporting and business intelligence
* **Set operations** for combining result sets

Mastering these techniques enables you to:
* Write sophisticated analytical queries
* Optimize report generation
* Solve complex business logic in SQL
* Build efficient data transformations
* Pass senior-level technical interviews

By the end of this part, you'll be able to construct production-grade queries that handle hierarchical data, perform running calculations, and execute complex analytical operations—all with optimal performance.

---

## 3.1 Complex Joins & Relationships

Joins are the foundation of relational databases, allowing you to combine data from multiple tables. Understanding join mechanics, algorithms, and optimization is crucial for performance.

### JOIN Types

SQL supports several join types, each serving specific use cases.

#### INNER JOIN

Returns only rows that have matching values in both tables:

```sql
-- Explicit INNER JOIN syntax (preferred)
SELECT 
    e.emp_id,
    e.first_name,
    e.last_name,
    d.dept_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.dept_id;

-- Implicit join syntax (avoid in production)
SELECT 
    e.emp_id,
    e.first_name,
    d.dept_name
FROM employees e, departments d
WHERE e.department_id = d.dept_id;

-- Multiple join conditions
SELECT 
    e.emp_id,
    e.first_name,
    d.dept_name
FROM employees e
INNER JOIN departments d 
    ON e.department_id = d.dept_id 
    AND e.location_id = d.location_id;
```

**When to use:**
* Default join type when you want only matching records
* Filtering out unmatched rows from either side
* Most common join in OLTP queries

#### LEFT/RIGHT/FULL OUTER JOIN

**LEFT OUTER JOIN** (or LEFT JOIN):
Returns all rows from the left table, with matching rows from the right table (NULL if no match):

```sql
-- Find all employees and their departments (including employees without departments)
SELECT 
    e.emp_id,
    e.first_name,
    COALESCE(d.dept_name, 'No Department') AS dept_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.dept_id;

-- Find employees without departments (unmatched rows)
SELECT 
    e.emp_id,
    e.first_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.dept_id
WHERE d.dept_id IS NULL;
```

**RIGHT OUTER JOIN** (less common, can always be rewritten as LEFT JOIN):

```sql
-- Same as LEFT JOIN with tables reversed
SELECT 
    e.emp_id,
    e.first_name,
    d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.dept_id;

-- Equivalent LEFT JOIN (preferred for readability)
SELECT 
    e.emp_id,
    e.first_name,
    d.dept_name
FROM departments d
LEFT JOIN employees e ON e.department_id = d.dept_id;
```

**FULL OUTER JOIN** (MySQL: not supported directly, PostgreSQL: supported):

```sql
-- PostgreSQL: Returns all rows from both tables
SELECT 
    COALESCE(e.emp_id::TEXT, 'No Employee') AS emp_id,
    COALESCE(d.dept_name, 'No Department') AS dept_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.dept_id;

-- MySQL workaround: UNION of LEFT and RIGHT joins
SELECT 
    e.emp_id,
    e.first_name,
    d.dept_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.dept_id
UNION
SELECT 
    e.emp_id,
    e.first_name,
    d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.dept_id
WHERE e.emp_id IS NULL;
```

#### CROSS JOIN

Cartesian product of two tables (every row from left table with every row from right table):

```sql
-- Explicit CROSS JOIN
SELECT 
    e.first_name,
    d.dept_name
FROM employees e
CROSS JOIN departments d;

-- Implicit cross join (no ON clause)
SELECT 
    e.first_name,
    d.dept_name
FROM employees e, departments d;

-- Useful example: Generate all date/product combinations
SELECT 
    d.date,
    p.product_name
FROM date_dimension d
CROSS JOIN products p
WHERE d.date BETWEEN '2024-01-01' AND '2024-01-31';
-- Result: 31 dates × N products = 31N rows
```

**Use cases:**
* Generate all combinations for testing
* Create calendar/dimension tables
* Matrix operations
* Generally rare in production queries

**⚠️ Warning:** CROSS JOIN of large tables can generate billions of rows!

```sql
-- ❌ DANGEROUS: Accidental cross join
SELECT *
FROM orders o, customers c, products p;
-- If orders: 1M rows, customers: 100K, products: 10K
-- Result: 1,000,000,000,000,000 rows!

-- ✅ CORRECT: Add join conditions
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id;
```

#### SELF JOIN

Joining a table to itself:

```sql
-- Find employees and their managers
SELECT 
    e.emp_id,
    e.first_name AS employee_name,
    m.first_name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.emp_id;

-- Find employees in same department
SELECT 
    e1.first_name AS employee1,
    e2.first_name AS employee2,
    e1.department_id
FROM employees e1
JOIN employees e2 
    ON e1.department_id = e2.department_id 
    AND e1.emp_id < e2.emp_id  -- Avoid duplicates and self-pairs
ORDER BY e1.department_id;

-- Find hierarchical paths (with CTE for better readability)
WITH RECURSIVE emp_hierarchy AS (
    -- Anchor: Top-level employees
    SELECT 
        emp_id,
        first_name,
        manager_id,
        1 as level,
        first_name as path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Add subordinates
    SELECT 
        e.emp_id,
        e.first_name,
        e.manager_id,
        eh.level + 1,
        eh.path || ' > ' || e.first_name
    FROM employees e
    JOIN emp_hierarchy eh ON e.manager_id = eh.emp_id
)
SELECT * FROM emp_hierarchy ORDER BY path;
```

#### Natural Joins (Avoid in Production)

```sql
-- NATURAL JOIN: Automatically joins on all columns with same name
SELECT *
FROM employees
NATURAL JOIN departments;
-- Implicitly joins on any columns with matching names

-- ❌ PROBLEMS:
-- 1. Ambiguous: Which columns are used?
-- 2. Fragile: Adding a column breaks the join
-- 3. Unreadable: Intent unclear

-- ✅ ALWAYS use explicit ON clause
SELECT *
FROM employees e
JOIN departments d ON e.department_id = d.dept_id;
```

### Join Algorithms

Understanding how databases execute joins helps you write better queries.

#### Nested Loop Join

Simplest join algorithm: For each row in outer table, scan inner table for matches.

```
for each row R1 in table1:
    for each row R2 in table2:
        if join_condition(R1, R2):
            output(R1, R2)
```

```sql
-- Example query
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date = '2024-01-15';

-- Nested Loop execution:
-- 1. Filter orders by date (assume 100 matching rows)
-- 2. For each of 100 orders, lookup customer by customer_id
-- 3. If customer found, output combined row

-- Cost: 100 outer rows × 1 inner lookup = 100 lookups
-- Efficient when outer table is small and inner table has index
```

**When used:**
* Small outer table
* Index on join column in inner table
* Highly selective WHERE clause on outer table

**Optimization:**

```sql
-- ✅ GOOD: Index on join column
CREATE INDEX idx_customer_id ON customers(customer_id);

-- Query becomes very efficient:
-- 100 orders × O(log n) index lookups
```

#### Block Nested Loop Join

Optimization of nested loop: Process outer table in blocks to reduce I/O.

```
for each block B1 in table1:
    for each block B2 in table2:
        for each row R1 in B1:
            for each row R2 in B2:
                if join_condition(R1, R2):
                    output(R1, R2)
```

**Controlled by:**
```sql
-- MySQL
SET SESSION join_buffer_size = 256K;  -- Default 256KB

-- PostgreSQL
SET work_mem = '256MB';  -- Affects all work areas including joins
```

#### Index Nested Loop Join

Uses index on inner table for lookups:

```sql
-- Example
SELECT *
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id;

-- With index on order_items.order_id:
-- For each order:
--   Use index to find matching order_items (O(log n) instead of O(n))

-- EXPLAIN output (MySQL)
-- type: ref (index lookup)
-- Extra: Using index

-- EXPLAIN output (PostgreSQL)
-- -> Index Scan on order_items
```

#### Hash Join

Build hash table from smaller table, probe with larger table:

```
Phase 1 (Build):
    for each row R in smaller_table:
        hash_table[hash(join_key)] = R

Phase 2 (Probe):
    for each row S in larger_table:
        if hash_table[hash(join_key)] exists:
            output(hash_table[hash(join_key)], S)
```

```sql
-- Example
SELECT *
FROM large_table l
JOIN small_table s ON l.join_key = s.join_key;

-- Hash Join execution:
-- 1. Build hash table from small_table (in memory)
-- 2. Scan large_table, probe hash table
-- 3. Output matches

-- Cost: O(smaller_table) + O(larger_table)
-- Very efficient for equi-joins on large tables

-- EXPLAIN output (MySQL 8.0.18+)
-- Extra: Using hash join

-- EXPLAIN output (PostgreSQL)
-- -> Hash Join
--    -> Seq Scan on large_table
--    -> Hash
--       -> Seq Scan on small_table
```

**Requirements:**
* Equi-join (equality condition)
* Sufficient memory for hash table
* Available in MySQL 8.0.18+, always available in PostgreSQL

**Memory considerations:**

```sql
-- If hash table doesn't fit in memory, uses disk (slower)
-- MySQL: Controlled by join_buffer_size
-- PostgreSQL: Controlled by work_mem

-- Monitor hash join performance
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM large_table l JOIN small_table s ON l.id = s.id;
-- Look for: "Batches: 1" (good) vs "Batches: 10" (spilled to disk)
```

#### Merge Join (Sort-Merge Join)

Sort both tables on join key, then merge:

```
Phase 1: Sort both tables on join_key
Phase 2: Merge sorted results
    pointer1 = first row of table1
    pointer2 = first row of table2
    while not end of either table:
        if table1[pointer1].key == table2[pointer2].key:
            output(table1[pointer1], table2[pointer2])
        else if table1[pointer1].key < table2[pointer2].key:
            pointer1++
        else:
            pointer2++
```

```sql
-- Example
SELECT *
FROM table1 t1
JOIN table2 t2 ON t1.sorted_col = t2.sorted_col;

-- Merge Join execution:
-- 1. Sort table1 by sorted_col (if not already sorted)
-- 2. Sort table2 by sorted_col (if not already sorted)
-- 3. Merge sorted sequences

-- Cost: O(n log n) + O(m log m) + O(n + m)
-- Efficient when tables are already sorted or have indexes

-- EXPLAIN output (MySQL)
-- Not explicitly shown, uses filesort

-- EXPLAIN output (PostgreSQL)
-- -> Merge Join
--    Merge Cond: (t1.sorted_col = t2.sorted_col)
--    -> Index Scan on table1
--    -> Index Scan on table2
```

**When used:**
* Equi-join on indexed columns
* Tables already sorted on join key
* Large tables where hash join would spill to disk

#### Algorithm Selection Factors

| Factor | Nested Loop | Hash Join | Merge Join |
|--------|-------------|-----------|------------|
| **Best for** | Small outer table | Large equi-joins | Sorted data |
| **Index required** | Yes (inner table) | No | Optional but helps |
| **Memory usage** | Low | High | Medium |
| **Join type** | Any | Equi-join only | Equi-join only |
| **Pre-sorted data** | No benefit | No benefit | Avoids sort |
| **Small tables** | Excellent | Overkill | Overkill |
| **Large tables** | Poor | Excellent | Excellent |

### Advanced Join Patterns

#### Star Schema Joins

Common in data warehouses: fact table joined to multiple dimension tables.

```sql
-- Typical star schema query
SELECT 
    d.date,
    p.product_name,
    c.customer_name,
    st.store_name,
    f.sales_amount,
    f.quantity
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_customer c ON f.customer_key = c.customer_key
JOIN dim_store st ON f.store_key = st.store_key
WHERE d.year = 2024
  AND d.quarter = 1;

-- Optimization: Filter dimensions first, then join to fact
SELECT 
    d.date,
    p.product_name,
    c.customer_name,
    f.sales_amount
FROM (
    SELECT * FROM dim_date WHERE year = 2024 AND quarter = 1
) d
JOIN fact_sales f ON f.date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_customer c ON f.customer_key = c.customer_key;
```

#### Snowflake Schema Joins

Normalized dimensions with multiple levels:

```sql
-- Product dimension normalized into category hierarchy
SELECT 
    f.sales_amount,
    p.product_name,
    sc.subcategory_name,
    c.category_name
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_subcategory sc ON p.subcategory_key = sc.subcategory_key
JOIN dim_category c ON sc.category_key = c.category_key
WHERE c.category_name = 'Electronics';
```

#### Multiple Table Joins (5+ Tables)

```sql
-- Complex business query joining many tables
SELECT 
    o.order_id,
    c.customer_name,
    c.email,
    p.product_name,
    cat.category_name,
    o.order_date,
    oi.quantity,
    oi.unit_price,
    s.store_name,
    s.city,
    emp.first_name AS sales_person
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
JOIN stores s ON o.store_id = s.store_id
JOIN employees emp ON o.sales_person_id = emp.emp_id
WHERE o.order_date >= '2024-01-01'
  AND cat.category_name IN ('Electronics', 'Computers')
ORDER BY o.order_date DESC;

-- Join order matters for performance!
-- Database optimizer reorders joins, but you can help:

-- ✅ GOOD: Filter early, reduce rows before joining
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name
FROM (
    SELECT * FROM orders 
    WHERE order_date >= '2024-01-01'
) o
JOIN (
    SELECT * FROM categories 
    WHERE category_name IN ('Electronics', 'Computers')
) cat
JOIN products p ON p.category_id = cat.category_id
JOIN order_items oi ON oi.product_id = p.product_id AND oi.order_id = o.order_id
JOIN customers c ON o.customer_id = c.customer_id;
```

#### Non-Equi Joins

Joins with conditions other than equality:

```sql
-- Find overlapping date ranges
SELECT 
    p1.project_name,
    p2.project_name,
    p1.start_date,
    p2.start_date
FROM projects p1
JOIN projects p2 
    ON p1.project_id < p2.project_id  -- Avoid duplicates
    AND p1.start_date <= p2.end_date
    AND p1.end_date >= p2.start_date;

-- Salary brackets
SELECT 
    e.first_name,
    e.salary,
    b.bracket_name
FROM employees e
JOIN salary_brackets b
    ON e.salary >= b.min_salary
    AND e.salary < b.max_salary;

-- Range joins (often slow, may need different approach)
SELECT 
    o.order_id,
    d.discount_rate
FROM orders o
JOIN discount_rules d
    ON o.order_total >= d.min_amount
    AND o.order_total < d.max_amount
    AND o.order_date BETWEEN d.start_date AND d.end_date;
```

**Performance warning:** Non-equi joins are slower than equi-joins (can't use hash join).

#### Lateral Joins (PostgreSQL) / Derived Tables with Correlation (MySQL)

**PostgreSQL LATERAL:**

```sql
-- For each department, get top 3 highest-paid employees
SELECT 
    d.dept_name,
    e.first_name,
    e.salary
FROM departments d
CROSS JOIN LATERAL (
    SELECT first_name, salary
    FROM employees
    WHERE department_id = d.dept_id
    ORDER BY salary DESC
    LIMIT 3
) e;

-- Equivalent without LATERAL (using window functions)
SELECT 
    dept_name,
    first_name,
    salary
FROM (
    SELECT 
        d.dept_name,
        e.first_name,
        e.salary,
        ROW_NUMBER() OVER (PARTITION BY d.dept_id ORDER BY e.salary DESC) as rn
    FROM departments d
    JOIN employees e ON e.department_id = d.dept_id
) ranked
WHERE rn <= 3;
```

**MySQL Equivalent (Correlated Derived Tables in MySQL 8.0+):**

```sql
-- MySQL 8.0+ supports derived tables with references to outer query
SELECT 
    d.dept_name,
    top_emp.first_name,
    top_emp.salary
FROM departments d
JOIN (
    SELECT 
        e.department_id,
        e.first_name,
        e.salary,
        ROW_NUMBER() OVER (PARTITION BY e.department_id ORDER BY e.salary DESC) as rn
    FROM employees e
) top_emp ON d.dept_id = top_emp.department_id AND top_emp.rn <= 3;
```

#### Join Elimination

Optimizer removes unnecessary joins:

```sql
-- Query only needs customer table
SELECT c.customer_id, c.customer_name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
-- Optimizer may eliminate orders table if:
-- - No columns from orders selected
-- - Foreign key guarantees every customer has orders
-- - Or LEFT JOIN where orders not used

-- Verify with EXPLAIN
EXPLAIN SELECT c.customer_id, c.customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
-- Look for: orders table not in execution plan
```

#### Semi-Joins and Anti-Joins

**Semi-join (EXISTS):**

```sql
-- Find customers who have placed orders
-- Semi-join: Only checks for existence, doesn't retrieve order data
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);

-- Optimizer may transform to semi-join
-- More efficient than:
SELECT DISTINCT c.customer_id, c.customer_name
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

**Anti-join (NOT EXISTS):**

```sql
-- Find customers who have NOT placed orders
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);

-- Equivalent LEFT JOIN with NULL check (may be less efficient)
SELECT c.customer_id, c.customer_name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

#### Outer Join Simplification

Optimizer may convert OUTER JOIN to INNER JOIN:

```sql
-- Written as LEFT JOIN
SELECT c.customer_name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= '2024-01-01';

-- WHERE clause filters out NULL order_date (unmatched customers)
-- Optimizer converts to INNER JOIN for better performance:
SELECT c.customer_name, o.order_id
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= '2024-01-01';

-- To preserve LEFT JOIN semantics, move condition to ON clause:
SELECT c.customer_name, o.order_id
FROM customers c
LEFT JOIN orders o 
    ON c.customer_id = o.customer_id 
    AND o.order_date >= '2024-01-01';
```

### MySQL vs PostgreSQL Join Differences

| Feature | MySQL 8.0+ | PostgreSQL 13+ |
|---------|------------|----------------|
| **Hash Join** | Available (8.0.18+) | Always available |
| **Merge Join** | Limited (uses sort) | Full support |
| **FULL OUTER JOIN** | Not supported | Supported |
| **LATERAL** | Not supported (use derived tables) | Full support |
| **Join buffer** | join_buffer_size | work_mem |
| **Optimizer hints** | STRAIGHT_JOIN, force index | Join order hints, enable/disable methods |
| **Join algorithms visible in EXPLAIN** | Limited | Detailed (Hash Join, Merge Join, Nested Loop) |

---

## Frequently Asked Questions

**Q1: When should I use LEFT JOIN vs EXISTS?**

**A:** It depends on what you need:

**Use LEFT JOIN when:**
* You need columns from both tables
* You want to count or aggregate unmatched rows

```sql
-- Get customers with their order count (including 0 orders)
SELECT 
    c.customer_name,
    COUNT(o.order_id) as order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

**Use EXISTS when:**
* You only need to know if a match exists
* You don't need data from the other table
* Better performance (stops at first match)

```sql
-- Find customers who have orders
SELECT customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders WHERE customer_id = c.customer_id
);
-- Faster: EXISTS stops after finding first match
```

**Related Concepts**: Semi-joins, query optimization

---

**Q2: Why is my 5-table join so slow?**

**A:** Common causes:

1. **Missing indexes on join columns:**
```sql
-- ❌ BAD: No indexes
SELECT *
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id;

-- ✅ GOOD: Add indexes
CREATE INDEX idx_order_id ON order_items(order_id);
CREATE INDEX idx_product_id ON order_items(product_id);
```

2. **Wrong join order (large Cartesian product):**
```sql
-- Check with EXPLAIN
EXPLAIN SELECT ...
-- Look for: rows column showing millions of intermediate rows
```

3. **No WHERE filtering before joins:**
```sql
-- ❌ BAD: Join all data first, filter after
SELECT *
FROM huge_table1 t1
JOIN huge_table2 t2 ON t1.id = t2.id
WHERE t1.date = '2024-01-01';

-- ✅ GOOD: Filter first
SELECT *
FROM (SELECT * FROM huge_table1 WHERE date = '2024-01-01') t1
JOIN huge_table2 t2 ON t1.id = t2.id;
```

4. **Implicit cross joins:**
```sql
-- ❌ DANGEROUS: Missing join condition
SELECT *
FROM t1, t2, t3
WHERE t1.id = t3.id;
-- Missing: t2 join condition → creates Cartesian product!
```

---

**Q3: What's the difference between CROSS JOIN and INNER JOIN without WHERE?**

**A:** They're identical:

```sql
-- These produce the same result:
SELECT * FROM t1 CROSS JOIN t2;
SELECT * FROM t1 INNER JOIN t2 ON 1=1;
SELECT * FROM t1, t2;

-- All create Cartesian product: every row from t1 with every row from t2
-- If t1 has 100 rows and t2 has 200 rows: 20,000 result rows
```

**Use CROSS JOIN explicitly** when you want Cartesian product to make intent clear:

```sql
-- ✅ GOOD: Intent is clear
SELECT d.date, p.product_name
FROM date_dimension d
CROSS JOIN products p
WHERE d.date BETWEEN '2024-01-01' AND '2024-01-31';

-- ❌ CONFUSING: Looks like missing ON clause
SELECT d.date, p.product_name
FROM date_dimension d
JOIN products p;
```

---

## Interview Questions

**Question 1: Explain how the database decides which join algorithm to use.**

**Difficulty:** Senior

**Answer:**

The query optimizer chooses join algorithms based on several factors:

**1. Table Sizes:**
```sql
-- Small table (< 1000 rows) × Large table
-- → Nested Loop Join with index on large table
SELECT * FROM small_table s
JOIN large_table l ON s.id = l.s_id;

-- Optimizer reasoning:
-- - Few outer rows → nested loop overhead acceptable
-- - Index on large_table.s_id → each lookup is O(log n)
-- - Total cost: 1000 × log(1M) ≈ 20,000 operations
```

**2. Index Availability:**
```sql
-- With index on join column:
-- → Index Nested Loop Join
CREATE INDEX idx_join_key ON inner_table(join_key);

-- Without index:
-- → Hash Join (if equi-join) or Block Nested Loop
```

**3. Join Type:**
```sql
-- Equi-join (equality condition):
-- → Hash Join or Merge Join possible
SELECT * FROM t1 JOIN t2 ON t1.id = t2.id;

-- Non-equi join:
-- → Only Nested Loop available
SELECT * FROM t1 JOIN t2 ON t1.amount > t2.threshold;
```

**4. Memory Available:**
```sql
-- If hash table fits in memory:
-- → Hash Join (very fast)

-- If hash table too large:
-- → May spill to disk (slower)
-- → Or choose Merge Join if data is sorted
```

**5. Data Distribution:**
```sql
-- Skewed data (most values match one key):
-- → Nested Loop may be better (hash join creates huge bucket)

-- Uniform distribution:
-- → Hash Join excellent
```

**Decision Matrix:**

| Scenario | Chosen Algorithm | Why |
|----------|-----------------|-----|
| Small outer (< 10K rows) with index | Index Nested Loop | Index lookups are O(log n) |
| Large equi-join, enough memory | Hash Join | O(n+m) scan, O(1) lookup |
| Large equi-join, data pre-sorted | Merge Join | Avoid sort cost |
| Non-equi join | Block Nested Loop | Only option |
| Very small tables (< 100 rows each) | Nested Loop | Overhead of hash/merge not worth it |

**Optimizer Hints (when optimizer chooses wrong):**

```sql
-- MySQL
SELECT /*+ HASH_JOIN(t1, t2) */ *
FROM t1 JOIN t2 ON t1.id = t2.id;

-- PostgreSQL
SET enable_hashjoin = off;  -- Force different algorithm
SET enable_mergejoin = off;
```

**Why This Matters:** Understanding join algorithm selection helps you:
* Create appropriate indexes
* Structure queries for better performance
* Diagnose slow join queries
* Override optimizer when it makes suboptimal choices

**Follow-up Questions:**
* How would you force the optimizer to use a specific join order?
* What statistics does the optimizer need to make good decisions?
* When might the optimizer choose a suboptimal plan?

---

**Question 2: You have a query joining 10 tables that takes 5 minutes. How do you optimize it?**

**Difficulty:** Senior

**Answer:**

**Step-by-step optimization approach:**

**1. Analyze the execution plan:**
```sql
-- MySQL
EXPLAIN FORMAT=JSON
SELECT ...;

-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT ...;

-- Look for:
-- - Sequential scans on large tables
-- - High row estimates (Cartesian products)
-- - Missing indexes
-- - Sort operations
-- - Temporary tables
```

**2. Identify the slowest operation:**
```sql
-- Look at EXPLAIN ANALYZE actual time
-- Find the node with highest "actual time" or "rows"

-- Example problem: Sequential scan on 10M row table
-- Seq Scan on orders (cost=0..150000 rows=10000000 actual time=45000ms)
```

**3. Add appropriate indexes:**
```sql
-- Index all join columns
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_items_order_id ON order_items(order_id);
CREATE INDEX idx_items_product_id ON order_items(product_id);

-- Covering indexes for frequently selected columns
CREATE INDEX idx_orders_covering 
ON orders(customer_id, order_date, total_amount);
```

**4. Filter early in the query:**
```sql
-- ❌ BAD: Join everything, filter after
SELECT ...
FROM table1 t1
JOIN table2 t2 ON t1.id = t2.t1_id
JOIN table3 t3 ON t2.id = t3.t2_id
...
WHERE t1.date = '2024-01-01'  -- Filter at the end!

-- ✅ GOOD: Filter first
SELECT ...
FROM (
    SELECT * FROM table1 WHERE date = '2024-01-01'
) t1
JOIN table2 t2 ON t1.id = t2.t1_id
JOIN table3 t3 ON t2.id = t3.t2_id
...
```

**5. Break into CTEs for clarity and potential materialization:**
```sql
-- Original complex query
SELECT ... FROM t1 
JOIN t2 ON ... 
JOIN t3 ON ... 
WHERE ... complex conditions ...

-- Refactored with CTEs
WITH filtered_t1 AS (
    SELECT * FROM t1 WHERE date >= '2024-01-01'
),
filtered_t2 AS (
    SELECT * FROM t2 WHERE status = 'active'
),
joined_data AS (
    SELECT ... FROM filtered_t1 f1
    JOIN filtered_t2 f2 ON f1.id = f2.t1_id
)
SELECT ... FROM joined_data JOIN t3 ...;

-- PostgreSQL: May materialize CTEs
-- MySQL 8.0+: Optimizes CTEs inline (use MATERIALIZED hint if needed)
```

**6. Consider denormalization for frequent queries:**
```sql
-- If joining 10 tables for every query:
-- Create materialized view or summary table

-- PostgreSQL
CREATE MATERIALIZED VIEW customer_order_summary AS
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) as order_count,
    SUM(o.total) as lifetime_value
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;

CREATE INDEX idx_summary_customer ON customer_order_summary(customer_id);

REFRESH MATERIALIZED VIEW customer_order_summary;

-- MySQL equivalent: Create physical table
CREATE TABLE customer_order_summary AS
SELECT ...;

-- Update periodically
TRUNCATE customer_order_summary;
INSERT INTO customer_order_summary SELECT ...;
```

**7. Partition large tables:**
```sql
-- If joining on date ranges
CREATE TABLE orders (
    order_id BIGINT,
    order_date DATE,
    ...
) PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025)
);

-- Query with date filter uses partition pruning
SELECT ... FROM orders WHERE order_date >= '2024-01-01';
-- Only scans p2024 partition
```

**8. Verify optimizer statistics are current:**
```sql
-- MySQL
ANALYZE TABLE table1, table2, table3;

-- PostgreSQL
ANALYZE table1;
VACUUM ANALYZE table1;

-- Outdated statistics → wrong query plans
```

**Real-world example optimization:**

```sql
-- Before (5 minutes):
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name,
    ...
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
JOIN ... (6 more tables)
WHERE o.order_date >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- After optimizations (5 seconds):
-- 1. Added indexes on all join columns
-- 2. Filtered orders early
-- 3. Used covering index on orders table
WITH recent_orders AS (
    SELECT order_id, customer_id, order_date
    FROM orders
    WHERE order_date >= DATE_SUB(NOW(), INTERVAL 7 DAY)
    -- Uses index: idx_orders_date_covering (order_date, order_id, customer_id)
)
SELECT 
    ro.order_id,
    c.customer_name,
    p.product_name
FROM recent_orders ro
JOIN customers c ON ro.customer_id = c.customer_id
JOIN order_items oi ON ro.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
...

-- Result: 300x faster (5 min → 1 sec)
```

**Why This Matters:** Multi-table joins are common in real applications. Systematic optimization is essential for production performance.

**Follow-up Questions:**
* How do you identify which table should be the driving table in joins?
* When would you consider rewriting the query entirely vs optimizing the existing one?
* How do you handle optimization when you can't add indexes (e.g., third-party database)?

---

## Key Takeaways

* **Join types serve different purposes**: INNER for matching only, LEFT/RIGHT for optional matches, CROSS for all combinations
* **Join algorithms matter**: Nested Loop for small tables with indexes, Hash Join for large equi-joins, Merge Join for sorted data
* **Index your join columns**: Missing indexes force sequential scans or hash joins
* **Filter early**: Reduce rows before joining to minimize intermediate result sizes
* **Understand execution plans**: EXPLAIN shows which algorithm is used and where time is spent
* **Semi-joins (EXISTS) are more efficient** than DISTINCT when you only need existence check
* **Avoid accidental Cartesian products**: Always specify join conditions for all tables
* **LATERAL joins** (PostgreSQL) enable powerful correlated subquery patterns
* **Join order optimization**: Optimizer reorders joins, but you can help with subqueries/CTEs
* **Consider denormalization**: Materialized views or summary tables for frequent complex joins

---

## 3.2 Subqueries & Common Table Expressions (CTEs)

Subqueries and CTEs allow complex queries to be broken into logical, readable parts. Understanding when and how to use each is crucial for writing maintainable SQL.

### Subquery Types

#### Scalar Subqueries

Returns single value (one row, one column):

```sql
-- Find employees earning more than average
SELECT 
    emp_id,
    first_name,
    salary,
    (SELECT AVG(salary) FROM employees) AS avg_salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);

-- Subquery in SELECT (must return single value)
SELECT 
    order_id,
    total_amount,
    (SELECT AVG(total_amount) FROM orders) AS avg_order_amount,
    total_amount - (SELECT AVG(total_amount) FROM orders) AS diff_from_avg
FROM orders;

-- ⚠️ Error if subquery returns multiple rows:
SELECT * FROM products
WHERE category_id = (SELECT category_id FROM categories);
-- ERROR: subquery returns more than one row
```

**Performance consideration:**

```sql
-- ❌ BAD: Subquery executed for each row
SELECT 
    emp_id,
    salary,
    (SELECT AVG(salary) FROM employees) AS avg_sal  -- Runs N times!
FROM employees;

-- ✅ GOOD: Calculate once with window function
SELECT 
    emp_id,
    salary,
    AVG(salary) OVER () AS avg_sal  -- Calculated once
FROM employees;

-- Or use CTE
WITH avg_salary AS (
    SELECT AVG(salary) AS avg_sal FROM employees
)
SELECT 
    e.emp_id,
    e.salary,
    a.avg_sal
FROM employees e
CROSS JOIN avg_salary a;
```

#### Column Subqueries

Returns multiple rows, single column:

```sql
-- IN subquery
SELECT * FROM orders
WHERE customer_id IN (
    SELECT customer_id 
    FROM customers 
    WHERE country = 'USA'
);

-- ANY/SOME subquery (synonyms)
SELECT * FROM products
WHERE price > ANY (
    SELECT price 
    FROM products 
    WHERE category = 'Electronics'
);
-- Returns products more expensive than ANY Electronics product

SELECT * FROM products
WHERE price > SOME (
    SELECT price 
    FROM products 
    WHERE category = 'Electronics'
);
-- Same as ANY

-- ALL subquery
SELECT * FROM products
WHERE price > ALL (
    SELECT price 
    FROM products 
    WHERE category = 'Electronics'
);
-- Returns products more expensive than ALL Electronics products
-- Equivalent to: price > MAX(Electronics prices)

-- EXISTS (usually faster than IN)
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id
);
-- Returns customers who have placed orders

-- NOT EXISTS
SELECT * FROM customers c
WHERE NOT EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id
);
-- Returns customers who have never placed orders
```

**IN vs EXISTS performance:**

```sql
-- When to use IN vs EXISTS:

-- Use IN when subquery returns small result set:
SELECT * FROM large_table
WHERE id IN (SELECT id FROM small_table WHERE condition);
-- Small result set built once, then used for lookups

-- Use EXISTS when outer table is small or has good filtering:
SELECT * FROM small_table st
WHERE EXISTS (
    SELECT 1 FROM large_table lt 
    WHERE lt.id = st.id AND lt.condition
);
-- Correlated subquery, but stops at first match (short-circuits)

-- EXISTS advantages:
-- 1. Can use indexes on both tables
-- 2. Stops at first match (doesn't need to fetch all rows)
-- 3. Better for semi-joins (checking existence)

-- IN advantages:
-- 1. Simpler syntax
-- 2. Better when subquery returns few distinct values
-- 3. Can be optimized to hash join
```

**Real-world example:**

```sql
-- Find customers who bought ALL products in category 'Electronics'
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE NOT EXISTS (
    -- Products in Electronics that this customer hasn't bought
    SELECT 1
    FROM products p
    WHERE p.category = 'Electronics'
      AND NOT EXISTS (
          SELECT 1
          FROM orders o
          JOIN order_items oi ON o.order_id = oi.order_id
          WHERE o.customer_id = c.customer_id
            AND oi.product_id = p.product_id
      )
);

-- Translation: No Electronics product exists that this customer hasn't bought
-- Double negation: NOT EXISTS (product NOT bought) = bought ALL products
```

#### Row Subqueries

Returns multiple columns (MySQL):

```sql
-- Single row, multiple columns
SELECT * FROM employees
WHERE (department_id, salary) = (
    SELECT department_id, MAX(salary)
    FROM employees
    GROUP BY department_id
    LIMIT 1
);

-- Multiple rows, multiple columns
SELECT * FROM employees
WHERE (department_id, salary) IN (
    SELECT department_id, MAX(salary)
    FROM employees
    GROUP BY department_id
);
-- Returns highest-paid employee in each department
```

#### Table Subqueries (Derived Tables)

Returns multiple rows and columns:

```sql
-- Subquery in FROM clause (derived table)
SELECT 
    dept,
    avg_salary,
    max_salary
FROM (
    SELECT 
        department_id AS dept,
        AVG(salary) AS avg_salary,
        MAX(salary) AS max_salary
    FROM employees
    GROUP BY department_id
) AS dept_stats
WHERE avg_salary > 50000;

-- Must have alias!
-- MySQL requires alias even if not referenced

-- Join with derived table
SELECT 
    e.emp_id,
    e.first_name,
    e.salary,
    ds.avg_salary,
    e.salary - ds.avg_salary AS diff_from_avg
FROM employees e
JOIN (
    SELECT 
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS ds ON e.department_id = ds.department_id;
```

#### Correlated Subqueries

Subquery references outer query:

```sql
-- Find employees earning more than their department average
SELECT 
    emp_id,
    first_name,
    department_id,
    salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id  -- Correlated!
);

-- Executed for each row of outer query
-- Can be slow on large tables

-- Find customers with above-average order totals
SELECT 
    c.customer_id,
    c.customer_name,
    (
        SELECT AVG(total_amount)
        FROM orders o
        WHERE o.customer_id = c.customer_id
    ) AS avg_order_amount
FROM customers c
WHERE (
    SELECT AVG(total_amount)
    FROM orders o
    WHERE o.customer_id = c.customer_id
) > (
    SELECT AVG(total_amount)
    FROM orders
);
```

**Rewriting correlated subqueries for performance:**

```sql
-- ❌ SLOW: Correlated subquery
SELECT 
    e.emp_id,
    e.first_name,
    e.salary
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e.department_id
);

-- ✅ FAST: JOIN with aggregation
SELECT 
    e.emp_id,
    e.first_name,
    e.salary
FROM employees e
JOIN (
    SELECT 
        department_id,
        AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
) AS dept_avg ON e.department_id = dept_avg.department_id
WHERE e.salary > dept_avg.avg_salary;

-- Or use window function (best for this case)
SELECT 
    emp_id,
    first_name,
    salary
FROM (
    SELECT 
        emp_id,
        first_name,
        salary,
        AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary
    FROM employees
) AS e
WHERE salary > dept_avg_salary;
```

### Common Table Expressions (CTEs)

CTEs improve readability and maintainability:

```sql
-- Basic CTE syntax
WITH cte_name AS (
    SELECT ...
    FROM ...
    WHERE ...
)
SELECT *
FROM cte_name;

-- Multiple CTEs
WITH 
    sales_summary AS (
        SELECT 
            customer_id,
            SUM(total_amount) AS total_sales,
            COUNT(*) AS order_count
        FROM orders
        WHERE order_date >= '2024-01-01'
        GROUP BY customer_id
    ),
    customer_categories AS (
        SELECT 
            customer_id,
            CASE 
                WHEN total_sales > 10000 THEN 'VIP'
                WHEN total_sales > 5000 THEN 'Premium'
                ELSE 'Standard'
            END AS category
        FROM sales_summary
    )
SELECT 
    c.customer_name,
    ss.total_sales,
    ss.order_count,
    cc.category
FROM customers c
JOIN sales_summary ss ON c.customer_id = ss.customer_id
JOIN customer_categories cc ON c.customer_id = cc.customer_id
ORDER BY ss.total_sales DESC;
```

**CTE vs Subquery:**

```sql
-- Subquery (less readable)
SELECT 
    c.customer_name,
    o.total_sales,
    o.order_count
FROM customers c
JOIN (
    SELECT 
        customer_id,
        SUM(total_amount) AS total_sales,
        COUNT(*) AS order_count
    FROM orders
    WHERE order_date >= '2024-01-01'
    GROUP BY customer_id
) AS o ON c.customer_id = o.customer_id
WHERE o.total_sales > (
    SELECT AVG(total_sales)
    FROM (
        SELECT 
            customer_id,
            SUM(total_amount) AS total_sales
        FROM orders
        WHERE order_date >= '2024-01-01'
        GROUP BY customer_id
    ) AS avg_calc
);

-- CTE (more readable)
WITH sales_summary AS (
    SELECT 
        customer_id,
        SUM(total_amount) AS total_sales,
        COUNT(*) AS order_count
    FROM orders
    WHERE order_date >= '2024-01-01'
    GROUP BY customer_id
),
avg_sales AS (
    SELECT AVG(total_sales) AS avg_total
    FROM sales_summary
)
SELECT 
    c.customer_name,
    ss.total_sales,
    ss.order_count
FROM customers c
JOIN sales_summary ss ON c.customer_id = ss.customer_id
CROSS JOIN avg_sales a
WHERE ss.total_sales > a.avg_total;
```

**CTE advantages:**
- Named, reusable within same query
- Can reference earlier CTEs
- More readable for complex queries
- Easier to debug (can run each CTE separately)

**CTE disadvantages:**
- May be materialized (optimization barrier in some databases)
- Cannot be indexed
- May not be as optimized as derived tables in some cases

### Recursive CTEs

Powerful for hierarchical data:

```sql
-- Employee hierarchy (manager-employee relationship)
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: Top-level managers (no manager)
    SELECT 
        emp_id,
        first_name,
        manager_id,
        1 AS level,
        CAST(first_name AS VARCHAR(1000)) AS path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: Employees with managers
    SELECT 
        e.emp_id,
        e.first_name,
        e.manager_id,
        eh.level + 1,
        CAST(eh.path || ' > ' || e.first_name AS VARCHAR(1000))
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.emp_id
)
SELECT 
    emp_id,
    first_name,
    level,
    path
FROM employee_hierarchy
ORDER BY path;

-- Output:
-- emp_id | first_name | level | path
-- 1      | Alice      | 1     | Alice
-- 2      | Bob        | 2     | Alice > Bob
-- 5      | Eve        | 3     | Alice > Bob > Eve
-- 3      | Carol      | 2     | Alice > Carol
```

**Recursive CTE structure:**

```sql
WITH RECURSIVE cte_name AS (
    -- 1. Base case (anchor member)
    SELECT ... FROM ... WHERE ...
    
    UNION ALL  -- or UNION
    
    -- 2. Recursive case (recursive member)
    SELECT ... 
    FROM ... 
    JOIN cte_name ON ...  -- References itself!
)
SELECT * FROM cte_name;
```

**More recursive examples:**

```sql
-- Number series (1 to 10)
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 10
)
SELECT * FROM numbers;

-- Date series (all dates in January 2024)
WITH RECURSIVE dates AS (
    SELECT DATE '2024-01-01' AS d
    UNION ALL
    SELECT d + INTERVAL '1 day' 
    FROM dates 
    WHERE d < DATE '2024-01-31'
)
SELECT * FROM dates;

-- Category tree (nested categories)
WITH RECURSIVE category_tree AS (
    -- Root categories
    SELECT 
        category_id,
        category_name,
        parent_category_id,
        1 AS level,
        category_name AS path
    FROM categories
    WHERE parent_category_id IS NULL
    
    UNION ALL
    
    -- Child categories
    SELECT 
        c.category_id,
        c.category_name,
        c.parent_category_id,
        ct.level + 1,
        ct.path || ' > ' || c.category_name
    FROM categories c
    JOIN category_tree ct ON c.parent_category_id = ct.category_id
)
SELECT * FROM category_tree
ORDER BY path;

-- Graph traversal (find all connected nodes)
WITH RECURSIVE graph_traversal AS (
    -- Starting node
    SELECT 
        node_id,
        connected_to,
        1 AS distance,
        ARRAY[node_id] AS path
    FROM graph
    WHERE node_id = 1  -- Start from node 1
    
    UNION
    
    -- Connected nodes (UNION prevents cycles)
    SELECT 
        g.node_id,
        g.connected_to,
        gt.distance + 1,
        gt.path || g.node_id
    FROM graph g
    JOIN graph_traversal gt ON g.node_id = gt.connected_to
    WHERE NOT (g.node_id = ANY(gt.path))  -- Prevent cycles
)
SELECT * FROM graph_traversal;
```

**Preventing infinite recursion:**

```sql
-- ⚠️ Dangerous: No termination condition
WITH RECURSIVE infinite AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM infinite  -- Never stops!
)
SELECT * FROM infinite;

-- ✅ Safe: Add WHERE condition
WITH RECURSIVE safe AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM safe WHERE n < 1000  -- Stops at 1000
)
SELECT * FROM safe;

-- PostgreSQL: Max recursion depth
-- SET max_recursion_depth is not a valid setting
-- PostgreSQL doesn't have a direct recursion depth limit setting

-- MySQL: cte_max_recursion_depth
SET cte_max_recursion_depth = 100;  -- Default: 1000
```

### Materialized CTEs (PostgreSQL 12+)

```sql
-- By default, PostgreSQL may inline CTEs (optimization barrier removed)
-- Force materialization with MATERIALIZED keyword

-- Not materialized (may be inlined)
WITH sales AS (
    SELECT customer_id, SUM(total_amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT * FROM sales WHERE total > 1000;

-- Forced materialization
WITH sales AS MATERIALIZED (
    SELECT customer_id, SUM(total_amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT * FROM sales WHERE total > 1000;

-- Prevent materialization (allow inlining)
WITH sales AS NOT MATERIALIZED (
    SELECT customer_id, SUM(total_amount) AS total
    FROM orders
    GROUP BY customer_id
)
SELECT * FROM sales WHERE total > 1000;

-- When to use MATERIALIZED:
-- - CTE used multiple times (compute once)
-- - CTE result is small
-- - Want to prevent optimizer from pushing down predicates

-- When to use NOT MATERIALIZED:
-- - CTE used once
-- - Want optimizer to push down WHERE conditions
-- - Subquery would be more efficient
```

### Lateral Joins (PostgreSQL) / Derived Tables (MySQL 8.0+)

LATERAL allows correlated derived tables:

```sql
-- PostgreSQL LATERAL
-- Get top 3 products per category
SELECT 
    c.category_name,
    p.product_name,
    p.price
FROM categories c
CROSS JOIN LATERAL (
    SELECT product_name, price
    FROM products
    WHERE category_id = c.category_id
    ORDER BY price DESC
    LIMIT 3
) AS p;

-- Equivalent without LATERAL (not possible directly)
-- Would need window functions or self-join

-- MySQL 8.0+ equivalent (automatic lateral)
SELECT 
    c.category_name,
    p.product_name,
    p.price
FROM categories c
JOIN (
    SELECT category_id, product_name, price
    FROM products
    ORDER BY price DESC
) AS p ON c.category_id = p.category_id
LIMIT 3;  -- But this limits total, not per category

-- MySQL workaround with window function
WITH ranked_products AS (
    SELECT 
        category_id,
        product_name,
        price,
        ROW_NUMBER() OVER (PARTITION BY category_id ORDER BY price DESC) AS rn
    FROM products
)
SELECT 
    c.category_name,
    rp.product_name,
    rp.price
FROM categories c
JOIN ranked_products rp ON c.category_id = rp.category_id
WHERE rp.rn <= 3;
```

**LATERAL use cases:**

```sql
-- For each customer, get their 5 most recent orders
SELECT 
    c.customer_id,
    c.customer_name,
    recent.order_id,
    recent.order_date,
    recent.total_amount
FROM customers c
CROSS JOIN LATERAL (
    SELECT order_id, order_date, total_amount
    FROM orders
    WHERE customer_id = c.customer_id
    ORDER BY order_date DESC
    LIMIT 5
) AS recent;

-- For each product, find similar products (by price range)
SELECT 
    p1.product_name,
    p1.price,
    similar.product_name AS similar_product,
    similar.price AS similar_price
FROM products p1
CROSS JOIN LATERAL (
    SELECT product_name, price
    FROM products p2
    WHERE p2.product_id != p1.product_id
      AND p2.price BETWEEN p1.price * 0.9 AND p1.price * 1.1
    ORDER BY ABS(p2.price - p1.price)
    LIMIT 3
) AS similar;
```

---

## Frequently Asked Questions

**Q1: When should I use a CTE vs a subquery?**

**A:**

**Use CTE when:**

```sql
-- 1. Multiple references to same logic
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', order_date) AS month,
        SUM(total_amount) AS total
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT 
    month,
    total,
    LAG(total) OVER (ORDER BY month) AS prev_month,
    total - LAG(total) OVER (ORDER BY month) AS growth
FROM monthly_sales;
-- monthly_sales defined once, used once
-- But with window functions, cleaner than subquery

-- 2. Building complex logic step-by-step
WITH 
    step1 AS (SELECT ... FROM ...),
    step2 AS (SELECT ... FROM step1 ...),  -- References step1
    step3 AS (SELECT ... FROM step2 ...)   -- References step2
SELECT * FROM step3;
-- Clear progression of logic

-- 3. Recursive queries (only option)
WITH RECURSIVE hierarchy AS (
    SELECT emp_id, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.emp_id, e.manager_id, h.level + 1
    FROM employees e
    JOIN hierarchy h ON e.manager_id = h.emp_id
)
SELECT * FROM hierarchy;
```

**Use subquery when:**

```sql
-- 1. One-time use, simple logic
SELECT * 
FROM orders
WHERE customer_id IN (SELECT customer_id FROM customers WHERE country = 'USA');
-- Simple, clear intent

-- 2. Correlated subquery for filtering
SELECT *
FROM products p
WHERE price > (
    SELECT AVG(price) 
    FROM products 
    WHERE category_id = p.category_id
);
-- Subquery depends on outer row

-- 3. Scalar value in SELECT
SELECT 
    order_id,
    (SELECT customer_name FROM customers WHERE customer_id = o.customer_id) AS name
FROM orders o;
-- Single value lookup
```

**Performance considerations:**

```sql
-- PostgreSQL 12+: CTEs can be inlined (optimized like subqueries)
-- Older PostgreSQL: CTEs are optimization barriers (always materialized)

-- MySQL 8.0+: CTEs and derived tables optimized similarly

-- When in doubt:
-- - Use CTE for readability
-- - Profile both approaches
-- - Check EXPLAIN plans
```

---

**Q2: My correlated subquery is very slow. How do I optimize it?**

**A:**

**Problem example:**

```sql
-- ❌ SLOW: Correlated subquery executed for EVERY row
SELECT 
    e.emp_id,
    e.first_name,
    e.salary,
    (
        SELECT AVG(salary) 
        FROM employees e2 
        WHERE e2.department_id = e.department_id
    ) AS dept_avg_salary,
    (
        SELECT COUNT(*) 
        FROM employees e3 
        WHERE e3.department_id = e.department_id
    ) AS dept_employee_count
FROM employees e
WHERE e.status = 'active';

-- If 10,000 active employees:
-- - Subquery 1 runs 10,000 times
-- - Subquery 2 runs 10,000 times
-- Total: 20,000 subquery executions!
```

**Solution 1: JOIN with pre-aggregated data**

```sql
-- ✅ FAST: Aggregate once, join
WITH dept_stats AS (
    SELECT 
        department_id,
        AVG(salary) AS avg_salary,
        COUNT(*) AS employee_count
    FROM employees
    GROUP BY department_id
)
SELECT 
    e.emp_id,
    e.first_name,
    e.salary,
    ds.avg_salary AS dept_avg_salary,
    ds.employee_count AS dept_employee_count
FROM employees e
JOIN dept_stats ds ON e.department_id = ds.department_id
WHERE e.status = 'active';

-- dept_stats computed once for all departments
-- Then joined with employees
-- Much faster!
```

**Solution 2: Window functions**

```sql
-- ✅ FAST: Window functions compute once per partition
SELECT 
    emp_id,
    first_name,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary,
    COUNT(*) OVER (PARTITION BY department_id) AS dept_employee_count
FROM employees
WHERE status = 'active';

-- Window functions processed after filtering
-- Each partition (department) processed once
-- Optimal for this use case
```

**Solution 3: Lateral join (PostgreSQL)**

```sql
-- When you need TOP N or complex logic per row
SELECT 
    c.customer_id,
    c.customer_name,
    recent.order_count,
    recent.total_spent
FROM customers c
CROSS JOIN LATERAL (
    SELECT 
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent
    FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.order_date >= CURRENT_DATE - INTERVAL '90 days'
) AS recent;

-- Better than correlated subquery because:
-- - Can use indexes efficiently
-- - Can return multiple columns
-- - Clearer intent
```

**Comparison:**

```sql
-- Benchmark (10,000 employees, 50 departments):

-- Correlated subquery: 5000ms
-- JOIN with CTE: 50ms (100x faster!)
-- Window function: 45ms (111x faster!)

-- Why so much faster?
-- Correlated: 10,000 × 2 = 20,000 aggregations
-- JOIN/Window: 50 aggregations (one per department)
```

---

**Q3: What's the difference between UNION and UNION ALL?**

**A:**

**UNION (removes duplicates):**

```sql
SELECT product_id, product_name FROM products WHERE category = 'Electronics'
UNION
SELECT product_id, product_name FROM products WHERE price < 100;

-- Process:
-- 1. Execute first query
-- 2. Execute second query
-- 3. Combine results
-- 4. Remove duplicates (DISTINCT operation)
-- 5. Return unique rows

-- If product is both Electronics AND < $100:
-- Returns only ONE row for that product
```

**UNION ALL (keeps duplicates):**

```sql
SELECT product_id, product_name FROM products WHERE category = 'Electronics'
UNION ALL
SELECT product_id, product_name FROM products WHERE price < 100;

-- Process:
-- 1. Execute first query
-- 2. Execute second query
-- 3. Combine results
-- 4. Return ALL rows (including duplicates)

-- If product is both Electronics AND < $100:
-- Returns TWO rows for that product
```

**Performance difference:**

```sql
-- UNION: Slower (must check for duplicates)
-- - Requires sort or hash to identify duplicates
-- - Extra memory for deduplication
-- - O(N log N) or O(N) depending on method

-- UNION ALL: Faster (no deduplication)
-- - Simple append of result sets
-- - O(N)
-- - Typically 2-10x faster

-- Example benchmark (1M rows each):
-- UNION: 2.5 seconds
-- UNION ALL: 0.3 seconds (8x faster!)
```

**When to use each:**

```sql
-- Use UNION when duplicates are possible AND undesired
SELECT email FROM employees
UNION
SELECT email FROM contractors;
-- Want unique list of all email addresses

-- Use UNION ALL when:
-- 1. Duplicates are impossible (disjoint sets)
SELECT * FROM orders_2023
UNION ALL
SELECT * FROM orders_2024;
-- No overlap between years, UNION ALL is safe and faster

-- 2. Duplicates are desired
SELECT 'Employee' AS type, COUNT(*) FROM employees
UNION ALL
SELECT 'Customer' AS type, COUNT(*) FROM customers;
-- Want both rows even if counts are same

-- 3. Performance-critical (and duplicates OK)
SELECT product_id FROM category_a_products
UNION ALL
SELECT product_id FROM category_b_products;
-- If overlap doesn't matter for your use case
```

**Combining multiple queries:**

```sql
-- Multiple UNIONs
SELECT product_id FROM products WHERE category = 'Electronics'
UNION
SELECT product_id FROM products WHERE price < 100
UNION
SELECT product_id FROM products WHERE rating > 4.5;

-- Mix UNION and UNION ALL (use parentheses for clarity)
(
    SELECT product_id FROM products WHERE category = 'Electronics'
    UNION
    SELECT product_id FROM products WHERE price < 100
)
UNION ALL
SELECT product_id FROM archived_products;
-- First two are deduplicated, then archived products appended
```

---

## Interview Questions

**Question 1: Write a query to find employees who earn more than all employees in the 'Sales' department.**

**Difficulty:** Mid-Level

**Answer:**

**Solution 1: Using ALL**

```sql
SELECT 
    emp_id,
    first_name,
    last_name,
    salary,
    department_id
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE department_id = (
        SELECT dept_id FROM departments WHERE dept_name = 'Sales'
    )
);
```

**Solution 2: Using MAX (more readable)**

```sql
SELECT 
    emp_id,
    first_name,
    last_name,
    salary,
    department_id
FROM employees
WHERE salary > (
    SELECT MAX(salary)
    FROM employees e
    JOIN departments d ON e.department_id = d.dept_id
    WHERE d.dept_name = 'Sales'
);
```

**Solution 3: Using CTE (most readable)**

```sql
WITH sales_max_salary AS (
    SELECT MAX(e.salary) AS max_salary
    FROM employees e
    JOIN departments d ON e.department_id = d.dept_id
    WHERE d.dept_name = 'Sales'
)
SELECT 
    e.emp_id,
    e.first_name,
    e.last_name,
    e.salary,
    e.department_id
FROM employees e
CROSS JOIN sales_max_salary sms
WHERE e.salary > sms.max_salary;
```

**Edge cases to consider:**

```sql
-- What if Sales department has no employees?
-- Subquery returns NULL
-- salary > NULL evaluates to NULL (not TRUE)
-- Result: No rows returned (correct behavior)

-- What if multiple departments named 'Sales'?
-- Need DISTINCT or rethink query

-- Better query with dept_name in result:
WITH sales_employees AS (
    SELECT salary
    FROM employees e
    JOIN departments d ON e.department_id = d.dept_id
    WHERE d.dept_name = 'Sales'
)
SELECT 
    e.emp_id,
    e.first_name,
    d.dept_name,
    e.salary,
    (SELECT MAX(salary) FROM sales_employees) AS sales_max_salary
FROM employees e
JOIN departments d ON e.department_id = d.dept_id
WHERE e.salary > (SELECT MAX(salary) FROM sales_employees);
```

**Why This Matters:** This demonstrates understanding of ALL, subqueries, and alternative approaches. In interviews, showing multiple solutions shows deeper knowledge.

**Follow-up Questions:**
* How would you modify this to find employees earning in the top 10% across the company?
* What indexes would optimize this query?

---

**Question 2: You have a self-referential employees table with manager_id. Write a recursive query to show the full reporting chain for a specific employee.**

**Difficulty:** Senior

**Answer:**

**Recursive CTE solution:**

```sql
-- Table structure:
-- employees (emp_id, first_name, last_name, manager_id)

-- Find reporting chain for employee 42 (upward to CEO)
WITH RECURSIVE reporting_chain AS (
    -- Base case: The specific employee
    SELECT 
        emp_id,
        first_name,
        last_name,
        manager_id,
        1 AS level,
        CAST(first_name || ' ' || last_name AS VARCHAR(1000)) AS chain
    FROM employees
    WHERE emp_id = 42
    
    UNION ALL
    
    -- Recursive case: Add each manager
    SELECT 
        e.emp_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        rc.level + 1,
        CAST(e.first_name || ' ' || e.last_name || ' > ' || rc.chain AS VARCHAR(1000))
    FROM employees e
    JOIN reporting_chain rc ON e.emp_id = rc.manager_id
)
SELECT 
    emp_id,
    first_name,
    last_name,
    level,
    chain
FROM reporting_chain
ORDER BY level DESC;  -- CEO at top
```

**Output example:**

```
emp_id | first_name | last_name | level | chain
-------|------------|-----------|-------|------------------------
1      | Alice      | CEO       | 3     | Alice CEO > Bob Manager > John Employee
5      | Bob        | Manager   | 2     | Bob Manager > John Employee
42     | John       | Employee  | 1     | John Employee
```

**Alternative: Downward hierarchy (all reports under a manager)**

```sql
-- Find all employees under manager 5
WITH RECURSIVE employee_tree AS (
    -- Base case: The manager
    SELECT 
        emp_id,
        first_name,
        last_name,
        manager_id,
        1 AS level,
        CAST(first_name || ' ' || last_name AS VARCHAR(1000)) AS path
    FROM employees
    WHERE emp_id = 5  -- Manager
    
    UNION ALL
    
    -- Recursive case: All direct and indirect reports
    SELECT 
        e.emp_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        et.level + 1,
        CAST(et.path || ' > ' || e.first_name || ' ' || e.last_name AS VARCHAR(1000))
    FROM employees e
    JOIN employee_tree et ON e.manager_id = et.emp_id
)
SELECT * FROM employee_tree
ORDER BY path;
```

**Enhanced version with titles and employee counts:**

```sql
WITH RECURSIVE org_chart AS (
    -- Base case
    SELECT 
        emp_id,
        first_name,
        last_name,
        job_title,
        manager_id,
        1 AS level,
        ARRAY[emp_id] AS path_ids,
        first_name || ' ' || last_name AS path
    FROM employees
    WHERE emp_id = 42
    
    UNION ALL
    
    -- Recursive case
    SELECT 
        e.emp_id,
        e.first_name,
        e.last_name,
        e.job_title,
        e.manager_id,
        oc.level + 1,
        oc.path_ids || e.emp_id,
        e.first_name || ' ' || e.last_name || ' > ' || oc.path
    FROM employees e
    JOIN org_chart oc ON e.emp_id = oc.manager_id
),
employee_counts AS (
    SELECT 
        manager_id,
        COUNT(*) AS direct_reports
    FROM employees
    WHERE manager_id IS NOT NULL
    GROUP BY manager_id
)
SELECT 
    oc.emp_id,
    oc.first_name,
    oc.last_name,
    oc.job_title,
    oc.level,
    COALESCE(ec.direct_reports, 0) AS direct_reports,
    oc.path
FROM org_chart oc
LEFT JOIN employee_counts ec ON oc.emp_id = ec.manager_id
ORDER BY oc.level DESC;
```

**MySQL version (same logic):**

```sql
-- MySQL 8.0+ supports recursive CTEs
WITH RECURSIVE reporting_chain AS (
    SELECT 
        emp_id,
        first_name,
        last_name,
        manager_id,
        1 AS level,
        CONCAT(first_name, ' ', last_name) AS chain
    FROM employees
    WHERE emp_id = 42
    
    UNION ALL
    
    SELECT 
        e.emp_id,
        e.first_name,
        e.last_name,
        e.manager_id,
        rc.level + 1,
        CONCAT(e.first_name, ' ', e.last_name, ' > ', rc.chain)
    FROM employees e
    JOIN reporting_chain rc ON e.emp_id = rc.manager_id
)
SELECT * FROM reporting_chain
ORDER BY level DESC;
```

**Preventing infinite loops (circular references):**

```sql
-- If data has cycle (A reports to B, B reports to A)
-- Add cycle detection:

WITH RECURSIVE reporting_chain AS (
    SELECT 
        emp_id,
        first_name,
        manager_id,
        1 AS level,
        ARRAY[emp_id] AS visited_ids  -- Track visited nodes
    FROM employees
    WHERE emp_id = 42
    
    UNION ALL
    
    SELECT 
        e.emp_id,
        e.first_name,
        e.manager_id,
        rc.level + 1,
        rc.visited_ids || e.emp_id
    FROM employees e
    JOIN reporting_chain rc ON e.emp_id = rc.manager_id
    WHERE NOT (e.emp_id = ANY(rc.visited_ids))  -- Prevent revisiting
)
SELECT * FROM reporting_chain;
```

**Why This Matters:** Hierarchical data is common (org charts, category trees, bill of materials). Recursive CTEs are the standard SQL approach, and demonstrating proper cycle prevention shows production-ready thinking.

**Follow-up Questions:**
* How would you find the common ancestor of two employees?
* How would you calculate the total salary cost for a manager and all their reports?
* What if the hierarchy is stored in a closure table instead?

---

## Key Takeaways

* **Scalar subqueries** return single values - use for simple lookups
* **EXISTS is often faster than IN** - short-circuits on first match
* **Correlated subqueries can be slow** - often better rewritten as JOINs or window functions
* **CTEs improve readability** - use for complex multi-step logic
* **Recursive CTEs handle hierarchies** - essential for tree structures
* **UNION removes duplicates** - UNION ALL is faster when duplicates are acceptable
* **LATERAL joins** enable correlated derived tables (PostgreSQL)
* **Window functions often replace** correlated subqueries more efficiently
* **Materialized CTEs** can improve performance when referenced multiple times
* **Always consider cycle prevention** in recursive queries on real-world data

---

## 3.3 Window Functions & Analytics

Window functions perform calculations across rows related to the current row without collapsing results like GROUP BY. They're essential for rankings, running totals, moving averages, and advanced analytics.

### Window Function Basics

**Syntax:**

```sql
function_name([arguments]) OVER (
    [PARTITION BY partition_expression]
    [ORDER BY sort_expression [ASC|DESC]]
    [ROWS|RANGE frame_specification]
)
```

**Key concepts:**
- **Window**: Set of rows related to current row
- **Partition**: Divides result set into groups
- **Frame**: Subset of partition for calculation
- **No row collapsing**: Returns one row per input row (unlike GROUP BY)

### Aggregate Window Functions

```sql
-- Running total
SELECT 
    order_id,
    order_date,
    total_amount,
    SUM(total_amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Output:
-- order_id | order_date | total_amount | running_total
-- 1        | 2024-01-01 | 100          | 100
-- 2        | 2024-01-02 | 150          | 250
-- 3        | 2024-01-03 | 200          | 450

-- Average by partition
SELECT 
    emp_id,
    department_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary,
    salary - AVG(salary) OVER (PARTITION BY department_id) AS diff_from_avg
FROM employees;

-- All aggregate functions work as window functions:
-- SUM, AVG, COUNT, MIN, MAX, STDDEV, VARIANCE

-- Department statistics
SELECT 
    emp_id,
    first_name,
    department_id,
    salary,
    COUNT(*) OVER (PARTITION BY department_id) AS dept_employee_count,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary,
    MIN(salary) OVER (PARTITION BY department_id) AS dept_min_salary,
    MAX(salary) OVER (PARTITION BY department_id) AS dept_max_salary
FROM employees;
```

### Ranking Functions

#### ROW_NUMBER()

```sql
-- Unique sequential number for each row
SELECT 
    product_id,
    product_name,
    category,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) AS overall_rank,
    ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS category_rank
FROM products;

-- Output:
-- product_id | product_name | category      | price | overall_rank | category_rank
-- 101        | Laptop Pro   | Electronics   | 1500  | 1            | 1
-- 102        | Phone X      | Electronics   | 1200  | 2            | 2
-- 201        | Desk Chair   | Furniture     | 800   | 3            | 1
-- 103        | Tablet       | Electronics   | 600   | 4            | 3

-- Use case: Top N per group
WITH ranked_products AS (
    SELECT 
        product_id,
        product_name,
        category,
        price,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS rn
    FROM products
)
SELECT product_id, product_name, category, price
FROM ranked_products
WHERE rn <= 3;  -- Top 3 products per category
```

#### RANK()

```sql
-- Rank with gaps for ties
SELECT 
    emp_id,
    first_name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees;

-- If two employees have same salary:
-- emp_id | first_name | salary | rank
-- 1      | Alice      | 100000 | 1
-- 2      | Bob        | 100000 | 1  ← Same rank (tie)
-- 3      | Carol      | 95000  | 3  ← Gap in ranking (no rank 2)
-- 4      | Dave       | 90000  | 4
```

#### DENSE_RANK()

```sql
-- Rank without gaps for ties
SELECT 
    emp_id,
    first_name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- If two employees have same salary:
-- emp_id | first_name | salary | dense_rank
-- 1      | Alice      | 100000 | 1
-- 2      | Bob        | 100000 | 1  ← Same rank (tie)
-- 3      | Carol      | 95000  | 2  ← No gap (consecutive)
-- 4      | Dave       | 90000  | 3
```

#### NTILE()

```sql
-- Divide rows into N buckets
SELECT 
    emp_id,
    first_name,
    salary,
    NTILE(4) OVER (ORDER BY salary DESC) AS salary_quartile
FROM employees;

-- Divides employees into 4 equal groups by salary
-- quartile 1: Highest 25%
-- quartile 2: Next 25%
-- quartile 3: Next 25%
-- quartile 4: Lowest 25%

-- Use case: Identify top/bottom performers
SELECT 
    emp_id,
    first_name,
    salary,
    CASE NTILE(10) OVER (ORDER BY salary DESC)
        WHEN 1 THEN 'Top 10%'
        WHEN 10 THEN 'Bottom 10%'
        ELSE 'Middle 80%'
    END AS performance_group
FROM employees;
```

**Ranking comparison:**

```sql
-- All ranking functions together
SELECT 
    product_name,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) AS row_num,
    RANK() OVER (ORDER BY price DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY price DESC) AS dense_rank,
    NTILE(3) OVER (ORDER BY price DESC) AS tertile
FROM products;

-- With ties at $100:
-- product_name | price | row_num | rank | dense_rank | tertile
-- Product A    | 150   | 1       | 1    | 1          | 1
-- Product B    | 100   | 2       | 2    | 2          | 1
-- Product C    | 100   | 3       | 2    | 2          | 2
-- Product D    | 75    | 4       | 4    | 3          | 2
-- Product E    | 50    | 5       | 5    | 4          | 3
```

### Value Functions

#### LAG() and LEAD()

```sql
-- Access previous/next row values
SELECT 
    order_date,
    total_amount,
    LAG(total_amount) OVER (ORDER BY order_date) AS prev_day_amount,
    LEAD(total_amount) OVER (ORDER BY order_date) AS next_day_amount,
    total_amount - LAG(total_amount) OVER (ORDER BY order_date) AS day_over_day_change
FROM daily_sales;

-- Output:
-- order_date | total_amount | prev_day | next_day | day_over_day_change
-- 2024-01-01 | 1000         | NULL     | 1200     | NULL
-- 2024-01-02 | 1200         | 1000     | 950      | 200
-- 2024-01-03 | 950          | 1200     | 1100     | -250

-- LAG/LEAD with offset and default
SELECT 
    order_date,
    total_amount,
    LAG(total_amount, 1, 0) OVER (ORDER BY order_date) AS prev_1_day,
    LAG(total_amount, 7, 0) OVER (ORDER BY order_date) AS prev_7_days,
    LEAD(total_amount, 1, 0) OVER (ORDER BY order_date) AS next_1_day
FROM daily_sales;
-- LAG(column, offset, default_value)
-- offset: How many rows back (default 1)
-- default_value: Return this if no row exists (default NULL)

-- Percent change from previous
SELECT 
    order_date,
    total_amount,
    LAG(total_amount) OVER (ORDER BY order_date) AS prev_amount,
    ROUND(
        100.0 * (total_amount - LAG(total_amount) OVER (ORDER BY order_date)) / 
        NULLIF(LAG(total_amount) OVER (ORDER BY order_date), 0),
        2
    ) AS pct_change
FROM daily_sales;
```

#### FIRST_VALUE() and LAST_VALUE()

```sql
-- First/last value in window
SELECT 
    order_date,
    total_amount,
    FIRST_VALUE(total_amount) OVER (ORDER BY order_date) AS first_day_amount,
    LAST_VALUE(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_day_amount
FROM daily_sales;

-- ⚠️ IMPORTANT: Default frame is "RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW"
-- This makes LAST_VALUE() return current row value (not useful!)
-- Must specify frame: ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING

-- Compare each day to first and last
SELECT 
    order_date,
    total_amount,
    FIRST_VALUE(total_amount) OVER w AS first_amount,
    LAST_VALUE(total_amount) OVER w AS last_amount,
    total_amount - FIRST_VALUE(total_amount) OVER w AS change_from_first,
    total_amount - LAST_VALUE(total_amount) OVER w AS change_from_last
FROM daily_sales
WINDOW w AS (
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
);
```

#### NTH_VALUE()

```sql
-- Nth value in window
SELECT 
    order_date,
    total_amount,
    NTH_VALUE(total_amount, 1) OVER w AS first_val,
    NTH_VALUE(total_amount, 2) OVER w AS second_val,
    NTH_VALUE(total_amount, 3) OVER w AS third_val
FROM daily_sales
WINDOW w AS (
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
);

-- Use case: Compare to benchmark (e.g., 90th percentile)
WITH ordered_sales AS (
    SELECT 
        product_id,
        total_sales,
        ROW_NUMBER() OVER (ORDER BY total_sales DESC) AS rn,
        COUNT(*) OVER () AS total_count
    FROM product_sales
)
SELECT 
    product_id,
    total_sales,
    NTH_VALUE(total_sales, CAST(total_count * 0.1 AS INT)) OVER (
        ORDER BY total_sales DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS p90_sales
FROM ordered_sales;
```

### Window Frames

Window frames define the subset of partition for calculation.

**Frame types:**
- **ROWS**: Physical rows
- **RANGE**: Logical range based on ORDER BY values

**Frame bounds:**
- **UNBOUNDED PRECEDING**: From start of partition
- **N PRECEDING**: N rows/range before current
- **CURRENT ROW**: Current row
- **N FOLLOWING**: N rows/range after current
- **UNBOUNDED FOLLOWING**: To end of partition

```sql
-- Default frame (if ORDER BY specified):
-- RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

-- Default frame (if no ORDER BY):
-- ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING

-- Moving average (3-day)
SELECT 
    order_date,
    total_amount,
    AVG(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3_day
FROM daily_sales;

-- Centered moving average (3-day: prev, current, next)
SELECT 
    order_date,
    total_amount,
    AVG(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS centered_moving_avg
FROM daily_sales;

-- Running total (from start to current)
SELECT 
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM daily_sales;

-- Total over entire partition
SELECT 
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS total_all
FROM daily_sales;
```

**ROWS vs RANGE:**

```sql
-- Sample data with ties
-- date       | amount
-- 2024-01-01 | 100
-- 2024-01-02 | 200
-- 2024-01-02 | 150  ← Same date (tie)
-- 2024-01-03 | 300

-- ROWS: Physical rows (counts each row)
SELECT 
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
    ) AS sum_rows
FROM orders;
-- date       | amount | sum_rows
-- 2024-01-01 | 100    | 100       (just current)
-- 2024-01-02 | 200    | 300       (100 + 200)
-- 2024-01-02 | 150    | 350       (200 + 150)
-- 2024-01-03 | 300    | 450       (150 + 300)

-- RANGE: Logical range (includes all ties)
SELECT 
    order_date,
    total_amount,
    SUM(total_amount) OVER (
        ORDER BY order_date
        RANGE BETWEEN INTERVAL '1' DAY PRECEDING AND CURRENT ROW
    ) AS sum_range
FROM orders;
-- date       | amount | sum_range
-- 2024-01-01 | 100    | 100       (just current day)
-- 2024-01-02 | 200    | 450       (100 + 200 + 150, includes all 01-02)
-- 2024-01-02 | 150    | 450       (same, all 01-02 rows get same value)
-- 2024-01-03 | 300    | 650       (200 + 150 + 300, includes all 01-02 and 01-03)
```

### Named Windows

Avoid repetition with WINDOW clause:

```sql
-- Without named window (repetitive)
SELECT 
    emp_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id ORDER BY salary) AS avg_sal,
    MIN(salary) OVER (PARTITION BY department_id ORDER BY salary) AS min_sal,
    MAX(salary) OVER (PARTITION BY department_id ORDER BY salary) AS max_sal
FROM employees;

-- With named window (cleaner)
SELECT 
    emp_id,
    salary,
    AVG(salary) OVER w AS avg_sal,
    MIN(salary) OVER w AS min_sal,
    MAX(salary) OVER w AS max_sal
FROM employees
WINDOW w AS (PARTITION BY department_id ORDER BY salary);

-- Multiple named windows
SELECT 
    emp_id,
    salary,
    AVG(salary) OVER dept_window AS dept_avg,
    RANK() OVER salary_rank AS rank,
    LAG(salary) OVER salary_rank AS prev_salary
FROM employees
WINDOW 
    dept_window AS (PARTITION BY department_id),
    salary_rank AS (ORDER BY salary DESC);
```

### Advanced Analytics Examples

#### Year-over-Year Growth

```sql
-- YoY sales growth
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', order_date) AS month,
        SUM(total_amount) AS monthly_total
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT 
    month,
    monthly_total,
    LAG(monthly_total, 12) OVER (ORDER BY month) AS same_month_last_year,
    monthly_total - LAG(monthly_total, 12) OVER (ORDER BY month) AS yoy_change,
    ROUND(
        100.0 * (monthly_total - LAG(monthly_total, 12) OVER (ORDER BY month)) /
        NULLIF(LAG(monthly_total, 12) OVER (ORDER BY month), 0),
        2
    ) AS yoy_growth_pct
FROM monthly_sales
ORDER BY month;
```

#### Cumulative Distribution

```sql
-- Percentile calculation
SELECT 
    emp_id,
    first_name,
    salary,
    PERCENT_RANK() OVER (ORDER BY salary) AS percentile_rank,
    CUME_DIST() OVER (ORDER BY salary) AS cumulative_dist,
    ROUND(100 * PERCENT_RANK() OVER (ORDER BY salary), 2) AS percentile
FROM employees;

-- PERCENT_RANK: (rank - 1) / (total_rows - 1)
-- CUME_DIST: (rows <= current) / total_rows

-- Find employees in top 10%
SELECT *
FROM (
    SELECT 
        emp_id,
        first_name,
        salary,
        PERCENT_RANK() OVER (ORDER BY salary DESC) AS pct_rank
    FROM employees
) ranked
WHERE pct_rank <= 0.10;
```

#### Moving Average with Multiple Windows

```sql
-- 7-day, 30-day, and 90-day moving averages
SELECT 
    order_date,
    daily_revenue,
    AVG(daily_revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS ma_7_day,
    AVG(daily_revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN 29 PRECEDING AND CURRENT ROW
    ) AS ma_30_day,
    AVG(daily_revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN 89 PRECEDING AND CURRENT ROW
    ) AS ma_90_day
FROM daily_revenue;
```

#### Session/Gap Analysis

```sql
-- Identify session gaps (time between events)
SELECT 
    user_id,
    event_timestamp,
    LAG(event_timestamp) OVER (PARTITION BY user_id ORDER BY event_timestamp) AS prev_event,
    event_timestamp - LAG(event_timestamp) OVER (PARTITION BY user_id ORDER BY event_timestamp) AS time_since_last_event,
    CASE 
        WHEN event_timestamp - LAG(event_timestamp) OVER (PARTITION BY user_id ORDER BY event_timestamp) > INTERVAL '30 minutes'
        THEN 1 
        ELSE 0 
    END AS new_session
FROM user_events;

-- Assign session IDs
SELECT 
    user_id,
    event_timestamp,
    SUM(new_session) OVER (PARTITION BY user_id ORDER BY event_timestamp) AS session_id
FROM (
    SELECT 
        user_id,
        event_timestamp,
        CASE 
            WHEN event_timestamp - LAG(event_timestamp) OVER (PARTITION BY user_id ORDER BY event_timestamp) > INTERVAL '30 minutes'
                OR LAG(event_timestamp) OVER (PARTITION BY user_id ORDER BY event_timestamp) IS NULL
            THEN 1 
            ELSE 0 
        END AS new_session
    FROM user_events
) sessions;
```

#### Cohort Analysis

```sql
-- Customer retention cohort analysis
WITH first_purchase AS (
    SELECT 
        customer_id,
        MIN(DATE_TRUNC('month', order_date)) AS cohort_month
    FROM orders
    GROUP BY customer_id
),
purchase_months AS (
    SELECT DISTINCT
        customer_id,
        DATE_TRUNC('month', order_date) AS purchase_month
    FROM orders
)
SELECT 
    fp.cohort_month,
    pm.purchase_month,
    EXTRACT(YEAR FROM AGE(pm.purchase_month, fp.cohort_month)) * 12 +
    EXTRACT(MONTH FROM AGE(pm.purchase_month, fp.cohort_month)) AS months_since_first,
    COUNT(DISTINCT pm.customer_id) AS customers
FROM first_purchase fp
JOIN purchase_months pm ON fp.customer_id = pm.customer_id
GROUP BY fp.cohort_month, pm.purchase_month
ORDER BY fp.cohort_month, pm.purchase_month;
```

#### Running Balance

```sql
-- Account balance over time
SELECT 
    transaction_id,
    transaction_date,
    amount,
    transaction_type,
    SUM(
        CASE 
            WHEN transaction_type = 'credit' THEN amount
            WHEN transaction_type = 'debit' THEN -amount
        END
    ) OVER (ORDER BY transaction_date, transaction_id) AS running_balance
FROM transactions
ORDER BY transaction_date, transaction_id;
```

### Filter with Window Functions

```sql
-- ❌ Cannot use window function directly in WHERE
SELECT 
    emp_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg
FROM employees
WHERE salary > AVG(salary) OVER (PARTITION BY department_id);  -- ERROR!

-- ✅ Use subquery or CTE
WITH emp_with_avg AS (
    SELECT 
        emp_id,
        salary,
        department_id,
        AVG(salary) OVER (PARTITION BY department_id) AS dept_avg
    FROM employees
)
SELECT *
FROM emp_with_avg
WHERE salary > dept_avg;

-- ✅ Or use QUALIFY (PostgreSQL 13+, MySQL doesn't support)
-- PostgreSQL:
SELECT 
    emp_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg
FROM employees
QUALIFY salary > AVG(salary) OVER (PARTITION BY department_id);
```

---

## Frequently Asked Questions

**Q1: What's the difference between window functions and GROUP BY?**

**A:**

**GROUP BY collapses rows:**

```sql
-- GROUP BY: Returns one row per department
SELECT 
    department_id,
    AVG(salary) AS avg_salary,
    COUNT(*) AS employee_count
FROM employees
GROUP BY department_id;

-- Output: 3 rows (one per department)
-- department_id | avg_salary | employee_count
-- 1             | 75000      | 10
-- 2             | 80000      | 15
-- 3             | 70000      | 8
```

**Window functions preserve rows:**

```sql
-- Window function: Returns one row per employee
SELECT 
    emp_id,
    first_name,
    department_id,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg_salary,
    COUNT(*) OVER (PARTITION BY department_id) AS dept_employee_count
FROM employees;

-- Output: 33 rows (one per employee)
-- emp_id | first_name | department_id | salary | dept_avg | dept_count
-- 1      | Alice      | 1             | 80000  | 75000    | 10
-- 2      | Bob        | 1             | 70000  | 75000    | 10
-- ...    | ...        | 1             | ...    | 75000    | 10
-- 11     | Carol      | 2             | 85000  | 80000    | 15
```

**Key differences:**

| Aspect | GROUP BY | Window Function |
|--------|----------|-----------------|
| **Rows returned** | One per group | One per input row |
| **Use with non-aggregated columns** | No (must be in GROUP BY) | Yes |
| **Can use in WHERE** | Via HAVING | No (use subquery) |
| **Can combine with other columns** | Limited | Full flexibility |
| **Partitioning** | All rows grouped | Logical partitions, rows preserved |

**When to use GROUP BY:**

```sql
-- Summary reports
SELECT 
    department_id,
    AVG(salary) AS avg_salary,
    MAX(salary) AS max_salary
FROM employees
GROUP BY department_id;
```

**When to use window functions:**

```sql
-- Detail with aggregates
SELECT 
    emp_id,
    first_name,
    salary,
    AVG(salary) OVER (PARTITION BY department_id) AS dept_avg,
    salary - AVG(salary) OVER (PARTITION BY department_id) AS diff_from_avg
FROM employees;

-- Rankings
SELECT 
    emp_id,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;

-- Running totals
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
```

**Can combine both:**

```sql
-- Group first, then use window function
SELECT 
    department_id,
    total_salary,
    AVG(total_salary) OVER () AS company_avg_dept_total,
    total_salary - AVG(total_salary) OVER () AS diff_from_avg
FROM (
    SELECT 
        department_id,
        SUM(salary) AS total_salary
    FROM employees
    GROUP BY department_id
) dept_totals;
```

---

**Q2: Why does LAST_VALUE() often return unexpected results?**

**A:**

**The problem:**

```sql
-- This seems like it should return the last value in partition
SELECT 
    order_date,
    daily_sales,
    LAST_VALUE(daily_sales) OVER (ORDER BY order_date) AS last_value
FROM sales;

-- But it returns current row value! Why?
-- order_date | daily_sales | last_value
-- 2024-01-01 | 100         | 100  ← Expected last day, got current!
-- 2024-01-02 | 150         | 150  ← Expected last day, got current!
-- 2024-01-03 | 200         | 200  ← Expected last day, got current!
```

**The reason: Default frame specification**

```sql
-- When you specify ORDER BY, the default frame is:
-- RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
-- This means: from start of partition to current row

-- So LAST_VALUE returns the "last value in the frame"
-- which is the current row!

-- Equivalent query showing implicit frame:
SELECT 
    order_date,
    daily_sales,
    LAST_VALUE(daily_sales) OVER (
        ORDER BY order_date
        RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW  -- Implicit!
    ) AS last_value
FROM sales;
```

**The solution: Explicit frame**

```sql
-- ✅ Specify full partition frame
SELECT 
    order_date,
    daily_sales,
    LAST_VALUE(daily_sales) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_value
FROM sales;

-- Now it works!
-- order_date | daily_sales | last_value
-- 2024-01-01 | 100         | 200  ← Correct!
-- 2024-01-02 | 150         | 200  ← Correct!
-- 2024-01-03 | 200         | 200  ← Correct!
```

**Frame defaults summary:**

```sql
-- No ORDER BY: Frame is entire partition
OVER (PARTITION BY dept)
-- Implicit: ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING

-- With ORDER BY: Frame is from start to current
OVER (ORDER BY date)
-- Implicit: RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

-- With ORDER BY and explicit frame: As specified
OVER (ORDER BY date ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)
-- Explicit: Exactly as written
```

**Practical implications:**

```sql
-- SUM with ORDER BY (running total)
SELECT 
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;
-- Works as expected (cumulative sum) due to default frame

-- LAST_VALUE with ORDER BY (last in partition)
SELECT 
    order_date,
    amount,
    LAST_VALUE(amount) OVER (ORDER BY order_date) AS last_amount
FROM orders;
-- Doesn't work as expected! Returns current row

-- ✅ Fix: Always specify frame for LAST_VALUE
SELECT 
    order_date,
    amount,
    LAST_VALUE(amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_amount
FROM orders;
```

**Best practice:**

```sql
-- Be explicit with frames for LAST_VALUE, FIRST_VALUE, NTH_VALUE
-- to avoid confusion

-- Bad (implicit frame)
LAST_VALUE(x) OVER (ORDER BY y)

-- Good (explicit frame)
LAST_VALUE(x) OVER (
    ORDER BY y
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)

-- Or use named window for readability
WINDOW w AS (
    ORDER BY y
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
```

---

**Q3: How do I calculate a moving average that handles missing dates?**

**A:**

**Problem: Date gaps in data**

```sql
-- Data has gaps (no sales on 2024-01-02)
-- date       | sales
-- 2024-01-01 | 100
-- 2024-01-03 | 150  ← Missing 2024-01-02
-- 2024-01-04 | 200

-- Naive moving average (ROWS-based)
SELECT 
    sale_date,
    sales,
    AVG(sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS ma_3_day
FROM sales;

-- Result:
-- date       | sales | ma_3_day
-- 2024-01-01 | 100   | 100      ← Only 1 value
-- 2024-01-03 | 150   | 125      ← Only 2 values (missing day not counted)
-- 2024-01-04 | 200   | 150      ← Only 3 values

-- Problem: Not a true 3-day average (gaps treated as missing, not zero)
```

**Solution 1: Generate date series and fill gaps**

```sql
-- PostgreSQL
WITH date_series AS (
    SELECT generate_series(
        DATE '2024-01-01',
        DATE '2024-01-31',
        INTERVAL '1 day'
    )::DATE AS sale_date
),
sales_with_gaps AS (
    SELECT 
        ds.sale_date,
        COALESCE(s.sales, 0) AS sales  -- Fill gaps with 0
    FROM date_series ds
    LEFT JOIN sales s ON ds.sale_date = s.sale_date
)
SELECT 
    sale_date,
    sales,
    AVG(sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS ma_3_day
FROM sales_with_gaps;

-- Result:
-- date       | sales | ma_3_day
-- 2024-01-01 | 100   | 100.00   ← (100) / 1
-- 2024-01-02 | 0     | 50.00    ← (100 + 0) / 2
-- 2024-01-03 | 150   | 83.33    ← (100 + 0 + 150) / 3
-- 2024-01-04 | 200   | 116.67   ← (0 + 150 + 200) / 3
```

**MySQL equivalent:**

```sql
-- MySQL 8.0+: Use recursive CTE for date series
WITH RECURSIVE date_series AS (
    SELECT DATE('2024-01-01') AS sale_date
    UNION ALL
    SELECT DATE_ADD(sale_date, INTERVAL 1 DAY)
    FROM date_series
    WHERE sale_date < DATE('2024-01-31')
),
sales_with_gaps AS (
    SELECT 
        ds.sale_date,
        COALESCE(s.sales, 0) AS sales
    FROM date_series ds
    LEFT JOIN sales s ON ds.sale_date = s.sale_date
)
SELECT 
    sale_date,
    sales,
    AVG(sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS ma_3_day
FROM sales_with_gaps;
```

**Solution 2: RANGE-based with interval (PostgreSQL)**

```sql
-- Use RANGE with interval instead of ROWS
SELECT 
    sale_date,
    sales,
    AVG(sales) OVER (
        ORDER BY sale_date
        RANGE BETWEEN INTERVAL '2 days' PRECEDING AND CURRENT ROW
    ) AS ma_3_day
FROM sales;

-- This includes all rows within 2 days prior
-- But doesn't fill gaps with zeros
-- date       | sales | ma_3_day
-- 2024-01-01 | 100   | 100      ← Only this row
-- 2024-01-03 | 150   | 125      ← 01-01 and 01-03 (within 2 days)
-- 2024-01-04 | 200   | 150      ← 01-03 and 01-04 (01-01 outside range)

-- Still not perfect (doesn't count missing day)
-- Best to use Solution 1 (generate complete date series)
```

**Solution 3: Custom window with date arithmetic**

```sql
-- For each row, manually calculate average over date range
SELECT 
    s1.sale_date,
    s1.sales,
    (
        SELECT AVG(COALESCE(s2.sales, 0))
        FROM generate_series(
            s1.sale_date - INTERVAL '2 days',
            s1.sale_date,
            INTERVAL '1 day'
        ) AS date_range(d)
        LEFT JOIN sales s2 ON s2.sale_date = date_range.d::DATE
    ) AS ma_3_day
FROM sales s1;

-- Accurate but slower (correlated subquery for each row)
```

**Best practice:**

```sql
-- 1. Generate complete date series (most accurate)
-- 2. Fill gaps with appropriate values (0, NULL, or carry forward)
-- 3. Then apply window function

-- Complete solution with gap filling
WITH RECURSIVE all_dates AS (
    SELECT MIN(sale_date) AS d FROM sales
    UNION ALL
    SELECT d + INTERVAL '1 day'
    FROM all_dates
    WHERE d < (SELECT MAX(sale_date) FROM sales)
),
sales_filled AS (
    SELECT 
        ad.d AS sale_date,
        COALESCE(s.sales, 0) AS sales,
        s.sales IS NOT NULL AS has_data
    FROM all_dates ad
    LEFT JOIN sales s ON ad.d = s.sale_date
)
SELECT 
    sale_date,
    sales,
    has_data,
    AVG(sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS ma_3_day,
    -- Also show count of days with actual data
    COUNT(CASE WHEN has_data THEN 1 END) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS days_with_data
FROM sales_filled;
```

---

## Interview Questions

**Question 1: Write a query to find the top 3 highest-paid employees in each department, including their rank within the department.**

**Difficulty:** Mid-Level

**Answer:**

**Solution 1: Using ROW_NUMBER()**

```sql
WITH ranked_employees AS (
    SELECT 
        emp_id,
        first_name,
        last_name,
        department_id,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT 
    emp_id,
    first_name,
    last_name,
    department_id,
    salary,
    rank
FROM ranked_employees
WHERE rank <= 3
ORDER BY department_id, rank;
```

**Solution 2: Using RANK() (handles ties differently)**

```sql
-- RANK() gives same rank to ties, with gaps
WITH ranked_employees AS (
    SELECT 
        emp_id,
        first_name,
        last_name,
        department_id,
        salary,
        RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT 
    emp_id,
    first_name,
    last_name,
    department_id,
    salary,
    rank
FROM ranked_employees
WHERE rank <= 3
ORDER BY department_id, rank;

-- If two employees tied for 2nd:
-- rank 1: Employee A
-- rank 2: Employee B (tie)
-- rank 2: Employee C (tie)
-- rank 4: Employee D
-- This query returns 4 employees (includes both rank 2)
```

**Solution 3: Using DENSE_RANK() (no gaps)**

```sql
-- DENSE_RANK() gives same rank to ties, no gaps
WITH ranked_employees AS (
    SELECT 
        emp_id,
        first_name,
        last_name,
        department_id,
        salary,
        DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dense_rank
    FROM employees
)
SELECT 
    emp_id,
    first_name,
    last_name,
    department_id,
    salary,
    dense_rank
FROM ranked_employees
WHERE dense_rank <= 3
ORDER BY department_id, dense_rank;

-- If two employees tied for 2nd:
-- rank 1: Employee A
-- rank 2: Employee B (tie)
-- rank 2: Employee C (tie)
-- rank 3: Employee D (no gap)
-- This query returns all 4 (ranks 1, 2, 2, 3)
```

**Comparison of approaches:**

```sql
-- Sample data:
-- dept | emp    | salary
-- 1    | Alice  | 100000
-- 1    | Bob    | 90000
-- 1    | Carol  | 90000  ← Tied with Bob
-- 1    | Dave   | 85000
-- 1    | Eve    | 80000

-- ROW_NUMBER() - Arbitrary ordering of ties:
-- rank 1: Alice (100000)
-- rank 2: Bob (90000)
-- rank 3: Carol (90000)  ← Only 3 returned

-- RANK() - Same rank for ties, with gaps:
-- rank 1: Alice (100000)
-- rank 2: Bob (90000)
-- rank 2: Carol (90000)
-- rank 4: Dave (85000)  ← Skips rank 3
-- Returns 3 employees (Alice, Bob, Carol)

-- DENSE_RANK() - Same rank for ties, no gaps:
-- rank 1: Alice (100000)
-- rank 2: Bob (90000)
-- rank 2: Carol (90000)
-- rank 3: Dave (85000)
-- Returns 4 employees (Alice, Bob, Carol, Dave)
```

**Which to use?**

```sql
-- ROW_NUMBER(): 
-- - Exactly 3 employees per department
-- - Ties broken arbitrarily (may not be fair)
-- Use when: You need exactly N results

-- RANK():
-- - May return more than 3 if ties exist
-- - Gaps in ranking
-- Use when: Ties should share rank, gaps acceptable

-- DENSE_RANK():
-- - May return more than 3 if ties exist
-- - No gaps in ranking
-- Use when: Ties should share rank, want continuous ranks
```

**Why This Matters:** Understanding ranking function differences is crucial for correct query results. Choosing wrong function can exclude tied employees or return too many results.

**Follow-up Questions:**
* How would you include the department name in results?
* What if you need exactly 3 employees even with ties (how to break ties deterministically)?
* How would you find top 10% instead of top 3?

---

**Question 2: Given a table of daily temperatures, write a query to calculate the 7-day moving average and identify days where the temperature was above this average.**

**Difficulty:** Senior

**Answer:**

**Complete solution:**

```sql
WITH daily_temps_with_ma AS (
    SELECT 
        measure_date,
        temperature,
        AVG(temperature) OVER (
            ORDER BY measure_date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) AS moving_avg_7_day,
        COUNT(*) OVER (
            ORDER BY measure_date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) AS days_in_average
    FROM temperature_readings
)
SELECT 
    measure_date,
    temperature,
    ROUND(moving_avg_7_day, 2) AS moving_avg_7_day,
    days_in_average,
    CASE 
        WHEN temperature > moving_avg_7_day THEN 'Above Average'
        WHEN temperature < moving_avg_7_day THEN 'Below Average'
        ELSE 'At Average'
    END AS comparison,
    ROUND(temperature - moving_avg_7_day, 2) AS deviation
FROM daily_temps_with_ma
ORDER BY measure_date;
```

**Output example:**

```
measure_date | temperature | moving_avg_7_day | days_in_average | comparison     | deviation
2024-01-01   | 32.5        | 32.50            | 1               | At Average     | 0.00
2024-01-02   | 34.2        | 33.35            | 2               | Above Average  | 0.85
2024-01-03   | 31.8        | 32.83            | 3               | Below Average  | -1.03
2024-01-07   | 35.6        | 33.42            | 7               | Above Average  | 2.18
2024-01-08   | 36.1        | 34.15            | 7               | Above Average  | 1.95
```

**Enhanced version: Handle missing dates**

```sql
WITH RECURSIVE date_series AS (
    -- Generate all dates
    SELECT MIN(measure_date) AS d FROM temperature_readings
    UNION ALL
    SELECT d + INTERVAL '1 day'
    FROM date_series
    WHERE d < (SELECT MAX(measure_date) FROM temperature_readings)
),
temps_filled AS (
    -- Fill missing dates
    SELECT 
        ds.d AS measure_date,
        tr.temperature,
        tr.temperature IS NOT NULL AS has_reading
    FROM date_series ds
    LEFT JOIN temperature_readings tr ON ds.d = tr.measure_date
),
temps_with_ma AS (
    SELECT 
        measure_date,
        temperature,
        has_reading,
        AVG(temperature) OVER (
            ORDER BY measure_date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) AS moving_avg_7_day,
        COUNT(temperature) OVER (
            ORDER BY measure_date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ) AS readings_in_average
    FROM temps_filled
)
SELECT 
    measure_date,
    temperature,
    has_reading,
    ROUND(moving_avg_7_day, 2) AS moving_avg_7_day,
    readings_in_average,
    CASE 
        WHEN has_reading AND temperature > moving_avg_7_day THEN 'Above'
        WHEN has_reading AND temperature < moving_avg_7_day THEN 'Below'
        WHEN has_reading THEN 'At Average'
        ELSE 'No Reading'
    END AS comparison
FROM temps_with_ma
ORDER BY measure_date;
```

**Advanced version: Multiple moving averages and trend detection**

```sql
WITH temps_with_stats AS (
    SELECT 
        measure_date,
        temperature,
        -- Moving averages
        AVG(temperature) OVER w7 AS ma_7_day,
        AVG(temperature) OVER w30 AS ma_30_day,
        -- Standard deviation
        STDDEV(temperature) OVER w7 AS stddev_7_day,
        -- Previous day
        LAG(temperature) OVER (ORDER BY measure_date) AS prev_day_temp,
        -- Trend (7-day MA slope)
        AVG(temperature) OVER w7 - 
        LAG(AVG(temperature) OVER w7) OVER (ORDER BY measure_date) AS ma_7_day_trend
    FROM temperature_readings
    WINDOW 
        w7 AS (ORDER BY measure_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW),
        w30 AS (ORDER BY measure_date ROWS BETWEEN 29 PRECEDING AND CURRENT ROW)
)
SELECT 
    measure_date,
    temperature,
    ROUND(ma_7_day, 2) AS ma_7_day,
    ROUND(ma_30_day, 2) AS ma_30_day,
    ROUND(stddev_7_day, 2) AS stddev_7_day,
    ROUND(temperature - ma_7_day, 2) AS deviation_from_ma,
    ROUND((temperature - ma_7_day) / NULLIF(stddev_7_day, 0), 2) AS z_score,
    CASE 
        WHEN temperature > ma_7_day + (2 * stddev_7_day) THEN 'Unusually Hot'
        WHEN temperature > ma_7_day THEN 'Above Average'
        WHEN temperature < ma_7_day - (2 * stddev_7_day) THEN 'Unusually Cold'
        WHEN temperature < ma_7_day THEN 'Below Average'
        ELSE 'Average'
    END AS classification,
    CASE 
        WHEN ma_7_day_trend > 0.5 THEN 'Warming Trend'
        WHEN ma_7_day_trend < -0.5 THEN 'Cooling Trend'
        ELSE 'Stable'
    END AS trend
FROM temps_with_stats
WHERE measure_date >= (SELECT MAX(measure_date) - INTERVAL '30 days' FROM temperature_readings)
ORDER BY measure_date;
```

**Optimizations:**

```sql
-- For large datasets, consider:
-- 1. Index on measure_date
CREATE INDEX idx_temp_date ON temperature_readings(measure_date);

-- 2. Materialized view for historical data
CREATE MATERIALIZED VIEW temperature_with_ma AS
SELECT 
    measure_date,
    temperature,
    AVG(temperature) OVER (
        ORDER BY measure_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS ma_7_day
FROM temperature_readings;

-- Refresh daily
REFRESH MATERIALIZED VIEW temperature_with_ma;

-- 3. Partitioning by date range (for very large tables)
-- PostgreSQL 10+
CREATE TABLE temperature_readings (
    measure_date DATE NOT NULL,
    temperature NUMERIC(5,2)
) PARTITION BY RANGE (measure_date);

CREATE TABLE temp_2023 PARTITION OF temperature_readings
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');
    
CREATE TABLE temp_2024 PARTITION OF temperature_readings
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

**Why This Matters:** Moving averages are fundamental to time series analysis. This demonstrates understanding of window frames, gap handling, and statistical concepts. Production systems need to handle missing data gracefully.

**Follow-up Questions:**
* How would you optimize this for a table with billions of rows?
* What if you need weighted moving average (recent days weighted more)?
* How would you detect anomalies (outliers) in the temperature data?

---

## Key Takeaways

* **Window functions preserve rows** - unlike GROUP BY which collapses them
* **PARTITION BY divides data** - creates independent windows per partition
* **ORDER BY determines calculation sequence** - affects running totals, LAG/LEAD
* **Frame specification critical** - especially for LAST_VALUE and moving calculations
* **Ranking functions handle ties differently** - ROW_NUMBER, RANK, DENSE_RANK
* **LAG/LEAD access adjacent rows** - perfect for time series analysis
* **Named windows reduce repetition** - use WINDOW clause for cleaner SQL
* **Cannot filter window functions in WHERE** - use subquery or CTE
* **Default frames can surprise** - be explicit with ROWS/RANGE for clarity
* **Window functions enable advanced analytics** - cohorts, trends, moving averages

---

## 3.4 Advanced Aggregation & Grouping

Beyond basic GROUP BY, SQL offers powerful aggregation features for multi-dimensional analysis and complex grouping scenarios.

### GROUPING SETS

Generate multiple grouping levels in single query:

```sql
-- Traditional approach (multiple queries with UNION)
SELECT department_id, NULL AS job_title, SUM(salary) AS total_salary
FROM employees
GROUP BY department_id
UNION ALL
SELECT NULL, job_title, SUM(salary)
FROM employees
GROUP BY job_title
UNION ALL
SELECT department_id, job_title, SUM(salary)
FROM employees
GROUP BY department_id, job_title;

-- ✅ Better: GROUPING SETS (single query)
SELECT 
    department_id,
    job_title,
    SUM(salary) AS total_salary,
    COUNT(*) AS employee_count
FROM employees
GROUP BY GROUPING SETS (
    (department_id),           -- By department
    (job_title),               -- By job title
    (department_id, job_title) -- By both
);

-- Result includes all three grouping levels:
-- dept_id | job_title | total_salary | employee_count
-- 1       | NULL      | 500000       | 10  ← Department total
-- NULL    | Engineer  | 300000       | 6   ← Job title total
-- 1       | Engineer  | 150000       | 3   ← Both
```

**GROUPING() function - identifies which columns are aggregated:**

```sql
SELECT 
    department_id,
    job_title,
    SUM(salary) AS total_salary,
    GROUPING(department_id) AS dept_grouping,
    GROUPING(job_title) AS title_grouping
FROM employees
GROUP BY GROUPING SETS (
    (department_id),
    (job_title),
    (department_id, job_title),
    ()  -- Grand total
);

-- GROUPING() returns:
-- 0: Column is in GROUP BY for this row
-- 1: Column is aggregated (NULL in this row is placeholder)

-- dept_id | job_title | total | dept_grouping | title_grouping
-- 1       | NULL      | 500K  | 0             | 1  ← Grouped by dept
-- NULL    | Engineer  | 300K  | 1             | 0  ← Grouped by title
-- 1       | Engineer  | 150K  | 0             | 0  ← Grouped by both
-- NULL    | NULL      | 800K  | 1             | 1  ← Grand total

-- Use GROUPING() to replace NULL with meaningful labels:
SELECT 
    CASE WHEN GROUPING(department_id) = 1 THEN 'All Departments' 
         ELSE CAST(department_id AS VARCHAR) 
    END AS department,
    CASE WHEN GROUPING(job_title) = 1 THEN 'All Titles'
         ELSE job_title
    END AS title,
    SUM(salary) AS total_salary
FROM employees
GROUP BY GROUPING SETS ((department_id), (job_title), ());
```

### ROLLUP

Creates hierarchical subtotals:

```sql
-- ROLLUP generates aggregates at each level of hierarchy
SELECT 
    department_id,
    job_title,
    SUM(salary) AS total_salary
FROM employees
GROUP BY ROLLUP(department_id, job_title);

-- Equivalent to GROUPING SETS:
GROUP BY GROUPING SETS (
    (department_id, job_title),  -- Most detailed
    (department_id),              -- Subtotal by department
    ()                            -- Grand total
);

-- Result:
-- dept_id | job_title | total_salary
-- 1       | Engineer  | 150000  ← Dept 1, Engineers
-- 1       | Manager   | 200000  ← Dept 1, Managers
-- 1       | NULL      | 350000  ← Dept 1 subtotal
-- 2       | Engineer  | 180000  ← Dept 2, Engineers
-- 2       | NULL      | 180000  ← Dept 2 subtotal
-- NULL    | NULL      | 530000  ← Grand total

-- Multi-level ROLLUP (sales reporting)
SELECT 
    year,
    quarter,
    month,
    SUM(sales) AS total_sales
FROM sales_data
GROUP BY ROLLUP(year, quarter, month)
ORDER BY year, quarter, month;

-- Generates: year/quarter/month, year/quarter, year, grand total
```

### CUBE

All possible combinations of grouping columns:

```sql
-- CUBE generates all possible grouping combinations
SELECT 
    department_id,
    job_title,
    SUM(salary) AS total_salary
FROM employees
GROUP BY CUBE(department_id, job_title);

-- Equivalent to GROUPING SETS:
GROUP BY GROUPING SETS (
    (department_id, job_title),  -- Both
    (department_id),              -- Dept only
    (job_title),                  -- Title only
    ()                            -- Grand total
);

-- Result includes all dimension combinations:
-- dept_id | job_title | total_salary
-- 1       | Engineer  | 150000  ← Dept 1, Engineers
-- 1       | Manager   | 200000  ← Dept 1, Managers
-- 2       | Engineer  | 180000  ← Dept 2, Engineers
-- 1       | NULL      | 350000  ← Dept 1 total
-- 2       | NULL      | 180000  ← Dept 2 total
-- NULL    | Engineer  | 330000  ← All Engineers
-- NULL    | Manager   | 200000  ← All Managers
-- NULL    | NULL      | 530000  ← Grand total

-- 3-column CUBE generates 2^3 = 8 grouping sets
SELECT 
    region,
    product,
    quarter,
    SUM(sales)
FROM sales
GROUP BY CUBE(region, product, quarter);
-- Generates all 8 combinations
```

### FILTER Clause (PostgreSQL, Modern SQL)

Apply WHERE to specific aggregates:

```sql
-- Traditional approach (CASE inside aggregate)
SELECT 
    department_id,
    COUNT(*) AS total_employees,
    COUNT(CASE WHEN salary > 100000 THEN 1 END) AS high_earners,
    COUNT(CASE WHEN status = 'active' THEN 1 END) AS active_employees,
    AVG(CASE WHEN status = 'active' THEN salary END) AS avg_active_salary
FROM employees
GROUP BY department_id;

-- ✅ Better: FILTER clause (PostgreSQL 9.4+)
SELECT 
    department_id,
    COUNT(*) AS total_employees,
    COUNT(*) FILTER (WHERE salary > 100000) AS high_earners,
    COUNT(*) FILTER (WHERE status = 'active') AS active_employees,
    AVG(salary) FILTER (WHERE status = 'active') AS avg_active_salary,
    SUM(salary) FILTER (WHERE hire_date >= '2024-01-01') AS new_hire_total
FROM employees
GROUP BY department_id;

-- More complex example: Multiple metrics
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*) AS total_orders,
    COUNT(*) FILTER (WHERE total_amount > 1000) AS large_orders,
    COUNT(*) FILTER (WHERE status = 'completed') AS completed_orders,
    COUNT(*) FILTER (WHERE status = 'cancelled') AS cancelled_orders,
    AVG(total_amount) FILTER (WHERE status = 'completed') AS avg_completed_amount,
    SUM(total_amount) FILTER (WHERE status = 'completed') AS revenue,
    SUM(total_amount) FILTER (WHERE status = 'refunded') AS refunds
FROM orders
GROUP BY DATE_TRUNC('month', order_date);
```

### Conditional Aggregation

```sql
-- Pivot-style aggregation
SELECT 
    department_id,
    COUNT(*) AS total,
    COUNT(*) FILTER (WHERE job_title = 'Engineer') AS engineers,
    COUNT(*) FILTER (WHERE job_title = 'Manager') AS managers,
    COUNT(*) FILTER (WHERE job_title = 'Designer') AS designers,
    AVG(salary) FILTER (WHERE job_title = 'Engineer') AS avg_eng_salary,
    AVG(salary) FILTER (WHERE job_title = 'Manager') AS avg_mgr_salary
FROM employees
GROUP BY department_id;

-- MySQL equivalent (no FILTER clause)
SELECT 
    department_id,
    COUNT(*) AS total,
    SUM(CASE WHEN job_title = 'Engineer' THEN 1 ELSE 0 END) AS engineers,
    SUM(CASE WHEN job_title = 'Manager' THEN 1 ELSE 0 END) AS managers,
    AVG(CASE WHEN job_title = 'Engineer' THEN salary END) AS avg_eng_salary
FROM employees
GROUP BY department_id;
```

### HAVING vs WHERE

```sql
-- WHERE: Filters before grouping
-- HAVING: Filters after grouping

-- Find departments with >10 employees earning >50K on average
SELECT 
    department_id,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary
FROM employees
WHERE status = 'active'  -- ✓ WHERE: Filter individual rows
GROUP BY department_id
HAVING COUNT(*) > 10     -- ✓ HAVING: Filter groups
   AND AVG(salary) > 50000;

-- Execution order:
-- 1. WHERE filters individual rows (status = 'active')
-- 2. GROUP BY groups remaining rows
-- 3. Aggregates calculated (COUNT, AVG)
-- 4. HAVING filters groups (>10 employees, avg > 50K)

-- ❌ Cannot use aggregate in WHERE
SELECT department_id, AVG(salary)
FROM employees
WHERE AVG(salary) > 50000  -- ERROR!
GROUP BY department_id;

-- ✅ Use HAVING for aggregate conditions
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 50000;  -- Correct
```

### Statistical Aggregates

```sql
-- Beyond SUM/AVG/COUNT
SELECT 
    department_id,
    COUNT(*) AS n,
    AVG(salary) AS mean,
    STDDEV(salary) AS std_dev,
    VARIANCE(salary) AS variance,
    MIN(salary) AS min,
    MAX(salary) AS max,
    MAX(salary) - MIN(salary) AS range,
    -- Percentiles (PostgreSQL)
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY salary) AS p25,
    PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY salary) AS median,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY salary) AS p75,
    PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY salary) AS p90
FROM employees
GROUP BY department_id;

-- Mode (most common value)
SELECT 
    department_id,
    MODE() WITHIN GROUP (ORDER BY job_title) AS most_common_title
FROM employees
GROUP BY department_id;

-- String aggregation (combine values into string)
-- PostgreSQL:
SELECT 
    department_id,
    STRING_AGG(first_name, ', ' ORDER BY first_name) AS employee_names
FROM employees
GROUP BY department_id;

-- MySQL:
SELECT 
    department_id,
    GROUP_CONCAT(first_name ORDER BY first_name SEPARATOR ', ') AS employee_names
FROM employees
GROUP BY department_id;
```

---

## 3.5 Set Operations & Data Manipulation

Set operations combine results from multiple queries. Understanding their behavior is essential for complex data analysis.

### UNION and UNION ALL

```sql
-- UNION removes duplicates
SELECT customer_id, email FROM customers WHERE country = 'USA'
UNION
SELECT customer_id, email FROM customers WHERE status = 'VIP';

-- UNION ALL keeps duplicates (faster)
SELECT customer_id FROM orders WHERE order_date >= '2024-01-01'
UNION ALL
SELECT customer_id FROM orders WHERE order_date >= '2024-06-01';

-- Requirements:
-- - Same number of columns
-- - Compatible data types
-- - Column names from first SELECT

-- Performance comparison:
-- UNION: Slower (must deduplicate with sort/hash)
-- UNION ALL: Faster (simple append)

-- Use UNION when: Duplicates must be removed
-- Use UNION ALL when: Duplicates OK or impossible
```

### INTERSECT

```sql
-- Rows in both result sets
SELECT customer_id FROM customers WHERE country = 'USA'
INTERSECT
SELECT customer_id FROM orders WHERE order_date >= '2024-01-01';
-- Returns: USA customers who ordered in 2024

-- MySQL equivalent (no INTERSECT until 8.0.31):
SELECT DISTINCT c.customer_id
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id
WHERE c.country = 'USA' AND o.order_date >= '2024-01-01';

-- Or using EXISTS:
SELECT customer_id FROM customers WHERE country = 'USA'
AND customer_id IN (SELECT customer_id FROM orders WHERE order_date >= '2024-01-01');
```

### EXCEPT (PostgreSQL) / MINUS (Oracle)

```sql
-- Rows in first result set but not in second
SELECT customer_id FROM customers WHERE country = 'USA'
EXCEPT
SELECT customer_id FROM orders WHERE order_date >= '2024-01-01';
-- Returns: USA customers who did NOT order in 2024

-- MySQL equivalent:
SELECT c.customer_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id AND o.order_date >= '2024-01-01'
WHERE c.country = 'USA' AND o.order_id IS NULL;

-- Or using NOT EXISTS:
SELECT customer_id FROM customers WHERE country = 'USA'
AND customer_id NOT IN (SELECT customer_id FROM orders WHERE order_date >= '2024-01-01');
```

### INSERT Variations

```sql
-- Basic INSERT
INSERT INTO customers (customer_id, name, email)
VALUES (1, 'Alice', 'alice@example.com');

-- Multiple rows
INSERT INTO customers (customer_id, name, email)
VALUES 
    (2, 'Bob', 'bob@example.com'),
    (3, 'Carol', 'carol@example.com'),
    (4, 'Dave', 'dave@example.com');

-- INSERT from SELECT
INSERT INTO archived_orders (order_id, customer_id, total_amount, order_date)
SELECT order_id, customer_id, total_amount, order_date
FROM orders
WHERE order_date < '2023-01-01';

-- INSERT with DEFAULT values
INSERT INTO products (product_name, created_at)
VALUES ('New Product', DEFAULT);  -- Uses column default

-- UPSERT (INSERT or UPDATE)
-- PostgreSQL: INSERT ... ON CONFLICT
INSERT INTO product_inventory (product_id, quantity)
VALUES (101, 50)
ON CONFLICT (product_id)
DO UPDATE SET quantity = product_inventory.quantity + EXCLUDED.quantity;

-- MySQL: INSERT ... ON DUPLICATE KEY UPDATE
INSERT INTO product_inventory (product_id, quantity)
VALUES (101, 50)
ON DUPLICATE KEY UPDATE quantity = quantity + VALUES(quantity);

-- MySQL modern: INSERT ... AS ... ON DUPLICATE
INSERT INTO product_inventory (product_id, quantity)
VALUES (101, 50) AS new
ON DUPLICATE KEY UPDATE quantity = quantity + new.quantity;
```

### UPDATE Patterns

```sql
-- Basic UPDATE
UPDATE employees
SET salary = salary * 1.1
WHERE department_id = 5;

-- UPDATE with JOIN (PostgreSQL)
UPDATE employees e
SET salary = salary * 1.1
FROM departments d
WHERE e.department_id = d.dept_id
  AND d.dept_name = 'Engineering';

-- MySQL UPDATE with JOIN
UPDATE employees e
INNER JOIN departments d ON e.department_id = d.dept_id
SET e.salary = e.salary * 1.1
WHERE d.dept_name = 'Engineering';

-- UPDATE with subquery
UPDATE products
SET category_id = (
    SELECT category_id 
    FROM categories 
    WHERE category_name = 'Electronics'
)
WHERE product_type = 'Gadget';

-- Conditional UPDATE (CASE)
UPDATE employees
SET bonus = CASE 
    WHEN performance_rating >= 4.5 THEN salary * 0.15
    WHEN performance_rating >= 3.5 THEN salary * 0.10
    WHEN performance_rating >= 2.5 THEN salary * 0.05
    ELSE 0
END;
```

### DELETE Patterns

```sql
-- Basic DELETE
DELETE FROM orders WHERE order_date < '2020-01-01';

-- DELETE with JOIN (PostgreSQL)
DELETE FROM orders o
USING customers c
WHERE o.customer_id = c.customer_id
  AND c.status = 'inactive';

-- MySQL DELETE with JOIN
DELETE o
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
WHERE c.status = 'inactive';

-- DELETE with subquery
DELETE FROM products
WHERE category_id IN (
    SELECT category_id FROM categories WHERE status = 'discontinued'
);

-- TRUNCATE (faster for deleting all rows)
TRUNCATE TABLE temp_data;  -- Faster than DELETE, resets auto-increment
```

### MERGE (PostgreSQL 15+) / Replace Pattern

```sql
-- PostgreSQL MERGE (15+)
MERGE INTO product_inventory t
USING product_updates s ON t.product_id = s.product_id
WHEN MATCHED THEN
    UPDATE SET quantity = t.quantity + s.quantity_change
WHEN NOT MATCHED THEN
    INSERT (product_id, quantity)
    VALUES (s.product_id, s.quantity_change);

-- MySQL pattern (before MERGE support)
INSERT INTO product_inventory (product_id, quantity)
SELECT product_id, quantity_change FROM product_updates
ON DUPLICATE KEY UPDATE 
    quantity = product_inventory.quantity + VALUES(quantity);
```

---

## Frequently Asked Questions

**Q1: When should I use GROUPING SETS vs multiple UNION queries?**

**A:** 

**GROUPING SETS advantages:**
- Single table scan (more efficient)
- Cleaner syntax
- Easier maintenance

**Example comparison:**

```sql
-- ❌ Multiple UNIONs: 3 table scans
SELECT department_id, NULL AS location, SUM(salary) FROM employees GROUP BY department_id
UNION ALL
SELECT NULL, location, SUM(salary) FROM employees GROUP BY location
UNION ALL
SELECT department_id, location, SUM(salary) FROM employees GROUP BY department_id, location;
-- Scans employees table 3 times!

-- ✅ GROUPING SETS: 1 table scan
SELECT department_id, location, SUM(salary)
FROM employees
GROUP BY GROUPING SETS ((department_id), (location), (department_id, location));
-- Scans employees table once!

-- Performance: GROUPING SETS typically 2-3x faster
```

**Use GROUPING SETS when:**
- Multiple grouping levels from same table
- Performance matters
- Need subtotals/grand totals

**Use UNION when:**
- Combining different tables
- Different WHERE conditions per query
- Need different aggregations

---

**Q2: What's the difference between UNION, INTERSECT, and EXCEPT?**

**A:**

**Visual representation:**

```
Set A: {1, 2, 3, 4}
Set B: {3, 4, 5, 6}

UNION: A ∪ B = {1, 2, 3, 4, 5, 6}  (all unique values)
INTERSECT: A ∩ B = {3, 4}          (common values)
EXCEPT: A - B = {1, 2}              (in A but not in B)
```

**SQL examples:**

```sql
-- UNION: All customers (deduplicated)
SELECT customer_id FROM usa_customers
UNION
SELECT customer_id FROM canada_customers;
-- Returns: Customers in USA or Canada (no duplicates)

-- INTERSECT: Customers in both
SELECT customer_id FROM usa_customers
INTERSECT
SELECT customer_id FROM canada_customers;
-- Returns: Customers in both USA and Canada

-- EXCEPT: USA-only customers
SELECT customer_id FROM usa_customers
EXCEPT
SELECT customer_id FROM canada_customers;
-- Returns: Customers in USA but not Canada
```

---

**Q3: How do I pivot data (convert rows to columns)?**

**A:**

**Problem: Convert this:**
```
dept | job_title | count
1    | Engineer  | 5
1    | Manager   | 2
2    | Engineer  | 3
```

**To this:**
```
dept | Engineer | Manager
1    | 5        | 2
2    | 3        | 0
```

**Solution: Conditional aggregation**

```sql
-- PostgreSQL & MySQL
SELECT 
    department_id,
    COUNT(*) FILTER (WHERE job_title = 'Engineer') AS engineers,
    COUNT(*) FILTER (WHERE job_title = 'Manager') AS managers,
    COUNT(*) FILTER (WHERE job_title = 'Designer') AS designers
FROM employees
GROUP BY department_id;

-- MySQL (no FILTER)
SELECT 
    department_id,
    SUM(CASE WHEN job_title = 'Engineer' THEN 1 ELSE 0 END) AS engineers,
    SUM(CASE WHEN job_title = 'Manager' THEN 1 ELSE 0 END) AS managers,
    SUM(CASE WHEN job_title = 'Designer' THEN 1 ELSE 0 END) AS designers
FROM employees
GROUP BY department_id;

-- Dynamic pivot (PostgreSQL with crosstab)
CREATE EXTENSION tablefunc;

SELECT * FROM crosstab(
    'SELECT department_id, job_title, COUNT(*) 
     FROM employees 
     GROUP BY department_id, job_title 
     ORDER BY 1,2',
    'SELECT DISTINCT job_title FROM employees ORDER BY 1'
) AS ct(department_id INT, Engineer BIGINT, Manager BIGINT, Designer BIGINT);
```

---

## Interview Questions

**Question 1: Write a query to generate a sales report showing total sales by year, quarter, and month, with subtotals at each level.**

**Difficulty:** Senior

**Answer:**

```sql
SELECT 
    EXTRACT(YEAR FROM order_date) AS year,
    EXTRACT(QUARTER FROM order_date) AS quarter,
    EXTRACT(MONTH FROM order_date) AS month,
    SUM(total_amount) AS total_sales,
    COUNT(*) AS order_count,
    -- Identify level
    CASE 
        WHEN GROUPING(EXTRACT(YEAR FROM order_date)) = 1 THEN 'Grand Total'
        WHEN GROUPING(EXTRACT(QUARTER FROM order_date)) = 1 THEN 'Year Total'
        WHEN GROUPING(EXTRACT(MONTH FROM order_date)) = 1 THEN 'Quarter Total'
        ELSE 'Month Detail'
    END AS aggregation_level
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY ROLLUP(
    EXTRACT(YEAR FROM order_date),
    EXTRACT(QUARTER FROM order_date),
    EXTRACT(MONTH FROM order_date)
)
ORDER BY year NULLS LAST, quarter NULLS LAST, month NULLS LAST;
```

**Why This Matters:** ROLLUP is essential for financial reporting and hierarchical summaries. Shows understanding of GROUPING and subtotal generation.

---

**Question 2: Find customers who purchased in January 2024 but not in February 2024.**

**Difficulty:** Mid-Level

**Answer:**

```sql
-- Solution 1: EXCEPT
SELECT customer_id FROM orders WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31'
EXCEPT
SELECT customer_id FROM orders WHERE order_date BETWEEN '2024-02-01' AND '2024-02-29';

-- Solution 2: NOT EXISTS
SELECT DISTINCT customer_id
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31'
  AND NOT EXISTS (
      SELECT 1 FROM orders o2 
      WHERE o2.customer_id = orders.customer_id 
        AND o2.order_date BETWEEN '2024-02-01' AND '2024-02-29'
  );

-- Solution 3: LEFT JOIN (MySQL compatible)
SELECT DISTINCT o1.customer_id
FROM orders o1
LEFT JOIN orders o2 ON o1.customer_id = o2.customer_id 
    AND o2.order_date BETWEEN '2024-02-01' AND '2024-02-29'
WHERE o1.order_date BETWEEN '2024-01-01' AND '2024-01-31'
  AND o2.order_id IS NULL;
```

**Why This Matters:** Set operations are fundamental for data analysis. Multiple solutions show SQL fluency and cross-database compatibility awareness.

---

## Key Takeaways

* **GROUPING SETS generates multiple aggregation levels** in single query
* **ROLLUP creates hierarchical subtotals** (useful for financial reports)
* **CUBE generates all combinations** of grouping dimensions
* **FILTER clause simplifies conditional aggregation** (PostgreSQL)
* **HAVING filters groups, WHERE filters rows** - understand execution order
* **UNION removes duplicates, UNION ALL faster** - choose appropriately
* **INTERSECT finds common rows** across result sets
* **EXCEPT finds differences** between result sets
* **UPSERT patterns** (ON CONFLICT, ON DUPLICATE KEY) handle conflicts
* **Statistical aggregates** (STDDEV, PERCENTILE) enable advanced analytics

---

*This concludes Part 3: Advanced SQL Techniques*
