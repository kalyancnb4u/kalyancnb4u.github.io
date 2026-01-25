---
title: "SQL Mastery - Supplement 7: SQL Quick Reference & Interview Prep"
date: 2024-02-16 00:00:00 +0530
categories: [SQL, SQL Mastery]
tags: [SQL, Database, Cheat-sheet, Interview, Quick-reference, MySQL, PostgreSQL]
---

# SQL Mastery Quick Reference & Interview Prep

Comprehensive cheat sheet, common patterns, and interview question bank for SQL mastery.

---

## Table of Contents

1. [SQL Execution Order Quick Reference](#sql-execution-order-quick-reference)
2. [Common SQL Patterns](#common-sql-patterns)
3. [Performance Optimization Checklist](#performance-optimization-checklist)
4. [MySQL vs PostgreSQL Quick Comparison](#mysql-vs-postgresql-quick-comparison)
5. [Comprehensive Interview Question Bank](#comprehensive-interview-question-bank)
6. [Common Anti-Patterns to Avoid](#common-anti-patterns-to-avoid)
7. [Production Readiness Checklist](#production-readiness-checklist)

---

## SQL Execution Order Quick Reference

### Logical Processing Order

```
1. FROM + JOINs        → Create source dataset
2. WHERE               → Filter rows
3. GROUP BY            → Group rows
4. HAVING              → Filter groups
5. SELECT              → Choose/compute columns
6. DISTINCT            → Remove duplicates
7. ORDER BY            → Sort results
8. LIMIT/OFFSET        → Paginate results
```

**Key Implications:**

- **WHERE executes before SELECT** → Can't use SELECT aliases in WHERE
- **HAVING executes after GROUP BY** → Can use aggregate functions
- **ORDER BY executes after SELECT** → Can use SELECT aliases
- **GROUP BY executes before SELECT** → Usually can't use aliases (MySQL allows as extension)

---

## Common SQL Patterns

### 1. Top N Per Group

**Problem:** Get top 3 products per category by price.

```sql
-- Using window functions (best)
WITH ranked AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) AS rn
    FROM products
)
SELECT * FROM ranked WHERE rn <= 3;

-- PostgreSQL DISTINCT ON
SELECT DISTINCT ON (category)
    *
FROM products
ORDER BY category, price DESC;
```

### 2. Running Totals

**Problem:** Calculate cumulative sum of daily sales.

```sql
SELECT 
    sale_date,
    daily_revenue,
    SUM(daily_revenue) OVER (ORDER BY sale_date) AS cumulative_revenue
FROM daily_sales;
```

### 3. Finding Gaps in Sequences

**Problem:** Find missing IDs in a sequence.

```sql
-- Generate full sequence and anti-join
WITH RECURSIVE seq AS (
    SELECT 1 AS num
    UNION ALL
    SELECT num + 1 FROM seq WHERE num < (SELECT MAX(id) FROM records)
)
SELECT num AS missing_id
FROM seq
WHERE num NOT IN (SELECT id FROM records);

-- Or using LAG
SELECT 
    id AS current_id,
    LEAD(id) OVER (ORDER BY id) AS next_id,
    LEAD(id) OVER (ORDER BY id) - id AS gap_size
FROM records
WHERE LEAD(id) OVER (ORDER BY id) - id > 1;
```

### 4. Pivot (Rows to Columns)

**Problem:** Transform monthly sales rows into columns.

```sql
SELECT 
    product_id,
    SUM(CASE WHEN month = 1 THEN sales ELSE 0 END) AS jan_sales,
    SUM(CASE WHEN month = 2 THEN sales ELSE 0 END) AS feb_sales,
    SUM(CASE WHEN month = 3 THEN sales ELSE 0 END) AS mar_sales
    -- ... more months
FROM monthly_sales
GROUP BY product_id;
```

### 5. Finding Duplicates

**Problem:** Find duplicate email addresses.

```sql
SELECT email, COUNT(*) AS duplicate_count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;

-- Get actual duplicate records
SELECT u.*
FROM users u
INNER JOIN (
    SELECT email FROM users GROUP BY email HAVING COUNT(*) > 1
) dups ON u.email = dups.email
ORDER BY u.email, u.user_id;
```

### 6. Orphaned Records

**Problem:** Find orders without customers.

```sql
SELECT o.*
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;

-- Alternative with NOT EXISTS (often faster)
SELECT *
FROM orders o
WHERE NOT EXISTS (
    SELECT 1 FROM customers c WHERE c.customer_id = o.customer_id
);
```

### 7. Conditional Aggregation

**Problem:** Count orders by status in one query.

```sql
SELECT 
    COUNT(*) AS total_orders,
    COUNT(CASE WHEN status = 'completed' THEN 1 END) AS completed,
    COUNT(CASE WHEN status = 'pending' THEN 1 END) AS pending,
    COUNT(CASE WHEN status = 'cancelled' THEN 1 END) AS cancelled,
    SUM(CASE WHEN status = 'completed' THEN amount ELSE 0 END) AS completed_revenue
FROM orders;
```

### 8. Hierarchical Queries

**Problem:** Get employee reporting chain.

```sql
WITH RECURSIVE emp_hierarchy AS (
    -- Anchor: Start with CEO
    SELECT employee_id, employee_name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Find direct reports
    SELECT e.employee_id, e.employee_name, e.manager_id, eh.level + 1
    FROM employees e
    JOIN emp_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM emp_hierarchy ORDER BY level, employee_name;
```

### 9. Deduplication

**Problem:** Keep only newest record per email.

```sql
-- Using window function
DELETE FROM users
WHERE user_id IN (
    SELECT user_id
    FROM (
        SELECT 
            user_id,
            ROW_NUMBER() OVER (PARTITION BY email ORDER BY created_at DESC) AS rn
        FROM users
    ) ranked
    WHERE rn > 1
);

-- PostgreSQL using DISTINCT ON
DELETE FROM users
WHERE user_id NOT IN (
    SELECT DISTINCT ON (email) user_id
    FROM users
    ORDER BY email, created_at DESC
);
```

### 10. Date Range Overlap

**Problem:** Find overlapping reservations.

```sql
SELECT r1.*, r2.*
FROM reservations r1
JOIN reservations r2 
    ON r1.room_id = r2.room_id
    AND r1.reservation_id < r2.reservation_id
    AND r1.start_date < r2.end_date
    AND r2.start_date < r1.end_date;
```

---

## Performance Optimization Checklist

### Query-Level Optimizations

- [ ] **SELECT only needed columns** (not SELECT *)
- [ ] **Use appropriate indexes** on WHERE, JOIN, ORDER BY columns
- [ ] **Avoid functions on indexed columns** in WHERE clause
- [ ] **Use EXISTS instead of IN** for large subqueries
- [ ] **Replace correlated subqueries** with JOINs or window functions
- [ ] **Use LIMIT** to restrict result set size
- [ ] **Avoid SELECT DISTINCT** to hide join problems
- [ ] **Use UNION ALL** instead of UNION when duplicates don't matter
- [ ] **Batch INSERT/UPDATE/DELETE** operations
- [ ] **Use prepared statements** to prevent SQL injection and improve performance

### Index Optimization

- [ ] **Index foreign keys** used in JOINs
- [ ] **Create composite indexes** for multi-column filters
- [ ] **Use covering indexes** to avoid table lookups
- [ ] **Order composite index columns** by selectivity (most selective first)
- [ ] **Don't over-index** (each index slows writes)
- [ ] **Remove unused indexes** (check with EXPLAIN)
- [ ] **Use partial indexes** for common filters (PostgreSQL)
- [ ] **Consider expression indexes** for computed columns

### Data Model Optimization

- [ ] **Normalize to reduce redundancy** (3NF typically)
- [ ] **Denormalize for read-heavy workloads** when appropriate
- [ ] **Use appropriate data types** (SMALLINT vs INT, VARCHAR(50) vs VARCHAR(255))
- [ ] **Add NOT NULL constraints** where possible
- [ ] **Partition large tables** by date or range
- [ ] **Archive old data** to keep tables small
- [ ] **Use INTEGER for IDs** not VARCHAR

### Application-Level Optimizations

- [ ] **Implement caching** (Redis, Memcached)
- [ ] **Use connection pooling**
- [ ] **Avoid N+1 queries** (eager load relationships)
- [ ] **Paginate results** (don't fetch all rows)
- [ ] **Use read replicas** for read-heavy workloads
- [ ] **Queue long-running operations** (background jobs)
- [ ] **Monitor slow query log**

### EXPLAIN Analysis

- [ ] **Check query execution plan** with EXPLAIN
- [ ] **Look for full table scans** (type: ALL)
- [ ] **Verify index usage** (type: ref, eq_ref, range)
- [ ] **Check rows examined** (lower is better)
- [ ] **Look for filesort/temporary** (slower operations)
- [ ] **Verify join type** (nested loop, hash, merge)

---

## MySQL vs PostgreSQL Quick Comparison

| Feature | MySQL | PostgreSQL |
|---------|-------|------------|
| **ACID compliance** | ✅ InnoDB | ✅ Always |
| **Window functions** | ✅ 8.0+ | ✅ Always |
| **CTEs** | ✅ 8.0+ | ✅ Always |
| **Recursive CTEs** | ✅ 8.0+ | ✅ Always |
| **FULL OUTER JOIN** | ❌ Use UNION | ✅ Native |
| **LATERAL joins** | ⚠️ 8.0.14+ (limited) | ✅ Full support |
| **BOOLEAN type** | `TINYINT(1)` | True `BOOLEAN` |
| **JSONB** | ❌ Only `JSON` | ✅ Binary JSON |
| **Arrays** | ❌ Not supported | ✅ Native arrays |
| **String concatenation** | `CONCAT(a,b)` | `a || b` |
| **String aggregation** | `GROUP_CONCAT()` | `STRING_AGG()` |
| **NULLS FIRST/LAST** | ❌ Workaround | ✅ Native |
| **DISTINCT ON** | ❌ Not supported | ✅ Supported |
| **Materialized views** | ❌ Not native | ✅ Native |
| **Partial indexes** | ❌ Not supported | ✅ Supported |
| **Expression indexes** | ⚠️ Limited | ✅ Full support |
| **Row-level security** | ❌ Not native | ✅ Native |
| **Parallel queries** | ⚠️ Limited (8.0.14+) | ✅ Mature |
| **Case sensitivity** | Case-insensitive (default) | Case-sensitive |
| **Replication** | Master-slave (async) | Streaming (async/sync) |
| **License** | GPL (or commercial) | PostgreSQL License (permissive) |

**When to Use MySQL:**
- Simpler deployment/management
- Read-heavy workloads with master-slave replication
- Need commercial support (Oracle)
- Existing MySQL expertise in team
- Compatibility with legacy applications

**When to Use PostgreSQL:**
- Complex queries and analytics
- Need for advanced features (arrays, JSON, full-text search)
- Strong ACID requirements
- Data integrity critical
- Advanced indexing needed (partial, expression)
- Open-source preference

---

## Comprehensive Interview Question Bank

### Junior Level (1-2 years experience)

**1. What's the difference between INNER JOIN and LEFT JOIN?**

**Answer:** INNER JOIN returns only matching rows from both tables. LEFT JOIN returns all rows from the left table plus matching rows from the right table (NULL for non-matches).

```sql
-- INNER JOIN: Only customers with orders
SELECT c.name, o.order_id
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id;

-- LEFT JOIN: All customers, even without orders
SELECT c.name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id;
```

---

**2. How do you find duplicate records in a table?**

**Answer:** Use GROUP BY with HAVING COUNT(*) > 1:

```sql
SELECT email, COUNT(*) AS count
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

---

**3. What's the difference between WHERE and HAVING?**

**Answer:** WHERE filters rows before GROUP BY; HAVING filters groups after aggregation. WHERE cannot use aggregate functions; HAVING can.

```sql
SELECT department, AVG(salary)
FROM employees
WHERE status = 'active'  -- Filter individuals
GROUP BY department
HAVING AVG(salary) > 50000;  -- Filter groups
```

---

**4. Explain the difference between DELETE and TRUNCATE.**

**Answer:**
- **DELETE:** Removes rows one by one, can use WHERE clause, slower, can be rolled back, triggers fire
- **TRUNCATE:** Removes all rows, fast, can't use WHERE, can't be rolled back (in some databases), triggers don't fire

```sql
DELETE FROM users WHERE inactive_days > 365;  -- Conditional
TRUNCATE TABLE temp_data;  -- All rows, fast
```

---

**5. What does NULL mean in SQL?**

**Answer:** NULL represents unknown or missing data. It's not zero, not empty string, not false—it's "unknown."

Key behaviors:
- NULL = NULL returns NULL (not TRUE)
- Use IS NULL / IS NOT NULL to check
- Aggregate functions ignore NULLs
- NULL in arithmetic returns NULL

---

### Mid-Level (3-5 years experience)

**6. Explain the difference between UNION and UNION ALL.**

**Answer:**
- **UNION:** Combines results and removes duplicates (slower, requires sorting)
- **UNION ALL:** Combines results keeping duplicates (faster)

```sql
-- UNION: Removes duplicate rows
SELECT product_id FROM orders_2023
UNION
SELECT product_id FROM orders_2024;

-- UNION ALL: Keeps all rows
SELECT product_id FROM orders_2023
UNION ALL
SELECT product_id FROM orders_2024;
```

Use UNION ALL when you know there are no duplicates or duplicates don't matter (faster).

---

**7. How would you optimize this slow query?**

```sql
SELECT * FROM orders
WHERE YEAR(order_date) = 2024;
```

**Answer:** Function on indexed column prevents index usage. Rewrite as sargable predicate:

```sql
SELECT * FROM orders
WHERE order_date >= '2024-01-01' 
  AND order_date < '2025-01-01';

-- Also:
-- 1. SELECT specific columns, not *
-- 2. Ensure index on order_date
-- 3. Use EXPLAIN to verify index usage
```

---

**8. What's the N+1 query problem and how do you fix it?**

**Answer:** N+1 occurs when you fetch a list of N items, then execute one query per item (N additional queries).

```sql
-- ❌ N+1 Problem: 1 query for customers + N queries for order counts
SELECT id, name FROM customers;  -- Returns 1000 customers
-- Then for each customer:
SELECT COUNT(*) FROM orders WHERE customer_id = ?;  -- 1000 queries!

-- ✅ Fix: Single JOIN
SELECT 
    c.id,
    c.name,
    COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.id = o.customer_id
GROUP BY c.id, c.name;
```

---

**9. Explain window functions and when you'd use them.**

**Answer:** Window functions perform calculations across rows related to the current row without collapsing them (unlike GROUP BY).

Use cases:
- Rankings (ROW_NUMBER, RANK)
- Running totals (SUM() OVER)
- Moving averages
- Comparisons to previous/next rows (LAG, LEAD)

```sql
-- Rank products by price per category
SELECT 
    product_name,
    category,
    price,
    RANK() OVER (PARTITION BY category ORDER BY price DESC) AS price_rank
FROM products;
```

---

**10. What's the difference between RANK and DENSE_RANK?**

**Answer:**
- **RANK:** Skips ranks after ties (1, 1, 3, 4)
- **DENSE_RANK:** Consecutive ranks (1, 1, 2, 3)

```sql
SELECT 
    score,
    RANK() OVER (ORDER BY score DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM test_scores;

-- Scores: 95, 95, 90, 85
-- rank: 1, 1, 3, 4 (skips 2)
-- dense_rank: 1, 1, 2, 3 (consecutive)
```

---

### Senior Level (5+ years experience)

**11. Explain query execution plans and how to read them.**

**Answer:** Execution plans show how the database will execute a query. Use EXPLAIN (MySQL) or EXPLAIN ANALYZE (PostgreSQL).

Key things to check:
- **Type:** ALL (table scan) vs ref/eq_ref (index use)
- **Rows:** Estimated rows examined
- **Extra:** Using index, Using filesort, Using temporary
- **Join type:** Nested loop, hash join, merge join

```sql
EXPLAIN SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.order_date > '2024-01-01';

-- Look for:
-- type: ref (good) vs ALL (bad)
-- Extra: Using index (good) vs Using filesort (slower)
```

---

**12. Design a database schema for a multi-tenant SaaS application. How would you handle data isolation?**

**Answer:** Three main approaches:

**1. Separate Database per Tenant (strongest isolation)**
```sql
CREATE DATABASE tenant_acme;
CREATE DATABASE tenant_widgetco;
-- Pros: Strong isolation, easy backup/restore per tenant
-- Cons: Management overhead, doesn't scale to thousands of tenants
```

**2. Shared Database, Separate Schemas (middle ground)**
```sql
CREATE SCHEMA tenant_acme;
CREATE SCHEMA tenant_widgetco;
-- Pros: Better resource sharing, moderate isolation
-- Cons: Still management overhead
```

**3. Shared Database, Shared Schema with tenant_id column (scalable)**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    tenant_id INT NOT NULL,
    username VARCHAR(100),
    -- ...
    INDEX idx_tenant (tenant_id)
);

-- All queries filter by tenant_id
SELECT * FROM users WHERE tenant_id = 123 AND username = 'john';
```

Row-level security (PostgreSQL):
```sql
CREATE POLICY tenant_isolation ON users
USING (tenant_id = current_setting('app.current_tenant')::INT);

ALTER TABLE users ENABLE ROW LEVEL SECURITY;
```

**Trade-offs:**
- Separate DB: Best isolation, limited scale (100s of tenants)
- Shared schema: Best scalability (1000s+ tenants), requires careful query filtering
- Middle: Separate schemas for medium scale

---

**13. How would you implement efficient pagination for a table with 100 million rows?**

**Answer:** Use keyset (seek) pagination instead of OFFSET.

```sql
-- ❌ Slow: OFFSET-based pagination
SELECT * FROM posts
ORDER BY created_at DESC
LIMIT 10 OFFSET 1000000;  -- Skips 1M rows!

-- ✅ Fast: Keyset pagination
-- Page 1
SELECT * FROM posts
ORDER BY created_at DESC, id DESC
LIMIT 10;
-- Last row: created_at = '2024-03-15', id = 12345

-- Page 2
SELECT * FROM posts
WHERE (created_at, id) < ('2024-03-15', 12345)
ORDER BY created_at DESC, id DESC
LIMIT 10;

-- Required index:
CREATE INDEX idx_created_id ON posts(created_at DESC, id DESC);
```

Why it's fast:
- OFFSET reads and discards rows: O(n + offset)
- Keyset seeks to position: O(log n)
- Page 1 and page 100,000 take the same time

---

**14. Explain database normalization. When would you denormalize?**

**Answer:** **Normalization** reduces data redundancy by organizing into related tables.

**Normal forms:**
- **1NF:** Atomic values, no repeating groups
- **2NF:** 1NF + no partial dependencies
- **3NF:** 2NF + no transitive dependencies
- **BCNF:** Every determinant is a candidate key

**Denormalization** trades redundancy for performance:

When to denormalize:
- Read-heavy workloads (100:1 read:write ratio)
- Complex joins hurt performance
- Caching isn't sufficient
- Data rarely changes

```sql
-- Normalized (3NF)
orders (order_id, customer_id, order_date)
customers (customer_id, customer_name, email)

-- Every query needs JOIN
SELECT o.*, c.customer_name 
FROM orders o JOIN customers c ON o.customer_id = c.id;

-- Denormalized
orders (order_id, customer_id, customer_name, order_date)
-- Faster reads, but customer_name duplicated and can get stale
```

Solutions for staleness:
- Triggers to update denormalized data
- Event-driven updates
- Eventual consistency (acceptable for some use cases)

---

**15. How do you handle high-concurrency updates to the same row?**

**Answer:** Several strategies depending on requirements:

**1. Optimistic Locking (version number)**
```sql
-- Add version column
ALTER TABLE inventory ADD COLUMN version INT NOT NULL DEFAULT 1;

-- Update with version check
UPDATE inventory
SET quantity = quantity - 5, version = version + 1
WHERE product_id = 123 AND version = 10;

-- If affected rows = 0, version changed (conflict), retry
```

**2. Pessimistic Locking**
```sql
-- Lock row for update
BEGIN;
SELECT quantity FROM inventory
WHERE product_id = 123
FOR UPDATE;  -- Locks row until commit

UPDATE inventory SET quantity = quantity - 5
WHERE product_id = 123;
COMMIT;
```

**3. Atomic Operations**
```sql
-- Single atomic UPDATE (best for simple increments)
UPDATE inventory
SET quantity = quantity - 5
WHERE product_id = 123 AND quantity >= 5;

-- Check affected rows to verify success
```

**4. Queue-Based (for very high concurrency)**
```
Application → Queue (Redis/RabbitMQ) → Single worker processes updates
-- Serializes updates, no database-level conflicts
```

**Trade-offs:**
- Optimistic: Better throughput, retry on conflict
- Pessimistic: Guaranteed consistency, can cause lock contention
- Atomic: Simple, works for counters/increments
- Queue: Best for extreme concurrency, adds complexity

---

**16. Explain MVCC and how it affects transaction isolation.**

**Answer:** **Multi-Version Concurrency Control (MVCC)** allows multiple versions of rows to exist simultaneously, enabling readers and writers to work without blocking each other.

How it works:
- Each transaction sees a consistent snapshot of data
- Readers see old versions while writers create new versions
- No read locks needed (read never blocks write, write never blocks read)

**Isolation levels in MVCC:**

```sql
-- Read Uncommitted (dirty reads possible)
-- MySQL/PostgreSQL: Not recommended

-- Read Committed (default in PostgreSQL)
-- Sees only committed data, may see different data in same transaction

-- Repeatable Read (default in MySQL)
-- Consistent snapshot within transaction

-- Serializable (strictest)
-- Transactions appear to execute sequentially
```

**Example:**

```sql
-- Transaction 1
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- Returns 1000
-- Transaction 2 updates balance to 500
SELECT balance FROM accounts WHERE id = 1;  -- Repeatable Read: still 1000
COMMIT;

-- After commit: sees 500
```

**Implications:**
- **Phantom reads:** Can occur at Read Committed (new rows added by other transactions)
- **Write skew:** Can occur at Repeatable Read (two transactions read same data, make conflicting updates)
- **Deadlocks:** Possible when multiple transactions lock different rows in different orders

---

**17. Design a system to track user activity history (100M+ events per day) with efficient querying.**

**Answer:** Design considerations:

**Schema:**
```sql
CREATE TABLE user_events (
    event_id BIGSERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    event_type VARCHAR(50) NOT NULL,
    event_time TIMESTAMP NOT NULL,
    event_data JSONB,
    -- Partitioning key
    created_date DATE NOT NULL DEFAULT CURRENT_DATE
) PARTITION BY RANGE (created_date);

-- Create monthly partitions
CREATE TABLE user_events_2024_01 PARTITION OF user_events
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE user_events_2024_02 PARTITION OF user_events
FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

**Indexes:**
```sql
-- On each partition
CREATE INDEX idx_user_time ON user_events_2024_01(user_id, event_time DESC);
CREATE INDEX idx_event_type ON user_events_2024_01(event_type);
CREATE INDEX idx_event_data ON user_events_2024_01 USING GIN (event_data);
```

**Optimizations:**
1. **Time-series partitioning** (by month): Query pruning, easy archival
2. **Composite index** (user_id, event_time): Efficient user history queries
3. **JSONB** for flexible event_data: Structured yet flexible
4. **Archive old data**: Move 6+ month old partitions to cold storage
5. **Write optimizations:**
   - Batch inserts (1000s per transaction)
   - Disable unnecessary triggers/constraints
   - Use UNLOGGED tables for non-critical data (PostgreSQL)

**Query patterns:**
```sql
-- User history (uses partition pruning + index)
SELECT * FROM user_events
WHERE user_id = 12345 
  AND event_time >= '2024-01-01'
  AND event_time < '2024-02-01'
ORDER BY event_time DESC
LIMIT 100;

-- Event type aggregation
SELECT event_type, COUNT(*)
FROM user_events
WHERE created_date = '2024-01-15'
GROUP BY event_type;
```

**Scaling further:**
- Read replicas for queries
- Time-series databases (TimescaleDB, InfluxDB) for specialized workloads
- Data warehouse (Redshift, BigQuery) for analytics

---

**18. How would you migrate a large production table (1B rows) with zero downtime?**

**Answer:** Use online schema migration techniques:

**Approach: Dual-Write Strategy**

```sql
-- Step 1: Create new table with desired schema
CREATE TABLE users_new (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    name VARCHAR(100),
    new_column VARCHAR(50),  -- New column
    INDEX idx_email (email)
);

-- Step 2: Backfill data in batches
-- Run this in background (avoid locking entire table)
INSERT INTO users_new (id, email, name, new_column)
SELECT id, email, name, NULL
FROM users
WHERE id BETWEEN ? AND ?
ON DUPLICATE KEY UPDATE  -- In case of overlap
    email = VALUES(email),
    name = VALUES(name);

-- Step 3: Enable dual-write in application
-- Application writes to BOTH tables
INSERT INTO users (...);
INSERT INTO users_new (...);  -- Application change

-- Step 4: Verify data consistency
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM users_new;

-- Step 5: Cutover (rename tables atomically)
RENAME TABLE users TO users_old, users_new TO users;

-- Step 6: Drop old table after verification
DROP TABLE users_old;
```

**Alternative: pt-online-schema-change (Percona Toolkit)**
```bash
pt-online-schema-change \
  --alter "ADD COLUMN new_column VARCHAR(50)" \
  --execute \
  D=mydb,t=users
```

How it works:
- Creates new table with changes
- Copies data in chunks
- Uses triggers to maintain consistency during migration
- Atomic rename at the end

**Key principles:**
- Batch processing (avoid long locks)
- Monitor replication lag
- Test on staging first
- Have rollback plan
- Use application-level dual writes for complex changes

---

## Common Anti-Patterns to Avoid

### 1. SELECT * in Production

```sql
-- ❌ Bad
SELECT * FROM users;

-- ✅ Good
SELECT id, username, email FROM users;
```

**Why:** Wastes I/O, memory, network bandwidth. Breaks when schema changes.

---

### 2. Not Using Prepared Statements

```sql
-- ❌ Bad: SQL injection risk
query = "SELECT * FROM users WHERE username = '" + input + "'";

-- ✅ Good: Parameterized query
query = "SELECT * FROM users WHERE username = ?";
execute(query, [input]);
```

---

### 3. Implicit Type Conversion

```sql
-- ❌ Bad: user_id is INT
SELECT * FROM orders WHERE user_id = '123';  -- String comparison

-- ✅ Good
SELECT * FROM orders WHERE user_id = 123;  -- INT comparison
```

---

### 4. Function on Indexed Column

```sql
-- ❌ Bad: Can't use index
WHERE YEAR(order_date) = 2024

-- ✅ Good: Sargable
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01'
```

---

### 5. Using OR on Different Columns

```sql
-- ❌ Bad: Can use only one index
WHERE category = 'Electronics' OR price > 1000

-- ✅ Good: Use UNION to leverage both indexes
SELECT * FROM products WHERE category = 'Electronics'
UNION
SELECT * FROM products WHERE price > 1000;
```

---

### 6. NOT IN with NULL

```sql
-- ❌ Bad: Returns zero rows if blacklist contains NULL
WHERE customer_id NOT IN (SELECT customer_id FROM blacklist)

-- ✅ Good
WHERE NOT EXISTS (
    SELECT 1 FROM blacklist WHERE customer_id = customers.customer_id
)
```

---

### 7. Correlated Subquery in SELECT

```sql
-- ❌ Bad: N+1 queries
SELECT 
    customer_id,
    (SELECT COUNT(*) FROM orders WHERE customer_id = c.customer_id)
FROM customers c;

-- ✅ Good: Single JOIN
SELECT c.customer_id, COUNT(o.order_id)
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id;
```

---

### 8. Using DISTINCT to Fix Bad Joins

```sql
-- ❌ Bad: DISTINCT hiding join problem
SELECT DISTINCT users.name
FROM users
JOIN orders ON users.id = orders.user_id;

-- ✅ Good: Fix the root cause
SELECT users.name
FROM users
WHERE EXISTS (SELECT 1 FROM orders WHERE user_id = users.id);
```

---

### 9. Forgetting to Index Foreign Keys

```sql
-- ❌ Bad: Slow joins
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT  -- No index!
);

-- ✅ Good
CREATE INDEX idx_customer_id ON orders(customer_id);
```

---

### 10. Not Paginating Results

```sql
-- ❌ Bad: Fetching millions of rows
SELECT * FROM products;

-- ✅ Good: Keyset pagination
SELECT * FROM products
WHERE id > last_id
ORDER BY id
LIMIT 100;
```

---

## Production Readiness Checklist

### Before Deploying to Production

#### Schema Design
- [ ] All foreign keys indexed
- [ ] Appropriate data types (no VARCHAR for dates!)
- [ ] NOT NULL constraints where applicable
- [ ] Proper primary keys (prefer BIGINT for high-volume tables)
- [ ] Composite indexes for multi-column filters

#### Query Optimization
- [ ] All queries tested with EXPLAIN
- [ ] No full table scans on large tables
- [ ] Pagination implemented for large result sets
- [ ] No SELECT * in application code
- [ ] Prepared statements used for all dynamic queries

#### Performance
- [ ] Queries complete in < 100ms (target)
- [ ] Indexes don't exceed 10-15% of table size
- [ ] Statistics up to date (ANALYZE)
- [ ] Slow query log enabled and monitored

#### Security
- [ ] Parameterized queries (prevent SQL injection)
- [ ] Least privilege access (app user ≠ admin)
- [ ] Sensitive data encrypted at rest
- [ ] SSL/TLS for connections
- [ ] Row-level security if multi-tenant

#### Monitoring
- [ ] Query performance monitoring
- [ ] Replication lag monitoring
- [ ] Disk space alerts
- [ ] Connection pool monitoring
- [ ] Slow query alerts

#### Backup & Recovery
- [ ] Automated daily backups
- [ ] Backup restore tested
- [ ] Point-in-time recovery configured
- [ ] Backup retention policy defined
- [ ] Disaster recovery plan documented

#### Scaling
- [ ] Read replicas for read-heavy workloads
- [ ] Connection pooling configured
- [ ] Caching layer (Redis, Memcached)
- [ ] Partition large tables if needed
- [ ] Archive strategy for old data

---

## Key Takeaways

1. **SQL execution order matters:** Understand why you can't use SELECT aliases in WHERE
2. **Window functions > self-joins:** For analytics, always prefer window functions
3. **EXISTS > IN:** Especially for large subqueries and with NOT
4. **Index your joins:** Foreign keys should always be indexed
5. **Keyset > Offset:** For pagination beyond small datasets
6. **Avoid functions on indexed columns:** Makes queries non-sargable
7. **CTEs improve readability:** Use them to break complex queries into steps
8. **NULL is not zero:** Handle NULLs explicitly with IS NULL/COALESCE
9. **EXPLAIN is your friend:** Always check execution plans before deploying
10. **Normalize, then denormalize:** Start with 3NF, denormalize only when performance requires it

---

This comprehensive guide covers SQL from fundamentals through production optimization. Practice these patterns, understand the "why" behind each concept, and you'll be well-equipped for any SQL challenge in interviews or production systems.
