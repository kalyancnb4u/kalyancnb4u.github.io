---
title: "SQL Interview Flashcards - 200 Questions"
date: 2026-01-01 16:00:00 +0000
categories: [Database, SQL, Interview]
tags: [sql, flashcards, interview, study, review]
---

# SQL Interview Flashcards - 200 Questions

Quick-review flashcards for SQL interview preparation. Organized by topic and difficulty.

**Usage:** Cover the answer, try to explain the concept, then check your response.

---

## How to Use These Flashcards

1. **Spaced Repetition:** Review daily for 15-20 minutes
2. **Active Recall:** Try to answer before looking
3. **Difficulty Tracking:** Mark cards you struggle with
4. **Teach Back:** Explain concepts out loud
5. **Mix It Up:** Shuffle topics for better retention

---

## SECTION 1: BASICS (Cards 1-40)

### Card 1
**Q:** What does SQL stand for?
**A:** Structured Query Language - a domain-specific language for managing relational databases.

### Card 2
**Q:** What are the main SQL command categories?
**A:** 
- **DDL** (Data Definition Language): CREATE, ALTER, DROP
- **DML** (Data Manipulation Language): SELECT, INSERT, UPDATE, DELETE
- **DCL** (Data Control Language): GRANT, REVOKE
- **TCL** (Transaction Control Language): COMMIT, ROLLBACK, SAVEPOINT

### Card 3
**Q:** What's the difference between CHAR and VARCHAR?
**A:** 
- **CHAR(n):** Fixed-length, pads with spaces, faster for fixed-size data
- **VARCHAR(n):** Variable-length, saves storage, flexible

### Card 4
**Q:** How do you handle NULL values in SQL?
**A:** 
- Use `IS NULL` / `IS NOT NULL` for comparisons
- Use `COALESCE(col, default)` for default values
- Use `NULLIF(val1, val2)` to return NULL if values equal
- Remember: NULL in any arithmetic = NULL

### Card 5
**Q:** What's the logical execution order of a SELECT query?
**A:** FROM → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT

### Card 6
**Q:** Why can't you use a SELECT alias in the WHERE clause?
**A:** Because WHERE executes before SELECT in the logical processing order. The alias doesn't exist yet when WHERE is evaluated.

### Card 7
**Q:** What's the difference between WHERE and HAVING?
**A:**
- **WHERE:** Filters rows before GROUP BY, cannot use aggregates
- **HAVING:** Filters groups after aggregation, can use aggregates

### Card 8
**Q:** Name the aggregate functions in SQL.
**A:** COUNT(), SUM(), AVG(), MIN(), MAX(), GROUP_CONCAT()/STRING_AGG()

### Card 9
**Q:** How does COUNT(*) differ from COUNT(column)?
**A:**
- **COUNT(*):** Counts all rows including NULLs
- **COUNT(column):** Counts non-NULL values only
- **COUNT(DISTINCT column):** Counts unique non-NULL values

### Card 10
**Q:** What does DISTINCT do?
**A:** Removes duplicate rows from result set. Applies to entire row, not individual columns.

### Card 11
**Q:** Explain the BETWEEN operator.
**A:** Inclusive range check. `price BETWEEN 100 AND 200` means `price >= 100 AND price <= 200`.

### Card 12
**Q:** What wildcards are used with LIKE?
**A:**
- **%:** Zero or more characters
- **_:** Exactly one character

### Card 13
**Q:** What's the difference between IN and EXISTS?
**A:**
- **IN:** Tests membership in a set, evaluates subquery fully
- **EXISTS:** Tests for existence, short-circuits after first match (often faster)

### Card 14
**Q:** How do you prevent SQL injection?
**A:** Use parameterized queries (prepared statements), never concatenate user input directly into SQL strings.

### Card 15
**Q:** What's the difference between DELETE and TRUNCATE?
**A:**
- **DELETE:** Row-by-row, can use WHERE, slower, logged, rollbackable
- **TRUNCATE:** All rows, fast, minimal logging, can't use WHERE

### Card 16
**Q:** What's a primary key?
**A:** Unique identifier for rows. Automatically creates unique index. Cannot be NULL. Enforces entity integrity.

### Card 17
**Q:** What's a foreign key?
**A:** Column(s) that reference primary key in another table. Enforces referential integrity.

### Card 18
**Q:** What's the difference between UNION and UNION ALL?
**A:**
- **UNION:** Combines results, removes duplicates (slower)
- **UNION ALL:** Combines results, keeps duplicates (faster)

### Card 19
**Q:** How do you sort results in descending order?
**A:** `ORDER BY column DESC` (ASC is default for ascending)

### Card 20
**Q:** What does LIMIT do?
**A:** Restricts number of rows returned. `LIMIT 10` returns first 10 rows.

### Card 21
**Q:** What's the difference between = and LIKE?
**A:**
- **=:** Exact match
- **LIKE:** Pattern matching with wildcards (%, _)

### Card 22
**Q:** What does COALESCE do?
**A:** Returns first non-NULL value from a list. `COALESCE(col1, col2, 0)` returns col1 if not NULL, else col2, else 0.

### Card 23
**Q:** What's a composite key?
**A:** Primary key made of multiple columns. Example: (student_id, course_id) in enrollment table.

### Card 24
**Q:** What's the difference between DROP and DELETE?
**A:**
- **DROP:** Removes entire table structure
- **DELETE:** Removes rows, keeps structure

### Card 25
**Q:** What's an index?
**A:** Database structure that improves query performance by creating ordered reference to table data. Like a book index.

### Card 26
**Q:** When should you NOT use SELECT *?
**A:** Production code. Reasons: Wastes I/O, breaks when schema changes, can't use covering indexes, unclear intent.

### Card 27
**Q:** What's the difference between INT and BIGINT?
**A:**
- **INT:** 4 bytes, ±2.1 billion range
- **BIGINT:** 8 bytes, ±9.2 quintillion range

### Card 28
**Q:** What's the purpose of GROUP BY?
**A:** Groups rows with same values into summary rows. Required when mixing aggregate functions with regular columns.

### Card 29
**Q:** Can you use ORDER BY with an alias?
**A:** Yes. ORDER BY executes after SELECT, so aliases are available.

### Card 30
**Q:** What's a subquery?
**A:** Query nested inside another query. Can be scalar (single value), row, or table subquery.

### Card 31
**Q:** What's the difference between INNER JOIN and LEFT JOIN?
**A:**
- **INNER JOIN:** Returns only matching rows from both tables
- **LEFT JOIN:** Returns all rows from left table, NULL for non-matches from right

### Card 32
**Q:** What does AS keyword do?
**A:** Creates alias for column or table. Makes queries more readable. Optional in most cases.

### Card 33
**Q:** What's a view?
**A:** Virtual table based on a query. Doesn't store data, only the query definition.

### Card 34
**Q:** What's ACID in databases?
**A:**
- **Atomicity:** All-or-nothing transactions
- **Consistency:** Data integrity maintained
- **Isolation:** Transactions don't interfere
- **Durability:** Committed changes persist

### Card 35
**Q:** What's the default sort order for NULL values?
**A:** MySQL: NULLs first in ASC, last in DESC. PostgreSQL: Can specify NULLS FIRST/LAST.

### Card 36
**Q:** What does DISTINCT ON do? (PostgreSQL)
**A:** Returns first row for each distinct value. `DISTINCT ON (category)` returns one row per category.

### Card 37
**Q:** What's a transaction?
**A:** Sequence of SQL operations treated as single unit. Either all succeed (COMMIT) or all fail (ROLLBACK).

### Card 38
**Q:** What's the difference between DECIMAL and FLOAT?
**A:**
- **DECIMAL:** Exact precision, for money
- **FLOAT:** Approximate, for scientific data

### Card 39
**Q:** What does CASE expression do?
**A:** Implements conditional logic (if-then-else). Returns different values based on conditions.

### Card 40
**Q:** What's the difference between simple CASE and searched CASE?
**A:**
- **Simple:** Compares one expression: `CASE status WHEN 'active' THEN...`
- **Searched:** Independent conditions: `CASE WHEN price > 100 THEN...`

---

## SECTION 2: JOINS (Cards 41-70)

### Card 41
**Q:** What are the main types of JOINs?
**A:** INNER, LEFT (OUTER), RIGHT (OUTER), FULL OUTER, CROSS, SELF

### Card 42
**Q:** What's a CROSS JOIN?
**A:** Cartesian product - every row from first table paired with every row from second table.

### Card 43
**Q:** What's a SELF JOIN?
**A:** Joining a table to itself. Used for hierarchical data (employees-managers) or comparing rows within same table.

### Card 44
**Q:** When do you need to use table aliases?
**A:** Required for self joins, recommended for all joins for clarity. Makes queries more readable.

### Card 45
**Q:** What's the difference between ON and WHERE in joins?
**A:** 
- **ON:** Join condition (executes during join)
- **WHERE:** Filter results after join
- For OUTER joins, this matters significantly!

### Card 46
**Q:** What's an equi-join vs non-equi join?
**A:**
- **Equi-join:** Uses = in condition (most common)
- **Non-equi join:** Uses !=, <, >, BETWEEN (less common)

### Card 47
**Q:** What's a natural join?
**A:** Automatically joins on columns with same name. Generally avoided - too implicit and error-prone.

### Card 48
**Q:** How do you find records in table A not in table B?
**A:** `LEFT JOIN B ON A.id = B.id WHERE B.id IS NULL` or `NOT EXISTS` subquery.

### Card 49
**Q:** What are the join algorithms?
**A:** Nested Loop, Hash Join, Merge Join. Database chooses based on table size and indexes.

### Card 50
**Q:** What's a LATERAL join? (PostgreSQL)
**A:** Allows subquery on right side to reference columns from left side. Like a for-each loop.

### Card 51
**Q:** Can you JOIN more than two tables?
**A:** Yes. JOINs are left-associative: (A JOIN B) JOIN C.

### Card 52
**Q:** What's the purpose of USING clause?
**A:** Shorthand for join when column names match. `JOIN USING (customer_id)` instead of `ON a.customer_id = b.customer_id`.

### Card 53
**Q:** What happens if you forget the ON clause?
**A:** Creates accidental CROSS JOIN (Cartesian product) - usually unintended and slow.

### Card 54
**Q:** How do you join three tables?
**A:** Chain joins: `FROM a JOIN b ON a.id = b.id JOIN c ON b.id = c.id`

### Card 55
**Q:** What's the difference between JOIN and INNER JOIN?
**A:** They're identical. JOIN defaults to INNER JOIN.

### Card 56
**Q:** When should you use RIGHT JOIN?
**A:** Almost never. Use LEFT JOIN and swap table order instead for consistency.

### Card 57
**Q:** What's a semi-join?
**A:** Returns rows from left table where match exists in right table (like EXISTS). Doesn't multiply rows.

### Card 58
**Q:** What's an anti-join?
**A:** Returns rows from left table where NO match exists in right table (like NOT EXISTS).

### Card 59
**Q:** Why index foreign keys?
**A:** Critical for join performance. Without index, database does full table scan for each join.

### Card 60
**Q:** What's the difference between these two queries?
```sql
-- A
WHERE a.type = 'X' AND b.status = 'Y'
-- B  
ON a.id = b.id AND a.type = 'X' AND b.status = 'Y'
```
**A:** For INNER JOIN, same result. For OUTER JOIN, different - ON filters before join, WHERE filters after.

### Card 61
**Q:** How do you optimize slow joins?
**A:** 
1. Index join columns
2. Filter before joining (subquery/WHERE)
3. Use appropriate join type
4. Check execution plan with EXPLAIN

### Card 62
**Q:** What's a many-to-many relationship?
**A:** Relationship where multiple records in table A relate to multiple in table B. Requires junction table.

### Card 63
**Q:** What's a one-to-many relationship?
**A:** One record in table A relates to multiple records in table B. Example: Customer has many Orders.

### Card 64
**Q:** What's a junction table?
**A:** Table that connects two tables in many-to-many relationship. Contains foreign keys to both tables.

### Card 65
**Q:** Can you JOIN on multiple columns?
**A:** Yes. `ON a.col1 = b.col1 AND a.col2 = b.col2`

### Card 66
**Q:** What's the result of LEFT JOIN when right table has no matches?
**A:** All left table rows returned, right table columns filled with NULL.

### Card 67
**Q:** How do you find rows that exist in both tables?
**A:** INNER JOIN, or INTERSECT (PostgreSQL).

### Card 68
**Q:** What's the difference between implicit and explicit joins?
**A:**
- **Implicit:** `FROM a, b WHERE a.id = b.id` (old style, avoid)
- **Explicit:** `FROM a JOIN b ON a.id = b.id` (modern, preferred)

### Card 69
**Q:** Can you join a table to a subquery?
**A:** Yes. Subquery acts as derived table: `FROM orders o JOIN (SELECT...) sq ON o.id = sq.id`

### Card 70
**Q:** What causes duplicate rows in joins?
**A:** One-to-many relationship: one row from left table matches multiple rows from right table.

---

## SECTION 3: SUBQUERIES & CTEs (Cards 71-100)

### Card 71
**Q:** What's a correlated subquery?
**A:** Subquery that references columns from outer query. Executes once per outer row (can be slow).

### Card 72
**Q:** What's an uncorrelated subquery?
**A:** Subquery independent of outer query. Executes once, result cached.

### Card 73
**Q:** What's a scalar subquery?
**A:** Subquery that returns single value (1 row, 1 column). Used where single value expected.

### Card 74
**Q:** What happens if scalar subquery returns multiple rows?
**A:** Error! Scalar subquery must return exactly one row and one column.

### Card 75
**Q:** What's a derived table?
**A:** Subquery in FROM clause. Acts as temporary table.

### Card 76
**Q:** What's a CTE (Common Table Expression)?
**A:** Temporary named result set defined using WITH clause. Improves readability.

### Card 77
**Q:** What's the syntax for a CTE?
**A:** `WITH cte_name AS (SELECT...) SELECT * FROM cte_name;`

### Card 78
**Q:** Can you define multiple CTEs?
**A:** Yes. Separate with commas: `WITH cte1 AS (...), cte2 AS (...)`

### Card 79
**Q:** What's a recursive CTE?
**A:** CTE that references itself. Used for hierarchical data, graphs, sequences.

### Card 80
**Q:** What are the parts of a recursive CTE?
**A:** 
1. Anchor member (base case)
2. UNION ALL
3. Recursive member (references CTE)
4. Termination condition

### Card 81
**Q:** When should you use EXISTS over IN?
**A:** 
- Large inner table (EXISTS can short-circuit)
- Checking existence (not actual values)
- Avoiding NULL problems with NOT IN

### Card 82
**Q:** What's the NULL problem with NOT IN?
**A:** If subquery contains NULL, NOT IN returns zero rows. Use NOT EXISTS instead.

### Card 83
**Q:** What does ANY operator do?
**A:** Returns TRUE if condition is true for at least one value. `> ANY (...)` = greater than minimum.

### Card 84
**Q:** What does ALL operator do?
**A:** Returns TRUE if condition is true for all values. `> ALL (...)` = greater than maximum.

### Card 85
**Q:** How do you avoid N+1 query problem?
**A:** Use JOINs or batch queries instead of subqueries in SELECT clause.

### Card 86
**Q:** What's the difference between CTE and subquery?
**A:** 
- **CTE:** Named, can reference multiple times, better readability
- **Subquery:** Anonymous, repeated if used twice
- Functionally similar in simple cases

### Card 87
**Q:** When are CTEs materialized?
**A:** PostgreSQL: Default materialized (can use NOT MATERIALIZED). MySQL: Optimized automatically.

### Card 88
**Q:** What's the advantage of CTEs over temporary tables?
**A:** CTEs exist only for query duration, no creation/cleanup overhead, cleaner syntax.

### Card 89
**Q:** Can CTEs reference earlier CTEs?
**A:** Yes. `WITH a AS (...), b AS (SELECT * FROM a)` - b can reference a.

### Card 90
**Q:** What's the maximum recursion depth in CTEs?
**A:** Database-specific. MySQL: 1000 (configurable), PostgreSQL: 100 (configurable).

### Card 91
**Q:** How do you prevent infinite recursion?
**A:** Include termination condition in recursive member's WHERE clause.

### Card 92
**Q:** Can you UPDATE/DELETE with CTEs?
**A:** PostgreSQL: Yes. `WITH cte AS (...) DELETE FROM table USING cte...` MySQL: No.

### Card 93
**Q:** What's the difference between IN and = ANY?
**A:** They're equivalent. `col IN (1, 2, 3)` = `col = ANY(ARRAY[1,2,3])`

### Card 94
**Q:** When should you use a temp table instead of CTE?
**A:** When you need indexes, statistics, or reuse across multiple queries.

### Card 95
**Q:** Can you nest subqueries?
**A:** Yes, but gets hard to read. Use CTEs for better readability.

### Card 96
**Q:** What's a lateral reference?
**A:** Subquery that references columns from earlier in FROM clause (requires LATERAL keyword in PostgreSQL).

### Card 97
**Q:** How do you optimize correlated subqueries?
**A:** Convert to JOIN or use window functions instead.

### Card 98
**Q:** Can subqueries be used in SELECT, WHERE, and HAVING?
**A:** Yes, in all three. Also in FROM (derived tables).

### Card 99
**Q:** What's the difference between UNION and UNION ALL in recursive CTEs?
**A:** Must use UNION ALL in recursive CTEs. UNION (which removes duplicates) not allowed.

### Card 100
**Q:** How do you generate a sequence with recursive CTE?
**A:** 
```sql
WITH RECURSIVE seq AS (
    SELECT 1 as n
    UNION ALL
    SELECT n+1 FROM seq WHERE n < 100
) SELECT * FROM seq;
```

---

## SECTION 4: WINDOW FUNCTIONS (Cards 101-130)

### Card 101
**Q:** What's a window function?
**A:** Function that performs calculation across set of rows related to current row WITHOUT collapsing rows (unlike GROUP BY).

### Card 102
**Q:** What's the basic syntax of a window function?
**A:** `function() OVER (PARTITION BY ... ORDER BY ... ROWS/RANGE ...)`

### Card 103
**Q:** What's the difference between ROW_NUMBER, RANK, and DENSE_RANK?
**A:**
- **ROW_NUMBER:** Unique (1,2,3,4,5)
- **RANK:** Skips after ties (1,1,3,4,5)
- **DENSE_RANK:** Consecutive (1,1,2,3,4)

### Card 104
**Q:** What does PARTITION BY do?
**A:** Divides rows into groups (like GROUP BY) but doesn't collapse rows. Function applies within each partition.

### Card 105
**Q:** What does OVER() without arguments do?
**A:** Applies function over entire result set (no partitioning).

### Card 106
**Q:** What's the difference between GROUP BY and PARTITION BY?
**A:**
- **GROUP BY:** Collapses rows into one per group
- **PARTITION BY:** Keeps all rows, computes within groups

### Card 107
**Q:** What does LAG function do?
**A:** Accesses value from previous row. `LAG(col, 1)` gets previous row's value.

### Card 108
**Q:** What does LEAD function do?
**A:** Accesses value from next row. `LEAD(col, 1)` gets next row's value.

### Card 109
**Q:** What's the default offset for LAG/LEAD?
**A:** 1 (previous/next row). Can specify: `LAG(col, 2)` for two rows back.

### Card 110
**Q:** What does FIRST_VALUE do?
**A:** Returns first value in window frame. Requires explicit frame for correct results.

### Card 111
**Q:** What does LAST_VALUE do?
**A:** Returns last value in window frame. **Must** specify `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`.

### Card 112
**Q:** Why doesn't LAST_VALUE work without a frame clause?
**A:** Default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, so "last value" = current row.

### Card 113
**Q:** What does NTILE do?
**A:** Divides rows into N equal-sized buckets. `NTILE(4)` creates quartiles.

### Card 114
**Q:** Can you use aggregate functions as window functions?
**A:** Yes. SUM() OVER, AVG() OVER, COUNT() OVER, etc. create running/cumulative calculations.

### Card 115
**Q:** What's a running total?
**A:** Cumulative sum: `SUM(amount) OVER (ORDER BY date)` adds up values progressively.

### Card 116
**Q:** How do you calculate a moving average?
**A:** `AVG(value) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` for 7-day average.

### Card 117
**Q:** What's the difference between ROWS and RANGE?
**A:**
- **ROWS:** Physical row positions
- **RANGE:** Logical value range (includes ties/peers)

### Card 118
**Q:** What are the frame clause options?
**A:** UNBOUNDED PRECEDING, N PRECEDING, CURRENT ROW, N FOLLOWING, UNBOUNDED FOLLOWING

### Card 119
**Q:** What's the default window frame?
**A:** With ORDER BY: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` Without ORDER BY: Entire partition

### Card 120
**Q:** Can you use window functions in WHERE clause?
**A:** No! Window functions execute after WHERE. Use subquery/CTE instead.

### Card 121
**Q:** How do you get top N per group with window functions?
**A:** 
```sql
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY category ORDER BY price DESC) as rn
    FROM products
)
SELECT * FROM ranked WHERE rn <= 3;
```

### Card 122
**Q:** What does CUME_DIST function do?
**A:** Cumulative distribution - percentage of values <= current value (0.0 to 1.0).

### Card 123
**Q:** What does PERCENT_RANK function do?
**A:** Relative rank as percentage (0.0 to 1.0). (rank - 1) / (total_rows - 1)

### Card 124
**Q:** Can you have multiple window functions in one query?
**A:** Yes! Each can have different OVER clause.

### Card 125
**Q:** What's the WINDOW clause? (PostgreSQL)
**A:** Define window once, reuse: `SELECT ... FROM ... WINDOW w AS (PARTITION BY ...)`

### Card 126
**Q:** When should you use window functions over GROUP BY?
**A:** When you need both row-level detail AND aggregate calculations in same result.

### Card 127
**Q:** How do you calculate period-over-period change?
**A:** `value - LAG(value) OVER (ORDER BY period)` or `(value - LAG(value)) / LAG(value) * 100` for percentage.

### Card 128
**Q:** What's NTH_VALUE function?
**A:** Returns value at Nth position in window. `NTH_VALUE(price, 2)` gets 2nd price in frame.

### Card 129
**Q:** Can window functions be nested?
**A:** No. Can't put window function inside another window function.

### Card 130
**Q:** How do you calculate running percentage of total?
**A:** `value * 100.0 / SUM(value) OVER ()`

---

## SECTION 5: PERFORMANCE & OPTIMIZATION (Cards 131-170)

### Card 131
**Q:** What's an index?
**A:** Database structure that improves query speed. Like a book index - helps find data without scanning everything.

### Card 132
**Q:** What's the most common index type?
**A:** B-tree (balanced tree). Default in most databases.

### Card 133
**Q:** What's a composite index?
**A:** Index on multiple columns. Order matters! Index on (A, B) helps queries filtering by A or (A and B), but not just B.

### Card 134
**Q:** What's a covering index?
**A:** Index that contains all columns needed for query. Database can satisfy query from index alone (index-only scan).

### Card 135
**Q:** When should you create an index?
**A:** 
- Foreign keys (join columns)
- WHERE clause columns
- ORDER BY columns
- Columns used frequently in queries

### Card 136
**Q:** When should you NOT create an index?
**A:**
- Small tables (< 1000 rows)
- Columns with low cardinality (few unique values)
- Frequently updated columns
- Too many indexes slow INSERTs/UPDATEs

### Card 137
**Q:** What's index selectivity?
**A:** Ratio of unique values to total rows. High selectivity (many unique values) = good index candidate.

### Card 138
**Q:** What's a partial index? (PostgreSQL)
**A:** Index on subset of rows: `CREATE INDEX ON users(email) WHERE status = 'active'`

### Card 139
**Q:** What's a functional/expression index?
**A:** Index on function result: `CREATE INDEX ON users(LOWER(email))` for case-insensitive searches.

### Card 140
**Q:** What does EXPLAIN do?
**A:** Shows query execution plan. Reveals if indexes are used, join methods, estimated costs.

### Card 141
**Q:** What does EXPLAIN ANALYZE do? (PostgreSQL)
**A:** Runs query AND shows execution plan with actual statistics (time, rows).

### Card 142
**Q:** What's a full table scan?
**A:** Reading every row in table. Shown as "type: ALL" in MySQL EXPLAIN. Slow for large tables.

### Card 143
**Q:** What's an index seek?
**A:** Using index to jump directly to matching rows. Fast. Shown as "type: ref" or "type: eq_ref" in MySQL.

### Card 144
**Q:** What's a sargable query?
**A:** Search ARGument ABLE - query that can use index. Example: `WHERE date >= '2024-01-01'` (sargable) vs `WHERE YEAR(date) = 2024` (not sargable).

### Card 145
**Q:** Why does function on column prevent index use?
**A:** Index stores column values, not function results. `WHERE YEAR(date) = 2024` can't use date index.

### Card 146
**Q:** How do you make date range queries sargable?
**A:** Use range operators: `WHERE date >= '2024-01-01' AND date < '2025-01-01'` instead of `WHERE YEAR(date) = 2024`

### Card 147
**Q:** What's query cardinality?
**A:** Number of rows returned. Low cardinality (few rows) benefits from indexes. High cardinality (many rows) might not.

### Card 148
**Q:** What's index cardinality?
**A:** Number of unique values in indexed column. High cardinality = better index selectivity.

### Card 149
**Q:** What's the difference between hash and B-tree index?
**A:**
- **B-tree:** Supports ranges (<, >, BETWEEN), most common
- **Hash:** Only equality (=), faster for exact matches

### Card 150
**Q:** What's a clustered index?
**A:** Table data physically sorted by index. One per table (usually primary key).

### Card 151
**Q:** What's a non-clustered index?
**A:** Separate structure pointing to table data. Can have many per table.

### Card 152
**Q:** What's ANALYZE TABLE?
**A:** Updates database statistics about table. Helps optimizer make better decisions.

### Card 153
**Q:** What are table statistics?
**A:** Metadata about table: row count, data distribution, null percentage. Used by optimizer.

### Card 154
**Q:** What's query plan caching?
**A:** Database saves execution plan for reuse. Prepared statements use this.

### Card 155
**Q:** What's the N+1 query problem?
**A:** Executing N queries in loop instead of one query with JOIN. Extremely slow.

### Card 156
**Q:** How do you fix N+1 queries?
**A:** Use JOIN or batch queries to fetch all needed data at once.

### Card 157
**Q:** What's query batching?
**A:** Combining multiple operations into single query/transaction for efficiency.

### Card 158
**Q:** What's pagination performance issue with OFFSET?
**A:** Large OFFSET requires scanning and discarding rows. `LIMIT 10 OFFSET 1000000` scans 1M rows to return 10.

### Card 159
**Q:** What's keyset pagination?
**A:** Pagination using WHERE clause instead of OFFSET: `WHERE id > last_seen_id LIMIT 10`. Much faster.

### Card 160
**Q:** What's a slow query log?
**A:** Database log of queries exceeding time threshold. Essential for finding performance bottlenecks.

### Card 161
**Q:** What's database normalization?
**A:** Organizing data to reduce redundancy. Typically aim for 3rd Normal Form (3NF).

### Card 162
**Q:** When should you denormalize?
**A:** For read-heavy workloads where query performance > data redundancy concerns. Requires careful consideration.

### Card 163
**Q:** What's a prepared statement?
**A:** Pre-compiled SQL with placeholders. Prevents SQL injection, improves performance through plan caching.

### Card 164
**Q:** What's connection pooling?
**A:** Reusing database connections instead of creating new ones. Critical for application performance.

### Card 165
**Q:** What's database partitioning?
**A:** Splitting large table into smaller pieces. Example: partition by date range.

### Card 166
**Q:** What's the cost of an index?
**A:** Slower writes (INSERT/UPDATE/DELETE), more storage. Balance against query performance gains.

### Card 167
**Q:** How do you identify missing indexes?
**A:** 
- Check slow query log
- EXPLAIN slow queries
- Look for full table scans
- Database-specific tools

### Card 168
**Q:** What's join order optimization?
**A:** Database chooses which table to scan first in joins. Based on statistics and indexes.

### Card 169
**Q:** What's predicate pushdown?
**A:** Optimization where database applies WHERE filters as early as possible in query execution.

### Card 170
**Q:** What's materialized view?
**A:** View with stored results. Faster queries but requires refresh/maintenance. PostgreSQL supports natively.

---

## SECTION 6: TRANSACTIONS & ADVANCED (Cards 171-200)

### Card 171
**Q:** What's a database transaction?
**A:** Sequence of operations treated as single unit. All succeed (COMMIT) or all fail (ROLLBACK).

### Card 172
**Q:** What's COMMIT?
**A:** Saves all transaction changes permanently to database.

### Card 173
**Q:** What's ROLLBACK?
**A:** Undoes all transaction changes, restoring to pre-transaction state.

### Card 174
**Q:** What's a SAVEPOINT?
**A:** Checkpoint within transaction. Can rollback to savepoint without rolling back entire transaction.

### Card 175
**Q:** What are the ACID properties?
**A:**
- **Atomicity:** All-or-nothing
- **Consistency:** Valid state transitions
- **Isolation:** Transactions don't interfere
- **Durability:** Committed changes persist

### Card 176
**Q:** What are the transaction isolation levels?
**A:** Read Uncommitted, Read Committed, Repeatable Read, Serializable (increasingly strict).

### Card 177
**Q:** What's Read Committed isolation?
**A:** See only committed data. Most common default. Prevents dirty reads but allows non-repeatable reads.

### Card 178
**Q:** What's Repeatable Read isolation?
**A:** Consistent snapshot within transaction. Prevents dirty and non-repeatable reads. Allows phantom reads.

### Card 179
**Q:** What's Serializable isolation?
**A:** Strictest level. Transactions appear to execute sequentially. Prevents all anomalies but slowest.

### Card 180
**Q:** What's a dirty read?
**A:** Reading uncommitted changes from another transaction. Prevented by Read Committed and higher.

### Card 181
**Q:** What's a phantom read?
**A:** Different rows returned in repeated read due to inserts/deletes by other transactions. Prevented by Serializable.

### Card 182
**Q:** What's MVCC?
**A:** Multi-Version Concurrency Control. Allows readers and writers to not block each other. PostgreSQL uses this.

### Card 183
**Q:** What's a deadlock?
**A:** Two transactions waiting for each other's locks. Database detects and aborts one transaction.

### Card 184
**Q:** How do you prevent deadlocks?
**A:** 
- Access resources in consistent order
- Keep transactions short
- Use appropriate isolation level
- Implement retry logic

### Card 185
**Q:** What's row-level locking?
**A:** Locking specific rows, not entire table. Allows better concurrency.

### Card 186
**Q:** What's table-level locking?
**A:** Locking entire table. Simpler but reduces concurrency.

### Card 187
**Q:** What's optimistic locking?
**A:** Assume no conflicts, check version at commit. Good for low-contention scenarios.

### Card 188
**Q:** What's pessimistic locking?
**A:** Lock resources immediately (SELECT FOR UPDATE). Good for high-contention scenarios.

### Card 189
**Q:** What's SELECT FOR UPDATE?
**A:** Locks selected rows for update within transaction. Prevents other transactions from modifying them.

### Card 190
**Q:** What's a stored procedure?
**A:** Precompiled SQL code stored in database. Can contain logic (IF, WHILE), parameters.

### Card 191
**Q:** What's a trigger?
**A:** Automatic SQL code execution when event occurs (INSERT, UPDATE, DELETE).

### Card 192
**Q:** When should you use triggers?
**A:** Audit logging, denormalization maintenance, enforcing complex constraints. Use sparingly - can hide logic.

### Card 193
**Q:** What's a schema in PostgreSQL?
**A:** Namespace within database. Organize tables into logical groups.

### Card 194
**Q:** What's referential integrity?
**A:** Foreign keys enforce relationships - can't have orphaned records. Maintains data consistency.

### Card 195
**Q:** What's CASCADE in foreign keys?
**A:** Automatically propagate deletes/updates. `ON DELETE CASCADE` deletes child rows when parent deleted.

### Card 196
**Q:** What's the difference between CHECK and FOREIGN KEY constraints?
**A:**
- **CHECK:** Validates column values against condition
- **FOREIGN KEY:** Enforces relationship between tables

### Card 197
**Q:** What's database replication?
**A:** Copying data to multiple servers. Master-slave or master-master. For high availability and read scaling.

### Card 198
**Q:** What's read replica?
**A:** Copy of database that handles read-only queries. Reduces load on primary database.

### Card 199
**Q:** What's sharding?
**A:** Horizontal partitioning across multiple servers. Different rows on different servers.

### Card 200
**Q:** What's the CAP theorem?
**A:** Can have only 2 of 3: Consistency, Availability, Partition tolerance. Relevant for distributed databases.

---

## Study Tips for Flashcards

### Daily Routine
- **Morning:** Review 20 cards from different sections
- **Evening:** Review cards you got wrong

### Spaced Repetition Schedule
- **Day 1:** Learn cards 1-50
- **Day 2:** Review 1-50, learn 51-100
- **Day 3:** Review all, focus on weak areas
- **Day 4:** Learn 101-150
- **Day 5:** Review 101-150, revisit Day 1 cards
- **Week 2:** Full review, focus on difficult cards
- **Before interview:** Speed review all 200

### Active Learning Techniques
1. **Explain Out Loud:** Don't just read, speak the answer
2. **Write Examples:** For each concept, write a query
3. **Connect Concepts:** Link related flashcards
4. **Test Yourself:** Cover answer before looking
5. **Teach Someone:** Best way to solidify knowledge

### Difficulty Tracking
Mark each card:
- ⭐ **Easy:** Know it cold
- ⚠️ **Medium:** Got it but not instant
- ❌ **Hard:** Struggled or wrong

Focus extra time on ⚠️ and ❌ cards.

---

## Digital Flashcard Options

If you prefer digital flashcards:

1. **Anki** (Desktop/Mobile)
   - Spaced repetition algorithm
   - Free and open-source
   - Import these as deck

2. **Quizlet** (Web/Mobile)
   - Easy to use
   - Study modes: flashcards, tests, games
   - Can share decks

3. **RemNote** (Web/Mobile)
   - Integrated note-taking and spaced repetition
   - Great for connecting concepts

**Create your own deck:**
- Copy question-answer pairs
- Import to your chosen app
- Study using their algorithms

---

**Mastery takes repetition. Review these cards daily for 2-3 weeks before your interview, and you'll have SQL concepts solidly internalized. Good luck!** 🚀
