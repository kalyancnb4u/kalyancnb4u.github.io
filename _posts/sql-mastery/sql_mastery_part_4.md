---
title: "Complete SQL Mastery Part 4: Query Optimization & Performance"
date: 2024-02-05 00:00:00 +0530
categories: [SQL, Mastery]
tags: [SQL, Database, PostgreSQL, MySQL, Performance, Optimization, Indexing, Explain, Explain Analyze, Query-tuning]
---

# Complete SQL Mastery Part 4: Query Optimization & Performance

## Introduction

Performance optimization is where SQL knowledge transforms into production value. A well-optimized query can be 1000x faster than its naive equivalent. This isn't hyperbole—it's a daily reality for database engineers.

In Parts 1-3, we covered SQL fundamentals, database internals, and advanced techniques. Now we focus on making queries fast. You'll learn to:
* **Read and interpret execution plans** to understand query behavior
* **Design optimal index strategies** that accelerate queries without bloating storage
* **Recognize and fix anti-patterns** that kill performance
* **Optimize schema design** for both read and write performance
* **Debug slow queries** systematically using proven methodologies

This knowledge is essential because:
* **User experience depends on it**: Slow queries mean slow applications
* **Cost reduction**: Efficient queries need fewer resources
* **Scalability**: Optimized systems handle 10x more load
* **Career advancement**: Performance tuning is a highly valued skill

By the end of this part, you'll think like a database performance engineer, seeing not just what a query does, but how efficiently it executes and where optimization opportunities exist.

---

## 4.1 Execution Plan Analysis

Execution plans reveal how the database will execute your query. Reading them is the single most important skill for query optimization.

### Understanding EXPLAIN

EXPLAIN shows the query execution plan without actually running the query.

**Why execution plans matter:**
* Reveals which indexes are used (or not used)
* Shows estimated vs actual row counts
* Identifies expensive operations (sorts, temporary tables)
* Exposes join order and algorithms
* Highlights bottlenecks

### EXPLAIN Syntax

**MySQL:**

```sql
-- Basic EXPLAIN
EXPLAIN SELECT * FROM employees WHERE department_id = 5;

-- JSON format (more detailed)
EXPLAIN FORMAT=JSON 
SELECT * FROM employees WHERE department_id = 5;

-- Traditional format
EXPLAIN FORMAT=TRADITIONAL
SELECT * FROM employees WHERE department_id = 5;

-- Tree format (MySQL 8.0.16+)
EXPLAIN FORMAT=TREE
SELECT * FROM employees WHERE department_id = 5;

-- EXPLAIN ANALYZE (MySQL 8.0.18+) - actually executes the query
EXPLAIN ANALYZE
SELECT * FROM employees WHERE department_id = 5;
```

**PostgreSQL:**

```sql
-- Basic EXPLAIN (estimated costs only)
EXPLAIN SELECT * FROM employees WHERE department_id = 5;

-- EXPLAIN ANALYZE (actual execution)
EXPLAIN ANALYZE SELECT * FROM employees WHERE department_id = 5;

-- With additional details
EXPLAIN (ANALYZE, BUFFERS, VERBOSE, TIMING)
SELECT * FROM employees WHERE department_id = 5;

-- Options:
-- ANALYZE: Execute and show actual times
-- BUFFERS: Show buffer usage (cache hits/misses)
-- VERBOSE: Show output column details
-- TIMING: Show timing for each node (default with ANALYZE)
-- COSTS: Show cost estimates (default on)
-- FORMAT: TEXT (default), JSON, XML, YAML
```

### MySQL EXPLAIN Output

**Column-by-column explanation:**

```sql
EXPLAIN SELECT * FROM employees WHERE department_id = 5;
```

**Output:**

| id | select_type | table | type | possible_keys | key | key_len | ref | rows | filtered | Extra |
|----|-------------|-------|------|---------------|-----|---------|-----|------|----------|-------|
| 1 | SIMPLE | employees | ref | idx_dept | idx_dept | 4 | const | 50 | 100.00 | NULL |

**Column Meanings:**

**1. id (Query Execution Order)**

```sql
-- Simple query: id = 1
EXPLAIN SELECT * FROM employees WHERE emp_id = 1;
-- id: 1

-- Subquery: different ids
EXPLAIN 
SELECT * FROM employees 
WHERE department_id IN (
    SELECT dept_id FROM departments WHERE location = 'NY'
);
-- id: 1 (outer query)
-- id: 2 (subquery)

-- Join: same id
EXPLAIN
SELECT * FROM employees e JOIN departments d ON e.department_id = d.dept_id;
-- id: 1 (for both tables)

-- Union: sequential ids
EXPLAIN
SELECT * FROM employees WHERE status = 'active'
UNION
SELECT * FROM employees WHERE status = 'pending';
-- id: 1 (first SELECT)
-- id: 2 (second SELECT)
```

**2. select_type (Query Type)**

| Type | Meaning | Example |
|------|---------|---------|
| SIMPLE | Simple SELECT without subqueries or UNIONs | `SELECT * FROM t` |
| PRIMARY | Outermost SELECT in complex query | Outer query with subquery |
| SUBQUERY | Subquery in SELECT or WHERE | `WHERE x IN (SELECT...)` |
| DERIVED | Derived table (FROM subquery) | `FROM (SELECT...) AS dt` |
| UNION | Second or later SELECT in UNION | `SELECT... UNION SELECT...` |
| DEPENDENT SUBQUERY | Subquery dependent on outer query | Correlated subquery |
| MATERIALIZED | Materialized subquery | Optimized IN subquery |

```sql
-- PRIMARY and SUBQUERY
EXPLAIN
SELECT * FROM employees
WHERE department_id = (
    SELECT dept_id FROM departments WHERE dept_name = 'IT'
);
-- select_type: PRIMARY (employees)
-- select_type: SUBQUERY (departments)

-- DERIVED (subquery in FROM)
EXPLAIN
SELECT dept, avg_sal FROM (
    SELECT department_id AS dept, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department_id
) AS dept_avg;
-- select_type: PRIMARY
-- select_type: DERIVED

-- DEPENDENT SUBQUERY (correlated)
EXPLAIN
SELECT e.first_name
FROM employees e
WHERE salary > (
    SELECT AVG(salary) FROM employees WHERE department_id = e.department_id
);
-- select_type: PRIMARY
-- select_type: DEPENDENT SUBQUERY
```

**3. table (Table Name)**

The table being accessed. Can be:
* Actual table name
* Derived table alias (`<derivedN>`)
* Union table (`<unionM,N>`)

**4. type (Access Type) - MOST IMPORTANT**

Access type shows how MySQL accesses the table. **Order from best to worst:**

| Type | Description | Performance |
|------|-------------|-------------|
| **system** | Table has 0 or 1 row | Excellent |
| **const** | At most one matching row (PK or unique index with constant) | Excellent |
| **eq_ref** | One row from this table for each row combination from previous tables | Excellent |
| **ref** | All rows matching an index value | Good |
| **fulltext** | Uses FULLTEXT index | Good |
| **ref_or_null** | Like ref but also searches for NULL | Good |
| **index_merge** | Multiple indexes used and merged | Okay |
| **range** | Index range scan | Okay |
| **index** | Full index scan | Poor |
| **ALL** | Full table scan | Very Poor |

**Examples:**

```sql
-- const: Primary key lookup
EXPLAIN SELECT * FROM employees WHERE emp_id = 123;
-- type: const

-- eq_ref: Joining on primary key or unique index
EXPLAIN 
SELECT * FROM orders o 
JOIN customers c ON o.customer_id = c.customer_id;
-- type: eq_ref (customers)

-- ref: Non-unique index lookup
EXPLAIN SELECT * FROM employees WHERE department_id = 5;
-- type: ref

-- range: Range scan on index
EXPLAIN SELECT * FROM employees WHERE salary BETWEEN 50000 AND 100000;
-- type: range

EXPLAIN SELECT * FROM employees WHERE emp_id IN (1, 2, 3, 4, 5);
-- type: range

-- index: Full index scan (reading entire index)
EXPLAIN SELECT emp_id FROM employees;
-- type: index (index-only scan)

-- ALL: Full table scan
EXPLAIN SELECT * FROM employees WHERE YEAR(hire_date) = 2024;
-- type: ALL (function on indexed column prevents index usage)
```

**5. possible_keys and key**

* **possible_keys**: Indexes that could be used
* **key**: Index actually chosen by optimizer

```sql
-- Multiple indexes available
CREATE INDEX idx_dept ON employees(department_id);
CREATE INDEX idx_status ON employees(status);
CREATE INDEX idx_dept_status ON employees(department_id, status);

EXPLAIN SELECT * FROM employees WHERE department_id = 5 AND status = 'active';
-- possible_keys: idx_dept, idx_status, idx_dept_status
-- key: idx_dept_status (most selective)

-- NULL key means no index used
EXPLAIN SELECT * FROM employees WHERE UPPER(first_name) = 'JOHN';
-- possible_keys: NULL
-- key: NULL
-- type: ALL (full table scan)
```

**6. key_len (Key Length)**

Number of bytes of the index used:

```sql
-- INT (4 bytes)
CREATE INDEX idx_dept ON employees(department_id);
EXPLAIN SELECT * FROM employees WHERE department_id = 5;
-- key_len: 4

-- INT NULL (4 bytes + 1 byte NULL flag)
ALTER TABLE employees MODIFY department_id INT NULL;
EXPLAIN SELECT * FROM employees WHERE department_id = 5;
-- key_len: 5

-- VARCHAR(100) with utf8mb4 (400 bytes + 2 length + 1 NULL)
CREATE INDEX idx_email ON employees(email);  -- email VARCHAR(100)
EXPLAIN SELECT * FROM employees WHERE email = 'test@example.com';
-- key_len: 403

-- Composite index: Shows how much is used
CREATE INDEX idx_composite ON employees(department_id, status, salary);
EXPLAIN SELECT * FROM employees WHERE department_id = 5;
-- key_len: 4 (only first column used)

EXPLAIN SELECT * FROM employees WHERE department_id = 5 AND status = 'active';
-- key_len: 24 (first two columns used)
```

**7. ref (Reference Columns)**

Shows which columns or constants are compared to the index:

```sql
-- const: Constant value
EXPLAIN SELECT * FROM employees WHERE department_id = 5;
-- ref: const

-- Column from another table
EXPLAIN 
SELECT * FROM employees e 
JOIN departments d ON e.department_id = d.dept_id;
-- ref: database_name.d.dept_id

-- Function result
EXPLAIN 
SELECT * FROM employees 
WHERE department_id = (SELECT MAX(dept_id) FROM departments);
-- ref: const
```

**8. rows (Estimated Rows)**

Number of rows MySQL expects to examine:

```sql
-- Low row count (good)
EXPLAIN SELECT * FROM employees WHERE emp_id = 123;
-- rows: 1

-- High row count (potentially problematic)
EXPLAIN SELECT * FROM employees WHERE first_name LIKE '%John%';
-- rows: 50000 (full table scan)

-- Multiplication in joins
EXPLAIN 
SELECT * FROM orders o JOIN order_items oi ON o.order_id = oi.order_id;
-- orders rows: 1000
-- order_items rows: 5 (per order)
-- Total examined: 1000 + (1000 × 5) = 6000
```

**9. filtered (Filter Percentage)**

Percentage of rows that will be filtered by WHERE conditions:

```sql
EXPLAIN 
SELECT * FROM employees 
WHERE department_id = 5 AND salary > 50000;

-- rows: 100 (estimated rows with department_id = 5)
-- filtered: 30.00 (30% also match salary > 50000)
-- Actual rows returned: 100 × 0.30 = 30

-- Low filtered percentage indicates poor selectivity
EXPLAIN SELECT * FROM employees WHERE status = 'active';
-- rows: 10000
-- filtered: 90.00 (most rows match)
```

**10. Extra (Additional Information)**

Critical optimization hints:

| Extra | Meaning | Action |
|-------|---------|--------|
| **Using index** | Index-only scan (covering index) | Excellent! |
| **Using where** | WHERE filtering after reading rows | Normal |
| **Using index condition** | Index condition pushdown | Good |
| **Using temporary** | Temporary table needed | Slow, consider optimization |
| **Using filesort** | External sort needed | Slow, add ORDER BY index |
| **Using join buffer** | Join buffer used (no index) | Consider adding index |
| **Impossible WHERE** | WHERE always false | Query can be optimized away |
| **Select tables optimized away** | Query optimized to constant | Excellent! |

**Examples:**

```sql
-- Using index (covering index)
CREATE INDEX idx_dept_salary ON employees(department_id, salary);
EXPLAIN SELECT department_id, salary FROM employees WHERE department_id = 5;
-- Extra: Using index

-- Using filesort (needs optimization)
EXPLAIN SELECT * FROM employees ORDER BY hire_date;
-- Extra: Using filesort
-- Fix: CREATE INDEX idx_hire_date ON employees(hire_date);

-- Using temporary (needs optimization)
EXPLAIN 
SELECT DISTINCT department_id FROM employees ORDER BY salary;
-- Extra: Using temporary; Using filesort
-- Fix: Rewrite or add appropriate indexes

-- Impossible WHERE
EXPLAIN SELECT * FROM employees WHERE 1=0;
-- Extra: Impossible WHERE

-- Select tables optimized away
EXPLAIN SELECT MAX(emp_id) FROM employees;
-- Extra: Select tables optimized away
-- Optimizer uses index statistics instead of scanning
```

### PostgreSQL EXPLAIN Output

PostgreSQL EXPLAIN shows a tree structure with costs and row estimates:

```sql
EXPLAIN SELECT * FROM employees WHERE department_id = 5;
```

**Output:**

```
Seq Scan on employees  (cost=0.00..458.00 rows=50 width=100)
  Filter: (department_id = 5)
```

**Reading the output:**

```
Node Type on table_name  (cost=startup..total rows=estimate width=average_bytes)
  Additional information
```

**Components:**

**1. Node Types (Operation Types)**

| Node Type | Description | Performance |
|-----------|-------------|-------------|
| **Seq Scan** | Sequential table scan | Slow for large tables |
| **Index Scan** | Index lookup with table access | Good |
| **Index Only Scan** | Index-only (covering index) | Excellent |
| **Bitmap Heap Scan** | Two-phase index scan | Good for multiple conditions |
| **Nested Loop** | Nested loop join | Good for small outer table |
| **Hash Join** | Hash join | Good for large equi-joins |
| **Merge Join** | Merge join | Good for sorted data |
| **Sort** | External sort operation | Expensive |
| **Aggregate** | Grouping operation | Depends on input size |
| **CTE Scan** | Common Table Expression scan | Depends on CTE |

**Examples:**

```sql
-- Sequential Scan
EXPLAIN SELECT * FROM employees WHERE first_name = 'John';
-- Seq Scan on employees  (cost=0.00..458.00 rows=10 width=100)
--   Filter: ((first_name)::text = 'John'::text)

-- Index Scan
CREATE INDEX idx_dept ON employees(department_id);
EXPLAIN SELECT * FROM employees WHERE department_id = 5;
-- Index Scan using idx_dept on employees  (cost=0.29..8.31 rows=50 width=100)
--   Index Cond: (department_id = 5)

-- Index Only Scan
CREATE INDEX idx_dept_sal ON employees(department_id, salary);
EXPLAIN SELECT department_id, salary FROM employees WHERE department_id = 5;
-- Index Only Scan using idx_dept_sal on employees  (cost=0.29..4.36 rows=50 width=8)
--   Index Cond: (department_id = 5)

-- Bitmap Heap Scan
CREATE INDEX idx_dept ON employees(department_id);
CREATE INDEX idx_status ON employees(status);
EXPLAIN SELECT * FROM employees 
WHERE department_id = 5 OR status = 'active';
-- Bitmap Heap Scan on employees  (cost=4.73..458.00 rows=150 width=100)
--   Recheck Cond: ((department_id = 5) OR ...)
--   ->  BitmapOr
--         ->  Bitmap Index Scan on idx_dept
--         ->  Bitmap Index Scan on idx_status
```

**2. Cost Notation (startup..total)**

```
cost=0.29..8.31

startup cost = 0.29  (cost to get first row)
total cost = 8.31    (cost to get all rows)
```

**Cost units:**
* Arbitrary units (not milliseconds)
* Sequential page read = 1.0
* Random page read = 4.0 (configurable)
* CPU operations have smaller costs

```sql
-- Low cost (good)
EXPLAIN SELECT * FROM employees WHERE emp_id = 1;
-- Index Scan using employees_pkey on employees  (cost=0.29..8.31 rows=1 width=100)

-- High cost (check for optimization)
EXPLAIN SELECT * FROM large_table;
-- Seq Scan on large_table  (cost=0.00..169248.00 rows=10000000 width=100)
```

**3. rows (Estimated Rows)**

Optimizer's estimate of rows this node will produce:

```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE department_id = 5;
-- Index Scan using idx_dept on employees  
--   (cost=0.29..8.31 rows=50 width=100)  <- Estimated
--   (actual time=0.012..0.034 rows=47 loops=1)  <- Actual
```

**4. width (Average Row Width)**

Average number of bytes per row:

```sql
EXPLAIN SELECT emp_id FROM employees;  -- INT (4 bytes)
-- width=4

EXPLAIN SELECT emp_id, first_name, last_name FROM employees;
-- width=40 (approximate)
```

### EXPLAIN ANALYZE (Actual Execution)

**MySQL 8.0.18+:**

```sql
EXPLAIN ANALYZE
SELECT e.first_name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.dept_id
WHERE e.salary > 50000;
```

**Output (tree format with actual times):**

```
-> Nested loop inner join  (actual time=0.123..5.234 rows=150 loops=1)
    -> Filter: (e.salary > 50000)  (actual time=0.089..3.456 rows=150 loops=1)
        -> Table scan on e  (actual time=0.045..2.345 rows=1000 loops=1)
    -> Single-row index lookup on d using PRIMARY (dept_id=e.department_id)  
       (actual time=0.001..0.002 rows=1 loops=150)
```

**PostgreSQL:**

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT e.first_name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.dept_id
WHERE e.salary > 50000;
```

**Output:**

```
Hash Join  (cost=5.50..458.00 rows=150 width=32) 
           (actual time=0.123..5.234 rows=147 loops=1)
  Hash Cond: (e.department_id = d.dept_id)
  Buffers: shared hit=45 read=12
  ->  Seq Scan on employees e  (cost=0.00..458.00 rows=150 width=28) 
                                (actual time=0.012..3.456 rows=147 loops=1)
        Filter: (salary > '50000'::numeric)
        Rows Removed by Filter: 853
        Buffers: shared hit=40
  ->  Hash  (cost=3.50..3.50 rows=50 width=20) 
            (actual time=0.089..0.089 rows=50 loops=1)
        Buckets: 1024  Batches: 1  Memory Usage: 10kB
        Buffers: shared hit=5
        ->  Seq Scan on departments d  (cost=0.00..3.50 rows=50 width=20) 
                                       (actual time=0.006..0.045 rows=50 loops=1)
              Buffers: shared hit=5

Planning Time: 0.234 ms
Execution Time: 5.567 ms
```

**Key metrics:**

* **actual time=0.123..5.234**: Actual startup and total time in milliseconds
* **rows=147**: Actual number of rows returned
* **loops=1**: Number of times this node executed
* **Buffers: shared hit=45**: Pages found in cache (good)
* **Buffers: read=12**: Pages read from disk (slower)
* **Planning Time**: Time to plan the query
* **Execution Time**: Total execution time

### Reading Complex Plans

**Example: Multi-join query with subquery**

```sql
EXPLAIN ANALYZE
SELECT 
    c.customer_name,
    o.order_date,
    (SELECT COUNT(*) FROM order_items WHERE order_id = o.order_id) as item_count
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE c.country = 'USA'
  AND o.order_date >= '2024-01-01'
ORDER BY o.order_date DESC
LIMIT 10;
```

**PostgreSQL output (simplified):**

```
Limit  (cost=1245.67..1245.69 rows=10 width=48) 
       (actual time=12.345..12.378 rows=10 loops=1)
  ->  Sort  (cost=1245.67..1248.17 rows=1000 width=48) 
            (actual time=12.344..12.356 rows=10 loops=1)
        Sort Key: o.order_date DESC
        Sort Method: top-N heapsort  Memory: 25kB
        ->  Hash Join  (cost=45.00..1223.45 rows=1000 width=48) 
                       (actual time=1.234..10.567 rows=985 loops=1)
              Hash Cond: (o.customer_id = c.customer_id)
              ->  Seq Scan on orders o  (cost=0.00..1100.00 rows=5000 width=20) 
                                        (actual time=0.012..5.678 rows=4932 loops=1)
                    Filter: (order_date >= '2024-01-01'::date)
                    Rows Removed by Filter: 15068
              ->  Hash  (cost=40.00..40.00 rows=400 width=32) 
                        (actual time=1.123..1.123 rows=387 loops=1)
                    Buckets: 1024  Batches: 1  Memory Usage: 25kB
                    ->  Seq Scan on customers c  (cost=0.00..40.00 rows=400 width=32) 
                                                 (actual time=0.023..0.987 rows=387 loops=1)
                          Filter: ((country)::text = 'USA'::text)
                          Rows Removed by Filter: 613
  SubPlan 1
    ->  Aggregate  (cost=0.50..0.51 rows=1 width=8) 
                   (actual time=0.010..0.010 rows=1 loops=10)
          ->  Seq Scan on order_items  (cost=0.00..0.48 rows=8 width=0) 
                                       (actual time=0.002..0.008 rows=8 loops=10)
                Filter: (order_id = o.order_id)
                Rows Removed by Filter: 992

Planning Time: 1.234 ms
Execution Time: 12.456 ms
```

**Reading strategy:**

1. **Start from innermost operations** (bottom of tree)
2. **Follow the flow upward**
3. **Compare estimated vs actual rows** (large differences indicate stale statistics)
4. **Look for expensive operations** (Seq Scan on large tables, Sort, Subplans in loops)
5. **Check BUFFERS** (many reads from disk indicate cache issues)

**Issues in this plan:**

```sql
-- ❌ PROBLEM 1: Seq Scan on orders with filter removing many rows
-- Filter: (order_date >= '2024-01-01'::date)
-- Rows Removed by Filter: 15068

-- FIX: Add index
CREATE INDEX idx_orders_date ON orders(order_date);

-- ❌ PROBLEM 2: Seq Scan on customers with filter
-- Filter: ((country)::text = 'USA'::text)
-- Rows Removed by Filter: 613

-- FIX: Add index
CREATE INDEX idx_customers_country ON customers(country);

-- ❌ PROBLEM 3: Correlated subquery runs for each row (loops=10)
-- SubPlan 1 ... (loops=10)

-- FIX: Rewrite with JOIN and window function
SELECT 
    c.customer_name,
    o.order_date,
    COUNT(oi.item_id) as item_count
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN order_items oi ON o.order_id = oi.order_id
WHERE c.country = 'USA'
  AND o.order_date >= '2024-01-01'
GROUP BY c.customer_id, c.customer_name, o.order_id, o.order_date
ORDER BY o.order_date DESC
LIMIT 10;
```

### Plan Node Types Comparison

**MySQL:**

| Type | Meaning |
|------|---------|
| Table scan | Full table scan (type: ALL) |
| Index scan | Index lookup (type: ref, eq_ref) |
| Index range scan | Range scan (type: range) |
| Nested loop | Nested loop join |
| Hash join | Hash join (8.0.18+) |
| Filesort | External sort |
| Temporary table | Temporary table created |

**PostgreSQL:**

| Node Type | Meaning |
|-----------|---------|
| Seq Scan | Sequential table scan |
| Index Scan | B-tree index lookup |
| Index Only Scan | Covering index scan |
| Bitmap Index Scan | Bitmap index scan |
| Bitmap Heap Scan | Heap scan using bitmap |
| Nested Loop | Nested loop join |
| Hash Join | Hash join algorithm |
| Merge Join | Sort-merge join |
| Sort | Explicit sort operation |
| Aggregate | Grouping/aggregation |

### Performance Profiling Tools

**MySQL Performance Schema:**

```sql
-- Enable performance schema
UPDATE performance_schema.setup_instruments 
SET ENABLED = 'YES', TIMED = 'YES' 
WHERE NAME LIKE 'statement/%';

UPDATE performance_schema.setup_consumers 
SET ENABLED = 'YES' 
WHERE NAME LIKE '%statements%';

-- View slow queries
SELECT 
    DIGEST_TEXT,
    COUNT_STAR,
    AVG_TIMER_WAIT / 1000000000000 AS avg_seconds,
    SUM_ROWS_EXAMINED,
    SUM_ROWS_SENT
FROM performance_schema.events_statements_summary_by_digest
ORDER BY AVG_TIMER_WAIT DESC
LIMIT 10;

-- Query profiling (session)
SET profiling = 1;

SELECT * FROM large_table WHERE condition = 'value';

SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;

-- Detailed profiling
SHOW PROFILE CPU, BLOCK IO FOR QUERY 1;
```

**MySQL Slow Query Log:**

```sql
-- my.cnf configuration
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 1  -- Queries taking > 1 second
log_queries_not_using_indexes = 1

-- Analyze slow log with mysqldumpslow
$ mysqldumpslow -s at -t 10 /var/log/mysql/slow.log
```

**PostgreSQL pg_stat_statements:**

```sql
-- Enable extension
CREATE EXTENSION pg_stat_statements;

-- postgresql.conf
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.track = all

-- View slow queries
SELECT 
    query,
    calls,
    total_exec_time,
    mean_exec_time,
    max_exec_time,
    rows
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Reset statistics
SELECT pg_stat_statements_reset();
```

**PostgreSQL System Views:**

```sql
-- Current queries
SELECT 
    pid,
    now() - query_start AS duration,
    query,
    state
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY duration DESC;

-- Table statistics
SELECT 
    schemaname,
    relname,
    seq_scan,
    seq_tup_read,
    idx_scan,
    idx_tup_fetch,
    n_tup_ins,
    n_tup_upd,
    n_tup_del
FROM pg_stat_user_tables
ORDER BY seq_scan DESC;

-- Index usage
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Missing indexes (tables with high seq_scan and low idx_scan)
SELECT 
    schemaname,
    relname,
    seq_scan,
    seq_tup_read,
    idx_scan,
    seq_tup_read / seq_scan AS avg_seq_read
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_scan DESC
LIMIT 20;
```

---

## Frequently Asked Questions

**Q1: What's the difference between "rows" in EXPLAIN and actual rows returned?**

**A:** 

**"rows" in EXPLAIN** is the optimizer's **estimate** of how many rows will be examined (not returned).

**Actual rows** is what the query actually returns after filtering.

```sql
-- Example
EXPLAIN SELECT * FROM employees WHERE department_id = 5;
-- rows: 100 (optimizer estimates 100 rows with department_id = 5)

-- But query might return fewer after additional filters:
SELECT * FROM employees WHERE department_id = 5 AND salary > 80000;
-- Examines: 100 rows
-- Returns: 15 rows (after salary filter)

-- EXPLAIN ANALYZE shows both:
EXPLAIN ANALYZE SELECT * FROM employees 
WHERE department_id = 5 AND salary > 80000;

-- MySQL output:
-- rows=100 (estimate for department_id = 5)
-- actual rows=15

-- PostgreSQL output:
-- (cost=... rows=30 ...) (actual time=... rows=15 ...)
--  ^^^estimated            ^^^actual
```

**Why estimates can be wrong:**
* Outdated statistics (run ANALYZE)
* Correlation between columns not known
* Complex WHERE conditions
* Data skew (uneven distribution)

**When to worry:**
* Estimate: 10 rows, Actual: 10,000 rows → optimizer chose wrong plan
* Fix: Update statistics with ANALYZE/ANALYZE TABLE

**Related Concepts**: Query optimizer, cardinality estimation

---

**Q2: How do I know if my query is using an index efficiently?**

**A:**

**MySQL - Check these indicators:**

```sql
EXPLAIN SELECT * FROM employees WHERE department_id = 5;
```

**✅ GOOD signs:**
* `type`: const, eq_ref, ref, range (NOT ALL or index)
* `key`: Shows an index name (NOT NULL)
* `Extra`: "Using index" (covering index)
* `rows`: Low number

**❌ BAD signs:**
* `type`: ALL (full table scan)
* `type`: index (full index scan - only marginally better than ALL)
* `key`: NULL (no index used)
* `Extra`: "Using filesort" or "Using temporary"
* `rows`: High number (many rows examined)

**PostgreSQL - Check these:**

```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE department_id = 5;
```

**✅ GOOD signs:**
* Node type: "Index Scan" or "Index Only Scan" (NOT "Seq Scan")
* Low cost estimates
* Few "Buffers: read" (disk reads), more "Buffers: shared hit" (cache hits)

**❌ BAD signs:**
* Node type: "Seq Scan" on large table
* Filter removing many rows
* High buffer reads

**Example comparison:**

```sql
-- ❌ BAD: Full table scan
EXPLAIN SELECT * FROM employees WHERE YEAR(hire_date) = 2024;
-- MySQL: type=ALL, key=NULL, rows=100000
-- PostgreSQL: Seq Scan, Filter: (EXTRACT(year FROM hire_date) = 2024)

-- ✅ GOOD: Index range scan
CREATE INDEX idx_hire_date ON employees(hire_date);
EXPLAIN SELECT * FROM employees 
WHERE hire_date >= '2024-01-01' AND hire_date < '2025-01-01';
-- MySQL: type=range, key=idx_hire_date, rows=5000
-- PostgreSQL: Index Scan using idx_hire_date
```

---

**Q3: Why does adding more indexes sometimes make queries slower?**

**A:**

Indexes have costs:

**1. Write Performance:**
```sql
-- Every INSERT/UPDATE/DELETE must update all indexes
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2),
    stock INT,
    description TEXT
);

-- With 5 indexes:
CREATE INDEX idx_name ON products(name);
CREATE INDEX idx_category ON products(category);
CREATE INDEX idx_price ON products(price);
CREATE INDEX idx_stock ON products(stock);
CREATE INDEX idx_cat_price ON products(category, price);

-- Single INSERT now:
INSERT INTO products VALUES (...);
-- Must update:
-- 1. Primary key index
-- 2. idx_name
-- 3. idx_category
-- 4. idx_price
-- 5. idx_stock
-- 6. idx_cat_price
-- Total: 6 index updates per INSERT (slower writes)
```

**2. Storage Cost:**
```sql
-- Indexes consume disk space
-- Large table: 10GB data, 15GB indexes → 25GB total

-- Query overhead: Optimizer must consider many options
SELECT * FROM products WHERE category = 'Electronics' AND price < 100;

-- Optimizer evaluates:
-- Option 1: idx_category
-- Option 2: idx_price
-- Option 3: idx_cat_price
-- Option 4: Index merge (idx_category + idx_price)
-- More indexes → longer planning time
```

**3. Maintenance Cost:**
```sql
-- Index bloat requires maintenance
-- PostgreSQL: REINDEX
-- MySQL: OPTIMIZE TABLE

-- Fragmented indexes slow down queries
```

**4. Wrong Index Selection:**
```sql
-- Optimizer might choose less optimal index
CREATE INDEX idx_a ON table(column_a);
CREATE INDEX idx_b ON table(column_b);

SELECT * FROM table WHERE column_a = 1 AND column_b = 2;

-- Optimizer might choose idx_a when idx_b is better
-- Or vice versa (depending on data distribution)

-- Solution: Create composite index
CREATE INDEX idx_a_b ON table(column_a, column_b);
DROP INDEX idx_a;  -- If only used for this query
DROP INDEX idx_b;  -- If only used for this query
```

**Best Practices:**
* Only create indexes for frequently used queries
* Monitor index usage (pg_stat_user_indexes)
* Remove unused indexes
* Use covering indexes instead of multiple single-column indexes
* Consider write vs read ratio

---

## Interview Questions

**Question 1: A query was fast last week but is slow today with the same data volume. What could cause this?**

**Difficulty:** Mid-Level

**Answer:**

**Several potential causes:**

**1. Outdated Statistics (Most Common)**

```sql
-- Statistics become stale as data changes
-- Optimizer makes wrong decisions

-- Check last analyze time
-- PostgreSQL:
SELECT 
    schemaname,
    relname,
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
WHERE relname = 'slow_table';

-- MySQL:
SELECT 
    TABLE_NAME,
    UPDATE_TIME
FROM information_schema.TABLES
WHERE TABLE_NAME = 'slow_table';

-- FIX: Update statistics
-- PostgreSQL:
ANALYZE slow_table;

-- MySQL:
ANALYZE TABLE slow_table;
```

**2. Index Bloat / Fragmentation**

```sql
-- PostgreSQL: Indexes become bloated
-- Check index size vs table size
SELECT 
    pg_size_pretty(pg_relation_size('slow_table')) AS table_size,
    pg_size_pretty(pg_indexes_size('slow_table')) AS index_size;

-- FIX: Rebuild indexes
-- PostgreSQL:
REINDEX TABLE slow_table;

-- MySQL:
OPTIMIZE TABLE slow_table;
```

**3. Lock Contention**

```sql
-- Other queries holding locks
-- PostgreSQL: Check locks
SELECT 
    pid,
    usename,
    query,
    state,
    wait_event_type,
    wait_event
FROM pg_stat_activity
WHERE wait_event IS NOT NULL;

-- MySQL: Check processlist
SHOW PROCESSLIST;
SHOW ENGINE INNODB STATUS\G  -- Look for TRANSACTIONS section
```

**4. Plan Regression (Optimizer Changed Plan)**

```sql
-- PostgreSQL: Optimizer chose different plan
-- Check query with EXPLAIN ANALYZE
-- Compare with previous plan (if you have it saved)

-- Temporary fix: Use hints to force old plan
-- PostgreSQL:
SET enable_seqscan = off;  -- Force index scan

-- MySQL:
SELECT /*+ INDEX(table_name index_name) */ ...

-- Permanent fix: Add index or adjust statistics
```

**5. Cache Warm-Up**

```sql
-- First query after restart is slow (cold cache)
-- Subsequent queries fast (warm cache)

-- Check buffer pool hit ratio
-- MySQL:
SHOW STATUS LIKE 'Innodb_buffer_pool_read%';
-- Calculate hit ratio:
-- (Innodb_buffer_pool_read_requests - Innodb_buffer_pool_reads) 
-- / Innodb_buffer_pool_read_requests

-- PostgreSQL:
SELECT 
    sum(heap_blks_hit) / nullif(sum(heap_blks_hit + heap_blks_read), 0) AS cache_hit_ratio
FROM pg_statio_user_tables;
```

**6. Concurrent Load**

```sql
-- More concurrent queries competing for resources
-- Check active connections

-- PostgreSQL:
SELECT count(*) FROM pg_stat_activity WHERE state = 'active';

-- MySQL:
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Threads_running';
```

**Systematic Debugging Process:**

```sql
-- 1. Get current execution plan
EXPLAIN ANALYZE [query];

-- 2. Update statistics
ANALYZE TABLE table_name;  -- MySQL
ANALYZE table_name;  -- PostgreSQL

-- 3. Compare plans (if you have historical plan)
-- Look for:
--   - Different access methods (Seq Scan vs Index Scan)
--   - Different join orders
--   - Different row estimates

-- 4. Check for locks
-- [See queries above]

-- 5. Check system resources
-- CPU, disk I/O, memory

-- 6. Check for schema changes
-- New indexes, dropped indexes, table alterations
```

**Why This Matters:** Performance regressions are common in production. Systematic debugging saves hours of troubleshooting.

**Follow-up Questions:**
* How would you prevent plan regressions in production?
* What monitoring would you implement to catch this earlier?

---

**Question 2: You run EXPLAIN and see "type: ALL" on a 100-million-row table. The query still needs to return most rows. How do you optimize it?**

**Difficulty:** Senior

**Answer:**

**First, verify if optimization is actually needed:**

```sql
-- Query returning 90% of table:
SELECT * FROM huge_table WHERE status != 'deleted';
-- 90 million out of 100 million rows returned

-- In this case, full table scan might be OPTIMAL!
-- Reading 90M rows via index would be:
-- 1. Index scan: 90M index lookups (random I/O)
-- 2. Table access: 90M row fetches (more random I/O)
-- vs.
-- Sequential scan: Single pass through table (sequential I/O)

-- Sequential I/O is 10-100x faster than random I/O
```

**When full table scan IS optimal:**
* Query returns > 10-20% of table
* Index selectivity is poor
* Table fits in buffer pool

**When full table scan IS NOT optimal:**

**1. Query can use covering index:**

```sql
-- ❌ BAD: Full table scan
EXPLAIN SELECT id, name, status FROM huge_table WHERE status = 'active';
-- type: ALL, rows: 100000000

-- ✅ GOOD: Create covering index
CREATE INDEX idx_status_covering ON huge_table(status, id, name);

EXPLAIN SELECT id, name, status FROM huge_table WHERE status = 'active';
-- type: index (or ref), key: idx_status_covering
-- Now reads only index (much smaller than table)
```

**2. Partition the table:**

```sql
-- Partition by common filter column
CREATE TABLE huge_table (
    id BIGINT,
    status VARCHAR(20),
    created_date DATE,
    ...
) PARTITION BY RANGE (YEAR(created_date)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025)
);

-- Query with partition filter
SELECT * FROM huge_table 
WHERE created_date >= '2024-01-01';
-- Only scans p2024 partition (much smaller)
```

**3. Use materialized view / summary table:**

```sql
-- If frequently querying aggregated data:
-- ❌ SLOW: Aggregate 100M rows each time
SELECT status, COUNT(*) 
FROM huge_table 
GROUP BY status;

-- ✅ FAST: Create summary table
CREATE TABLE status_summary AS
SELECT status, COUNT(*) as count
FROM huge_table
GROUP BY status;

-- Update summary periodically
TRUNCATE status_summary;
INSERT INTO status_summary 
SELECT status, COUNT(*) FROM huge_table GROUP BY status;

-- Or use triggers to maintain in real-time
```

**4. Add selective index with good cardinality:**

```sql
-- Check cardinality
SELECT 
    status,
    COUNT(*) as count,
    COUNT(*) * 100.0 / SUM(COUNT(*)) OVER () as percentage
FROM huge_table
GROUP BY status;

-- If status has good selectivity (e.g., 'active' is 5%):
CREATE INDEX idx_status ON huge_table(status);

SELECT * FROM huge_table WHERE status = 'pending';  -- 1% of rows
-- Now uses index (5% threshold typically)
```

**5. Parallel query execution:**

```sql
-- PostgreSQL: Use parallel workers
SET max_parallel_workers_per_gather = 4;

EXPLAIN SELECT * FROM huge_table WHERE expensive_condition;
-- Look for: "Parallel Seq Scan"
-- Uses multiple CPU cores for single query

-- MySQL 8.0+: Automatic parallelization for some queries
```

**6. Rewrite query to avoid returning all rows:**

```sql
-- ❌ Application filters after fetching all rows
SELECT * FROM huge_table WHERE status != 'deleted';
-- App then filters further

-- ✅ Push filters to database
SELECT * FROM huge_table 
WHERE status != 'deleted' 
  AND created_date >= '2024-01-01'
  AND category IN ('electronics', 'computers');
```

**7. Consider if query pattern is wrong:**

```sql
-- ❌ ANTI-PATTERN: Export all data frequently
SELECT * FROM huge_table;  -- 100M rows

-- ✅ SOLUTION: Incremental export
SELECT * FROM huge_table 
WHERE updated_at > '2024-01-01 00:00:00';  -- Only recent changes

-- Or: Use replication/CDC for data sync
```

**Real-world decision tree:**

```
100M row table, full scan detected
│
├─ Returns > 20% of table?
│  ├─ Yes → Full scan OK, consider partitioning or parallel scan
│  └─ No → Add selective index
│
├─ Needs all columns?
│  ├─ Yes → Full scan may be necessary
│  └─ No → Create covering index
│
├─ Frequent aggregation?
│  ├─ Yes → Create summary table
│  └─ No → Continue analysis
│
└─ Can partition by common filter?
   ├─ Yes → Partition table
   └─ No → Review query pattern
```

**Why This Matters:** Not all full scans are bad. Understanding when to optimize and when to accept full scans requires deep knowledge of database internals and I/O patterns.

**Follow-up Questions:**
* How would you determine the optimal index selectivity threshold?
* When would you choose table partitioning over indexing?
* How do you handle tables that grow by millions of rows per day?

---

## Key Takeaways

* **EXPLAIN is your best friend** - always analyze query plans before optimizing
* **Understand access types** - const/eq_ref/ref are excellent, ALL is often problematic
* **Compare estimated vs actual rows** - large differences indicate stale statistics
* **Full table scans aren't always bad** - they're optimal when returning most rows
* **Look for key indicators**: Using filesort, Using temporary, Seq Scan on large tables
* **MySQL and PostgreSQL EXPLAIN differ** - learn both formats for cross-database work
* **EXPLAIN ANALYZE shows reality** - estimates can be wrong, actual execution reveals truth
* **Monitor with system views** - pg_stat_statements, Performance Schema show real-world performance
* **Update statistics regularly** - ANALYZE prevents plan regressions
* **Profile slow queries systematically** - use EXPLAIN ANALYZE, slow query logs, and profiling tools

---

## 4.2 Index Strategy & Optimization

Indexes are the most powerful performance tool, but poor index design can harm more than help. Understanding when, where, and how to index is critical.

### Index Design Principles

#### Selectivity and Cardinality

```sql
-- High selectivity (good for indexing)
-- email: 1,000,000 rows, 999,000 unique values
-- Selectivity = 999,000 / 1,000,000 = 99.9%
CREATE INDEX idx_email ON users(email);  -- ✓ Excellent

-- Low selectivity (poor for indexing)
-- gender: 1,000,000 rows, 3 unique values (M, F, Other)
-- Selectivity = 3 / 1,000,000 = 0.0003%
CREATE INDEX idx_gender ON users(gender);  -- ✗ Wasteful

-- Check selectivity (PostgreSQL)
SELECT 
    attname AS column_name,
    n_distinct,
    CASE 
        WHEN n_distinct > 0 THEN n_distinct
        WHEN n_distinct < 0 THEN ABS(n_distinct) * reltuples
    END AS distinct_values,
    reltuples AS total_rows,
    CASE 
        WHEN n_distinct > 0 THEN n_distinct / reltuples
        WHEN n_distinct < 0 THEN ABS(n_distinct)
    END AS selectivity
FROM pg_stats
JOIN pg_class ON pg_stats.tablename = pg_class.relname
WHERE tablename = 'users'
ORDER BY selectivity DESC;

-- MySQL: Check cardinality
SHOW INDEX FROM users;
-- High Cardinality = Good selectivity
```

**Selectivity guidelines:**
- **> 95%**: Excellent candidate for index (email, username, order_id)
- **50-95%**: Good candidate (zip_code, product_sku)
- **10-50%**: Consider composite index or partial index
- **< 10%**: Poor candidate (gender, boolean flags, status with few values)

#### Composite Index Column Order

```sql
-- Rule: Most selective column first? NO!
-- Rule: Match query patterns!

-- Query patterns:
-- 1. WHERE country = 'USA' (frequent)
-- 2. WHERE country = 'USA' AND city = 'NYC' (less frequent)
-- 3. WHERE country = 'USA' AND city = 'NYC' AND status = 'active' (rare)

-- ✓ CORRECT: Order by query frequency and filtering
CREATE INDEX idx_country_city_status ON users(country, city, status);

-- This index supports:
-- - WHERE country = 'USA'                                    ✓
-- - WHERE country = 'USA' AND city = 'NYC'                  ✓
-- - WHERE country = 'USA' AND city = 'NYC' AND status = 'active' ✓

-- ✗ WRONG: Most selective first
CREATE INDEX idx_status_city_country ON users(status, city, country);
-- This does NOT efficiently support:
-- - WHERE country = 'USA'                                    ✗
-- - WHERE city = 'NYC'                                       ✗

-- Leftmost prefix rule:
-- Index on (A, B, C) can be used for:
-- - WHERE A
-- - WHERE A AND B
-- - WHERE A AND B AND C
-- - WHERE A AND C (uses A only)
-- But NOT efficiently for:
-- - WHERE B
-- - WHERE C
-- - WHERE B AND C
```

**Column ordering strategy:**

```sql
-- 1. Equality conditions before range conditions
CREATE INDEX idx_orders_optimal 
ON orders(customer_id, status, order_date);

-- Good for:
SELECT * FROM orders 
WHERE customer_id = 123           -- Equality
  AND status = 'pending'          -- Equality
  AND order_date > '2024-01-01';  -- Range

-- ✗ BAD ordering:
CREATE INDEX idx_orders_bad 
ON orders(order_date, customer_id, status);
-- Range first means customer_id and status can't be used efficiently

-- 2. High-equality columns before low-equality
CREATE INDEX idx_filter_order 
ON orders(customer_id, product_id, order_date);
-- customer_id used in 90% of queries
-- product_id used in 50% of queries
-- order_date used in 30% of queries

-- 3. Consider sort requirements
CREATE INDEX idx_sort_aware 
ON orders(customer_id, order_date DESC);
-- Supports: WHERE customer_id = X ORDER BY order_date DESC
-- Without additional sort!
```

### Covering Indexes

```sql
-- Query that requires table lookup
CREATE INDEX idx_email ON users(email);

SELECT user_id, first_name, last_name
FROM users
WHERE email = 'alice@example.com';

-- Execution:
-- 1. Search idx_email for 'alice@example.com' → get row pointer
-- 2. Fetch row from table → get user_id, first_name, last_name
-- 2 I/O operations

-- ✓ Covering index (includes all needed columns)
CREATE INDEX idx_email_covering 
ON users(email, user_id, first_name, last_name);

SELECT user_id, first_name, last_name
FROM users
WHERE email = 'alice@example.com';

-- Execution:
-- 1. Search idx_email_covering → get all data from index
-- 1 I/O operation (2x faster!)

-- PostgreSQL: Index-Only Scan
EXPLAIN ANALYZE
SELECT user_id, first_name FROM users WHERE email = 'alice@example.com';
-- Index Only Scan using idx_email_covering
-- Heap Fetches: 0  ← No table access!

-- MySQL: Using index
EXPLAIN
SELECT user_id, first_name FROM users WHERE email = 'alice@example.com';
-- Extra: Using index  ← Covering index used
```

**When to use covering indexes:**

```sql
-- ✓ Frequent queries with few columns
SELECT order_id, total_amount 
FROM orders 
WHERE customer_id = 123;

CREATE INDEX idx_customer_covering 
ON orders(customer_id, order_id, total_amount);

-- ✓ Analytical queries
SELECT customer_id, COUNT(*), SUM(total_amount)
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY customer_id;

CREATE INDEX idx_analytics_covering 
ON orders(order_date, customer_id, total_amount);

-- ✗ Too many columns (bloated index)
CREATE INDEX idx_bloated 
ON users(email, user_id, first_name, last_name, address, city, state, zip, phone, created_at);
-- Index becomes as large as table!

-- ✗ Volatile columns (frequently updated)
CREATE INDEX idx_volatile 
ON products(sku, price, inventory_count, last_updated);
-- Every price/inventory update rebuilds index
```

### Partial Indexes (PostgreSQL) / Filtered Indexes (SQL Server)

```sql
-- Index only subset of rows
-- Problem: Index on status (low selectivity)
CREATE INDEX idx_status ON orders(status);
-- 95% of orders are 'completed'
-- 5% are 'pending' or 'cancelled'
-- Index is 95% useless!

-- ✓ Solution: Partial index on active statuses only
CREATE INDEX idx_status_active 
ON orders(status)
WHERE status IN ('pending', 'processing');

-- Index size: 5% of full index
-- Query performance: Same or better for relevant queries

-- Use cases:
-- 1. Soft deletes
CREATE INDEX idx_active_users 
ON users(email)
WHERE deleted_at IS NULL;

-- 2. Time-based
CREATE INDEX idx_recent_orders 
ON orders(order_date)
WHERE order_date >= '2024-01-01';

-- 3. Status-based
CREATE INDEX idx_failed_payments 
ON payments(payment_date)
WHERE status = 'failed';

-- 4. Type-based
CREATE INDEX idx_premium_customers 
ON customers(customer_id, last_order_date)
WHERE tier = 'premium';
```

**Partial index benefits:**
- Smaller index size (faster scans, less storage)
- Faster writes (fewer index entries to update)
- Better cache utilization
- Focused on relevant data

**MySQL alternative (virtual columns):**

```sql
-- MySQL doesn't support partial indexes directly
-- Workaround: Virtual column + index

ALTER TABLE orders 
ADD COLUMN is_active BOOLEAN 
GENERATED ALWAYS AS (status IN ('pending', 'processing')) STORED;

CREATE INDEX idx_active_orders ON orders(is_active, order_date);

SELECT * FROM orders 
WHERE is_active = 1 
  AND order_date >= '2024-01-01';
```

### Expression/Functional Indexes

```sql
-- Problem: Function on indexed column prevents index usage
CREATE INDEX idx_email ON users(email);

SELECT * FROM users 
WHERE LOWER(email) = 'alice@example.com';
-- Index not used! (function applied)

-- ✓ Solution: Expression index
-- PostgreSQL:
CREATE INDEX idx_email_lower ON users(LOWER(email));

SELECT * FROM users 
WHERE LOWER(email) = 'alice@example.com';
-- Uses idx_email_lower!

-- MySQL 8.0+:
CREATE INDEX idx_email_lower ON users((LOWER(email)));

-- Common use cases:
-- 1. Case-insensitive searches
CREATE INDEX idx_username_lower ON users(LOWER(username));

-- 2. Date truncation
CREATE INDEX idx_order_month ON orders((DATE_TRUNC('month', order_date)));

SELECT * FROM orders 
WHERE DATE_TRUNC('month', order_date) = '2024-01-01';

-- 3. Computed values
CREATE INDEX idx_total_price ON order_items((quantity * unit_price));

SELECT * FROM order_items 
WHERE quantity * unit_price > 1000;

-- 4. JSON extraction (PostgreSQL)
CREATE INDEX idx_metadata_country ON users((metadata->>'country'));

SELECT * FROM users 
WHERE metadata->>'country' = 'USA';
```

### Index Maintenance

#### Detecting Unused Indexes

```sql
-- PostgreSQL: Find unused indexes
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan AS number_of_scans,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    pg_size_pretty(pg_relation_size(relid)) AS table_size
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
  AND idx_scan = 0  -- Never used
  AND indexrelname NOT LIKE '%_pkey'  -- Exclude primary keys
ORDER BY pg_relation_size(indexrelid) DESC;

-- Drop unused indexes (carefully!)
DROP INDEX idx_unused;

-- MySQL: Check index usage
SELECT 
    object_schema,
    object_name,
    index_name,
    count_star AS total_accesses,
    count_read AS reads,
    count_fetch AS fetches
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE object_schema = 'your_database'
  AND index_name IS NOT NULL
  AND count_star = 0
ORDER BY object_name;
```

#### Detecting Duplicate/Redundant Indexes

```sql
-- Redundant indexes:
-- idx_a: (user_id)
-- idx_ab: (user_id, created_at)
-- idx_a is redundant! (idx_ab covers all queries idx_a would handle)

-- PostgreSQL: Find duplicates
SELECT 
    pg_size_pretty(SUM(pg_relation_size(idx))::BIGINT) AS size,
    (array_agg(idx))[1] AS idx1,
    (array_agg(idx))[2] AS idx2
FROM (
    SELECT 
        indexrelid::regclass AS idx,
        (indrelid::text || E'\n' || indclass::text || E'\n' || 
         indkey::text || E'\n' || COALESCE(indexprs::text, '') || 
         E'\n' || COALESCE(indpred::text, '')) AS key
    FROM pg_index
) sub
GROUP BY key 
HAVING COUNT(*) > 1
ORDER BY SUM(pg_relation_size(idx)) DESC;

-- Find redundant (left-prefix) indexes
-- This is more complex - requires analyzing index definitions
-- Manual review recommended
```

#### Index Bloat

```sql
-- PostgreSQL: Check index bloat
CREATE EXTENSION pgstattuple;

SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    round(100 * pgstatindex(indexrelid)::numeric / 
          NULLIF(pg_relation_size(indexrelid), 0), 2) AS bloat_pct
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY pg_relation_size(indexrelid) DESC;

-- Fix bloat: REINDEX
REINDEX INDEX idx_bloated;
REINDEX TABLE users;  -- All indexes

-- Or rebuild index concurrently (no table lock)
CREATE INDEX CONCURRENTLY idx_new ON users(email);
DROP INDEX idx_old;
ALTER INDEX idx_new RENAME TO idx_old;

-- MySQL: Optimize table (rebuilds indexes)
OPTIMIZE TABLE users;
```

### Index Anti-Patterns

#### Over-Indexing

```sql
-- ❌ BAD: Index on every column
CREATE INDEX idx_user_id ON orders(user_id);
CREATE INDEX idx_product_id ON orders(product_id);
CREATE INDEX idx_order_date ON orders(order_date);
CREATE INDEX idx_status ON orders(status);
CREATE INDEX idx_total ON orders(total_amount);
CREATE INDEX idx_created ON orders(created_at);
-- 6 indexes! Every INSERT/UPDATE/DELETE updates all 6

-- ✓ GOOD: Indexes for actual query patterns
CREATE INDEX idx_user_date ON orders(user_id, order_date);
CREATE INDEX idx_product ON orders(product_id);
CREATE INDEX idx_status_date ON orders(status, order_date) 
WHERE status != 'completed';
-- 3 targeted indexes
```

**Cost of over-indexing:**
- Slower writes (every write updates all indexes)
- Increased storage (indexes can be larger than table)
- Slower optimizer (more options to evaluate)
- Cache pollution (indexes compete for buffer pool)

#### Single-Column Indexes for Multi-Column Queries

```sql
-- Query pattern:
SELECT * FROM orders 
WHERE customer_id = 123 
  AND order_date >= '2024-01-01';

-- ❌ BAD: Two single-column indexes
CREATE INDEX idx_customer ON orders(customer_id);
CREATE INDEX idx_date ON orders(order_date);

-- Database can:
-- Option 1: Use idx_customer, then filter by date (slow if many orders)
-- Option 2: Use idx_date, then filter by customer (slow if many dates)
-- Option 3: Index merge (scan both indexes, intersect) - overhead

-- ✓ GOOD: Composite index
CREATE INDEX idx_customer_date ON orders(customer_id, order_date);
-- Direct lookup: customer_id = 123, then scan date range
```

#### Indexing Low-Cardinality Columns

```sql
-- ❌ BAD: Index on boolean or low-cardinality column
CREATE INDEX idx_is_active ON users(is_active);
-- Only 2 values: true/false
-- If 90% are active, index returns 90% of table
-- Full table scan is faster!

-- ✓ GOOD: Partial index on minority value
CREATE INDEX idx_inactive_users ON users(user_id)
WHERE is_active = false;
-- Only indexes 10% of rows
```

### Index Selection Debugging

```sql
-- Why isn't my index being used?

-- 1. Check if index exists
-- PostgreSQL:
\d users  -- psql command
-- or
SELECT * FROM pg_indexes WHERE tablename = 'users';

-- MySQL:
SHOW INDEXES FROM users;

-- 2. Check query can use index
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';

-- 3. Common reasons index not used:
-- a) Function on indexed column
SELECT * FROM users WHERE LOWER(email) = 'alice@example.com';
-- Solution: Expression index or rewrite query

-- b) Implicit type conversion
-- Column: email VARCHAR
SELECT * FROM users WHERE email = 123;  -- Converts email to INT
-- Solution: Use correct type: email = '123'

-- c) Leading wildcard in LIKE
SELECT * FROM users WHERE email LIKE '%@example.com';
-- Solution: Reverse index or full-text search

-- d) OR conditions on different columns
SELECT * FROM users WHERE email = 'alice@example.com' OR phone = '555-1234';
-- Solution: UNION or separate queries

-- e) Optimizer thinks full scan is faster
SELECT * FROM users WHERE status = 'active';  -- 95% are active
-- Solution: May be correct! Verify with EXPLAIN ANALYZE

-- 4. Force index usage (for testing)
-- PostgreSQL:
SET enable_seqscan = off;  -- Force index scans
EXPLAIN ANALYZE SELECT ...;
SET enable_seqscan = on;   -- Reset

-- MySQL:
SELECT * FROM users FORCE INDEX (idx_email) WHERE email = 'alice@example.com';
```

### Monitoring Index Performance

```sql
-- PostgreSQL: Index usage statistics
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan DESC;

-- idx_scan: Number of index scans
-- idx_tup_read: Tuples read from index
-- idx_tup_fetch: Tuples fetched from table

-- Efficiency ratio
SELECT 
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    CASE 
        WHEN idx_tup_read > 0 
        THEN round(100.0 * idx_tup_fetch / idx_tup_read, 2)
        ELSE 0
    END AS fetch_pct
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
  AND idx_scan > 0
ORDER BY idx_scan DESC;

-- High fetch_pct: Index selective (good)
-- Low fetch_pct: Index scanning many entries (check if needed)

-- MySQL: Index statistics
SELECT 
    TABLE_SCHEMA,
    TABLE_NAME,
    INDEX_NAME,
    CARDINALITY,
    ROUND(STAT_VALUE * @@innodb_page_size / 1024 / 1024, 2) AS size_mb
FROM information_schema.STATISTICS
JOIN information_schema.INNODB_TABLESPACES_BRIEF 
    ON CONCAT(TABLE_SCHEMA, '/', TABLE_NAME) = NAME
WHERE TABLE_SCHEMA = 'your_database'
ORDER BY size_mb DESC;
```

---

## 4.3 Query Optimization Patterns

Recognizing and fixing common query anti-patterns dramatically improves performance.

### N+1 Query Problem

```sql
-- ❌ ANTI-PATTERN: N+1 queries
-- Application code (pseudo-code):
orders = SELECT * FROM orders WHERE customer_id = 123;
for each order in orders:
    items = SELECT * FROM order_items WHERE order_id = order.id;
    // Process items

-- Executes: 1 + N queries (1 for orders, N for items)
-- 100 orders = 101 queries!

-- ✓ SOLUTION 1: JOIN
SELECT 
    o.order_id,
    o.order_date,
    o.total_amount,
    oi.item_id,
    oi.product_id,
    oi.quantity,
    oi.price
FROM orders o
LEFT JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.customer_id = 123;
-- 1 query!

-- ✓ SOLUTION 2: Subquery with aggregation
SELECT 
    o.order_id,
    o.order_date,
    o.total_amount,
    (
        SELECT COUNT(*) 
        FROM order_items 
        WHERE order_id = o.order_id
    ) AS item_count,
    (
        SELECT JSON_AGG(
            JSON_BUILD_OBJECT(
                'product_id', product_id,
                'quantity', quantity,
                'price', price
            )
        )
        FROM order_items
        WHERE order_id = o.order_id
    ) AS items
FROM orders o
WHERE o.customer_id = 123;

-- ✓ SOLUTION 3: Batch query
-- Fetch order IDs first
order_ids = SELECT order_id FROM orders WHERE customer_id = 123;

-- Then fetch all items in one query
SELECT * FROM order_items 
WHERE order_id IN (1, 2, 3, ..., 100);
-- 2 queries total (much better than 101!)
```

### SELECT * Anti-Pattern

```sql
-- ❌ BAD: SELECT *
SELECT * FROM users WHERE user_id = 123;

-- Problems:
-- 1. Fetches unnecessary data (network overhead)
-- 2. Prevents covering index usage
-- 3. Breaks when schema changes
-- 4. Unclear what data is actually needed

-- ✓ GOOD: Explicit columns
SELECT user_id, first_name, last_name, email
FROM users
WHERE user_id = 123;

-- Benefits:
-- 1. Smaller result set
-- 2. Can use covering index
-- 3. Self-documenting
-- 4. Stable against schema changes

-- Example impact:
-- Table: users (50 columns, avg row size 2KB)
-- Query: SELECT * WHERE user_id IN (1, 2, ..., 1000)
-- Data transferred: 2MB

-- vs.
-- Query: SELECT user_id, email WHERE user_id IN (1, 2, ..., 1000)
-- Data transferred: 100KB (20x less!)
```

### Implicit Type Conversions

```sql
-- ❌ BAD: Type mismatch prevents index usage
-- Column: customer_id INT
SELECT * FROM orders WHERE customer_id = '123';
-- Converts '123' to INT (ok, index used)

-- But this:
-- Column: order_id VARCHAR
SELECT * FROM orders WHERE order_id = 123;
-- Converts ALL order_id values to INT (index not used!)

-- ✓ GOOD: Matching types
SELECT * FROM orders WHERE order_id = '123';

-- Date comparisons
-- ❌ BAD:
SELECT * FROM orders 
WHERE DATE(created_at) = '2024-01-15';
-- Function on column prevents index usage

-- ✓ GOOD:
SELECT * FROM orders 
WHERE created_at >= '2024-01-15' 
  AND created_at < '2024-01-16';
-- Range scan uses index
```

### Correlated Subqueries

```sql
-- ❌ SLOW: Correlated subquery
SELECT 
    p.product_id,
    p.product_name,
    (
        SELECT AVG(unit_price)
        FROM order_items
        WHERE product_id = p.product_id
    ) AS avg_price
FROM products p;
-- Subquery executes for EVERY product

-- ✓ FAST: JOIN with aggregation
SELECT 
    p.product_id,
    p.product_name,
    COALESCE(oi.avg_price, 0) AS avg_price
FROM products p
LEFT JOIN (
    SELECT 
        product_id,
        AVG(unit_price) AS avg_price
    FROM order_items
    GROUP BY product_id
) oi ON p.product_id = oi.product_id;
-- Aggregation executes once

-- ✓ FAST: Window function
SELECT DISTINCT
    p.product_id,
    p.product_name,
    AVG(oi.unit_price) OVER (PARTITION BY p.product_id) AS avg_price
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id;
```

### OR Conditions on Different Columns

```sql
-- ❌ SLOW: OR on different columns
SELECT * FROM users 
WHERE email = 'alice@example.com' 
   OR phone = '555-1234';
-- Can't use index efficiently (must check both conditions)

-- ✓ FAST: UNION
SELECT * FROM users WHERE email = 'alice@example.com'
UNION
SELECT * FROM users WHERE phone = '555-1234';
-- Each query uses its index, then combine

-- Or if duplicates acceptable:
SELECT * FROM users WHERE email = 'alice@example.com'
UNION ALL
SELECT * FROM users WHERE phone = '555-1234';
-- Faster (no deduplication)
```

### NOT IN with NULLs

```sql
-- ❌ DANGEROUS: NOT IN with potential NULLs
SELECT * FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id FROM orders
);

-- If orders.customer_id has ANY NULL:
-- Returns ZERO rows! (unexpected)

-- Why? NULL logic:
-- customer_id NOT IN (1, 2, NULL)
-- = customer_id != 1 AND customer_id != 2 AND customer_id != NULL
-- = customer_id != 1 AND customer_id != 2 AND UNKNOWN
-- = UNKNOWN (not TRUE, so filtered out)

-- ✓ SAFE: NOT EXISTS
SELECT * FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.customer_id = c.customer_id
);

-- ✓ SAFE: NOT IN with NULL filter
SELECT * FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id FROM orders
    WHERE customer_id IS NOT NULL
);

-- ✓ SAFE: LEFT JOIN
SELECT c.*
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

### Pagination Anti-Patterns

```sql
-- ❌ SLOW: OFFSET for deep pagination
SELECT * FROM products
ORDER BY product_id
LIMIT 100 OFFSET 500000;
-- Scans 500,100 rows, returns 100
-- Gets slower as offset increases

-- ✓ FAST: Keyset pagination (seek method)
-- First page:
SELECT * FROM products
WHERE product_id > 0
ORDER BY product_id
LIMIT 100;
-- Returns: product_id 1-100, last_id = 100

-- Next page:
SELECT * FROM products
WHERE product_id > 100  -- Use last_id from previous page
ORDER BY product_id
LIMIT 100;
-- Returns: product_id 101-200

-- Benefits:
-- - Constant time (no offset scanning)
-- - Works for any page depth
-- - More efficient

-- For complex sorting:
SELECT * FROM products
WHERE (category, price, product_id) > ('Electronics', 99.99, 12345)
ORDER BY category, price, product_id
LIMIT 100;

-- Composite key comparison ensures consistent ordering
```

### Counting Rows

```sql
-- ❌ SLOW: COUNT(*) on large tables
SELECT COUNT(*) FROM orders;
-- Full table scan

-- ✓ FAST: Use statistics (approximate)
-- PostgreSQL:
SELECT reltuples::BIGINT AS estimate
FROM pg_class
WHERE relname = 'orders';

-- MySQL:
SELECT TABLE_ROWS
FROM information_schema.TABLES
WHERE TABLE_NAME = 'orders';

-- ✓ FAST: Index-only count
SELECT COUNT(order_id) FROM orders;
-- Can use index-only scan if order_id indexed

-- ✓ FAST: Maintain counter table
CREATE TABLE table_stats (
    table_name VARCHAR(50),
    row_count BIGINT,
    updated_at TIMESTAMP
);

-- Update via triggers or scheduled job
-- Then query is instant:
SELECT row_count FROM table_stats WHERE table_name = 'orders';
```

---

## 4.4 Schema Design for Performance

Schema design choices have long-term performance implications. Optimizing schema upfront prevents costly refactoring later.

### Normalization vs Denormalization

**Normalization benefits:**
- No data duplication
- Easier updates (single source of truth)
- Smaller table sizes
- Referential integrity

**Denormalization benefits:**
- Fewer joins
- Faster reads
- Simpler queries
- Better for read-heavy workloads

```sql
-- Normalized (3NF)
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    country VARCHAR(50)
);

-- Query requires JOIN:
SELECT o.order_id, c.customer_name, c.country
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

-- Denormalized (duplicates customer data)
CREATE TABLE orders_denorm (
    order_id INT PRIMARY KEY,
    customer_id INT,
    customer_name VARCHAR(100),  -- Duplicated
    customer_country VARCHAR(50), -- Duplicated
    order_date DATE
);

-- No JOIN needed:
SELECT order_id, customer_name, customer_country
FROM orders_denorm;

-- Trade-offs:
-- Normalized: Consistent data, slower reads (JOIN)
-- Denormalized: Fast reads, risk of inconsistency, harder updates
```

**When to denormalize:**
- Read-heavy workload (100:1 read:write ratio)
- Data rarely changes (country codes, product categories)
- JOIN performance is critical bottleneck
- You can maintain consistency (triggers, app logic)

**Partial denormalization strategies:**

```sql
-- 1. Summary/aggregation tables
CREATE TABLE daily_sales_summary (
    sale_date DATE PRIMARY KEY,
    total_orders INT,
    total_revenue DECIMAL(15,2),
    avg_order_value DECIMAL(10,2)
);

-- Refreshed nightly via job
-- Queries are instant instead of aggregating millions of rows

-- 2. Materialized views
CREATE MATERIALIZED VIEW customer_order_summary AS
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS total_orders,
    SUM(o.total_amount) AS lifetime_value,
    MAX(o.order_date) AS last_order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;

CREATE INDEX idx_customer_summary_ltv 
ON customer_order_summary(lifetime_value);

REFRESH MATERIALIZED VIEW customer_order_summary;

-- 3. Cached computed columns (MySQL 5.7+)
ALTER TABLE orders ADD COLUMN items_count INT;
ALTER TABLE orders ADD COLUMN avg_item_price DECIMAL(10,2);

UPDATE orders o
SET items_count = (SELECT COUNT(*) FROM order_items WHERE order_id = o.order_id),
    avg_item_price = (SELECT AVG(unit_price) FROM order_items WHERE order_id = o.order_id);

-- Or use GENERATED columns
ALTER TABLE orders 
ADD COLUMN total_calculated DECIMAL(10,2)
GENERATED ALWAYS AS (quantity * unit_price) STORED;
```

### Data Type Selection

```sql
-- ❌ BAD: Oversized types
CREATE TABLE users (
    user_id BIGINT,          -- 8 bytes (supports 9 quintillion users)
    username VARCHAR(255),   -- Variable, but probably 20 chars max
    age INT,                 -- 4 bytes (supports age up to 2 billion)
    is_active VARCHAR(10)    -- Storing 'true'/'false' as string
);

-- ✓ GOOD: Right-sized types
CREATE TABLE users (
    user_id INT,             -- 4 bytes (supports 2 billion users)
    username VARCHAR(50),    -- Right-sized
    age SMALLINT,            -- 2 bytes (supports 0-65535)
    is_active BOOLEAN        -- 1 byte
);

-- Impact on 10 million rows:
-- Bad:  (8 + 255 + 4 + 10) × 10M = 2.77 GB
-- Good: (4 + 50 + 2 + 1) × 10M = 570 MB
-- Savings: 2.2 GB (79% smaller!)

-- Plus:
-- - Better index performance (smaller indexes)
-- - More rows fit in buffer pool
-- - Faster network transfer
```

**Type selection guide:**

```sql
-- Integers:
-- TINYINT: -128 to 127 (1 byte)
-- SMALLINT: -32,768 to 32,767 (2 bytes)
-- INT: -2B to 2B (4 bytes)
-- BIGINT: -9 quintillion to 9 quintillion (8 bytes)

-- Strings:
-- CHAR(n): Fixed length, padded (fast comparisons)
-- VARCHAR(n): Variable length (saves space)
-- TEXT: Large text (use sparingly, can't be indexed fully)

-- Use CHAR for:
-- - Fixed-length codes (country codes: CHAR(2))
-- - Status fields with fixed values (CHAR(10))

-- Use VARCHAR for:
-- - Names, emails, addresses (variable length)
-- - User-generated content with max length

-- Dates/Times:
-- DATE: Date only (3 bytes)
-- DATETIME: Date and time (8 bytes MySQL, 8 bytes PostgreSQL)
-- TIMESTAMP: Unix timestamp with timezone (4 bytes)
-- TIME: Time only (3 bytes)

-- Use DATE for: birth_date, event_date
-- Use TIMESTAMP for: created_at, updated_at (auto-timezone)

-- Decimals:
-- DECIMAL(p, s): Exact decimal (DECIMAL(10,2) for money)
-- FLOAT/DOUBLE: Approximate (scientific calculations)

-- Always use DECIMAL for money (no rounding errors)
```

### Partitioning

```sql
-- Horizontal partitioning: Split table by rows
-- Vertical partitioning: Split table by columns

-- Range partitioning (PostgreSQL 10+)
CREATE TABLE orders (
    order_id BIGINT,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2)
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2023 
    PARTITION OF orders 
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

CREATE TABLE orders_2024 
    PARTITION OF orders 
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');

-- Benefits:
-- - Queries on recent data only scan relevant partitions
-- - Archiving old data: DROP TABLE orders_2022
-- - Faster queries (partition pruning)

-- Query automatically uses right partition:
SELECT * FROM orders 
WHERE order_date >= '2024-06-01';
-- Only scans orders_2024 partition

-- List partitioning
CREATE TABLE users (
    user_id BIGINT,
    country VARCHAR(2),
    username VARCHAR(50)
) PARTITION BY LIST (country);

CREATE TABLE users_us PARTITION OF users FOR VALUES IN ('US');
CREATE TABLE users_uk PARTITION OF users FOR VALUES IN ('UK');
CREATE TABLE users_other PARTITION OF users DEFAULT;

-- Hash partitioning (distribute evenly)
CREATE TABLE events (
    event_id BIGINT,
    user_id BIGINT,
    event_type VARCHAR(50),
    created_at TIMESTAMP
) PARTITION BY HASH (user_id);

CREATE TABLE events_p0 PARTITION OF events 
    FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE events_p1 PARTITION OF events 
    FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE events_p2 PARTITION OF events 
    FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE events_p3 PARTITION OF events 
    FOR VALUES WITH (MODULUS 4, REMAINDER 3);
```

**When to partition:**
- Tables > 100 GB
- Time-series data (logs, events, orders)
- Clear partitioning key (date, region, tenant_id)
- Queries filter on partition key
- Need to archive old data regularly

### Vertical Partitioning (Column Stores)

```sql
-- Problem: Wide table with rarely-accessed columns
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    sku VARCHAR(50),
    name VARCHAR(200),
    price DECIMAL(10,2),
    -- Frequently accessed ↑
    
    description TEXT,          -- 10KB average
    specifications TEXT,       -- 5KB average
    marketing_copy TEXT,       -- 8KB average
    -- Rarely accessed ↓
);

-- 90% of queries: SELECT product_id, sku, name, price
-- But table scan loads ALL columns (including 23KB of rarely-used text)

-- ✓ Solution: Vertical partitioning
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    sku VARCHAR(50),
    name VARCHAR(200),
    price DECIMAL(10,2)
);

CREATE TABLE product_details (
    product_id INT PRIMARY KEY,
    description TEXT,
    specifications TEXT,
    marketing_copy TEXT,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Common query (fast):
SELECT product_id, sku, name, price FROM products;

-- Detailed view (when needed):
SELECT p.*, pd.description, pd.specifications
FROM products p
JOIN product_details pd ON p.product_id = pd.product_id
WHERE p.product_id = 123;
```

---

## Frequently Asked Questions

**Q1: Should I index foreign keys?**

**A:** Yes, almost always.

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,  -- Foreign key
    product_id INT,   -- Foreign key
    order_date DATE
);

-- ✓ Index foreign keys
CREATE INDEX idx_customer ON orders(customer_id);
CREATE INDEX idx_product ON orders(product_id);

-- Why?
-- 1. JOINs use foreign keys
SELECT * FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
-- Without index: Full table scan of orders

-- 2. Referential integrity checks
DELETE FROM customers WHERE customer_id = 123;
-- Database checks: SELECT 1 FROM orders WHERE customer_id = 123
-- Without index: Full table scan

-- 3. Cascading updates/deletes
UPDATE customers SET customer_id = 999 WHERE customer_id = 123;
-- Updates: UPDATE orders SET customer_id = 999 WHERE customer_id = 123
-- Without index: Full table scan

-- Exception: Don't index if FK column has very low cardinality
-- AND you never filter/join on it
```

---

**Q2: How do I optimize a slow COUNT(*) query?**

**A:** Multiple strategies depending on requirements.

**Strategy 1: Approximate count (statistics)**
```sql
-- PostgreSQL (instant, approximate):
SELECT reltuples::BIGINT AS estimate
FROM pg_class
WHERE relname = 'orders';

-- MySQL (instant, approximate):
SELECT TABLE_ROWS
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'mydb' AND TABLE_NAME = 'orders';

-- Accuracy: ±10-20%
-- Use when: Exact count not critical (dashboards, estimates)
```

**Strategy 2: Index-only count**
```sql
-- If counting specific column with index:
SELECT COUNT(customer_id) FROM orders;
-- Uses index-only scan if customer_id indexed

-- Better than:
SELECT COUNT(*) FROM orders;
-- May require table scan
```

**Strategy 3: Materialized counter**
```sql
-- Create counter table
CREATE TABLE table_counts (
    table_name VARCHAR(50) PRIMARY KEY,
    row_count BIGINT,
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Update via triggers
CREATE TRIGGER update_orders_count
AFTER INSERT OR DELETE OR UPDATE ON orders
FOR EACH STATEMENT
EXECUTE FUNCTION update_count_function();

-- Or scheduled job (less real-time, more efficient):
-- Cron: Every 5 minutes
INSERT INTO table_counts (table_name, row_count)
VALUES ('orders', (SELECT COUNT(*) FROM orders))
ON CONFLICT (table_name)
DO UPDATE SET row_count = EXCLUDED.row_count, updated_at = NOW();

-- Query is instant:
SELECT row_count FROM table_counts WHERE table_name = 'orders';
```

**Strategy 4: Conditional count with index**
```sql
-- Instead of counting all rows:
SELECT COUNT(*) FROM orders WHERE status = 'pending';

-- Create partial index:
CREATE INDEX idx_pending ON orders(status) WHERE status = 'pending';

-- Count becomes index-only operation
SELECT COUNT(*) FROM orders WHERE status = 'pending';
```

---

**Q3: When should I use a covering index vs just filtering columns in SELECT?**

**A:** Use covering indexes when read performance is critical and the extra index size is acceptable.

**Covering index benefits:**
```sql
-- Without covering index:
CREATE INDEX idx_customer ON orders(customer_id);

SELECT customer_id, order_date, total_amount
FROM orders
WHERE customer_id = 123;

-- Execution:
-- 1. Index scan finds matching rows → get row pointers
-- 2. Random access to table → fetch order_date, total_amount
-- Cost: Index scan + table lookups

-- With covering index:
CREATE INDEX idx_customer_covering 
ON orders(customer_id, order_date, total_amount);

SELECT customer_id, order_date, total_amount
FROM orders
WHERE customer_id = 123;

-- Execution:
-- 1. Index scan finds matching rows → all data in index
-- Cost: Index scan only (no table access)
-- Typically 2-5x faster
```

**When to use covering indexes:**

✓ **Use when:**
- Query is very frequent (executed 1000s of times/second)
- Few additional columns needed (2-4 extra columns)
- Columns are not frequently updated
- Read performance is critical

❌ **Don't use when:**
- Many columns needed (index becomes huge)
- Columns frequently updated (every update rebuilds index)
- Query is rare
- Table is small (full scan is fast anyway)

**Example decision matrix:**

```sql
-- Scenario 1: User login check (millions/day)
SELECT user_id, password_hash, salt
FROM users
WHERE email = 'user@example.com';

-- ✓ Covering index worth it
CREATE INDEX idx_email_covering 
ON users(email, user_id, password_hash, salt);
-- Only 4 columns, query is critical, data rarely changes

-- Scenario 2: Admin report (once/day)
SELECT * FROM orders WHERE created_at > NOW() - INTERVAL '90 days';

-- ✗ Covering index not worth it
CREATE INDEX idx_created ON orders(created_at);
-- Query is rare, SELECT * needs many columns
-- Simple index is sufficient
```

---

## Interview Questions

**Question 1: You notice queries on users table are slow. EXPLAIN shows "Using filesort". What does this mean and how do you fix it?**

**Difficulty:** Mid-Level

**Answer:**

**What "Using filesort" means:**
- Database must sort results (not using index for sorting)
- Creates temporary table, sorts data, returns results
- Expensive operation, especially for large result sets

**Example:**

```sql
-- Query
SELECT * FROM users
WHERE country = 'USA'
ORDER BY created_at DESC
LIMIT 100;

-- EXPLAIN shows:
-- Extra: Using where; Using filesort

-- Means:
-- 1. Filter by country (Using where)
-- 2. Sort filtered results by created_at (Using filesort)
-- 3. Return top 100
```

**Fixing strategies:**

**Solution 1: Add index with ORDER BY column**
```sql
-- Current index (doesn't help with sort):
CREATE INDEX idx_country ON users(country);

-- ✓ Better: Include sort column
CREATE INDEX idx_country_created ON users(country, created_at DESC);

-- Now EXPLAIN shows:
-- Extra: Using index condition
-- No filesort! Index provides both filtering and sorting
```

**Solution 2: Adjust existing composite index order**
```sql
-- If you have:
CREATE INDEX idx_created_country ON users(created_at, country);

-- Query: WHERE country = 'USA' ORDER BY created_at
-- Can't use index efficiently (country is second column)

-- Fix: Reorder index
DROP INDEX idx_created_country;
CREATE INDEX idx_country_created ON users(country, created_at);
```

**Solution 3: Reduce result set before sorting**
```sql
-- If sorting many rows is unavoidable,
-- reduce rows first:

-- Bad: Sort millions, return 100
SELECT * FROM users
ORDER BY created_at DESC
LIMIT 100;

-- Better: Filter first, then sort
SELECT * FROM users
WHERE status = 'active'  -- Reduces to 100K rows
ORDER BY created_at DESC
LIMIT 100;

CREATE INDEX idx_active_created ON users(status, created_at);
```

**Verification:**
```sql
-- After creating index, verify:
EXPLAIN SELECT * FROM users
WHERE country = 'USA'
ORDER BY created_at DESC
LIMIT 100;

-- Should show:
-- type: ref or range
-- key: idx_country_created
-- Extra: Using index (or no filesort mentioned)
```

**Why This Matters:** "Using filesort" is one of the most common performance issues. Understanding index column ordering for sort optimization is essential.

---

**Question 2: Your application has a slow query that joins 5 tables. How do you systematically optimize it?**

**Difficulty:** Senior

**Answer:**

**Systematic optimization process:**

**Step 1: Understand current performance**
```sql
-- Original query
EXPLAIN ANALYZE
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name,
    s.store_name,
    sh.shipped_date
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN stores s ON o.store_id = s.store_id
LEFT JOIN shipments sh ON o.order_id = sh.order_id
WHERE o.order_date >= '2024-01-01'
  AND c.country = 'USA';

-- Record:
-- - Execution time
-- - Row counts (estimated vs actual)
-- - Join algorithms used
-- - Indexes used
```

**Step 2: Analyze execution plan**
```sql
-- Look for red flags:
-- ✗ Seq Scan on large tables
-- ✗ Hash Join on large tables without indexes
-- ✗ Estimated rows << actual rows (bad statistics)
-- ✗ Nested Loop with many iterations
-- ✗ Using temporary; Using filesort
```

**Step 3: Fix missing indexes**
```sql
-- Check each join condition has index:
CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_store ON orders(store_id);
CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_product ON order_items(product_id);
CREATE INDEX idx_shipments_order ON shipments(order_id);

-- Check WHERE conditions have indexes:
CREATE INDEX idx_orders_date ON orders(order_date);
CREATE INDEX idx_customers_country ON customers(country);

-- Or composite indexes for multiple conditions:
CREATE INDEX idx_orders_date_customer_store 
ON orders(order_date, customer_id, store_id);
```

**Step 4: Update statistics**
```sql
-- PostgreSQL:
ANALYZE orders;
ANALYZE customers;
ANALYZE order_items;
ANALYZE products;
ANALYZE stores;
ANALYZE shipments;

-- MySQL:
ANALYZE TABLE orders;
ANALYZE TABLE customers;
-- ... etc
```

**Step 5: Rewrite query if needed**
```sql
-- Option 1: Filter early with CTE
WITH recent_orders AS (
    SELECT order_id, customer_id, store_id
    FROM orders
    WHERE order_date >= '2024-01-01'
),
usa_customers AS (
    SELECT customer_id, customer_name
    FROM customers
    WHERE country = 'USA'
)
SELECT 
    ro.order_id,
    uc.customer_name,
    p.product_name,
    s.store_name,
    sh.shipped_date
FROM recent_orders ro
JOIN usa_customers uc ON ro.customer_id = uc.customer_id
JOIN order_items oi ON ro.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN stores s ON ro.store_id = s.store_id
LEFT JOIN shipments sh ON ro.order_id = sh.order_id;

-- Option 2: Denormalize if query is critical
-- Add customer_country to orders table
ALTER TABLE orders ADD COLUMN customer_country VARCHAR(2);

CREATE INDEX idx_orders_country_date 
ON orders(customer_country, order_date);

-- Update via trigger or application
-- Then query becomes simpler:
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name,
    s.store_name,
    sh.shipped_date
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN stores s ON o.store_id = s.store_id
LEFT JOIN shipments sh ON o.order_id = sh.order_id
WHERE o.customer_country = 'USA'
  AND o.order_date >= '2024-01-01';
```

**Step 6: Consider materialized view**
```sql
-- If query is very frequent and data changes slowly:
CREATE MATERIALIZED VIEW usa_order_details AS
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name,
    s.store_name,
    sh.shipped_date,
    o.order_date
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN stores s ON o.store_id = s.store_id
LEFT JOIN shipments sh ON o.order_id = sh.order_id
WHERE c.country = 'USA';

CREATE INDEX idx_mv_order_date ON usa_order_details(order_date);

-- Refresh periodically
REFRESH MATERIALIZED VIEW usa_order_details;

-- Query is now simple and fast:
SELECT * FROM usa_order_details
WHERE order_date >= '2024-01-01';
```

**Step 7: Measure improvement**
```sql
-- Run EXPLAIN ANALYZE again
-- Compare:
-- - Execution time (before vs after)
-- - Rows examined
-- - Index usage
-- - Join algorithms

-- Document improvement:
-- Before: 5000ms
-- After: 50ms (100x faster)
```

**Why This Matters:** Multi-table joins are common in production. Systematic optimization methodology prevents random "trying things" and ensures sustainable performance improvements.

---

## Key Takeaways

* **Index selectivity matters** - high cardinality columns make good candidates
* **Composite index order is critical** - match query patterns, equality before range
* **Covering indexes avoid table lookups** - 2-5x faster for read-heavy queries
* **Partial indexes reduce size and cost** - index only relevant subset
* **Expression indexes enable function-based queries** - LOWER(email), DATE_TRUNC()
* **Monitor index usage** - drop unused indexes, detect duplicates
* **N+1 queries kill performance** - always batch or JOIN
* **SELECT * wastes resources** - specify needed columns
* **Avoid correlated subqueries** - rewrite as JOIN or window function
* **Schema design impacts performance** - right-size types, consider denormalization
* **Partitioning helps huge tables** - time-series data, multi-tenant systems
* **Systematic optimization** - measure, analyze, fix, verify

---

*This concludes Part 4: Query Optimization & Performance*
