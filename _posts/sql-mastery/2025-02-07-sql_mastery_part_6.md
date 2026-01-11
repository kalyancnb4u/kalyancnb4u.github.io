---
title: "SQL Mastery Part 6: Interview Preparation & Career Mastery"
date: 2024-02-07 00:00:00 +0530
categories: [SQL, SQL Mastery]
tags: [SQL, Database, PostgreSQL, MySQL, Interview, Career, System-design, Troubleshooting, Best-practices]
---

# Complete SQL Mastery Part 6: Interview Preparation & Career Mastery

## Introduction

Technical interviews test not just what you know, but how you think, communicate, and solve problems under pressure. Database interviews range from basic SQL syntax to complex system design, requiring both breadth and depth of knowledge.

In Parts 1-5, we built comprehensive SQL and database expertise. Now we transform that knowledge into interview success and career advancement.

This final part covers:
* **Interview questions by level** - junior to staff engineer questions with detailed answers
* **System design with databases** - designing scalable database architectures
* **Troubleshooting scenarios** - diagnosing production issues under pressure
* **Best practices reference** - quick review of key concepts
* **Career development** - advancing from junior to senior to staff engineer

This knowledge is critical because:
* **Interviews are gatekeepers**: Top companies require strong database skills
* **System design separates levels**: Senior+ roles require architectural thinking
* **Troubleshooting demonstrates expertise**: Production debugging skills are highly valued
* **Communication matters**: Explaining your thinking is as important as the solution
* **Continuous learning**: Technology evolves, staying current is essential

By the end of this part, you'll confidently tackle any database interview and understand the career path from junior developer to principal engineer.

---

## 6.1 Interview Questions by Level

### Junior Level (0-2 years)

These questions test fundamental SQL knowledge and basic database concepts.

#### Question 1: Write a query to find the second highest salary in the employees table.

**What they're testing:** Basic SQL, subqueries, LIMIT/OFFSET

**Answer:**

```sql
-- Solution 1: Using LIMIT and OFFSET (MySQL, PostgreSQL)
SELECT DISTINCT salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

-- Solution 2: Using subquery (all databases)
SELECT MAX(salary) AS second_highest
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Solution 3: Using DENSE_RANK (handles ties)
WITH ranked_salaries AS (
    SELECT 
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT salary
FROM ranked_salaries
WHERE rank = 2;

-- Edge cases to discuss:
-- 1. What if there's no second highest? (Only one salary)
-- 2. What if multiple employees have the same highest salary?
-- 3. Should we return NULL or no rows?

-- Robust solution handling edge cases:
SELECT 
    CASE 
        WHEN COUNT(DISTINCT salary) >= 2 THEN
            (SELECT DISTINCT salary 
             FROM employees 
             ORDER BY salary DESC 
             LIMIT 1 OFFSET 1)
        ELSE NULL
    END AS second_highest_salary
FROM employees;
```

**Follow-up questions:**
- How would you find the Nth highest salary?
- What's the difference between RANK() and DENSE_RANK()?
- How would you handle NULL salaries?

---

#### Question 2: Explain the difference between WHERE and HAVING.

**What they're testing:** Understanding of query execution order

**Answer:**

```sql
-- WHERE: Filters rows BEFORE grouping
-- HAVING: Filters groups AFTER grouping

-- Example table: sales
-- product | quantity | price
-- A       | 10       | 100
-- A       | 5        | 100
-- B       | 20       | 50
-- B       | 10       | 50

-- Wrong: Using HAVING for row filter
SELECT product, SUM(quantity) AS total_qty
FROM sales
HAVING price > 75  -- ❌ Error: price not in GROUP BY
GROUP BY product;

-- Correct: Using WHERE for row filter
SELECT product, SUM(quantity) AS total_qty
FROM sales
WHERE price > 75   -- ✓ Filters rows first
GROUP BY product;
-- Result: Only product A (price=100)

-- Wrong: Using WHERE for aggregate filter
SELECT product, SUM(quantity) AS total_qty
FROM sales
GROUP BY product
WHERE SUM(quantity) > 15;  -- ❌ Error: Can't use aggregate in WHERE

-- Correct: Using HAVING for aggregate filter
SELECT product, SUM(quantity) AS total_qty
FROM sales
GROUP BY product
HAVING SUM(quantity) > 15;  -- ✓ Filters groups
-- Result: Only product B (total=30)

-- Execution order:
-- 1. FROM
-- 2. WHERE (row filter)
-- 3. GROUP BY
-- 4. Aggregate functions (SUM, COUNT, etc.)
-- 5. HAVING (group filter)
-- 6. SELECT
-- 7. ORDER BY
-- 8. LIMIT

-- Combined example:
SELECT 
    product,
    SUM(quantity) AS total_qty
FROM sales
WHERE price >= 50              -- Filter: Only consider products ≥ $50
GROUP BY product
HAVING SUM(quantity) > 10      -- Filter: Only groups with total > 10
ORDER BY total_qty DESC
LIMIT 5;                       -- Return top 5
```

**Key points to emphasize:**
- WHERE is more efficient (filters early)
- HAVING can use aggregate functions
- Can use both in same query
- Understanding execution order is critical

---

#### Question 3: What's the difference between INNER JOIN and LEFT JOIN?

**What they're testing:** Understanding of join types and NULL handling

**Answer:**

```sql
-- Sample data:
-- customers:
-- customer_id | name
-- 1           | Alice
-- 2           | Bob
-- 3           | Carol

-- orders:
-- order_id | customer_id | total
-- 101      | 1           | 100
-- 102      | 1           | 150
-- 103      | 2           | 200

-- INNER JOIN: Only returns matching rows from both tables
SELECT c.name, o.order_id, o.total
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;

-- Result:
-- name  | order_id | total
-- Alice | 101      | 100
-- Alice | 102      | 150
-- Bob   | 103      | 200
-- Carol is excluded (no orders)

-- LEFT JOIN: Returns all rows from left table, with NULLs for non-matches
SELECT c.name, o.order_id, o.total
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;

-- Result:
-- name  | order_id | total
-- Alice | 101      | 100
-- Alice | 102      | 150
-- Bob   | 103      | 200
-- Carol | NULL     | NULL  ← Included with NULL values

-- Practical use cases:

-- Find customers who have NOT placed orders:
SELECT c.name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
-- Result: Carol

-- Count orders per customer (including 0):
SELECT 
    c.name,
    COUNT(o.order_id) AS order_count
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.name;
-- Result:
-- Alice: 2
-- Bob: 1
-- Carol: 0

-- With INNER JOIN, Carol would be excluded entirely
```

**Visual representation to draw:**
```
INNER JOIN (Intersection):
   ┌─────┐
   │  A  │
   │  ┌──┼──┐
   │  │██│  │
   └──┼──┘  │
      │  B  │
      └─────┘

LEFT JOIN (All of A + Intersection):
   ┌─────┐
   │█████│
   │█┌──┼──┐
   │█│██│  │
   └─┼──┘  │
     │  B  │
     └─────┘
```

---

### Mid-Level (2-5 years)

These questions test deeper SQL knowledge, optimization awareness, and database internals.

#### Question 4: How would you optimize this slow query?

**Setup:**
```sql
-- Table: orders (10 million rows)
-- Indexes: PRIMARY KEY (order_id)
-- No other indexes

-- Slow query (5 seconds):
SELECT 
    customer_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_spent
FROM orders
WHERE order_date >= '2024-01-01'
  AND status = 'completed'
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

**What they're testing:** 
- Understanding of execution plans
- Index design
- Query optimization techniques

**Answer:**

```sql
-- Step 1: Analyze current execution plan
EXPLAIN ANALYZE
SELECT 
    customer_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_spent
FROM orders
WHERE order_date >= '2024-01-01'
  AND status = 'completed'
GROUP BY customer_id
HAVING COUNT(*) > 5;

-- Likely shows:
-- Seq Scan on orders (cost=0.00..250000.00 rows=1000000 ...)
-- Execution time: 5000 ms

-- Problem: Full table scan

-- Step 2: Add appropriate indexes
CREATE INDEX idx_orders_date_status ON orders(order_date, status);
-- Covers WHERE clause

CREATE INDEX idx_orders_customer ON orders(customer_id);
-- Helps with GROUP BY

-- Or better: Covering index
CREATE INDEX idx_orders_covering 
ON orders(order_date, status, customer_id, total_amount);
-- Includes all columns needed

-- Step 3: Verify improvement
EXPLAIN ANALYZE
SELECT 
    customer_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_spent
FROM orders
WHERE order_date >= '2024-01-01'
  AND status = 'completed'
GROUP BY customer_id
HAVING COUNT(*) > 5;

-- Should now show:
-- Index Scan using idx_orders_covering (cost=0.00..5000.00 rows=10000 ...)
-- Execution time: 50 ms

-- 100x faster!

-- Step 4: Additional optimizations if still slow

-- Option 1: Partial index (if status='completed' is rare)
CREATE INDEX idx_orders_completed
ON orders(order_date, customer_id, total_amount)
WHERE status = 'completed';

-- Option 2: Materialized view (if query runs frequently)
CREATE MATERIALIZED VIEW customer_order_summary AS
SELECT 
    customer_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_spent,
    MAX(order_date) AS last_order_date
FROM orders
WHERE status = 'completed'
GROUP BY customer_id;

CREATE INDEX idx_summary_date ON customer_order_summary(last_order_date);

-- Query becomes:
SELECT *
FROM customer_order_summary
WHERE last_order_date >= '2024-01-01'
  AND order_count > 5;

-- Execution time: < 1 ms

-- Refresh materialized view periodically
REFRESH MATERIALIZED VIEW customer_order_summary;
```

**Discussion points:**
- Why covering index is better
- Trade-offs of adding indexes (write performance)
- When to use materialized views
- Monitoring query performance over time

---

#### Question 5: Explain database normalization with an example.

**What they're testing:** Understanding of database design principles

**Answer:**

```sql
-- Unnormalized table (1NF violation):
CREATE TABLE orders_bad (
    order_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_phone VARCHAR(20),
    products VARCHAR(500),  -- "Laptop, Mouse, Keyboard"
    quantities VARCHAR(100), -- "1, 2, 1"
    prices VARCHAR(100)      -- "1000, 25, 50"
);

-- Problems:
-- 1. Data duplication (customer info repeated for each order)
-- 2. Update anomalies (change email → update all customer's orders)
-- 3. Insertion anomalies (can't add customer without order)
-- 4. Deletion anomalies (delete last order → lose customer)
-- 5. Multiple values in single column (can't query products easily)

-- First Normal Form (1NF): Atomic values
CREATE TABLE orders_1nf (
    order_id INT,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_phone VARCHAR(20),
    product VARCHAR(100),
    quantity INT,
    price DECIMAL(10,2),
    PRIMARY KEY (order_id, product)
);

-- Better: Each column has atomic value
-- But still duplicates customer info

-- Second Normal Form (2NF): No partial dependencies
-- (All non-key columns depend on entire primary key)

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_phone VARCHAR(20)
);

CREATE TABLE orders_2nf (
    order_id INT,
    customer_id INT,
    product VARCHAR(100),
    quantity INT,
    price DECIMAL(10,2),
    PRIMARY KEY (order_id, product),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Better: Customer info not duplicated
-- But price is duplicated for same product

-- Third Normal Form (3NF): No transitive dependencies
-- (Non-key columns depend only on primary key)

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_phone VARCHAR(20)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Now in 3NF:
-- ✓ No duplicate data
-- ✓ No update anomalies
-- ✓ No insertion anomalies
-- ✓ No deletion anomalies

-- Trade-offs:

-- Normalized (3NF):
-- Pros: No redundancy, data integrity
-- Cons: More joins (slower queries)

-- Example query (requires 3 joins):
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name,
    oi.quantity,
    p.price,
    oi.quantity * p.price AS line_total
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id;

-- Denormalized (intentional redundancy for performance):
CREATE TABLE order_items_denorm (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),  -- Denormalized
    quantity INT,
    price DECIMAL(10,2),        -- Denormalized (snapshot)
    PRIMARY KEY (order_id, product_id)
);

-- Pros: Faster queries (no joins to products table)
-- Cons: Redundancy, must maintain consistency

-- When to denormalize:
-- 1. Read-heavy workload (100:1 read:write ratio)
-- 2. Query performance is critical
-- 3. Data rarely changes
-- 4. Can maintain consistency (triggers, application logic)
```

**Key points to discuss:**
- Each normal form builds on previous
- Normalization reduces redundancy
- Sometimes denormalization is appropriate
- Balance between consistency and performance

---

### Senior Level (5-10 years)

These questions test system design, advanced optimization, and production expertise.

#### Question 6: Design a database schema for a Twitter-like social media platform. Consider scalability.

**What they're testing:**
- Schema design for complex systems
- Understanding of scaling challenges
- Denormalization strategies
- Sharding and partitioning

**Answer:**

```sql
-- Requirements analysis:
-- 1. Users can post tweets (280 chars)
-- 2. Users can follow other users
-- 3. Timeline shows tweets from followed users
-- 4. Tweets can have likes, retweets, replies
-- 5. Scale: 500M users, 500M tweets/day

-- Core schema:

CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    display_name VARCHAR(100),
    bio TEXT,
    profile_image_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT NOW(),
    follower_count INT DEFAULT 0,
    following_count INT DEFAULT 0,
    tweet_count INT DEFAULT 0
);

CREATE INDEX idx_users_username ON users(username);

CREATE TABLE tweets (
    tweet_id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    content VARCHAR(280) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    like_count INT DEFAULT 0,
    retweet_count INT DEFAULT 0,
    reply_count INT DEFAULT 0,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE INDEX idx_tweets_user_created ON tweets(user_id, created_at DESC);
CREATE INDEX idx_tweets_created ON tweets(created_at DESC);

CREATE TABLE follows (
    follower_id BIGINT NOT NULL,
    following_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (follower_id, following_id),
    FOREIGN KEY (follower_id) REFERENCES users(user_id),
    FOREIGN KEY (following_id) REFERENCES users(user_id)
);

CREATE INDEX idx_follows_following ON follows(following_id);

CREATE TABLE likes (
    user_id BIGINT NOT NULL,
    tweet_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (user_id, tweet_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (tweet_id) REFERENCES tweets(tweet_id)
);

CREATE INDEX idx_likes_tweet ON likes(tweet_id);

-- Scalability challenges:

-- Challenge 1: Timeline query (feed generation)
-- Naive approach:
SELECT t.*
FROM tweets t
JOIN follows f ON t.user_id = f.following_id
WHERE f.follower_id = ?  -- Current user
ORDER BY t.created_at DESC
LIMIT 20;

-- Problem: 
-- - If user follows 1000 people, must scan many tweets
-- - Hot users (1M followers) create hotspots
-- - Can't scale to 500M users

-- Solution 1: Fan-out on write (pre-compute timelines)
CREATE TABLE timelines (
    user_id BIGINT NOT NULL,
    tweet_id BIGINT NOT NULL,
    created_at TIMESTAMP NOT NULL,
    PRIMARY KEY (user_id, created_at, tweet_id)
);

-- When user posts tweet:
-- 1. Insert into tweets table
-- 2. Insert into timelines for all followers
INSERT INTO timelines (user_id, tweet_id, created_at)
SELECT follower_id, :new_tweet_id, :created_at
FROM follows
WHERE following_id = :author_id;

-- Timeline query becomes:
SELECT t.*
FROM timelines tl
JOIN tweets t ON tl.tweet_id = t.tweet_id
WHERE tl.user_id = ?
ORDER BY tl.created_at DESC
LIMIT 20;

-- Fast! But...
-- Problem: Celebrities with 10M followers → 10M inserts per tweet!

-- Solution 2: Hybrid approach
-- - Normal users: Fan-out on write (pre-compute)
-- - Celebrities: Fan-out on read (query on demand)

-- Mark celebrity accounts:
ALTER TABLE users ADD COLUMN is_celebrity BOOLEAN DEFAULT FALSE;

-- Timeline query (hybrid):
-- 1. Get timeline from timelines table (pre-computed)
-- 2. Get tweets from celebrities user follows (on-demand)
-- 3. Merge and sort

WITH pre_computed AS (
    SELECT t.*
    FROM timelines tl
    JOIN tweets t ON tl.tweet_id = t.tweet_id
    WHERE tl.user_id = ?
    AND tl.created_at >= NOW() - INTERVAL '7 days'
),
celebrity_tweets AS (
    SELECT t.*
    FROM tweets t
    JOIN follows f ON t.user_id = f.following_id
    JOIN users u ON t.user_id = u.user_id
    WHERE f.follower_id = ?
      AND u.is_celebrity = TRUE
      AND t.created_at >= NOW() - INTERVAL '7 days'
)
SELECT * FROM pre_computed
UNION ALL
SELECT * FROM celebrity_tweets
ORDER BY created_at DESC
LIMIT 20;

-- Challenge 2: Partitioning for scale

-- Partition tweets by date (time-series data):
CREATE TABLE tweets (
    tweet_id BIGINT,
    user_id BIGINT,
    content VARCHAR(280),
    created_at TIMESTAMP,
    -- ...
) PARTITION BY RANGE (created_at);

CREATE TABLE tweets_2024_01 PARTITION OF tweets
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE tweets_2024_02 PARTITION OF tweets
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- Benefits:
-- - Old tweets archived/dropped easily
-- - Queries on recent tweets only scan relevant partitions

-- Challenge 3: Sharding users

-- Shard by user_id hash:
-- Shard 0: user_id % 64 = 0
-- Shard 1: user_id % 64 = 1
-- ...
-- Shard 63: user_id % 64 = 63

-- Application-level routing:
def get_shard(user_id):
    return user_id % 64

def get_user(user_id):
    shard = get_shard(user_id)
    return shard_connections[shard].query(
        "SELECT * FROM users WHERE user_id = ?", user_id
    )

-- Challenges:
-- - Cross-shard queries expensive
-- - Resharding difficult (changing % 64 to % 128)
-- - Hotspots if celebrity on one shard

-- Challenge 4: Caching strategy

-- Cache hot data in Redis:
-- - User profiles (1 hour TTL)
-- - Tweet content (permanent)
-- - Timelines (15 minutes TTL)
-- - Follower counts (5 minutes TTL)

-- Example:
def get_timeline(user_id):
    cache_key = f"timeline:{user_id}"
    
    # Try cache
    cached = redis.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Cache miss - query database
    timeline = db.query("SELECT ... FROM timelines WHERE user_id = ?", user_id)
    
    # Store in cache
    redis.setex(cache_key, 900, json.dumps(timeline))  # 15 min TTL
    
    return timeline

-- Additional considerations:

-- 1. Counters (follower_count, like_count):
-- Denormalized for performance
-- Updated with background jobs (eventual consistency OK)

-- 2. Trending topics:
-- Separate analytics database (e.g., ClickHouse)
-- Stream processing (Kafka + Flink)

-- 3. Media storage:
-- Store in S3/object storage, not database
-- Only URLs in database

-- 4. Search:
-- Elasticsearch for full-text search
-- Sync tweets to Elasticsearch

-- 5. Rate limiting:
-- Redis for request counting
-- Prevent spam/abuse
```

**Discussion points:**
- Trade-offs between fan-out on write vs read
- When to use caching vs pre-computation
- Sharding strategies and challenges
- Consistency vs availability (CAP theorem)
- Monitoring and observability

---

#### Question 7: You're paged at 2 AM. The database is running slow and users are complaining. How do you diagnose and fix it?

**What they're testing:**
- Troubleshooting methodology
- Production debugging skills
- Stress management
- Communication

**Answer:**

```sql
-- Systematic troubleshooting process:

-- Step 1: Assess severity (2 minutes)
-- Check monitoring dashboard:
-- - Is the database up?
-- - What's the error rate?
-- - How many users affected?

-- If database is down:
-- Priority: Restore service (failover to replica if available)

-- If database is slow but functional:
-- Priority: Identify bottleneck

-- Step 2: Check system resources (3 minutes)

-- CPU usage:
top
# Look for:
# - CPU > 90%? (query bottleneck)
# - High iowait? (disk bottleneck)

-- Memory:
free -h
# Look for:
# - Out of memory?
# - Excessive swap usage?

-- Disk I/O:
iostat -x 1
# Look for:
# - %util near 100%? (disk saturation)
# - High await? (slow disk)

-- Network:
iftop
# Look for:
# - Bandwidth saturation?

-- Step 3: Check database-specific metrics (5 minutes)

-- PostgreSQL:
psql -c "SELECT * FROM pg_stat_activity WHERE state = 'active';"
# Look for:
# - Long-running queries (> 5 minutes)
# - Many connections (> 80% of max_connections)
# - Queries stuck in "idle in transaction"

psql -c "SELECT * FROM pg_stat_database;"
# Look for:
# - Sudden spike in queries
# - High number of conflicts

-- Active connections:
psql -c "SELECT COUNT(*) FROM pg_stat_activity WHERE state = 'active';"

-- Locks:
psql -c "
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocking_locks.pid AS blocking_pid,
    blocked_activity.query AS blocked_query,
    blocking_activity.query AS blocking_query
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
"

-- MySQL:
mysql -e "SHOW PROCESSLIST;"
# Look for similar issues

mysql -e "SHOW ENGINE INNODB STATUS\G"
# Check for deadlocks, lock waits

-- Step 4: Identify the root cause

-- Common scenarios:

-- Scenario A: Long-running query blocking others
-- Symptom: One query running for 30 minutes, many queries waiting
-- Solution:
psql -c "SELECT pg_terminate_backend(12345);"  -- Kill blocking query

-- Scenario B: Connection pool exhausted
-- Symptom: max_connections reached, new connections rejected
-- Quick fix: Increase max_connections
psql -c "ALTER SYSTEM SET max_connections = 500;"
psql -c "SELECT pg_reload_conf();"

-- Long-term fix: Implement connection pooling (PgBouncer)

-- Scenario C: Missing index causing table scans
-- Symptom: High CPU, slow SELECT queries
-- Diagnosis:
psql -c "SELECT query, calls, mean_exec_time 
         FROM pg_stat_statements 
         ORDER BY mean_exec_time DESC 
         LIMIT 5;"

-- Check EXPLAIN for slow query:
EXPLAIN ANALYZE <slow_query>;
-- Look for: Seq Scan on large table

-- Quick fix: Create index
CREATE INDEX CONCURRENTLY idx_temp ON orders(customer_id);

-- Scenario D: Autovacuum falling behind
-- Symptom: Slow queries, table bloat
-- Diagnosis:
psql -c "SELECT schemaname, relname, n_dead_tup, n_live_tup
         FROM pg_stat_user_tables 
         WHERE n_dead_tup > 10000
         ORDER BY n_dead_tup DESC;"

-- Quick fix: Manual VACUUM
psql -c "VACUUM ANALYZE orders;"

-- Scenario E: Disk full
-- Symptom: Writes failing
-- Diagnosis:
df -h
# /var/lib/postgresql: 99% used

-- Quick fix: Delete old WAL files (after archiving)
# Or increase disk space

-- Scenario F: Replication lag
-- Symptom: Stale data on replica
-- Diagnosis:
psql -c "SELECT pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_bytes
         FROM pg_stat_replication;"

-- If lag > 1GB: Issue with replica

-- Step 5: Apply fix and verify (5-10 minutes)

-- After applying fix:
-- 1. Monitor query performance
-- 2. Check error rates
-- 3. Verify user experience improved
-- 4. Document what happened

-- Step 6: Post-incident (next day)

-- 1. Write incident report
-- 2. Identify root cause
-- 3. Implement permanent fix
-- 4. Add monitoring/alerts to prevent recurrence
-- 5. Share learnings with team

-- Communication during incident:

-- Update stakeholders every 15-30 minutes:
-- "Identified issue: Missing index causing slow queries. 
--  Creating index now. ETA: 5 minutes."

-- After resolution:
-- "Issue resolved. Root cause: Missing index on orders table.
--  Performance restored. Monitoring for 24 hours."

-- Example incident timeline:

-- 2:00 AM: Paged (database slow)
-- 2:02 AM: Confirmed database up but slow
-- 2:05 AM: Identified CPU at 95%, many active queries
-- 2:10 AM: Found long-running query causing lock contention
-- 2:12 AM: Killed blocking query, performance improved
-- 2:15 AM: Added index to prevent recurrence
-- 2:20 AM: Verified normal operations
-- 2:30 AM: Sent all-clear notification
-- Next day: Post-mortem and permanent fixes
```

**Discussion points:**
- Prioritization (restore service first, investigate later)
- Communication with stakeholders
- Trade-offs between quick fixes and proper solutions
- Importance of monitoring and alerting
- Post-incident reviews and continuous improvement

---

### Staff/Principal Level (10+ years)

These questions test strategic thinking, architectural decisions, and technical leadership.

#### Question 8: Your company is growing from 1M to 100M users. How do you evolve the database architecture?

**What they're testing:**
- Long-term architectural planning
- Understanding of scale challenges
- Cost-benefit analysis
- Risk management

**Answer:**

```
# Current state (1M users):
- Single PostgreSQL server (16 cores, 128GB RAM)
- Daily backup to S3
- ~1000 queries/second
- 500GB database size

# Target state (100M users):
- ~100,000 queries/second (100x increase)
- 50TB+ database size (100x increase)
- Multi-region deployment
- 99.99% uptime SLA

# Migration path:

## Phase 1: Vertical scaling (1M → 5M users)
Timeline: Months 0-6
Investment: ~$10K/month

1. Upgrade to larger instance
   - 64 cores, 512GB RAM, 5TB NVMe SSD
   - Handles 5,000 queries/second
   - Cost: $5K/month

2. Add read replicas
   - 3 replicas for read scaling
   - Load balancer distributes reads
   - Cost: $3K/month × 3 = $9K/month

3. Implement connection pooling
   - PgBouncer in transaction mode
   - 10,000 app connections → 100 database connections
   - Cost: Minimal

Total: ~$19K/month
Capacity: 5M users

## Phase 2: Horizontal scaling - Read replicas (5M → 20M users)
Timeline: Months 6-18
Investment: ~$50K/month

1. Add more read replicas (10 total)
   - Geographic distribution (US-East, US-West, EU)
   - Reduce latency for global users
   - Cost: $5K/month × 10 = $50K/month

2. Implement caching layer
   - Redis cluster (10 nodes)
   - 80%+ cache hit ratio
   - Cost: $2K/month × 10 = $20K/month

3. CDN for static assets
   - Reduce database queries for images, etc.
   - Cost: $5K/month

Total: ~$75K/month
Capacity: 20M users, 20,000 QPS

## Phase 3: Sharding (20M → 100M users)
Timeline: Months 18-36
Investment: ~$200K/month

1. Shard by user_id
   - 64 shards (PostgreSQL instances)
   - Application-level routing
   - Cost: $3K/month × 64 = $192K/month

2. Sharding implementation:

```sql
-- User ID structure: 64-bit
-- [8 bits: shard] [56 bits: user sequence]

-- User creation:
def create_user():
    shard = get_least_loaded_shard()  # Load balancing
    user_id = (shard << 56) | get_next_sequence(shard)
    
    shard_db = get_shard_connection(shard)
    shard_db.execute(
        "INSERT INTO users (user_id, ...) VALUES (?, ...)",
        user_id, ...
    )
    return user_id

-- Routing:
def get_shard_for_user(user_id):
    return (user_id >> 56) & 0xFF  # Extract shard from top 8 bits

def get_user(user_id):
    shard = get_shard_for_user(user_id)
    shard_db = get_shard_connection(shard)
    return shard_db.execute(
        "SELECT * FROM users WHERE user_id = ?",
        user_id
    )
```

3. Cross-shard queries:
   - Implement scatter-gather for analytics
   - Use separate OLAP database (ClickHouse) for reporting
   - Streaming pipeline (Kafka) to sync data

4. Data consistency:
   - Within shard: ACID via PostgreSQL
   - Cross-shard: Eventual consistency
   - Implement Saga pattern for distributed transactions

```
## Phase 4: Multi-region (Global scale)
Timeline: Months 36-48
Investment: ~$500K/month

1. Multi-region active-active
   - US: 24 shards
   - EU: 24 shards
   - Asia: 16 shards
   - Cost: $3K × 64 = $192K/month

2. Geographic routing
   - Route users to nearest region
   - Reduce latency from 200ms to 20ms

3. Data synchronization
   - Async replication between regions
   - Conflict resolution strategy (last-write-wins)

4. Disaster recovery
   - Automatic failover to other regions
   - RTO: < 1 minute, RPO: < 1 minute

## Phase 5: Specialization (100M+ users)
Timeline: Ongoing

1. Separate databases by workload:
   - User service: PostgreSQL (user profiles, auth)
   - Social graph: Graph database (Neo4j) for follows
   - Timeline: Cassandra (wide-column store) for feeds
   - Analytics: ClickHouse (OLAP) for reporting
   - Search: Elasticsearch for full-text search
   - Cache: Redis for hot data

2. Event-driven architecture:
   - Kafka for event streaming
   - Microservices consume events
   - Eventual consistency across services

3. Cost optimization:
   - Auto-scaling based on load
   - Reserved instances for baseline
   - Spot instances for batch processing

# Key decisions and trade-offs:

## Decision 1: PostgreSQL vs NoSQL
Choice: Start with PostgreSQL, add specialized NoSQL later
Reasoning:
- PostgreSQL handles 20M users well
- ACID guarantees important for user data
- Team expertise
- Add NoSQL for specific use cases (Cassandra for timelines)

## Decision 2: Sharding vs managed service (AWS Aurora)
Choice: Sharding
Reasoning:
- Aurora max connections: ~5000 (not enough)
- Sharding allows infinite horizontal scale
- Cost at scale: Sharding cheaper than Aurora

Trade-off:
- Complexity (application-level routing)
- Cross-shard queries difficult
- But necessary for 100M users

## Decision 3: Multi-region vs single region
Choice: Multi-region
Reasoning:
- Global user base
- Latency critical for UX
- Regulatory compliance (GDPR data residency)

Trade-off:
- Complexity (data synchronization, conflict resolution)
- Cost (3x infrastructure)
- But required for global scale

## Decision 4: Consistency model
Choice: Hybrid
Reasoning:
- Strong consistency: User profiles, payments
- Eventual consistency: Timelines, follower counts
- Different data has different requirements

# Risk management:

## Risk 1: Data loss during migration
Mitigation:
- Dual-write during migration (old + new schema)
- Verify data consistency before cutover
- Rollback plan
- Gradual migration (1% → 10% → 50% → 100%)

## Risk 2: Performance regression
Mitigation:
- Load testing at 2x expected traffic
- Gradual rollout with feature flags
- Automated performance tests in CI/CD
- Rollback within 5 minutes

## Risk 3: Cost overruns
Mitigation:
- Budget alerts at 80% spend
- Reserved instances for predictable load
- Auto-scaling policies
- Quarterly cost reviews

# Monitoring and observability:

1. Metrics (Prometheus + Grafana):
   - Query latency (p50, p95, p99)
   - Error rates
   - Throughput (QPS)
   - Resource utilization

2. Logging (ELK stack):
   - Slow query logs
   - Error logs
   - Audit logs

3. Tracing (Jaeger):
   - Distributed tracing
   - Identify bottlenecks in microservices

4. Alerting (PagerDuty):
   - Critical: Database down, p99 > 1s
   - Warning: Error rate > 1%, cache hit ratio < 80%

# Team structure:

1M users: 2 backend engineers
5M users: 5 engineers (2 focused on database)
20M users: 15 engineers (5 on database/infra team)
100M users: 50+ engineers (dedicated data platform team)

# Estimated timeline and cost:

Phase 1 (0-6 months): $20K/month → 5M users
Phase 2 (6-18 months): $75K/month → 20M users
Phase 3 (18-36 months): $200K/month → 100M users
Phase 4 (36-48 months): $500K/month → Global scale

Total investment: ~$10M over 4 years
```

**Discussion points:**
- When to invest in scalability vs deliver features
- Build vs buy decisions (managed services vs self-hosted)
- Team scaling and hiring strategy
- Technical debt management
- Communication with executive team (business case for investment)

---

## 6.2 System Design Patterns

### Pattern 1: Read-Heavy Workload

**Problem:** Social media feed, product catalog, news site

**Solution:**

```
Architecture:

┌─────────┐
│ Client  │
└────┬────┘
     │
     ↓
┌─────────────────┐
│ Load Balancer   │
└────┬────────────┘
     │
     ↓
┌────────────────────────────────┐
│   CDN (CloudFlare/CloudFront)  │
│   - Static assets (images, JS) │
│   - HTML caching (short TTL)   │
└────┬───────────────────────────┘
     │
     ↓
┌────────────────────────────────┐
│   Application Servers (10x)    │
│   - Connection pooling         │
│   - API endpoints              │
└────┬───────────────────────────┘
     │
     ├──→ Redis Cluster (Cache)
     │    - User sessions
     │    - Hot data (80% hit rate)
     │    - TTL: 15 minutes
     │
     └──→ Database Layer
          │
          ├──→ Primary (Writes)
          │    - PostgreSQL
          │    - 1 instance
          │
          └──→ Replicas (Reads)
               - 10 read replicas
               - Load balanced
               - < 1 second lag
```

**Implementation:**

```sql
-- Caching strategy:
def get_user_profile(user_id):
    # L1: Application cache (in-memory)
    if user_id in app_cache:
        return app_cache[user_id]
    
    # L2: Redis cache
    cache_key = f"user:{user_id}"
    cached = redis.get(cache_key)
    if cached:
        app_cache[user_id] = json.loads(cached)
        return app_cache[user_id]
    
    # L3: Database (replica)
    replica = get_read_replica()
    user = replica.query("SELECT * FROM users WHERE user_id = ?", user_id)
    
    # Store in caches
    redis.setex(cache_key, 900, json.dumps(user))  # 15 min TTL
    app_cache[user_id] = user
    
    return user

-- Read replica configuration:
# Primary: All writes
primary_db = connect("postgres://primary:5432/mydb")

# Replicas: Round-robin for reads
read_replicas = [
    connect("postgres://replica1:5432/mydb"),
    connect("postgres://replica2:5432/mydb"),
    connect("postgres://replica3:5432/mydb"),
    # ... 10 total
]

replica_index = 0

def get_read_replica():
    global replica_index
    replica = read_replicas[replica_index]
    replica_index = (replica_index + 1) % len(read_replicas)
    return replica

def write_query(sql, params):
    return primary_db.execute(sql, params)

def read_query(sql, params):
    replica = get_read_replica()
    return replica.execute(sql, params)
```

**Metrics:**
- Read:Write ratio: 100:1
- Cache hit rate: 85%+
- Database load reduction: 85%
- Average response time: < 100ms

---

### Pattern 2: Write-Heavy Workload

**Problem:** Analytics, logging, IoT data ingestion

**Solution:**

```
Architecture:

┌─────────┐
│ Clients │ (1000s/sec)
└────┬────┘
     │
     ↓
┌────────────────────────────────┐
│   Message Queue (Kafka)        │
│   - Buffer writes              │
│   - Decouples producers/consumers │
│   - Partitioned (64 partitions) │
└────┬───────────────────────────┘
     │
     ↓
┌────────────────────────────────┐
│   Consumer Workers (20x)       │
│   - Batch processing           │
│   - 1000 events → 1 transaction │
└────┬───────────────────────────┘
     │
     ↓
┌────────────────────────────────┐
│   Time-Series Database         │
│   - ClickHouse or TimescaleDB  │
│   - Optimized for append-only  │
│   - Columnar storage           │
│   - Compressed                 │
└────────────────────────────────┘
```

**Implementation:**

```python
# Producer (application):
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers=['kafka1:9092', 'kafka2:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

def log_event(user_id, event_type, data):
    message = {
        'user_id': user_id,
        'event_type': event_type,
        'data': data,
        'timestamp': time.time()
    }
    
    # Partition by user_id for ordering
    producer.send(
        'events',
        value=message,
        key=str(user_id).encode('utf-8')
    )

# Consumer (worker):
from kafka import KafkaConsumer

consumer = KafkaConsumer(
    'events',
    bootstrap_servers=['kafka1:9092', 'kafka2:9092'],
    group_id='events-to-db',
    auto_offset_reset='earliest',
    enable_auto_commit=False
)

def batch_insert(events):
    # Batch 1000 events into single INSERT
    values = ','.join([
        f"({e['user_id']}, '{e['event_type']}', '{json.dumps(e['data'])}', '{e['timestamp']}')"
        for e in events
    ])
    
    db.execute(f"""
        INSERT INTO events (user_id, event_type, data, timestamp)
        VALUES {values}
    """)
    
    db.commit()

# Process in batches
batch = []
for message in consumer:
    event = json.loads(message.value)
    batch.append(event)
    
    if len(batch) >= 1000:
        batch_insert(batch)
        consumer.commit()  # Commit offset after successful insert
        batch = []
```

```sql
-- ClickHouse table (optimized for analytics):
CREATE TABLE events (
    user_id UInt64,
    event_type String,
    data String,
    timestamp DateTime
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (user_id, timestamp);

-- Benefits:
-- - Columnar storage (compressed)
-- - Partitioned by month (easy to drop old data)
-- - Ordered by user_id (efficient range queries)
-- - Append-only (no updates/deletes)

-- Query performance:
SELECT 
    event_type,
    COUNT(*) AS count
FROM events
WHERE timestamp >= now() - INTERVAL 1 DAY
GROUP BY event_type;

-- Scans only 1 day partition
-- Columnar scan of event_type column only
-- Result: < 1 second on billions of rows
```

**Metrics:**
- Write throughput: 100,000 events/second
- Batch size: 1,000 events
- Insert latency: < 10ms per batch
- Storage: 10:1 compression ratio
- Query latency: < 1s on 1B+ rows

---

## Key Takeaways

* **Junior interviews test fundamentals** - SQL syntax, basic joins, simple optimization
* **Mid-level interviews test depth** - execution plans, normalization, index design
* **Senior interviews test breadth** - system design, production troubleshooting, architecture
* **Communication is as important as correctness** - explain your thinking clearly
* **Practice common patterns** - second highest, N+1 queries, schema design
* **Understand trade-offs** - every decision has pros and cons
* **System design is iterative** - start simple, identify bottlenecks, scale appropriately
* **Read-heavy workloads** need caching and read replicas
* **Write-heavy workloads** need message queues and batch processing
* **Real-world systems are messy** - perfect solutions don't exist, balance is key

---

*This concludes the Complete SQL Mastery Series - thank you for learning with us!*
