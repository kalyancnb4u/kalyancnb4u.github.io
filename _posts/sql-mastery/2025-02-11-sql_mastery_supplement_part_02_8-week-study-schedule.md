---
title: "SQL Supplement - Part 2: 8-Week SQL Mastery Study Schedule"
date: 2024-02-11 00:00:00 +0530
categories: [SQL, SQL Mastery]
tags: [SQL, Database, Learning, Study-plan, Learning-path, Schedule]
---

# 8-Week SQL Mastery Study Schedule

Structured daily learning plan to master SQL from fundamentals to advanced optimization in 8 weeks.

**Time Commitment:** 1.5-2 hours per day, 5 days per week

---

## Overview

This schedule follows a progressive learning path:
- **Weeks 1-2:** Foundations (SELECT, WHERE, JOINs, basic aggregations)
- **Weeks 3-4:** Intermediate (Window functions, subqueries, CTEs)
- **Weeks 5-6:** Advanced (Indexing, optimization, complex patterns)
- **Weeks 7-8:** Production skills (Performance tuning, real-world projects)

**Daily Structure:**
- 30 min: Theory & Concepts (read guide)
- 45 min: Hands-on Practice (write queries)
- 15 min: Review & Quiz (test yourself)

---

## Week 1: SQL Foundations

### Day 1: SQL Basics & Data Types
**Topics:**
- Relational database concepts
- Data types (INT, VARCHAR, DATE, DECIMAL)
- NULL handling and three-valued logic

**Practice:**
```sql
-- Setup practice database
CREATE DATABASE sql_practice;
USE sql_practice;

-- Create first table
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2),
    stock_quantity INT,
    created_at DATE
);

-- Insert sample data
INSERT INTO products VALUES
    (1, 'iPhone 15', 'Electronics', 999.99, 50, '2024-01-15'),
    (2, 'MacBook Pro', 'Electronics', 2499.99, 25, '2024-01-20'),
    (3, 'Office Chair', 'Furniture', 299.99, 100, '2024-01-18');

-- Practice queries
SELECT * FROM products;
SELECT product_name, price FROM products WHERE price > 500;
SELECT * FROM products WHERE category = 'Electronics';
```

**Study Goals:**
- [ ] Understand different data types and when to use each
- [ ] Create a table with appropriate data types
- [ ] Insert data successfully
- [ ] Write basic SELECT queries

**Quiz Questions:**
1. What's the difference between CHAR and VARCHAR?
2. How does NULL differ from 0 or empty string?
3. When should you use DECIMAL vs FLOAT?

---

### Day 2: SELECT & Filtering Basics
**Topics:**
- SELECT clause variations
- WHERE clause operators
- BETWEEN, IN, LIKE patterns

**Practice:**
```sql
-- Create customers table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50),
    signup_date DATE,
    total_spent DECIMAL(10,2)
);

-- Filter exercises
SELECT * FROM customers WHERE city = 'New York';
SELECT * FROM customers WHERE signup_date > '2024-01-01';
SELECT * FROM customers WHERE total_spent BETWEEN 100 AND 500;
SELECT * FROM customers WHERE email LIKE '%@gmail.com';
SELECT * FROM customers WHERE city IN ('New York', 'Los Angeles', 'Chicago');
```

**Study Goals:**
- [ ] Master comparison operators (=, !=, >, <, >=, <=)
- [ ] Use BETWEEN for range queries
- [ ] Apply LIKE with wildcards (%, _)
- [ ] Filter with IN for multiple values

**Exercises:**
1. Find all products cheaper than $100
2. Find products with "Pro" in the name
3. Find customers who signed up in January 2024

---

### Day 3: Sorting & Pagination
**Topics:**
- ORDER BY (ASC, DESC)
- LIMIT and OFFSET
- Multi-column sorting

**Practice:**
```sql
-- Sorting exercises
SELECT * FROM products ORDER BY price DESC;
SELECT * FROM products ORDER BY category, price;
SELECT * FROM products ORDER BY created_at DESC LIMIT 10;

-- Pagination
SELECT * FROM products ORDER BY product_id LIMIT 10 OFFSET 0;  -- Page 1
SELECT * FROM products ORDER BY product_id LIMIT 10 OFFSET 10; -- Page 2
```

**Study Goals:**
- [ ] Sort results in ascending and descending order
- [ ] Implement basic pagination with LIMIT/OFFSET
- [ ] Understand multi-column sort priority

**Project:**
Build a simple product catalog query that:
- Shows products sorted by price
- Paginates 20 items per page
- Filters by category

---

### Day 4: Aggregate Functions
**Topics:**
- COUNT, SUM, AVG, MIN, MAX
- NULL handling in aggregates
- DISTINCT with aggregates

**Practice:**
```sql
-- Aggregate exercises
SELECT COUNT(*) FROM products;
SELECT COUNT(DISTINCT category) FROM products;
SELECT AVG(price) FROM products;
SELECT SUM(stock_quantity * price) AS total_inventory_value FROM products;
SELECT category, MIN(price), MAX(price) FROM products GROUP BY category;
```

**Study Goals:**
- [ ] Calculate basic statistics with aggregate functions
- [ ] Understand NULL behavior in aggregates
- [ ] Count distinct values
- [ ] Combine aggregates with WHERE

**Challenge:**
Create a query to find:
- Total revenue if all products sell
- Average price per category
- Number of products in each price range (<$100, $100-$500, >$500)

---

### Day 5: GROUP BY & HAVING
**Topics:**
- GROUP BY mechanics
- HAVING vs WHERE
- Grouping by multiple columns

**Practice:**
```sql
-- Create orders table
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),
    status VARCHAR(20)
);

-- GROUP BY exercises
SELECT category, COUNT(*) as product_count
FROM products
GROUP BY category;

SELECT status, COUNT(*), SUM(total_amount)
FROM orders
GROUP BY status
HAVING COUNT(*) > 10;

SELECT DATE(order_date) as order_day, COUNT(*), SUM(total_amount)
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY DATE(order_date)
HAVING SUM(total_amount) > 1000
ORDER BY order_day;
```

**Study Goals:**
- [ ] Group data by one or more columns
- [ ] Filter groups with HAVING
- [ ] Understand WHERE vs HAVING execution order
- [ ] Combine grouping with sorting

**Weekend Project:**
Create a sales report showing:
- Daily sales totals for last 30 days
- Only days with more than $1000 in sales
- Sorted by date descending

---

## Week 2: JOINs & Relationships

### Day 6: INNER JOIN
**Topics:**
- JOIN basics and syntax
- INNER JOIN mechanics
- Table aliases
- Join conditions

**Practice:**
```sql
-- Create related tables
CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2)
);

-- INNER JOIN exercises
SELECT 
    o.order_id,
    c.customer_name,
    o.order_date,
    o.total_amount
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id;

SELECT 
    oi.item_id,
    p.product_name,
    oi.quantity,
    oi.unit_price
FROM order_items oi
INNER JOIN products p ON oi.product_id = p.product_id;
```

**Study Goals:**
- [ ] Understand how INNER JOIN works
- [ ] Use table aliases for clarity
- [ ] Join multiple tables
- [ ] Write proper join conditions

**Exercises:**
1. List all orders with customer names and emails
2. Show order items with product details
3. Find total quantity sold per product

---

### Day 7: LEFT JOIN & Outer Joins
**Topics:**
- LEFT JOIN vs INNER JOIN
- Finding missing relationships
- NULL in outer joins

**Practice:**
```sql
-- LEFT JOIN to find customers without orders
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) as order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;

-- Find customers who never ordered
SELECT c.*
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

**Study Goals:**
- [ ] Understand LEFT JOIN behavior
- [ ] Find records without matches
- [ ] Handle NULL values from outer joins
- [ ] Know when to use INNER vs LEFT JOIN

**Challenge:**
Write queries to find:
- Products never ordered
- Customers with zero orders
- Categories with no products

---

### Day 8: Multiple JOINs & Advanced Patterns
**Topics:**
- Joining 3+ tables
- Self joins
- Many-to-many relationships

**Practice:**
```sql
-- Complex join: orders with customer and product details
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name,
    oi.quantity,
    oi.unit_price
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
INNER JOIN order_items oi ON o.order_id = oi.order_id
INNER JOIN products p ON oi.product_id = p.product_id
WHERE o.order_date >= '2024-01-01';

-- Self join: find employees and their managers
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    employee_name VARCHAR(100),
    manager_id INT
);

SELECT 
    e.employee_name as employee,
    m.employee_name as manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

**Study Goals:**
- [ ] Join multiple tables in one query
- [ ] Understand self-join patterns
- [ ] Handle complex relationships
- [ ] Optimize join order

---

### Day 9: CASE Expressions
**Topics:**
- Simple vs searched CASE
- Conditional aggregation
- Pivot operations with CASE

**Practice:**
```sql
-- Simple CASE
SELECT 
    product_name,
    price,
    CASE 
        WHEN price < 100 THEN 'Budget'
        WHEN price < 500 THEN 'Standard'
        ELSE 'Premium'
    END as price_tier
FROM products;

-- Conditional aggregation
SELECT 
    COUNT(*) as total_orders,
    COUNT(CASE WHEN status = 'completed' THEN 1 END) as completed,
    COUNT(CASE WHEN status = 'pending' THEN 1 END) as pending,
    SUM(CASE WHEN status = 'completed' THEN total_amount ELSE 0 END) as revenue
FROM orders;
```

**Study Goals:**
- [ ] Use CASE for conditional logic
- [ ] Implement conditional aggregation
- [ ] Create pivot tables with CASE
- [ ] Understand CASE execution order

---

### Day 10: Week 1-2 Review & Mini Project
**Topics:**
- Review all concepts from Weeks 1-2
- Integration project

**Mini Project: E-Commerce Sales Dashboard**

Create a comprehensive sales report that shows:

```sql
-- Requirements:
-- 1. Total revenue by category
-- 2. Top 10 customers by spending
-- 3. Daily sales trends (last 30 days)
-- 4. Products that need restocking (stock < 10)
-- 5. Conversion rate: customers who ordered vs total customers
-- 6. Average order value by customer city

-- Bonus: Create a single query that shows all this data
```

**Weekend Review:**
- [ ] Complete mini project
- [ ] Review difficult concepts
- [ ] Practice 10 random queries from memory
- [ ] Take Week 1-2 assessment quiz

---

## Week 3: Subqueries & Advanced Patterns

### Day 11: Scalar Subqueries
**Topics:**
- Subquery basics
- Scalar subqueries in SELECT, WHERE
- Performance considerations

**Practice:**
```sql
-- Scalar subquery in WHERE
SELECT * FROM products
WHERE price > (SELECT AVG(price) FROM products);

-- Scalar subquery in SELECT
SELECT 
    product_name,
    price,
    (SELECT AVG(price) FROM products) as avg_price,
    price - (SELECT AVG(price) FROM products) as price_diff
FROM products;
```

**Study Goals:**
- [ ] Write scalar subqueries
- [ ] Understand subquery execution
- [ ] Compare values to aggregates
- [ ] Identify performance issues

---

### Day 12: Correlated Subqueries & EXISTS
**Topics:**
- Correlated vs uncorrelated
- EXISTS and NOT EXISTS
- Performance comparisons

**Practice:**
```sql
-- Correlated subquery
SELECT p.*
FROM products p
WHERE price > (
    SELECT AVG(price)
    FROM products
    WHERE category = p.category
);

-- EXISTS
SELECT c.*
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders WHERE customer_id = c.customer_id
);

-- NOT EXISTS (find customers without orders)
SELECT c.*
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders WHERE customer_id = c.customer_id
);
```

**Study Goals:**
- [ ] Write correlated subqueries
- [ ] Use EXISTS for existence checks
- [ ] Compare IN vs EXISTS performance
- [ ] Avoid N+1 query patterns

---

### Day 13: Common Table Expressions (CTEs)
**Topics:**
- CTE syntax and benefits
- Multiple CTEs
- CTE vs subqueries

**Practice:**
```sql
-- Basic CTE
WITH high_value_customers AS (
    SELECT customer_id, SUM(total_amount) as total_spent
    FROM orders
    GROUP BY customer_id
    HAVING SUM(total_amount) > 1000
)
SELECT c.*, hvc.total_spent
FROM customers c
JOIN high_value_customers hvc ON c.customer_id = hvc.customer_id;

-- Multiple CTEs
WITH 
monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', order_date) as month,
        SUM(total_amount) as revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
),
growth_calc AS (
    SELECT 
        month,
        revenue,
        LAG(revenue) OVER (ORDER BY month) as prev_month
    FROM monthly_sales
)
SELECT 
    month,
    revenue,
    prev_month,
    revenue - prev_month as growth
FROM growth_calc;
```

**Study Goals:**
- [ ] Convert subqueries to CTEs
- [ ] Chain multiple CTEs
- [ ] Improve query readability
- [ ] Reference CTEs multiple times

---

### Day 14: Recursive CTEs
**Topics:**
- Recursive CTE structure
- Hierarchical data queries
- Termination conditions

**Practice:**
```sql
-- Generate number sequence
WITH RECURSIVE numbers AS (
    SELECT 1 as n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 100
)
SELECT * FROM numbers;

-- Employee hierarchy
WITH RECURSIVE emp_tree AS (
    SELECT employee_id, employee_name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    SELECT e.employee_id, e.employee_name, e.manager_id, et.level + 1
    FROM employees e
    JOIN emp_tree et ON e.manager_id = et.employee_id
)
SELECT * FROM emp_tree ORDER BY level;
```

**Study Goals:**
- [ ] Understand recursive CTE structure
- [ ] Query hierarchical data
- [ ] Implement termination logic
- [ ] Generate sequences

---

### Day 15: Set Operations
**Topics:**
- UNION and UNION ALL
- INTERSECT and EXCEPT
- Use cases for each

**Practice:**
```sql
-- UNION: Combine results
SELECT product_id FROM products WHERE category = 'Electronics'
UNION
SELECT product_id FROM products WHERE price > 500;

-- UNION ALL: Keep duplicates
SELECT customer_id FROM orders WHERE order_date = '2024-01-15'
UNION ALL
SELECT customer_id FROM orders WHERE order_date = '2024-01-16';

-- INTERSECT (PostgreSQL): Common elements
SELECT customer_id FROM orders WHERE YEAR(order_date) = 2023
INTERSECT
SELECT customer_id FROM orders WHERE YEAR(order_date) = 2024;
```

**Study Goals:**
- [ ] Combine queries with UNION
- [ ] Choose UNION vs UNION ALL
- [ ] Find common/different elements
- [ ] Understand performance implications

**Weekend Practice:**
Create complex queries using:
- CTEs with 3+ steps
- Recursive queries
- Set operations
- Mixed patterns

---

## Week 4: Window Functions

### Day 16: Ranking Functions
**Topics:**
- ROW_NUMBER, RANK, DENSE_RANK
- NTILE for percentiles
- Ranking within partitions

**Practice:**
```sql
-- Basic ranking
SELECT 
    product_name,
    category,
    price,
    ROW_NUMBER() OVER (ORDER BY price DESC) as row_num,
    RANK() OVER (ORDER BY price DESC) as rank,
    DENSE_RANK() OVER (ORDER BY price DESC) as dense_rank
FROM products;

-- Ranking within category
SELECT 
    product_name,
    category,
    price,
    RANK() OVER (PARTITION BY category ORDER BY price DESC) as category_rank
FROM products;

-- Top 3 per category
WITH ranked AS (
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as rn
    FROM products
)
SELECT * FROM ranked WHERE rn <= 3;
```

**Study Goals:**
- [ ] Understand ranking function differences
- [ ] Partition data for rankings
- [ ] Solve top-N per group problems
- [ ] Use NTILE for percentiles

---

### Day 17: Offset Functions (LAG, LEAD)
**Topics:**
- LAG and LEAD
- FIRST_VALUE, LAST_VALUE
- Frame clauses

**Practice:**
```sql
-- Month-over-month comparison
WITH monthly AS (
    SELECT 
        DATE_TRUNC('month', order_date) as month,
        SUM(total_amount) as revenue
    FROM orders
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT 
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) as prev_month,
    revenue - LAG(revenue) OVER (ORDER BY month) as change
FROM monthly;

-- Running comparison
SELECT 
    order_date,
    total_amount,
    LAG(total_amount, 1) OVER (ORDER BY order_date) as prev_order,
    LEAD(total_amount, 1) OVER (ORDER BY order_date) as next_order
FROM orders;
```

**Study Goals:**
- [ ] Access previous/next row values
- [ ] Calculate period-over-period changes
- [ ] Use FIRST_VALUE and LAST_VALUE correctly
- [ ] Understand frame clauses

---

### Day 18: Aggregate Window Functions
**Topics:**
- SUM, AVG, COUNT as window functions
- Running totals
- Moving averages

**Practice:**
```sql
-- Running total
SELECT 
    order_date,
    total_amount,
    SUM(total_amount) OVER (ORDER BY order_date) as running_total
FROM orders;

-- Moving average (7-day)
SELECT 
    order_date,
    total_amount,
    AVG(total_amount) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7day
FROM orders;

-- Cumulative within partitions
SELECT 
    category,
    product_name,
    price,
    SUM(price) OVER (PARTITION BY category ORDER BY price) as cumulative_price
FROM products;
```

**Study Goals:**
- [ ] Calculate running totals
- [ ] Compute moving averages
- [ ] Use window frames (ROWS, RANGE)
- [ ] Combine with partitioning

---

### Day 19: Advanced Window Patterns
**Topics:**
- Complex window frames
- Multiple window functions
- Performance optimization

**Practice:**
```sql
-- Multiple analytics in one query
SELECT 
    order_date,
    total_amount,
    -- Ranking
    ROW_NUMBER() OVER (ORDER BY total_amount DESC) as amount_rank,
    -- Running total
    SUM(total_amount) OVER (ORDER BY order_date) as running_total,
    -- Moving average
    AVG(total_amount) OVER (
        ORDER BY order_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg,
    -- Percent of total
    ROUND(total_amount * 100.0 / SUM(total_amount) OVER (), 2) as pct_of_total
FROM orders;
```

**Study Goals:**
- [ ] Combine multiple window functions
- [ ] Optimize window queries
- [ ] Choose appropriate frames
- [ ] Solve complex analytics problems

---

### Day 20: Week 3-4 Review & Project
**Project: Advanced Analytics Dashboard**

Create comprehensive analytical queries:

```sql
-- 1. Customer Segmentation (RFM Analysis)
-- - Recency: Days since last order
-- - Frequency: Number of orders
-- - Monetary: Total spent
-- - Segment customers into quartiles

-- 2. Product Performance Analysis
-- - Running total of sales per product
-- - Month-over-month growth rates
-- - Top 5 products per category by revenue
-- - Products in bottom 10% by sales

-- 3. Cohort Analysis
-- - Group customers by signup month
-- - Track retention month-over-month
-- - Calculate cohort lifetime value
```

**Weekend Review:**
- [ ] Complete analytics project
- [ ] Master all window functions
- [ ] Practice without references
- [ ] Take Weeks 3-4 assessment

---

## Week 5: Indexing & Performance (Part 1)

### Day 21: Index Fundamentals
**Topics:**
- Index types (B-tree, Hash)
- When to index
- Index selectivity

**Practice:**
```sql
-- Create indexes
CREATE INDEX idx_customer_email ON customers(email);
CREATE INDEX idx_order_date ON orders(order_date);
CREATE INDEX idx_product_category ON products(category);

-- Composite index
CREATE INDEX idx_category_price ON products(category, price);

-- Check index usage
EXPLAIN SELECT * FROM customers WHERE email = 'test@example.com';
EXPLAIN SELECT * FROM products WHERE category = 'Electronics' AND price > 500;
```

**Study Goals:**
- [ ] Understand index structures
- [ ] Create appropriate indexes
- [ ] Use EXPLAIN to verify index usage
- [ ] Know when NOT to index

**Exercises:**
- Identify which columns need indexes in your tables
- Measure query performance before/after indexing
- Analyze EXPLAIN output

---

### Day 22: Composite Indexes & Covering Indexes
**Topics:**
- Multi-column indexes
- Index column order
- Covering indexes

**Practice:**
```sql
-- Composite index optimization
CREATE INDEX idx_customer_date_status ON orders(customer_id, order_date, status);

-- Query that benefits from composite index
SELECT * FROM orders
WHERE customer_id = 123
  AND order_date >= '2024-01-01'
  AND status = 'completed';

-- Covering index (index-only scan)
CREATE INDEX idx_covering ON orders(customer_id, order_date, total_amount);

-- This query can be satisfied entirely from the index
SELECT customer_id, order_date, total_amount
FROM orders
WHERE customer_id = 123;
```

**Study Goals:**
- [ ] Order index columns correctly
- [ ] Create covering indexes
- [ ] Understand index-only scans
- [ ] Measure index effectiveness

---

### Day 23: Query Optimization Basics
**Topics:**
- Sargable queries
- Avoiding anti-patterns
- Reading execution plans

**Practice:**
```sql
-- ❌ Non-sargable (can't use index)
SELECT * FROM orders WHERE YEAR(order_date) = 2024;

-- ✅ Sargable (can use index)
SELECT * FROM orders 
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';

-- ❌ Function on column prevents index
SELECT * FROM customers WHERE UPPER(email) = 'TEST@EXAMPLE.COM';

-- ✅ Use functional index or compare correctly
CREATE INDEX idx_email_lower ON customers(LOWER(email));
SELECT * FROM customers WHERE LOWER(email) = 'test@example.com';
```

**Study Goals:**
- [ ] Write sargable predicates
- [ ] Avoid functions on indexed columns
- [ ] Understand execution plan costs
- [ ] Identify optimization opportunities

---

### Day 24: Advanced Optimization Techniques
**Topics:**
- Partition pruning
- Join optimization
- Subquery materialization

**Practice:**
```sql
-- Optimize complex queries
-- Before
SELECT c.*, COUNT(o.order_id)
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.city = 'New York'
GROUP BY c.customer_id;

-- Optimized: Filter before joining
WITH ny_customers AS (
    SELECT * FROM customers WHERE city = 'New York'
)
SELECT c.*, COUNT(o.order_id)
FROM ny_customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id;
```

**Study Goals:**
- [ ] Rewrite queries for better performance
- [ ] Understand optimizer behavior
- [ ] Use hints appropriately (advanced)
- [ ] Profile and measure improvements

---

### Day 25: Week 5 Review & Optimization Project
**Project: Performance Tuning Challenge**

Given a slow-running database:

```sql
-- 1. Identify slow queries from slow query log
-- 2. Add appropriate indexes
-- 3. Rewrite inefficient queries
-- 4. Measure before/after performance
-- 5. Document optimization decisions

-- Benchmark template
SET @start = NOW();
-- Your query here
SET @end = NOW();
SELECT TIMEDIFF(@end, @start) as execution_time;
```

**Weekend Tasks:**
- [ ] Optimize 5 slow queries
- [ ] Create optimal index strategy
- [ ] Document performance improvements
- [ ] Review execution plans

---

## Week 6: Production Skills

### Day 26: Transactions & ACID
**Topics:**
- Transaction basics
- Isolation levels
- Deadlocks

**Practice:**
```sql
-- Basic transaction
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;

-- Rollback on error
BEGIN;
INSERT INTO orders (customer_id, total_amount) VALUES (123, 500);
-- Check for error
ROLLBACK;  -- or COMMIT

-- Isolation levels
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- or REPEATABLE READ, SERIALIZABLE
```

**Study Goals:**
- [ ] Use transactions correctly
- [ ] Understand isolation levels
- [ ] Handle deadlocks
- [ ] Implement savepoints

---

### Day 27: Data Modification Best Practices
**Topics:**
- Batch operations
- UPSERT patterns
- Safe DELETE strategies

**Practice:**
```sql
-- Batch insert
INSERT INTO products (name, price) VALUES
    ('Product 1', 10.00),
    ('Product 2', 20.00),
    ('Product 3', 30.00);

-- UPSERT (MySQL)
INSERT INTO products (id, name, price)
VALUES (1, 'Updated Product', 15.00)
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    price = VALUES(price);

-- Safe DELETE with verification
SELECT COUNT(*) FROM orders WHERE status = 'cancelled'; -- Check first
BEGIN;
DELETE FROM orders WHERE status = 'cancelled';
SELECT COUNT(*) FROM orders WHERE status = 'cancelled'; -- Verify
-- COMMIT or ROLLBACK
```

**Study Goals:**
- [ ] Batch operations efficiently
- [ ] Implement UPSERT logic
- [ ] Delete data safely
- [ ] Use transactions for modifications

---

### Day 28: Security & Best Practices
**Topics:**
- SQL injection prevention
- Parameterized queries
- Least privilege access

**Practice:**
```sql
-- ❌ Vulnerable to SQL injection
query = "SELECT * FROM users WHERE username = '" + userInput + "'";

-- ✅ Parameterized query (pseudo-code)
query = "SELECT * FROM users WHERE username = ?";
execute(query, [userInput]);

-- Principle of least privilege
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT, UPDATE ON mydb.* TO 'app_user'@'localhost';
-- Don't grant DELETE or ALTER unless needed
```

**Study Goals:**
- [ ] Prevent SQL injection
- [ ] Use prepared statements
- [ ] Apply least privilege
- [ ] Audit database access

---

### Day 29: Monitoring & Maintenance
**Topics:**
- Slow query log
- ANALYZE and statistics
- Backup strategies

**Practice:**
```sql
-- Update statistics
ANALYZE TABLE products;
ANALYZE TABLE orders;

-- Check table status (MySQL)
SHOW TABLE STATUS LIKE 'orders';

-- Find unused indexes (PostgreSQL)
SELECT 
    schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY schemaname, tablename;
```

**Study Goals:**
- [ ] Enable slow query logging
- [ ] Maintain statistics
- [ ] Plan backups
- [ ] Monitor query performance

---

### Day 30: Week 6 Review & Integration
**Capstone Project: Production-Ready E-Commerce Database**

Build a complete e-commerce system:

```sql
-- Requirements:
-- 1. Complete schema with proper constraints
-- 2. Appropriate indexes on all tables
-- 3. Stored procedures for common operations
-- 4. Sample data (1000+ records)
-- 5. 10 optimized analytical queries
-- 6. Performance benchmarks documented
-- 7. Security considerations implemented

-- Deliverables:
-- - schema.sql
-- - indexes.sql
-- - sample_data.sql
-- - queries.sql
-- - performance_report.md
```

---

## Week 7-8: Real-World Projects & Interview Prep

### Days 31-35: Project Week
Choose one advanced project:

**Option A: Analytics Platform**
- Sales analytics dashboard
- Customer segmentation
- Product recommendations
- Cohort analysis

**Option B: Multi-Tenant SaaS Database**
- Tenant isolation design
- Row-level security
- Scaling strategies
- Migration patterns

**Option C: Time-Series Data System**
- Event logging at scale
- Partitioning strategy
- Efficient querying
- Archival system

---

### Days 36-40: Interview Preparation

**Day 36: Junior-Level Questions**
- Practice 20 foundational questions
- Mock whiteboard sessions
- Explain concepts clearly

**Day 37: Mid-Level Questions**
- Window functions deep dive
- Optimization scenarios
- Schema design questions

**Day 38: Senior-Level Questions**
- System design with SQL
- Scaling challenges
- Production war stories

**Day 39: Coding Challenges**
- LeetCode SQL problems (Hard)
- HackerRank SQL challenges
- Real interview questions

**Day 40: Final Review**
- Review all concepts
- Practice explaining out loud
- Mock interview session

---

## Progress Tracking

### Weekly Checkpoints

**Week 1-2 Checklist:**
- [ ] Can write complex SELECT queries
- [ ] Comfortable with JOINs
- [ ] Understand GROUP BY and aggregates
- [ ] Can use CASE expressions

**Week 3-4 Checklist:**
- [ ] Master window functions
- [ ] Write complex CTEs
- [ ] Optimize subqueries
- [ ] Solve top-N problems

**Week 5-6 Checklist:**
- [ ] Create appropriate indexes
- [ ] Read execution plans
- [ ] Optimize slow queries
- [ ] Write production-ready SQL

**Week 7-8 Checklist:**
- [ ] Complete capstone project
- [ ] Confident in interviews
- [ ] Understand scaling
- [ ] Can design schemas

---

## Study Tips

1. **Practice Daily:** Consistency beats intensity
2. **Type, Don't Copy:** Muscle memory matters
3. **Explain Out Loud:** Teaching solidifies learning
4. **Break Complex Queries:** Build incrementally
5. **Use Real Data:** Practice on meaningful datasets
6. **Review Regularly:** Spaced repetition works
7. **Join Communities:** Learn from others
8. **Build Projects:** Apply knowledge practically

---

## Resources

- **Practice Databases:** Download sample databases (see Resource Compilation doc)
- **Online Practice:** SQLZoo, Mode Analytics, LeetCode
- **Documentation:** MySQL/PostgreSQL official docs
- **Community:** Stack Overflow, Reddit r/SQL
- **Tools:** DBeaver, DataGrip, pgAdmin

---

## Next Steps After 8 Weeks

1. **Specialize:** Choose MySQL or PostgreSQL expertise
2. **Scale Up:** Learn about sharding, replication
3. **NoSQL:** Understand when to use alternatives
4. **Cloud:** AWS RDS, Azure SQL, Google Cloud SQL
5. **Advanced Topics:** Query optimization deep dives
6. **Certifications:** Consider professional certifications

---

**Remember:** SQL mastery is a journey, not a destination. These 8 weeks give you a strong foundation, but continuous practice and real-world application cement the skills. Good luck!
