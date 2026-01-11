---
title: "SQL Mastery Part 2: Database Internals & System Architecture"
date: 2024-02-03 00:00:00 +0530
categories: [SQL, SQL Mastery]
tags: [SQL, Database, PostgreSQL, MySQL, Performance, Optimization, Internals, Transactions, Storage-engine, Buffer-pool, MVCC, WAL]
---

# Complete SQL Mastery Part 2: Database Internals & System Architecture

## Introduction

Understanding how databases work internally is what separates developers who write SQL from database engineers who architect high-performance systems. In Part 1, we covered SQL fundamentals and command categories (DDL, DML, DQL, DCL, TCL). Now we dive deep into the internals—how MySQL and PostgreSQL actually store data, execute queries, manage transactions, and ensure data integrity.

This knowledge is crucial for:
* **Query optimization**: Understanding execution helps you write faster queries
* **Troubleshooting**: Diagnosing performance issues requires knowing what's happening under the hood
* **Capacity planning**: Memory and disk requirements depend on internal structures
* **Interview success**: Senior positions require deep technical knowledge
* **Architecture decisions**: Choosing between MySQL and PostgreSQL based on their implementations

In this part, we'll explore storage engines, memory management, query execution pipelines, optimizer internals, index structures, transaction management (MVCC), locking mechanisms, and write-ahead logging. By the end, you'll understand not just *what* databases do, but *how* they do it.

---

## 2.1 Storage Engine Architecture

Storage engines are the components responsible for storing, retrieving, and managing data on disk. Understanding storage architecture is fundamental to database performance tuning.

### MySQL InnoDB Internals

InnoDB is the default storage engine for MySQL (since version 5.5). It provides ACID-compliant transactions, foreign key support, and crash recovery.

#### InnoDB Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                  MySQL Server Layer                 │
│  (Query Parser, Optimizer, Executor)                │
└─────────────────────┬───────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────┐
│                  InnoDB Storage Engine               │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │         In-Memory Structures               │    │
│  │  ┌──────────────┐  ┌──────────────────┐   │    │
│  │  │ Buffer Pool  │  │  Change Buffer   │   │    │
│  │  │  (Data &     │  │  (Insert Buffer) │   │    │
│  │  │   Indexes)   │  │                  │   │    │
│  │  └──────────────┘  └──────────────────┘   │    │
│  │  ┌──────────────┐  ┌──────────────────┐   │    │
│  │  │ Adaptive Hash│  │   Log Buffer     │   │    │
│  │  │    Index     │  │                  │   │    │
│  │  └──────────────┘  └──────────────────┘   │    │
│  └────────────────────────────────────────────┘    │
│                                                      │
│  ┌────────────────────────────────────────────┐    │
│  │         On-Disk Structures                 │    │
│  │  ┌──────────────┐  ┌──────────────────┐   │    │
│  │  │  Tablespace  │  │  Redo Log Files  │   │    │
│  │  │  (ibdata,    │  │  (ib_logfile0,   │   │    │
│  │  │   .ibd files)│  │   ib_logfile1)   │   │    │
│  │  └──────────────┘  └──────────────────┘   │    │
│  │  ┌──────────────┐  ┌──────────────────┐   │    │
│  │  │  Undo Logs   │  │ Doublewrite      │   │    │
│  │  │  (Rollback   │  │ Buffer           │   │    │
│  │  │   Segments)  │  │                  │   │    │
│  │  └──────────────┘  └──────────────────┘   │    │
│  └────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────┘
```

#### Tablespace Structure

InnoDB stores data in tablespaces, which are logical storage units that contain tables and indexes.

**System Tablespace (ibdata1):**

```sql
-- Default system tablespace
-- Contains:
-- - Data dictionary
-- - Undo logs
-- - Change buffer
-- - Doublewrite buffer
-- - (Optionally) Table data

-- Configuration
[mysqld]
innodb_data_file_path = ibdata1:12M:autoextend
```

**File-Per-Table Tablespace (.ibd files):**

```sql
-- Each table stored in separate .ibd file (default since MySQL 5.6)
-- Advantages:
-- - Easier backup of individual tables
-- - TRUNCATE reclaims disk space immediately
-- - Better performance for some operations

-- Enable (default)
SET GLOBAL innodb_file_per_table = 1;

-- Create table (creates employees.ibd)
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100)
) ENGINE=InnoDB;

-- File location: database_name/employees.ibd
```

**General Tablespace (MySQL 5.7+):**

```sql
-- Create custom tablespace
CREATE TABLESPACE ts1 
ADD DATAFILE 'ts1.ibd' 
FILE_BLOCK_SIZE = 16384;

-- Create table in specific tablespace
CREATE TABLE t1 (id INT) TABLESPACE ts1;

-- Advantages:
-- - Group related tables
-- - Control storage location
-- - Share space between tables
```

#### Page Organization (16KB Pages)

InnoDB organizes data in fixed-size pages (default 16KB):

```
┌─────────────────────────────────────────────────┐
│              InnoDB Page (16KB)                 │
├─────────────────────────────────────────────────┤
│  File Header (38 bytes)                         │
│  - Page checksum                                │
│  - Page number                                  │
│  - Previous/Next page pointers                  │
│  - LSN (Log Sequence Number)                    │
├─────────────────────────────────────────────────┤
│  Page Header (56 bytes)                         │
│  - Number of records                            │
│  - Free space pointer                           │
│  - Last insert position                         │
├─────────────────────────────────────────────────┤
│  Infimum + Supremum Records (26 bytes)          │
│  - Virtual min/max records                      │
├─────────────────────────────────────────────────┤
│  User Records (Variable)                        │
│  ┌──────────────────────────────────┐          │
│  │ Record Header                     │          │
│  │ - Transaction ID (DB_TRX_ID)      │          │
│  │ - Rollback pointer (DB_ROLL_PTR)  │          │
│  │ - Row ID (DB_ROW_ID if no PK)     │          │
│  ├──────────────────────────────────┤          │
│  │ Column Data                        │          │
│  │ - Fixed-length columns             │          │
│  │ - Variable-length columns          │          │
│  └──────────────────────────────────┘          │
├─────────────────────────────────────────────────┤
│  Free Space (Variable)                          │
├─────────────────────────────────────────────────┤
│  Page Directory (Variable)                      │
│  - Slots pointing to records for quick search   │
├─────────────────────────────────────────────────┤
│  File Trailer (8 bytes)                         │
│  - Checksum (validates page integrity)          │
└─────────────────────────────────────────────────┘
```

**Key Page Characteristics:**

* **Fixed size**: Always 16KB (configurable via innodb_page_size: 4KB, 8KB, 16KB, 32KB, 64KB)
* **Linked list**: Pages are linked via previous/next pointers
* **Fill factor**: Typically 15/16 full to leave room for updates
* **Page splits**: When a page is full, InnoDB splits it into two pages

```sql
-- View page size
SHOW VARIABLES LIKE 'innodb_page_size';

-- Monitor page splits (indicates index fragmentation)
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_pages_split';
```

#### Clustered Index Structure

InnoDB uses a **clustered index** organization where table data is stored in the primary key index itself.

```
Primary Key (Clustered) Index:
┌─────────────────────────────────────────┐
│         Root Page                       │
│  [10] → [20] → [30] → [40]              │
└────┬─────────┬─────────┬─────────┬──────┘
     │         │         │         │
     ▼         ▼         ▼         ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ Leaf   │ │ Leaf   │ │ Leaf   │ │ Leaf   │
│ Page 1 │ │ Page 2 │ │ Page 3 │ │ Page 4 │
│────────│ │────────│ │────────│ │────────│
│ PK=1   │ │ PK=11  │ │ PK=21  │ │ PK=31  │
│ Data   │ │ Data   │ │ Data   │ │ Data   │
│────────│ │────────│ │────────│ │────────│
│ PK=2   │ │ PK=12  │ │ PK=22  │ │ PK=32  │
│ Data   │ │ Data   │ │ Data   │ │ Data   │
│────────│ │────────│ │────────│ │────────│
│ ...    │ │ ...    │ │ ...    │ │ ...    │
└────────┘ └────────┘ └────────┘ └────────┘
```

**Characteristics:**

* **Data stored in leaf pages**: Actual row data is in the B-tree leaf pages
* **Primary key order**: Rows are physically sorted by primary key
* **No separate data file**: The index *is* the table
* **Sequential primary keys are optimal**: Auto-increment IDs minimize page splits

**Implications:**

```sql
-- ✅ GOOD: Auto-increment primary key
CREATE TABLE orders (
    order_id INT AUTO_INCREMENT PRIMARY KEY,  -- Sequential
    customer_id INT,
    order_date DATE
);

-- ❌ BAD: UUID/GUID primary key (random order)
CREATE TABLE orders (
    order_id CHAR(36) PRIMARY KEY,  -- Random UUID
    customer_id INT,
    order_date DATE
);
-- Causes:
-- - Random page access (poor cache locality)
-- - Frequent page splits
-- - Index fragmentation
-- - Slower inserts
```

#### Secondary Indexes

Secondary indexes in InnoDB contain primary key values as pointers to data:

```
Secondary Index (on email):
┌─────────────────────────────────────────┐
│         Root Page                       │
│  [aaa@] → [mmm@] → [zzz@]               │
└────┬─────────────┬─────────────┬────────┘
     │             │             │
     ▼             ▼             ▼
┌────────────┐ ┌────────────┐ ┌────────────┐
│ Leaf Page  │ │ Leaf Page  │ │ Leaf Page  │
│────────────│ │────────────│ │────────────│
│ alice@x.com│ │ mike@x.com │ │ zoe@x.com  │
│ → PK: 5    │ │ → PK: 23   │ │ → PK: 89   │
│────────────│ │────────────│ │────────────│
│ bob@x.com  │ │ nancy@x.com│ │ zack@x.com │
│ → PK: 12   │ │ → PK: 45   │ │ → PK: 67   │
└────────────┘ └────────────┘ └────────────┘
```

**Two-step lookup for secondary indexes:**

```sql
-- Query using secondary index
SELECT * FROM users WHERE email = 'alice@x.com';

-- Execution:
-- 1. Secondary index lookup: email → finds PK = 5
-- 2. Primary key lookup: PK = 5 → retrieves full row data

-- This is why secondary indexes are slower than primary key lookups
```

**Covering indexes** avoid the second lookup:

```sql
-- Non-covering index (requires PK lookup)
CREATE INDEX idx_email ON users(email);
SELECT * FROM users WHERE email = 'alice@x.com';  -- Needs PK lookup

-- Covering index (all needed columns in index)
CREATE INDEX idx_email_name ON users(email, first_name, last_name);
SELECT email, first_name, last_name 
FROM users 
WHERE email = 'alice@x.com';  -- No PK lookup needed!

-- Verify covering index
EXPLAIN SELECT email, first_name, last_name 
FROM users WHERE email = 'alice@x.com';
-- Look for: Extra: Using index
```

#### Change Buffer

The change buffer optimizes secondary index updates for non-unique secondary indexes:

```sql
-- Without change buffer:
INSERT INTO users (email, name) VALUES ('new@x.com', 'New User');
-- Must:
-- 1. Read secondary index page for email
-- 2. Update secondary index page
-- 3. Write back to disk
-- Problem: Random I/O for each insert!

-- With change buffer:
-- 1. Buffer the change in memory
-- 2. Apply later when page is read or during merge
-- Benefit: Reduces random I/O, batches updates

-- Configuration
SHOW VARIABLES LIKE 'innodb_change_buffering';
-- Options: all, none, inserts, deletes, changes, purges

-- Monitor change buffer
SHOW ENGINE INNODB STATUS\G
-- Look for: INSERT BUFFER AND ADAPTIVE HASH INDEX section
```

**When change buffer helps:**
* Large batch inserts
* Non-unique secondary indexes
* Working set doesn't fit in buffer pool

**When it doesn't help:**
* Unique indexes (must check for duplicates immediately)
* Small working sets (pages already in buffer pool)

#### Doublewrite Buffer

Protects against partial page writes during crashes:

```
Write Process:
┌──────────────────────────────────────────────────┐
│ 1. Write pages to doublewrite buffer (sequential)│
├──────────────────────────────────────────────────┤
│ 2. Fsync doublewrite buffer                     │
├──────────────────────────────────────────────────┤
│ 3. Write pages to actual locations (random)     │
├──────────────────────────────────────────────────┤
│ 4. If crash during step 3:                      │
│    Recovery uses doublewrite buffer              │
└──────────────────────────────────────────────────┘
```

```sql
-- Configuration
SHOW VARIABLES LIKE 'innodb_doublewrite';

-- Disable for performance (ONLY on systems with atomic writes)
SET GLOBAL innodb_doublewrite = 0;

-- Monitor
SHOW GLOBAL STATUS LIKE 'Innodb_dblwr%';
```

#### Adaptive Hash Index

InnoDB automatically builds hash indexes for frequently accessed data:

```sql
-- Automatic optimization
-- InnoDB monitors access patterns
-- Builds in-memory hash index for hot pages
-- Provides O(1) lookup instead of O(log n) B-tree

-- Configuration
SHOW VARIABLES LIKE 'innodb_adaptive_hash_index';

-- Monitor effectiveness
SHOW ENGINE INNODB STATUS\G
-- Look for: hash searches/s, non-hash searches/s
-- Good ratio: high hash searches relative to non-hash

-- Disable if contention issues (rare)
SET GLOBAL innodb_adaptive_hash_index = 0;
```

#### Data Dictionary

InnoDB maintains metadata about tables, columns, and indexes:

```sql
-- MySQL 8.0: Data Dictionary in InnoDB
-- Stored in: mysql.ibd tablespace
-- Benefits:
-- - Atomic DDL (transactional)
-- - Better performance
-- - Crash-safe

-- Query data dictionary
SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLES;
SELECT * FROM INFORMATION_SCHEMA.INNODB_COLUMNS;
SELECT * FROM INFORMATION_SCHEMA.INNODB_INDEXES;

-- In older versions, stored in .frm files (non-transactional)
```

### PostgreSQL Storage Internals

PostgreSQL uses a different storage architecture based on heap storage and MVCC tuple versioning.

#### Heap Storage Structure

Unlike InnoDB's clustered index, PostgreSQL uses heap storage:

```
Heap Table (Unordered):
┌─────────────────────────────────────────┐
│         Page 0                          │
│  [Row 1] [Row 5] [Row 3] [Row 7]        │
│  (Insertion order, not sorted)          │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│         Page 1                          │
│  [Row 2] [Row 6] [Row 4] [Row 8]        │
└─────────────────────────────────────────┘

Primary Key Index:
┌─────────────────────────────────────────┐
│  PK Value → CTID (Page#, Offset)        │
│  1 → (0,1)  2 → (1,1)  3 → (0,3)        │
└─────────────────────────────────────────┘
```

**Key Differences from InnoDB:**

| Aspect | InnoDB | PostgreSQL |
|--------|--------|------------|
| Storage | Clustered index (PK order) | Heap (insertion order) |
| Primary key lookup | Direct (data in index) | Indirect (index → CTID → data) |
| Sequential scans | Slower (must traverse index) | Faster (sequential read) |
| Primary key range scans | Very fast (data is sorted) | Same as any index scan |
| Index entries | Contain PK value | Contain CTID (physical location) |

#### TOAST (The Oversized-Attribute Storage Technique)

PostgreSQL stores large values externally:

```sql
-- Automatic for values > ~2KB
-- Column types affected: TEXT, BYTEA, VARCHAR

-- TOAST strategies:
-- - PLAIN: No TOAST (for small types like INT)
-- - EXTENDED: Compress, then move to TOAST table if still large
-- - EXTERNAL: Move to TOAST table (no compression)
-- - MAIN: Compress but keep inline if possible

-- Example table with TOAST
CREATE TABLE documents (
    doc_id INT PRIMARY KEY,
    title VARCHAR(200),
    content TEXT  -- TOASTable
);

-- View TOAST tables
SELECT 
    relname,
    reltoastrelid,
    (SELECT relname FROM pg_class WHERE oid = c.reltoastrelid) as toast_table
FROM pg_class c
WHERE relname = 'documents';

-- TOAST table naming: pg_toast.pg_toast_<oid>
```

**TOAST Performance Implications:**

```sql
-- ❌ BAD: Fetching TOASTed columns you don't need
SELECT * FROM documents WHERE doc_id = 123;
-- Fetches entire content column (slow)

-- ✅ GOOD: Only select needed columns
SELECT doc_id, title FROM documents WHERE doc_id = 123;
-- Doesn't fetch content (fast)

-- ✅ GOOD: Use TOAST compression
ALTER TABLE documents ALTER COLUMN content SET STORAGE EXTENDED;
```

#### Tuple Structure

PostgreSQL rows (tuples) include MVCC metadata:

```
Tuple Structure:
┌─────────────────────────────────────────────┐
│ Tuple Header (23 bytes)                     │
│ ┌─────────────────────────────────────────┐ │
│ │ t_xmin (4 bytes)  - Creating transaction│ │
│ │ t_xmax (4 bytes)  - Deleting transaction│ │
│ │ t_cid (4 bytes)   - Command ID          │ │
│ │ t_ctid (6 bytes)  - Tuple location      │ │
│ │ t_infomask (2 bytes) - Flags            │ │
│ │ t_hoff (1 byte)   - Header offset       │ │
│ │ t_bits (variable) - NULL bitmap         │ │
│ └─────────────────────────────────────────┘ │
├─────────────────────────────────────────────┤
│ Column Data (Variable)                      │
│ ┌─────────────────────────────────────────┐ │
│ │ Fixed-length columns (aligned)          │ │
│ │ Variable-length columns (4-byte length) │ │
│ └─────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
```

**Key Fields:**

* **t_xmin**: Transaction ID that inserted this tuple
* **t_xmax**: Transaction ID that deleted this tuple (0 if still visible)
* **t_ctid**: Physical location (page number, offset)
* **t_infomask**: Bit flags (HEAP_XMIN_COMMITTED, HEAP_XMAX_INVALID, etc.)

```sql
-- View tuple information (requires pageinspect extension)
CREATE EXTENSION pageinspect;

-- Inspect page
SELECT * FROM heap_page_items(get_raw_page('employees', 0));

-- Columns:
-- lp: Line pointer (tuple slot)
-- t_xmin: Creating transaction
-- t_xmax: Deleting transaction
-- t_ctid: Tuple ID
```

#### Page Layout (8KB Pages)

PostgreSQL uses 8KB pages by default:

```
┌─────────────────────────────────────────────────┐
│         PostgreSQL Page (8KB)                   │
├─────────────────────────────────────────────────┤
│  Page Header (24 bytes)                         │
│  - Page LSN                                     │
│  - Checksum                                     │
│  - Lower/Upper free space pointers              │
├─────────────────────────────────────────────────┤
│  Line Pointer Array (4 bytes each)              │
│  - Offset to actual tuple                       │
│  - Length of tuple                              │
├─────────────────────────────────────────────────┤
│  Free Space (Variable)                          │
├─────────────────────────────────────────────────┤
│  Tuples (Variable, grows from bottom up)        │
│  ┌──────────────────────────────────┐          │
│  │ Tuple 1                           │          │
│  ├──────────────────────────────────┤          │
│  │ Tuple 2                           │          │
│  ├──────────────────────────────────┤          │
│  │ ...                               │          │
│  └──────────────────────────────────┘          │
├─────────────────────────────────────────────────┤
│  Special Space (Index-specific, if index page)  │
└─────────────────────────────────────────────────┘
```

```sql
-- View page size
SHOW block_size;  -- Returns 8192 (8KB)

-- Page size is compile-time option (rarely changed)
-- Typical: 8KB
-- Some use 32KB for data warehouse workloads
```

#### CTID (Tuple Identifier)

Every tuple has a physical identifier:

```sql
-- CTID format: (page_number, tuple_index)
SELECT ctid, * FROM employees LIMIT 5;
-- ctid    | emp_id | name
-- --------+--------+------
-- (0,1)   | 1      | John
-- (0,2)   | 2      | Jane
-- (1,1)   | 3      | Bob
-- (1,2)   | 4      | Alice

-- Direct tuple access (very fast)
SELECT * FROM employees WHERE ctid = '(0,1)';

-- ⚠️ CTID changes during VACUUM FULL
-- Don't store CTID as a reference!
```

**Use cases:**
* Debugging (identify specific tuple versions)
* Understanding table bloat
* Temporary deduplication

#### System Catalogs

PostgreSQL metadata stored in system tables:

```sql
-- Main catalogs
SELECT * FROM pg_class;       -- Tables, indexes, sequences
SELECT * FROM pg_attribute;   -- Columns
SELECT * FROM pg_index;       -- Index definitions
SELECT * FROM pg_constraint;  -- Constraints
SELECT * FROM pg_type;        -- Data types
SELECT * FROM pg_proc;        -- Functions and procedures

-- Useful queries
-- Table sizes
SELECT 
    relname,
    pg_size_pretty(pg_total_relation_size(oid)) AS size
FROM pg_class
WHERE relkind = 'r'
ORDER BY pg_total_relation_size(oid) DESC
LIMIT 10;

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
```

#### Tablespaces and Schemas

PostgreSQL organizes data into logical structures:

```sql
-- Tablespace: Physical storage location
CREATE TABLESPACE fast_storage 
LOCATION '/mnt/ssd/pgdata';

CREATE TABLE hot_data (id INT) TABLESPACE fast_storage;

-- Schema: Logical namespace within database
CREATE SCHEMA analytics;
CREATE TABLE analytics.metrics (id INT);

-- Default search path
SHOW search_path;  -- "$user", public
SET search_path TO analytics, public;
```

#### Heap-Only Tuples (HOT)

Optimization for UPDATE operations:

```sql
-- Regular UPDATE without HOT:
-- 1. Mark old tuple deleted (set t_xmax)
-- 2. Insert new tuple version
-- 3. Update all indexes to point to new tuple
-- Problem: Index updates are expensive!

-- HOT UPDATE (when possible):
-- 1. Mark old tuple deleted
-- 2. Insert new tuple in same page
-- 3. Link old → new tuple
-- 4. NO INDEX UPDATES!

-- Conditions for HOT:
-- - Updated columns not indexed
-- - New tuple fits in same page
-- - Page has free space

-- Example where HOT works
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    email VARCHAR(100),
    last_login TIMESTAMP,  -- Not indexed
    login_count INT         -- Not indexed
);

-- HOT update (updates non-indexed columns)
UPDATE users 
SET last_login = NOW(), login_count = login_count + 1
WHERE user_id = 123;

-- Check HOT effectiveness
SELECT 
    relname,
    n_tup_upd,
    n_tup_hot_upd,
    n_tup_hot_upd::FLOAT / NULLIF(n_tup_upd, 0) AS hot_ratio
FROM pg_stat_user_tables
WHERE relname = 'users';

-- hot_ratio close to 1.0 is excellent
```

### Clustered vs Heap Storage Comparison

| Aspect | InnoDB (Clustered) | PostgreSQL (Heap) |
|--------|-------------------|-------------------|
| **Data Organization** | Sorted by primary key | Insertion order |
| **Primary Key Lookups** | Very fast (direct) | Fast (index → CTID) |
| **Range Scans on PK** | Very fast (sequential) | Slower (random access) |
| **Secondary Index Lookups** | Slower (PK lookup needed) | Same as PK (CTID lookup) |
| **Table Scans** | Slower (traverse index) | Faster (sequential) |
| **Inserts** | Faster with sequential PK | Fast anywhere |
| **Updates** | Can cause page splits | Creates new tuple |
| **Space Efficiency** | Better (data in index) | Worse (separate indexes) |
| **Index Size** | Smaller (PK values) | Larger (CTID values) |
| **Best For** | OLTP, PK-heavy queries | Mixed workloads, table scans |

### Page Size Differences

| Database | Default Page Size | Typical Range | Impact |
|----------|------------------|---------------|---------|
| MySQL InnoDB | 16KB | 4KB - 64KB | Larger pages → better for scans, worse for random access |
| PostgreSQL | 8KB | 8KB (rarely changed) | Smaller pages → better for random access, more pages |

### Index Organization Differences

**InnoDB:**
```
Primary Key → Data stored in index leaves
Secondary Index → Stores PK values → Lookup PK index for data
```

**PostgreSQL:**
```
All Indexes → Store CTID → Lookup heap page for data
```

**Implication:**

```sql
-- InnoDB: Covering index avoids extra lookup
CREATE INDEX idx_email_name ON users(email, first_name, last_name);
SELECT email, first_name, last_name FROM users WHERE email = 'x@y.com';
-- Uses only idx_email_name, no PK lookup needed

-- PostgreSQL: Covering index still needs heap lookup for visibility check
CREATE INDEX idx_email_name ON users(email, first_name, last_name);
SELECT email, first_name, last_name FROM users WHERE email = 'x@y.com';
-- Uses idx_email_name, but still checks heap for tuple visibility (MVCC)

-- PostgreSQL 11+: Index-only scans with visibility map
-- Avoids heap lookup if visibility map indicates all tuples visible
```

---

## Frequently Asked Questions

**Q1: Why does InnoDB use 16KB pages while PostgreSQL uses 8KB?**

**A:** The page size choice reflects different design philosophies:

**InnoDB (16KB):**
* Optimized for sequential I/O patterns (OLTP with clustered indexes)
* Larger pages reduce number of page splits
* Better for range scans on primary key
* Works well with SSD's larger write units

**PostgreSQL (8KB):**
* Better for random access patterns
* Smaller pages reduce write amplification
* More granular locking possible
* Better cache hit ratio for random access

Neither is inherently better—it depends on workload. PostgreSQL can be recompiled with different page sizes (rare), while InnoDB supports 4KB-64KB pages via configuration.

**Related Concepts**: Buffer pool efficiency, I/O patterns, SSD optimization

---

**Q2: What happens to secondary indexes when InnoDB reorganizes a table?**

**A:** When you run operations that reorganize the clustered index:

```sql
-- Operations that rebuild table:
ALTER TABLE employees ENGINE=InnoDB;  -- Full rebuild
OPTIMIZE TABLE employees;             -- Rebuilds if fragmented
ALTER TABLE employees ADD COLUMN ...  -- May rebuild (depends on algorithm)

-- What happens:
-- 1. New clustered index is built with new structure
-- 2. All secondary indexes must be rebuilt (they store PK values)
-- 3. This can take a long time on large tables!

-- Online DDL helps:
ALTER TABLE employees ADD COLUMN notes TEXT, ALGORITHM=INPLACE, LOCK=NONE;
```

In PostgreSQL, there's no clustered index, so adding columns doesn't require rebuilding indexes (usually).

---

**Q3: How does TOAST affect query performance?**

**A:** TOAST can significantly impact performance:

```sql
-- ❌ SLOW: Query fetches TOASTed column
SELECT * FROM documents WHERE doc_id < 1000;
-- Must:
-- 1. Read main table pages
-- 2. Read TOAST table pages for content column
-- 3. Decompress (if EXTENDED strategy)

-- ✅ FAST: Query avoids TOAST column
SELECT doc_id, title FROM documents WHERE doc_id < 1000;
-- Only reads main table pages

-- Monitor TOAST impact
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM documents LIMIT 100;
-- Look for: Buffers: shared hit/read (high numbers indicate TOAST access)
```

**Best practices:**
* Select only needed columns
* Consider splitting large columns into separate tables
* Use EXTERNAL storage for frequently accessed large values (avoids compression overhead)
* Monitor table and TOAST table sizes separately

---

## Interview Questions

**Question 1: Explain why a UUID primary key causes performance issues in InnoDB but not as much in PostgreSQL.**

**Difficulty:** Senior

**Answer:**

**InnoDB (Clustered Index):**

```sql
-- UUID primary key
CREATE TABLE orders (
    order_id CHAR(36) PRIMARY KEY,  -- Random UUID
    customer_id INT,
    total DECIMAL(10,2)
) ENGINE=InnoDB;

INSERT INTO orders VALUES 
    (UUID(), 123, 99.99);  -- Random UUID
```

**Problems:**
1. **Random insertion position**: UUIDs are random, so inserts happen anywhere in the B-tree
2. **Page splits**: Frequent splits because new rows don't go at the end
3. **Poor cache locality**: Random access pattern means buffer pool thrashing
4. **Index fragmentation**: Pages only 50-70% full after splits
5. **Secondary index overhead**: All secondary indexes store the full 36-byte UUID as pointer

**Performance impact:**
* 2-3x slower inserts
* 40-50% more storage (fragmentation + UUID size in indexes)
* Worse cache hit ratio

**PostgreSQL (Heap Storage):**

```sql
CREATE TABLE orders (
    order_id UUID PRIMARY KEY,  -- Random UUID
    customer_id INT,
    total NUMERIC(10,2)
);
```

**Why it's less problematic:**
1. **Heap insert**: New tuples append to heap (no specific order needed)
2. **Index impact only**: Only the index experiences fragmentation, not the data
3. **CTID references**: Secondary indexes store small CTID (6 bytes) not UUID
4. **Separate data and index**: Index fragmentation doesn't affect heap storage

**Still has issues but less severe:**
* Index fragmentation (but can be rebuilt independently)
* Random B-tree access for index inserts

**Best Solution for Both:**

```sql
-- Sequential UUID (version 1 or custom)
-- MySQL
CREATE TABLE orders (
    order_id BINARY(16) PRIMARY KEY,  -- Use UUID_TO_BIN with swap_flag
    ...
);

INSERT INTO orders 
VALUES (UUID_TO_BIN(UUID(), 1), ...);  -- swap_flag=1 for sequential

-- PostgreSQL
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE orders (
    order_id UUID PRIMARY KEY DEFAULT uuid_generate_v1mc(),  -- Sequential
    ...
);

-- Or use BIGINT with Snowflake ID / Twitter Snowflake
```

**Why This Matters:** Understanding storage internals helps you make informed primary key choices. Sequential IDs are nearly always better for InnoDB, while PostgreSQL is more forgiving but still benefits from sequential ordering.

**Follow-up Questions:**
* How would you migrate from UUID to sequential IDs in a production system?
* What are the trade-offs of using composite primary keys?

---

**Question 2: A table in PostgreSQL is 10GB but `pg_relation_size()` shows 50GB. What's happening and how do you fix it?**

**Difficulty:** Senior

**Answer:**

**Problem Diagnosis:**

```sql
-- Check table and index sizes
SELECT 
    pg_size_pretty(pg_relation_size('users')) AS table_size,
    pg_size_pretty(pg_total_relation_size('users')) AS total_size,
    pg_size_pretty(pg_total_relation_size('users') - pg_relation_size('users')) AS index_size;

-- Check for bloat
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
    n_dead_tup,
    n_live_tup,
    round(n_dead_tup::numeric / NULLIF(n_live_tup + n_dead_tup, 0) * 100, 2) AS dead_ratio
FROM pg_stat_user_tables
WHERE tablename = 'users';
```

**Cause: Table Bloat**

PostgreSQL MVCC creates new tuple versions for UPDATEs and marks old versions as dead. These dead tuples occupy space until VACUUM reclaims them.

**Reasons for excessive bloat:**
1. **Insufficient autovacuum**: Not running frequently enough
2. **High UPDATE rate**: Faster than VACUUM can clean
3. **Long-running transactions**: Prevent VACUUM from removing old tuples
4. **Disabled autovacuum**: Manually disabled (bad idea)

**Solutions:**

**1. Manual VACUUM (immediate but blocks some operations):**
```sql
-- Regular VACUUM (removes dead tuples, updates statistics)
VACUUM VERBOSE users;

-- Aggressive VACUUM FULL (rewrites entire table, requires exclusive lock)
-- ⚠️ WARNING: Locks table for duration
VACUUM FULL users;

-- Analyze table stats
ANALYZE users;

-- Or combined
VACUUM (FULL, ANALYZE, VERBOSE) users;
```

**2. Tune Autovacuum (long-term fix):**
```sql
-- Check autovacuum settings
SHOW autovacuum;
SHOW autovacuum_vacuum_scale_factor;
SHOW autovacuum_vacuum_threshold;

-- Table-specific tuning (for high-churn tables)
ALTER TABLE users SET (
    autovacuum_vacuum_scale_factor = 0.05,  -- Vacuum at 5% dead tuples
    autovacuum_vacuum_threshold = 1000,
    autovacuum_vacuum_cost_delay = 10
);

-- Global tuning (postgresql.conf)
autovacuum_vacuum_scale_factor = 0.1
autovacuum_vacuum_threshold = 50
autovacuum_naptime = 30s
autovacuum_max_workers = 4
```

**3. Prevent Long Transactions:**
```sql
-- Find long-running transactions
SELECT 
    pid,
    now() - xact_start AS duration,
    state,
    query
FROM pg_stat_activity
WHERE xact_start IS NOT NULL
ORDER BY duration DESC;

-- Kill blocking transaction (if appropriate)
SELECT pg_terminate_backend(pid);
```

**4. pg_repack (online table rebuild):**
```sql
-- Install extension
CREATE EXTENSION pg_repack;

-- Repack table (no exclusive lock, runs online)
pg_repack -t users database_name

-- Advantages over VACUUM FULL:
-- - No exclusive lock on table
-- - Can cancel mid-operation
-- - More efficient space reclamation
```

**5. Partitioning (preventative):**
```sql
-- For time-series data, drop old partitions instead of DELETE
CREATE TABLE orders (
    order_id BIGINT,
    order_date DATE,
    ...
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2024_01 PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- Drop old partitions (instant, no bloat)
DROP TABLE orders_2023_12;
```

**Why This Matters:** Table bloat is one of the most common performance problems in PostgreSQL. Understanding MVCC and VACUUM is essential for maintaining production databases.

**Follow-up Questions:**
* How does HOT (Heap-Only Tuples) help reduce bloat?
* When would you choose pg_repack over VACUUM FULL?
* How do you monitor bloat proactively?

---

## Key Takeaways

* **InnoDB uses clustered indexes** where table data is stored in primary key order, making PK range scans very fast
* **PostgreSQL uses heap storage** with unordered data and separate indexes, better for table scans and mixed workloads
* **Page size matters**: InnoDB's 16KB pages favor sequential I/O; PostgreSQL's 8KB pages favor random access
* **Secondary indexes behave differently**: InnoDB stores PK values; PostgreSQL stores CTID (physical pointers)
* **UUIDs hurt InnoDB more** due to clustered index fragmentation
* **TOAST externalizes large values** in PostgreSQL, affecting query performance if you select TOASTed columns
* **HOT updates** in PostgreSQL avoid index updates when updating non-indexed columns
* **Table bloat** in PostgreSQL requires VACUUM to reclaim space from dead tuples
* **Clustered indexes** require rebuilding all secondary indexes when reorganizing the table
* **Understanding storage internals** is crucial for schema design, query optimization, and troubleshooting

---

## 2.2 Buffer Pool & Memory Management

Memory management is critical for database performance. Understanding buffer pools, caches, and memory structures helps you tune databases for optimal performance.

### InnoDB Buffer Pool

The InnoDB buffer pool is the most important memory structure in MySQL, caching table and index data to minimize disk I/O.

#### Buffer Pool Architecture

```
┌─────────────────────────────────────────────────────┐
│           InnoDB Buffer Pool                        │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │            LRU List                           │ │
│  │  (Least Recently Used - Aging Algorithm)     │ │
│  │                                               │ │
│  │  ┌─────────────────────────────────────┐    │ │
│  │  │     New Sublist (5/8)               │    │ │
│  │  │  (Young pages, recently accessed)   │    │ │
│  │  │  [Page1] [Page2] [Page3] ...        │    │ │
│  │  └─────────────────────────────────────┘    │ │
│  │           ▲ Midpoint (3/8 from tail)         │ │
│  │  ┌─────────────────────────────────────┐    │ │
│  │  │     Old Sublist (3/8)               │    │ │
│  │  │  (Older pages, aging out)           │    │ │
│  │  │  [Page4] [Page5] [Page6] ...        │    │ │
│  │  └─────────────────────────────────────┘    │ │
│  └──────────────────────────────────────────────┘ │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │            Free List                          │ │
│  │  (Available pages for new data)              │ │
│  │  [Free1] [Free2] [Free3] ...                 │ │
│  └──────────────────────────────────────────────┘ │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │            Flush List                         │ │
│  │  (Modified/dirty pages to be written)        │ │
│  │  [Dirty1] [Dirty2] [Dirty3] ...              │ │
│  └──────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

#### LRU Algorithm (Least Recently Used)

InnoDB uses a modified LRU with midpoint insertion to prevent table scans from flushing hot data:

```sql
-- Configuration
SHOW VARIABLES LIKE 'innodb_old_blocks_pct';
-- Default: 37 (37% of buffer pool is "old" sublist)

SHOW VARIABLES LIKE 'innodb_old_blocks_time';
-- Default: 1000 (1 second before page moves to "new" sublist)

-- How it works:
-- 1. New page read → Inserted at midpoint (head of old sublist)
-- 2. If accessed again within 1 second → Stays in old sublist
-- 3. If accessed after 1 second → Moves to new sublist (young)
-- 4. Pages age toward tail in both sublists
-- 5. Pages at tail of old sublist evicted first

-- Why midpoint insertion?
-- Prevents table scans from flushing hot data:
SELECT * FROM huge_table;  -- Full table scan
-- Without midpoint: Would push all hot pages out
-- With midpoint: Scan pages stay in old sublist, evicted quickly
```

**Algorithm in action:**

```sql
-- Hot page (frequently accessed)
-- 1. Read from disk → Inserted at midpoint
-- 2. Accessed again quickly → Moves to new sublist
-- 3. Accessed repeatedly → Stays at head of new sublist
-- 4. Eventually ages but slowly (new sublist is 5/8 of pool)

-- Cold page (scan, rarely accessed)
-- 1. Read from disk → Inserted at midpoint
-- 2. Not accessed again → Ages in old sublist
-- 3. Evicted when space needed

-- Example: Report query scanning millions of rows
SELECT * FROM logs WHERE log_date = '2024-01-01';
-- Reads millions of pages
-- All inserted at midpoint
-- Rarely accessed again
-- Evicted from old sublist
-- Hot data (frequently queried tables) remains in new sublist
```

#### Buffer Pool Sizing

```sql
-- View current size
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

-- Recommended: 60-80% of available RAM (dedicated database server)
-- Example: 64GB RAM → 48-52GB buffer pool

-- Set in my.cnf
[mysqld]
innodb_buffer_pool_size = 48G

-- Or dynamically (MySQL 5.7+)
SET GLOBAL innodb_buffer_pool_size = 48 * 1024 * 1024 * 1024;

-- Resize happens in chunks
SHOW VARIABLES LIKE 'innodb_buffer_pool_chunk_size';
-- Default: 128MB

-- Buffer pool size must be multiple of:
-- innodb_buffer_pool_instances × innodb_buffer_pool_chunk_size

-- Example calculation:
-- Instances: 8
-- Chunk size: 128MB
-- Minimum size: 8 × 128MB = 1GB
-- Valid sizes: 1GB, 2GB, 3GB, ... (multiples of 1GB)
```

#### Multiple Buffer Pool Instances

```sql
-- Reduce contention in multi-core systems
SHOW VARIABLES LIKE 'innodb_buffer_pool_instances';
-- Default: 8 (if buffer pool >= 1GB)

-- Each instance has its own:
-- - LRU list
-- - Free list
-- - Flush list
-- - Mutexes

-- Benefits:
-- - Reduced mutex contention
-- - Better concurrency
-- - Improved performance on multi-core CPUs

-- Configuration
[mysqld]
innodb_buffer_pool_instances = 8
innodb_buffer_pool_size = 48G
-- Each instance: 6GB

-- ⚠️ Note: Deprecated in MySQL 8.0, removed in future versions
-- MySQL 8.0+ automatically manages buffer pool partitioning
```

#### Monitoring Buffer Pool

```sql
-- Buffer pool status
SHOW ENGINE INNODB STATUS\G

-- Key metrics in output:
-- Total memory allocated
-- Free buffers
-- Database pages
-- Old database pages
-- Modified db pages (dirty pages)
-- Pending reads/writes
-- Buffer pool hit rate

-- Buffer pool statistics
SELECT * FROM information_schema.INNODB_BUFFER_POOL_STATS\G

-- Important columns:
-- POOL_SIZE: Total pages in buffer pool
-- FREE_BUFFERS: Available pages
-- DATABASE_PAGES: Pages containing data
-- OLD_DATABASE_PAGES: Pages in old sublist
-- MODIFIED_DATABASE_PAGES: Dirty pages
-- PAGES_MADE_YOUNG: Pages moved to new sublist
-- PAGES_NOT_MADE_YOUNG: Pages that stayed in old sublist

-- Calculate hit ratio
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_read%';

-- Hit ratio formula:
-- (Innodb_buffer_pool_read_requests - Innodb_buffer_pool_reads) 
-- / Innodb_buffer_pool_read_requests

-- Example:
-- read_requests: 1,000,000
-- reads (from disk): 10,000
-- Hit ratio: (1,000,000 - 10,000) / 1,000,000 = 0.99 = 99%

-- Target: > 99% hit ratio (90%+ acceptable for some workloads)
```

**Query for hit ratio:**

```sql
SELECT 
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status 
     WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests') -
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status 
     WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads')
    /
    (SELECT VARIABLE_VALUE FROM performance_schema.global_status 
     WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests')
    AS buffer_pool_hit_ratio;
```

#### Dirty Pages and Flushing

```sql
-- Dirty pages: Modified pages not yet written to disk
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_pages_dirty';

-- Flush behavior
SHOW VARIABLES LIKE 'innodb_max_dirty_pages_pct';
-- Default: 90 (flush aggressively when > 90% pages dirty)

SHOW VARIABLES LIKE 'innodb_max_dirty_pages_pct_lwm';
-- Default: 10 (low water mark, start flushing at 10%)

-- Adaptive flushing
SHOW VARIABLES LIKE 'innodb_adaptive_flushing';
-- Default: ON

SHOW VARIABLES LIKE 'innodb_io_capacity';
-- Default: 200 (I/O operations per second)
-- SSD: 2000-5000
-- NVMe: 10000+

-- Aggressive flushing for SSD
[mysqld]
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000
innodb_flush_neighbors = 0  -- Don't flush neighbors on SSD

-- Monitor flushing
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool_pages_flushed';
```

### PostgreSQL Shared Buffers

PostgreSQL's main memory cache for table and index data.

#### Shared Buffers Architecture

```
┌─────────────────────────────────────────────────────┐
│        PostgreSQL Shared Buffers                   │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │         Buffer Pool                           │ │
│  │  [Buffer 0] [Buffer 1] ... [Buffer N]        │ │
│  │  Each buffer: 8KB page                        │ │
│  └──────────────────────────────────────────────┘ │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │         Buffer Descriptors                    │ │
│  │  Metadata for each buffer:                   │ │
│  │  - Tag (relation, block number)              │ │
│  │  - State (free, pinned, dirty)               │ │
│  │  - Usage count (for eviction)                │ │
│  │  - Flags                                      │ │
│  └──────────────────────────────────────────────┘ │
│                                                     │
│  ┌──────────────────────────────────────────────┐ │
│  │         Buffer Hash Table                     │ │
│  │  Fast lookup: (relfilenode, block) → buffer  │ │
│  └──────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

#### Clock-Sweep Algorithm

PostgreSQL uses clock-sweep (approximation of LRU):

```
-- Each buffer has usage_count (0-5)
-- Clock hand sweeps through buffer pool
-- When buffer needed:
--   1. Sweep clockwise from last position
--   2. If usage_count > 0: Decrement and continue
--   3. If usage_count = 0: Evict and use buffer
--   4. When buffer accessed: Increment usage_count (max 5)

-- Benefits:
-- - O(1) eviction (no list sorting)
-- - Handles sequential scans well
-- - Simple and efficient

-- Example:
-- Buffer accessed → usage_count = 5
-- Not accessed → usage_count decrements with each sweep
-- usage_count reaches 0 → eligible for eviction
```

#### Shared Buffers Sizing

```sql
-- View current size
SHOW shared_buffers;

-- Recommended: 25% of RAM (for dedicated database server)
-- Example: 64GB RAM → 16GB shared buffers

-- PostgreSQL relies more on OS page cache
-- Total cache: shared_buffers + OS page cache

-- Configuration (postgresql.conf)
shared_buffers = 16GB

-- Restart required to change shared_buffers

-- Why only 25% (vs MySQL's 70-80%)?
-- PostgreSQL uses OS page cache effectively
-- Double-buffering (shared_buffers + OS cache) wastes memory
-- Better to allocate more to OS cache for flexibility
```

#### Background Writer Process

```sql
-- Background writer: Writes dirty buffers to disk gradually
-- Prevents checkpoint spikes

-- Configuration
SHOW bgwriter_delay;
-- Default: 200ms (how often bgwriter wakes)

SHOW bgwriter_lru_maxpages;
-- Default: 100 (max pages to write per round)

SHOW bgwriter_lru_multiplier;
-- Default: 2.0 (multiplier for next round)

-- Example:
-- Round 1: Write 100 pages
-- Round 2: Write 100 × 2.0 = 200 pages
-- Round 3: Write 200 × 2.0 = 400 pages (capped at maxpages)

-- Monitor bgwriter
SELECT * FROM pg_stat_bgwriter;

-- Key columns:
-- buffers_clean: Buffers written by bgwriter
-- buffers_backend: Buffers written by backends (should be low)
-- buffers_checkpoint: Buffers written during checkpoints
-- maxwritten_clean: Times bgwriter stopped due to maxpages limit

-- If buffers_backend is high:
-- Increase bgwriter_lru_maxpages or decrease bgwriter_delay
```

#### Checkpointer Process

```sql
-- Checkpointer: Writes all dirty buffers periodically
-- Ensures recovery point for crash recovery

-- Configuration
SHOW checkpoint_timeout;
-- Default: 5min (max time between checkpoints)

SHOW max_wal_size;
-- Default: 1GB (checkpoint when WAL reaches this size)

SHOW checkpoint_completion_target;
-- Default: 0.9 (spread checkpoint over 90% of checkpoint_timeout)

-- Example:
-- checkpoint_timeout: 5min
-- checkpoint_completion_target: 0.9
-- Checkpoint spread over: 5 × 0.9 = 4.5 minutes

-- Monitor checkpoints
SELECT * FROM pg_stat_bgwriter;

-- checkpoint_timed: Checkpoints due to timeout
-- checkpoint_req: Checkpoints due to WAL size (should be low)

-- If checkpoint_req is high:
-- Increase max_wal_size
-- Checkpoints happening too frequently due to WAL volume
```

#### Effective Cache Size

```sql
-- effective_cache_size: Hint to optimizer about available cache
-- Includes shared_buffers + OS page cache

SHOW effective_cache_size;

-- Recommended: 50-75% of total RAM
-- Example: 64GB RAM → 40-48GB effective_cache_size

-- This does NOT allocate memory
-- Only informs optimizer for query planning

-- Configuration
effective_cache_size = 40GB

-- Affects:
-- - Index vs sequential scan decisions
-- - Join algorithm selection
-- - Nested loop vs hash join choice

-- Example impact:
-- With low effective_cache_size:
--   Optimizer assumes data not in cache
--   Prefers sequential scans (fewer random I/Os)

-- With high effective_cache_size:
--   Optimizer assumes data in cache
--   Willing to use index scans (random I/O acceptable)
```

#### Page Cache Interaction

```sql
-- PostgreSQL architecture:
-- Shared Buffers (PostgreSQL) → OS Page Cache → Disk

-- Read path:
-- 1. Check shared_buffers
-- 2. If miss, check OS page cache
-- 3. If miss, read from disk

-- Double-buffering issue:
-- Same page in shared_buffers AND OS page cache
-- Wastes memory

-- That's why shared_buffers is smaller than MySQL buffer pool
-- Let OS page cache handle additional caching

-- Monitor cache hits
SELECT 
    sum(heap_blks_hit) AS heap_hit,
    sum(heap_blks_read) AS heap_read,
    sum(heap_blks_hit) / NULLIF(sum(heap_blks_hit + heap_blks_read), 0) AS ratio
FROM pg_statio_user_tables;

-- Target: > 0.99 (99% hit ratio)

-- Check individual tables
SELECT 
    schemaname,
    relname,
    heap_blks_hit,
    heap_blks_read,
    heap_blks_hit::FLOAT / NULLIF(heap_blks_hit + heap_blks_read, 0) AS cache_hit_ratio
FROM pg_statio_user_tables
ORDER BY heap_blks_read DESC
LIMIT 20;
```

### Memory Structures Comparison

| Memory Structure | MySQL InnoDB | PostgreSQL |
|------------------|--------------|------------|
| **Main cache** | Buffer pool (70-80% RAM) | Shared buffers (25% RAM) |
| **Algorithm** | Modified LRU with midpoint | Clock-sweep |
| **Page size** | 16KB | 8KB |
| **OS cache reliance** | Low | High |
| **Configuration** | innodb_buffer_pool_size | shared_buffers |
| **Dynamic resize** | Yes (5.7+) | No (requires restart) |
| **Hit ratio target** | > 99% | > 99% |

### Sort Buffers and Work Memory

#### MySQL Sort Buffer

```sql
-- Memory for sorting operations
SHOW VARIABLES LIKE 'sort_buffer_size';
-- Default: 256KB
-- Range: 32KB to 4GB

-- Allocated per connection when needed
-- ORDER BY, GROUP BY, CREATE INDEX

-- Too small: Multiple disk merge passes (slow)
-- Too large: Wastes memory, risk of OOM

-- Monitor
SHOW GLOBAL STATUS LIKE 'Sort_merge_passes';
-- High value indicates sort_buffer_size too small

-- Temporary setting for large sort
SET SESSION sort_buffer_size = 2 * 1024 * 1024;  -- 2MB
SELECT * FROM large_table ORDER BY column;
```

#### PostgreSQL Work Memory

```sql
-- Memory for query operations
SHOW work_mem;
-- Default: 4MB

-- Used for:
-- - Sorting (ORDER BY)
-- - Hash tables (Hash joins, Hash aggregates)
-- - Bitmap operations

-- Allocated per operation
-- Complex query might use work_mem multiple times!

-- Example:
SELECT ... 
FROM t1 
JOIN t2 USING (id)    -- Hash join: 1 × work_mem
GROUP BY category     -- Hash aggregate: 1 × work_mem
ORDER BY total DESC;  -- Sort: 1 × work_mem
-- Total: 3 × work_mem per query

-- Monitor
EXPLAIN (ANALYZE, BUFFERS) 
SELECT ... FROM large_table ORDER BY column;
-- Look for: "Sort Method: external merge" (used disk)
-- Want: "Sort Method: quicksort  Memory: ...KB" (in memory)

-- Session setting for large operation
SET work_mem = '256MB';
SELECT ... complex query ...;
RESET work_mem;

-- ⚠️ Caution with global setting:
-- work_mem = 256MB with 100 concurrent queries using 3 operations each
-- Potential memory: 256MB × 3 × 100 = 75GB!
```

#### Join Buffers

**MySQL:**

```sql
-- Join buffer for block nested loop joins
SHOW VARIABLES LIKE 'join_buffer_size';
-- Default: 256KB

-- Used when join doesn't have index
-- Stores rows from driving table

-- Monitor
EXPLAIN SELECT ... FROM t1 JOIN t2 ON t1.id = t2.id;
-- Look for: "Using join buffer (Block Nested Loop)"

-- If many joins use buffer:
SET GLOBAL join_buffer_size = 1 * 1024 * 1024;  -- 1MB
```

**PostgreSQL:**

```sql
-- Controlled by work_mem
-- Used for hash joins

EXPLAIN (ANALYZE, BUFFERS)
SELECT ... FROM t1 JOIN t2 ON t1.id = t2.id;

-- Look for: Hash buckets and batches
-- Hash Join
--   Hash Cond: (t1.id = t2.id)
--   ->  Hash
--         Buckets: 1024  Batches: 1  Memory Usage: 45kB

-- Batches = 1: Good (everything fits in work_mem)
-- Batches > 1: Spilled to disk (increase work_mem)
```

### Temporary Tables

**MySQL:**

```sql
-- Memory for temporary tables
SHOW VARIABLES LIKE 'tmp_table_size';
-- Default: 16MB

SHOW VARIABLES LIKE 'max_heap_table_size';
-- Default: 16MB

-- Temporary table uses lesser of these two

-- Monitor
SHOW GLOBAL STATUS LIKE 'Created_tmp%';
-- Created_tmp_tables: Total temporary tables
-- Created_tmp_disk_tables: Temporary tables on disk

-- High Created_tmp_disk_tables indicates:
-- Increase tmp_table_size and max_heap_table_size

-- ⚠️ Limitations:
-- TEXT, BLOB columns force disk temporary table
```

**PostgreSQL:**

```sql
-- Uses work_mem for temporary tables
-- No separate configuration

-- Monitor with EXPLAIN ANALYZE
-- Look for: "Sort Method: external merge" or disk-based operations
```

### Memory Tuning Guidelines

**MySQL Memory Configuration:**

```sql
-- For dedicated database server (64GB RAM)
[mysqld]
# InnoDB
innodb_buffer_pool_size = 48G          # 75% of RAM
innodb_buffer_pool_instances = 8       # For large pools
innodb_log_file_size = 2G              # 25% of buffer pool

# Per-connection buffers (careful with these)
sort_buffer_size = 256K                # Default usually fine
join_buffer_size = 256K                # Increase if many joins
read_buffer_size = 128K                # For sequential scans
read_rnd_buffer_size = 256K            # For sort operations

# Temporary tables
tmp_table_size = 64M                   # Larger for complex queries
max_heap_table_size = 64M              # Match tmp_table_size

# Total memory estimate:
# Buffer pool: 48GB
# Per-connection (100 connections): 100 × (256K + 256K + 128K + 256K) ≈ 90MB
# Temp tables: Varies
# OS: ~10GB
# Total: ~58-60GB
```

**PostgreSQL Memory Configuration:**

```sql
-- For dedicated database server (64GB RAM)
# postgresql.conf

# Shared memory
shared_buffers = 16GB                  # 25% of RAM
effective_cache_size = 48GB            # 75% of RAM (hint, not allocation)
maintenance_work_mem = 2GB             # For VACUUM, CREATE INDEX

# Per-operation memory
work_mem = 64MB                        # Careful: per operation!
                                       # 64MB × 10 operations × 50 connections = 32GB

# Background processes
wal_buffers = 16MB                     # WAL buffer
autovacuum_work_mem = 2GB              # For autovacuum

# Total memory estimate:
# Shared buffers: 16GB
# Work mem (peak): ~32GB (assuming 50 connections, 10 ops each)
# OS cache: ~10GB
# Total: ~58-60GB
```

---

## Frequently Asked Questions

**Q1: Why does PostgreSQL use less shared_buffers than MySQL uses buffer pool?**

**A:** Different design philosophies:

**MySQL InnoDB (70-80% RAM):**
* Does most caching itself
* Less reliance on OS page cache
* Direct control over caching behavior
* Optimized for its own caching algorithms

**PostgreSQL (25% RAM):**
* Leverages OS page cache heavily
* Avoids double-buffering (same page in shared_buffers + OS cache)
* OS cache handles additional caching
* Remaining RAM used for OS page cache, work_mem, etc.

**Total cache for both:**
* MySQL: ~80% RAM in buffer pool
* PostgreSQL: ~25% in shared_buffers + ~50% in OS cache = ~75% total

**Why PostgreSQL approach:**
* Flexibility: OS can allocate cache dynamically
* Simplicity: Less memory management overhead
* Efficiency: Avoids double-buffering waste

**Related Concepts**: OS page cache, double-buffering

---

**Q2: My buffer pool hit ratio is 85%. Is this bad?**

**A:** It depends on the workload, but generally **yes, this indicates room for improvement**:

**Target hit ratios:**
* **> 99%**: Excellent (most reads from memory)
* **95-99%**: Good (acceptable for some workloads)
* **< 95%**: Poor (too many disk reads)

**85% means:**
* 15 out of every 100 reads go to disk
* Disk I/O is 100-1000x slower than memory
* Significant performance impact

**Possible causes:**

```sql
-- 1. Buffer pool too small
-- Check current size vs working set
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SELECT SUM(data_length + index_length) / 1024 / 1024 / 1024 AS total_gb
FROM information_schema.TABLES;

-- If total data >> buffer pool → Increase buffer pool

-- 2. Inefficient queries
-- Check slow query log
-- Look for full table scans on large tables

-- 3. Working set larger than RAM
-- If data is 500GB but only 64GB RAM:
-- Can't cache everything
-- Focus on caching hot data
-- Add indexes to reduce data accessed

-- 4. Large sequential scans
-- Table scans pollute buffer pool
-- Check for queries scanning millions of rows

-- 5. Low cache reuse
-- Data accessed once then never again
-- Consider query pattern optimization
```

**Actions to improve:**
1. Increase buffer pool size (if RAM available)
2. Add indexes to reduce data scanned
3. Optimize queries to access less data
4. Partition tables to reduce scan sizes
5. Add more RAM if working set is too large

---

**Q3: What happens if I set work_mem too high in PostgreSQL?**

**A:** Risk of **out-of-memory (OOM)** errors:

**The problem:**

```sql
-- Scenario:
SET work_mem = '1GB';  -- Per operation!

-- Complex query with 5 operations:
SELECT ...
FROM t1 
JOIN t2 USING (id)           -- Hash join: 1GB
JOIN t3 USING (category)     -- Hash join: 1GB
GROUP BY customer            -- Hash aggregate: 1GB
HAVING total > 1000
ORDER BY total DESC          -- Sort: 1GB
LIMIT 100;

-- Single query uses: 4GB work_mem

-- With 50 concurrent connections:
-- Potential memory: 4GB × 50 = 200GB!
-- System OOM → database crash
```

**Safe approach:**

```sql
-- 1. Conservative global setting
ALTER SYSTEM SET work_mem = '64MB';

-- 2. Increase per-session for known large operations
-- Application code:
BEGIN;
SET LOCAL work_mem = '1GB';
-- Large analytical query
SELECT ... complex aggregation ...;
COMMIT;  -- work_mem reverts to global setting

-- 3. Monitor actual usage
EXPLAIN (ANALYZE, BUFFERS)
SELECT ... your query ...;

-- Look for memory usage:
-- Sort Method: quicksort  Memory: 45678kB
-- If frequently spilling to disk, increase work_mem slightly

-- 4. Calculate safe maximum
-- Formula: (Total RAM - shared_buffers - OS) / (max_connections × avg_operations_per_query)
-- Example: (64GB - 16GB - 8GB) / (100 connections × 3 operations)
--        = 40GB / 300 = ~136MB per operation
```

**Signs work_mem is too low:**
* EXPLAIN shows "external merge" or "external sort"
* Queries writing to disk frequently
* Poor performance on sorts and aggregations

**Signs work_mem is too high:**
* OOM killer messages in logs
* Database crashes under load
* Swap usage increasing

---

## Interview Questions

**Question 1: Explain how InnoDB's midpoint insertion strategy prevents table scans from evicting hot data.**

**Difficulty:** Senior

**Answer:**

**The Problem (without midpoint insertion):**

Traditional LRU algorithm inserts new pages at the head (most recently used position):

```
LRU List (head = most recent):
[Hot1] [Hot2] [Hot3] ... [Cold1] [Cold2] → evict

-- Table scan reads millions of pages:
SELECT * FROM huge_logs;

-- Without midpoint insertion:
-- Scan page 1 → Insert at head → [Scan1] [Hot1] [Hot2] ...
-- Scan page 2 → Insert at head → [Scan2] [Scan1] [Hot1] [Hot2] ...
-- ...
-- Scan page N → Insert at head → [ScanN] ... [Scan2] [Scan1] → evict Hot1, Hot2, etc.

-- Result: Hot pages evicted by scan pages (buffer pool pollution)
```

**InnoDB's Solution (midpoint insertion):**

```
LRU List with midpoint (3/8 from tail):

┌─────────── New Sublist (5/8) ──────────┐   ┌── Old Sublist (3/8) ──┐
[Hot1] [Hot2] [Hot3] ... [HotN] | Midpoint | [Old1] [Old2] ... [OldN] → evict
                                     ▲
                                     │
                            New pages inserted here

-- Table scan:
-- Scan page 1 → Insert at midpoint (head of old sublist)
-- Scan page 2 → Insert at midpoint
-- Scan pages age in old sublist
-- Evicted from tail of old sublist

-- Hot pages in new sublist: UNAFFECTED by scan
```

**Detailed mechanism:**

```sql
-- Configuration
innodb_old_blocks_pct = 37         -- 37% of pool is old sublist
innodb_old_blocks_time = 1000      -- 1 second threshold

-- Page lifecycle:
-- 1. New page read → Inserted at midpoint (head of old sublist)
-- 2. If accessed within 1 second → Stays in old sublist
--    (Prevents scan pages that are read once from moving to new)
-- 3. If accessed after 1 second → Moves to new sublist
--    (Real hot data accessed multiple times)
-- 4. Pages in new sublist age slowly (5/8 of pool)
-- 5. Pages in old sublist age quickly (3/8 of pool)

-- Example: Sequential scan
SELECT * FROM logs WHERE date = '2024-01-01';  -- 1M rows, 100MB

-- Pages read: 6,250 pages (100MB / 16KB)
-- All inserted at midpoint
-- Accessed once → Stay in old sublist
-- Age out quickly → Evicted from tail
-- Hot data (frequently accessed indexes, tables) → Remain in new sublist

-- Example: Hot page
SELECT * FROM products WHERE product_id = 123;  -- Accessed 1000x/second

-- First access: Insert at midpoint
-- Second access (< 1 second later): Move to new sublist
-- Subsequent accesses: Stay at head of new sublist
-- Protected from eviction by scan traffic
```

**Why this matters:**

```sql
-- Without midpoint insertion:
-- Single report query scanning logs → Evicts all product/customer cache
-- Next transaction query → Slow (needs to rebuild cache)

-- With midpoint insertion:
-- Report query → Uses old sublist, doesn't affect hot data
-- Transaction query → Still fast (hot data in new sublist)
```

**Tuning for different workloads:**

```sql
-- Heavy scan workload (data warehousing)
SET GLOBAL innodb_old_blocks_pct = 50;  -- More space for scans
SET GLOBAL innodb_old_blocks_time = 100; -- Move to new sublist faster

-- OLTP workload (transactional)
SET GLOBAL innodb_old_blocks_pct = 25;  -- Less space for scans
SET GLOBAL innodb_old_blocks_time = 2000; -- Keep scans in old sublist longer
```

**Why This Matters:** Understanding buffer pool eviction algorithms is critical for mixed workloads where analytical queries and transactional queries coexist. This prevents performance degradation.

**Follow-up Questions:**
* How would you monitor if scan pollution is occurring?
* What alternatives exist if midpoint insertion isn't sufficient?

---

**Question 2: Your PostgreSQL database is using 32GB RAM but pg_stat_bgwriter shows high buffers_backend. What's wrong and how do you fix it?**

**Difficulty:** Senior

**Answer:**

**Understanding the problem:**

```sql
-- Check bgwriter statistics
SELECT * FROM pg_stat_bgwriter;

-- Key columns:
-- buffers_clean: Buffers written by bgwriter (GOOD)
-- buffers_backend: Buffers written by backend processes (BAD)
-- buffers_checkpoint: Buffers written during checkpoints

-- High buffers_backend means:
-- Backend processes (query connections) are writing dirty buffers themselves
-- This should be bgwriter's job!
-- Indicates bgwriter can't keep up
```

**Why backends write buffers:**

```
Normal flow:
1. Query modifies data → Creates dirty buffer
2. Background writer → Gradually writes dirty buffers to disk
3. Buffer pool has space for new reads

When bgwriter can't keep up:
1. Query modifies data → Creates dirty buffer
2. Bgwriter too slow → Dirty buffers accumulate
3. Buffer pool fills up with dirty buffers
4. New read needs buffer → No clean buffers available
5. Backend must write dirty buffer itself (SLOW!)
6. Backend then uses buffer for new read
```

**Diagnosis:**

```sql
-- 1. Check bgwriter stats
SELECT 
    checkpoints_timed,
    checkpoints_req,
    buffers_checkpoint,
    buffers_clean,
    buffers_backend,
    buffers_backend * 100.0 / (buffers_checkpoint + buffers_clean + buffers_backend) AS backend_pct
FROM pg_stat_bgwriter;

-- Problem indicators:
-- backend_pct > 10%: Bgwriter underperforming
-- checkpoints_req > checkpoints_timed: Too many checkpoint requests

-- 2. Check current settings
SHOW bgwriter_delay;           -- Default: 200ms
SHOW bgwriter_lru_maxpages;    -- Default: 100
SHOW bgwriter_lru_multiplier;  -- Default: 2.0
```

**Solutions:**

**1. Tune bgwriter to be more aggressive:**

```sql
-- postgresql.conf

-- Increase pages written per round
bgwriter_lru_maxpages = 500   -- Up from 100

-- Decrease delay between rounds
bgwriter_delay = 100ms         -- Down from 200ms

-- Increase multiplier
bgwriter_lru_multiplier = 3.0  -- Up from 2.0

-- Result: Bgwriter writes more, more frequently
-- Reduces burden on backends
```

**2. Increase shared_buffers (if RAM available):**

```sql
-- More buffer space → Less pressure to write dirty buffers

-- Current: 8GB
-- Increase to: 16GB (if 64GB total RAM)

shared_buffers = 16GB

-- Requires restart
```

**3. Tune checkpoint behavior:**

```sql
-- Checkpoint too frequent → Many dirty buffers → Backends write
-- Increase checkpoint distance

max_wal_size = 4GB              -- Up from 1GB
checkpoint_timeout = 15min      -- Up from 5min
checkpoint_completion_target = 0.9  -- Spread checkpoint over 90% of timeout

-- Fewer checkpoints → Bgwriter has more time to clean buffers
```

**4. Reduce write load (application-level):**

```sql
-- Analyze write pattern
SELECT 
    schemaname,
    relname,
    n_tup_ins + n_tup_upd + n_tup_del AS total_writes,
    n_tup_upd,
    n_tup_hot_upd,
    n_tup_hot_upd::FLOAT / NULLIF(n_tup_upd, 0) AS hot_update_ratio
FROM pg_stat_user_tables
ORDER BY total_writes DESC
LIMIT 10;

-- Low hot_update_ratio → Index updates on every update
-- Consider:
-- - Remove unnecessary indexes
-- - Use partial indexes
-- - Optimize update frequency
```

**5. Use faster storage:**

```sql
-- SSD/NVMe can handle more writes
-- Configure for SSD:

random_page_cost = 1.1          -- Down from 4.0
effective_io_concurrency = 200  -- Up from 1

-- Increase I/O capacity
bgwriter_lru_maxpages = 1000    -- SSD can handle more
```

**Monitoring after changes:**

```sql
-- Reset statistics
SELECT pg_stat_reset_shared('bgwriter');

-- Wait for workload
-- Check again after 1 hour
SELECT 
    buffers_clean,
    buffers_backend,
    buffers_backend * 100.0 / (buffers_checkpoint + buffers_clean + buffers_backend) AS backend_pct
FROM pg_stat_bgwriter;

-- Target: backend_pct < 5%
```

**Example fix:**

```sql
-- Before:
-- buffers_clean: 1M
-- buffers_backend: 500K (33%!) → BAD
-- bgwriter_lru_maxpages: 100

-- Changes:
bgwriter_lru_maxpages = 500
bgwriter_delay = 100ms
max_wal_size = 2GB

-- After:
-- buffers_clean: 5M
-- buffers_backend: 100K (2%) → GOOD
```

**Why This Matters:** High buffers_backend indicates a performance problem where query processes are doing I/O work that should be handled by background processes. This slows down queries and increases latency.

**Follow-up Questions:**
* How would you determine the optimal bgwriter_lru_maxpages value?
* What's the relationship between checkpoint frequency and bgwriter performance?
* How does this relate to WAL volume?

---

## Key Takeaways

* **Buffer pools are critical** - most important memory structure for performance
* **InnoDB uses 70-80% RAM** for buffer pool; PostgreSQL uses 25% shared_buffers + OS cache
* **LRU variants prevent cache pollution** - midpoint insertion (InnoDB), clock-sweep (PostgreSQL)
* **Monitor hit ratios** - target > 99% for both databases
* **work_mem is per-operation** - set conservatively to avoid OOM
* **Background processes matter** - bgwriter, checkpointer keep buffers clean
* **Different design philosophies** - InnoDB self-caching vs PostgreSQL OS cache reliance
* **Tune for workload** - OLTP vs OLAP require different configurations
* **Buffer pool instances** reduce contention on multi-core systems
* **Memory sizing is critical** - too small hurts performance, too large wastes resources

---

## 2.3 Query Execution Pipeline

Understanding how databases execute queries reveals optimization opportunities and explains performance behavior.

### MySQL Query Execution

MySQL processes queries through several distinct phases:

```
Client Request
      ↓
┌─────────────────────────────────────────┐
│     1. Connection Handling              │
│  - Authentication                       │
│  - Connection pool management           │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     2. Query Parser                     │
│  - Syntax validation                    │
│  - Create parse tree                    │
│  - Semantic analysis                    │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     3. Query Rewriter                   │
│  - View expansion                       │
│  - Subquery flattening                  │
│  - Constant folding                     │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     4. Query Optimizer                  │
│  - Cost-based optimization              │
│  - Index selection                      │
│  - Join order determination             │
│  - Access method selection              │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     5. Execution Plan Generation        │
│  - Convert to executable plan           │
│  - Iterator-based model                 │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     6. Query Execution                  │
│  - Storage Engine API calls             │
│  - Row fetching and filtering           │
│  - Join processing                      │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     7. Result Formatting                │
│  - Convert to wire protocol             │
│  - Send to client                       │
└─────────────────────────────────────────┘
```

#### Phase 1: Connection Handling

```sql
-- Connection establishment
mysql -u username -p -h hostname database_name

-- What happens:
-- 1. TCP connection established
-- 2. Authentication (mysql.user table)
-- 3. Check max_connections limit
-- 4. Allocate thread/connection
-- 5. Initialize session variables

-- Monitor connections
SHOW PROCESSLIST;
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';

-- Connection configuration
SHOW VARIABLES LIKE 'max_connections';  -- Default: 151

-- Per-connection memory allocation:
-- - Thread stack
-- - Sort buffer
-- - Join buffer
-- - Connection buffer
-- Total: ~1-4MB per connection
```

#### Phase 2: Query Parser

```sql
-- Parser validates syntax and creates parse tree
-- Example query:
SELECT e.first_name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.dept_id
WHERE e.salary > 50000;

-- Parse tree structure:
-- SELECT
--   ├─ Column List
--   │   ├─ e.first_name
--   │   └─ d.dept_name
--   ├─ FROM
--   │   ├─ employees (alias: e)
--   │   └─ JOIN departments (alias: d)
--   │       └─ ON e.department_id = d.dept_id
--   └─ WHERE
--       └─ e.salary > 50000

-- Parser errors detected here:
SELECT * FROM non_existent_table;
-- ERROR 1146: Table doesn't exist

SELECT first_name, invalid_column FROM employees;
-- ERROR 1054: Unknown column
```

#### Phase 3: Query Rewriter

```sql
-- Rewrites queries for optimization

-- Example 1: Constant folding
-- Original:
SELECT * FROM orders WHERE created_at > DATE_SUB(NOW(), INTERVAL 7 DAY);

-- Rewritten (at execution time):
SELECT * FROM orders WHERE created_at > '2024-01-25 10:30:00';

-- Example 2: View expansion
CREATE VIEW active_employees AS
SELECT * FROM employees WHERE status = 'active';

SELECT * FROM active_employees WHERE department_id = 5;

-- Rewritten:
SELECT * FROM employees 
WHERE status = 'active' AND department_id = 5;

-- Example 3: Subquery flattening
-- Original:
SELECT * FROM employees 
WHERE department_id IN (SELECT dept_id FROM departments WHERE location = 'NY');

-- May be rewritten to semi-join:
SELECT e.* FROM employees e
WHERE EXISTS (SELECT 1 FROM departments d 
              WHERE d.dept_id = e.department_id AND d.location = 'NY');
```

#### Phase 4: Query Optimizer

```sql
-- Cost-based optimization
-- Considers:
-- - Table statistics (row counts, data distribution)
-- - Index availability and selectivity
-- - Join order possibilities
-- - Access methods (full scan, index scan, etc.)

-- Example optimization decision:
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date > '2024-01-01';

-- Optimizer evaluates:
-- Option 1: Filter orders first, then join
--   Cost: Filter(orders) + Join(filtered_orders, customers)
--
-- Option 2: Join all, then filter
--   Cost: Join(orders, customers) + Filter(result)
--
-- Chooses lower cost option (usually Option 1)

-- View optimizer decisions
SHOW WARNINGS;  -- After EXPLAIN
-- Shows optimizer notes, conversions, etc.
```

#### Phase 5: Execution Plan Generation

```sql
-- Converts optimizer plan to executable iterators
-- Iterator model: Each operation produces/consumes rows

-- Example plan for join:
-- NestedLoopIterator
--   ├─ FilterIterator (orders.order_date > '2024-01-01')
--   │   └─ TableScanIterator (orders)
--   └─ IndexLookupIterator (customers by customer_id)

-- Each iterator:
-- - Init(): Initialize
-- - Read(): Get next row
-- - End(): Cleanup

-- Visible in EXPLAIN FORMAT=TREE (MySQL 8.0.16+)
EXPLAIN FORMAT=TREE
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date > '2024-01-01';

-- Output shows iterator hierarchy
```

#### Phase 6: Query Execution

```sql
-- Storage engine API calls
-- InnoDB provides:
-- - handler::index_read() - Read by index
-- - handler::rnd_next() - Sequential read
-- - handler::index_next() - Next index entry
-- - handler::write_row() - Insert
-- - handler::update_row() - Update
-- - handler::delete_row() - Delete

-- Execution flow:
-- 1. Open tables and indexes
-- 2. Acquire locks
-- 3. Execute plan iterators
-- 4. Filter rows
-- 5. Process joins
-- 6. Apply sorts/aggregations
-- 7. Return results

-- Monitor execution
SHOW PROFILE FOR QUERY 1;
-- Shows time spent in each phase:
-- - Opening tables
-- - System lock
-- - Table lock
-- - Init
-- - Optimizing
-- - Statistics
-- - Preparing
-- - Executing
-- - Sending data
-- - End
```

### PostgreSQL Query Execution

PostgreSQL has a more complex pipeline with additional phases:

```
Client Request
      ↓
┌─────────────────────────────────────────┐
│     1. Connection Handling (Postmaster) │
│  - Fork backend process                 │
│  - Authentication                       │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     2. Parser                           │
│  - Lexical analysis                     │
│  - Syntax analysis                      │
│  - Create raw parse tree                │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     3. Analyzer (Parse Analysis)        │
│  - Semantic validation                  │
│  - Name resolution                      │
│  - Type checking                        │
│  - Transform parse tree                 │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     4. Rewriter (Rule System)           │
│  - View expansion                       │
│  - Rule application                     │
│  - Macro expansion                      │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     5. Planner/Optimizer                │
│  - Generate possible paths              │
│  - Cost estimation                      │
│  - Select cheapest path                 │
│  - Create execution plan                │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     6. Executor                         │
│  - Execute plan nodes                   │
│  - Tuple processing                     │
│  - MVCC visibility checks               │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│     7. Result Return                    │
│  - Format results                       │
│  - Send to client                       │
└─────────────────────────────────────────┘
```

#### Phase 1: Connection Handling (Postmaster)

```sql
-- PostgreSQL uses process-per-connection model
psql -U username -d database_name -h hostname

-- What happens:
-- 1. Connection to postmaster process
-- 2. Postmaster forks new backend process
-- 3. Authentication (pg_hba.conf rules)
-- 4. Initialize backend process
-- 5. Load database context

-- Process isolation benefits:
-- - Crash in one connection doesn't affect others
-- - Better security (process boundaries)
-- - Simpler memory management

-- Drawbacks:
-- - Higher memory per connection (~10MB)
-- - Fork overhead

-- Monitor connections
SELECT 
    pid,
    usename,
    application_name,
    client_addr,
    backend_start,
    state,
    query
FROM pg_stat_activity;

-- Connection configuration
SHOW max_connections;  -- Default: 100

-- Use connection pooling (PgBouncer) for high connection counts
```

#### Phase 2: Parser

```sql
-- Creates raw parse tree from SQL text
-- Example:
SELECT p.product_name, SUM(o.quantity) 
FROM products p
JOIN order_items o ON p.product_id = o.product_id
GROUP BY p.product_name;

-- Parser output (simplified):
-- SelectStmt
--   ├─ targetList: [p.product_name, SUM(o.quantity)]
--   ├─ fromClause: [RangeVar(products, alias=p)]
--   ├─ joinClause: [JoinExpr(order_items, condition=...)]
--   └─ groupClause: [p.product_name]

-- Parser errors:
SELECT * FROM;  -- Syntax error
-- ERROR: syntax error at or near ";"
```

#### Phase 3: Analyzer (Parse Analysis)

```sql
-- Validates semantics and resolves names
-- Adds type information
-- Transforms parse tree to query tree

-- Checks:
-- 1. Table/column existence
SELECT invalid_column FROM products;
-- ERROR: column "invalid_column" does not exist

-- 2. Type compatibility
SELECT * FROM products WHERE price > 'not_a_number';
-- ERROR: operator does not exist: numeric > text

-- 3. Aggregate validity
SELECT product_name, SUM(price) 
FROM products;  -- Missing GROUP BY
-- ERROR: column "product_name" must appear in GROUP BY

-- 4. Permission checks
SELECT * FROM sensitive_table;
-- ERROR: permission denied for table sensitive_table

-- Transformations:
-- - * expansion to column list
-- - Function call resolution
-- - Type coercion insertion
```

#### Phase 4: Rewriter (Rule System)

```sql
-- Applies view expansion and rules
-- Example 1: View expansion
CREATE VIEW high_value_orders AS
SELECT * FROM orders WHERE total > 1000;

SELECT * FROM high_value_orders WHERE customer_id = 123;

-- Rewritten to:
SELECT * FROM orders 
WHERE total > 1000 AND customer_id = 123;

-- Example 2: Rule application
CREATE RULE log_deletes AS
ON DELETE TO important_table
DO ALSO INSERT INTO audit_log VALUES (OLD.id, now());

DELETE FROM important_table WHERE id = 1;

-- Rewritten to:
-- DELETE FROM important_table WHERE id = 1;
-- INSERT INTO audit_log VALUES (1, now());

-- Example 3: Inherited tables
CREATE TABLE measurements (
    time TIMESTAMP,
    value NUMERIC
);

CREATE TABLE measurements_2024 () INHERITS (measurements);

SELECT * FROM measurements;

-- Rewritten to:
-- SELECT * FROM measurements
-- UNION ALL
-- SELECT * FROM measurements_2024
```

#### Phase 5: Planner/Optimizer

```sql
-- Generates paths and selects cheapest
-- Creates detailed execution plan

-- Path generation:
-- For each table, generates:
-- - Sequential scan path
-- - Index scan paths (for each usable index)
-- - Bitmap scan paths

-- For joins, generates:
-- - Nested loop paths
-- - Hash join paths
-- - Merge join paths

-- Cost estimation based on:
-- - pg_class (table row counts, pages)
-- - pg_statistic (column distributions)
-- - Configuration (random_page_cost, cpu_tuple_cost, etc.)

-- Example cost calculation:
-- Sequential scan cost:
-- startup_cost = 0
-- run_cost = pages_to_read × seq_page_cost + rows × cpu_tuple_cost
-- total_cost = startup_cost + run_cost

-- Index scan cost:
-- startup_cost = index_pages × random_page_cost
-- run_cost = table_pages × random_page_cost + rows × cpu_tuple_cost
-- total_cost = startup_cost + run_cost

-- View planning process
SET client_min_messages = DEBUG1;
EXPLAIN SELECT ...;
-- Shows detailed optimizer decisions in logs
```

#### Phase 6: Executor

```sql
-- Executes plan nodes recursively
-- Each node type has specific execution function

-- Common node types:
-- - SeqScan: Sequential table scan
-- - IndexScan: B-tree index scan
-- - BitmapHeapScan: Bitmap scan with heap lookup
-- - NestedLoop: Nested loop join
-- - HashJoin: Hash join
-- - MergeJoin: Merge join
-- - Sort: Sort operation
-- - Agg: Aggregation
-- - Group: Grouping

-- Execution model:
-- Each node implements:
-- - ExecInitNode(): Initialize
-- - ExecProcNode(): Get next tuple
-- - ExecEndNode(): Cleanup

-- MVCC visibility checks during execution:
-- For each tuple:
-- 1. Check tuple headers (xmin, xmax)
-- 2. Determine visibility for current transaction
-- 3. Return only visible tuples

-- Example execution:
EXPLAIN (ANALYZE, BUFFERS)
SELECT p.product_name, SUM(o.quantity)
FROM products p
JOIN order_items o ON p.product_id = o.product_id
GROUP BY p.product_name;

-- Shows:
-- - Actual execution times
-- - Rows processed
-- - Buffer usage
-- - Sort methods
```

### Execution Plan Nodes Deep Dive

#### Sequential Scan

```sql
-- Reads entire table page by page
EXPLAIN ANALYZE SELECT * FROM large_table;

-- Seq Scan on large_table  (cost=0.00..1500.00 rows=100000 width=50)
--                           (actual time=0.010..15.234 rows=100000 loops=1)

-- When used:
-- - No suitable index
-- - Returning large % of rows
-- - Table is small
-- - Index scan would be more expensive

-- Optimization:
-- Add WHERE clause to reduce rows
-- Add index if selective filter
-- Increase shared_buffers if table fits in memory
```

#### Index Scan

```sql
-- Traverses B-tree index, fetches heap tuples
CREATE INDEX idx_product_id ON order_items(product_id);

EXPLAIN ANALYZE 
SELECT * FROM order_items WHERE product_id = 123;

-- Index Scan using idx_product_id on order_items
--   (cost=0.29..8.31 rows=5 width=30)
--   (actual time=0.015..0.025 rows=5 loops=1)
--   Index Cond: (product_id = 123)

-- Steps:
-- 1. Traverse B-tree to find product_id = 123
-- 2. Follow index pointers to heap tuples
-- 3. Check tuple visibility (MVCC)
-- 4. Return visible tuples

-- Cost components:
-- - Index traversal (log N)
-- - Heap access (random I/O)
-- - Visibility checks
```

#### Index Only Scan

```sql
-- Reads only index (covering index)
CREATE INDEX idx_product_covering ON products(category, product_id, price);

EXPLAIN ANALYZE
SELECT product_id, price 
FROM products 
WHERE category = 'Electronics';

-- Index Only Scan using idx_product_covering on products
--   (cost=0.29..4.36 rows=50 width=12)
--   (actual time=0.012..0.045 rows=50 loops=1)
--   Index Cond: (category = 'Electronics'::text)
--   Heap Fetches: 0

-- Advantages:
-- - No heap access needed
-- - Faster (no random I/O to heap)
-- - Uses less memory

-- Requires:
-- - All selected columns in index
-- - Visibility map indicates all tuples visible
--   (or heap fetch to check visibility)

-- Heap Fetches > 0:
-- Some tuples need visibility check from heap
-- VACUUM creates visibility map
```

#### Bitmap Heap Scan

```sql
-- Two-phase scan for multiple conditions or ranges
CREATE INDEX idx_price ON products(price);
CREATE INDEX idx_category ON products(category);

EXPLAIN ANALYZE
SELECT * FROM products
WHERE price < 100 OR category = 'Books';

-- Bitmap Heap Scan on products
--   Recheck Cond: ((price < 100) OR (category = 'Books'))
--   ->  BitmapOr
--         ->  Bitmap Index Scan on idx_price
--               Index Cond: (price < 100)
--         ->  Bitmap Index Scan on idx_category
--               Index Cond: (category = 'Books')

-- Phase 1 (Bitmap Index Scan):
-- - Scan indexes
-- - Build bitmap of heap page numbers
-- - Combine bitmaps (OR/AND)

-- Phase 2 (Bitmap Heap Scan):
-- - Sort page numbers
-- - Sequential read of pages
-- - Check actual conditions (recheck)

-- Benefits:
-- - Combines multiple indexes
-- - Converts random I/O to sequential I/O
-- - More efficient than multiple index scans
```

#### Nested Loop Join

```sql
-- For each outer row, scan inner table
EXPLAIN ANALYZE
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date = '2024-01-15';

-- Nested Loop  (cost=0.29..85.45 rows=10 width=100)
--   ->  Seq Scan on orders o  (cost=0.00..15.00 rows=10 width=50)
--         Filter: (order_date = '2024-01-15')
--   ->  Index Scan using customers_pkey on customers c
--         (cost=0.29..8.31 rows=1 width=50)
--         Index Cond: (customer_id = o.customer_id)

-- Pseudocode:
-- for each row in orders (where order_date = '2024-01-15'):
--     lookup customer in customers by customer_id
--     output joined row

-- Efficient when:
-- - Small outer table
-- - Index on inner table join column
-- - Early filtering reduces outer rows
```

#### Hash Join

```sql
-- Build hash table from smaller table, probe with larger
EXPLAIN ANALYZE
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

-- Hash Join  (cost=45.00..1250.00 rows=10000 width=100)
--   Hash Cond: (o.customer_id = c.customer_id)
--   ->  Seq Scan on orders o  (cost=0.00..1000.00 rows=10000)
--   ->  Hash  (cost=25.00..25.00 rows=1000 width=50)
--         Buckets: 1024  Batches: 1  Memory Usage: 65kB
--         ->  Seq Scan on customers c  (cost=0.00..25.00 rows=1000)

-- Phase 1 (Build):
-- - Scan customers (smaller table)
-- - Build hash table: customer_id → customer_row
-- - Store in work_mem

-- Phase 2 (Probe):
-- - Scan orders
-- - For each order, lookup customer_id in hash table
-- - Output matches

-- Batches = 1: Everything fits in work_mem (good)
-- Batches > 1: Spilled to disk (slower)

-- Efficient for:
-- - Large equi-joins
-- - Sufficient work_mem
-- - No suitable indexes
```

---

## Frequently Asked Questions

**Q1: Why does PostgreSQL fork a new process for each connection while MySQL uses threads?**

**A:**

**PostgreSQL (Process-per-connection):**
```
Client 1 → Backend Process 1 (isolated memory space)
Client 2 → Backend Process 2 (isolated memory space)
Client 3 → Backend Process 3 (isolated memory space)
```

**Advantages:**
* **Isolation**: Crash in one connection doesn't affect others
* **Security**: Process boundaries provide security isolation
* **Simpler memory management**: No shared memory corruption risks
* **Better for extensions**: Safer for third-party code

**Disadvantages:**
* **Memory overhead**: ~10MB per connection (separate memory space)
* **Fork cost**: Process creation overhead
* **Context switching**: More expensive than thread switching

**MySQL (Thread-per-connection):**
```
MySQL Server Process
  ├─ Thread 1 (Client 1) ─┐
  ├─ Thread 2 (Client 2) ─┼→ Shared memory space
  └─ Thread 3 (Client 3) ─┘
```

**Advantages:**
* **Lower memory**: ~1-4MB per connection (shared memory)
* **Faster creation**: Thread creation cheaper than process fork
* **Efficient communication**: Shared memory between threads

**Disadvantages:**
* **Less isolation**: Bug in one thread can crash entire server
* **Synchronization complexity**: Mutexes needed for shared structures
* **Security**: All threads in same address space

**Why the difference?**
* **PostgreSQL**: Prioritizes stability and correctness (ACID focus)
* **MySQL**: Prioritizes efficiency and connection scalability

**Solutions for high connection counts:**

```sql
-- PostgreSQL: Use connection pooler
-- PgBouncer reduces backend processes
-- 1000 client connections → 50 backend processes

-- MySQL: Usually handles more direct connections
-- But still benefits from connection pooling for very high loads
```

**Related Concepts**: Connection pooling, process vs thread model

---

**Q2: What's the difference between the Planner and Executor in PostgreSQL?**

**A:**

**Planner (Planning phase):**
* Figures out **HOW** to execute the query
* Doesn't actually execute anything
* Generates multiple possible plans (paths)
* Estimates costs for each plan
* Selects cheapest plan

```sql
-- Planner work:
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;

-- Output is the PLAN (not results):
-- Index Scan using idx_customer_id on orders
--   (cost=0.29..8.31 rows=5 width=50)
--   Index Cond: (customer_id = 123)

-- Planner considered:
-- 1. Sequential scan: cost = 450.00
-- 2. Index scan: cost = 8.31 ← chosen (cheaper)
```

**Executor (Execution phase):**
* Executes the plan created by planner
* Actually reads data
* Applies filters
* Performs joins
* Returns results

```sql
-- Executor work:
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 123;

-- Shows actual execution:
-- Index Scan using idx_customer_id on orders
--   (cost=0.29..8.31 rows=5 width=50)              ← Planner estimate
--   (actual time=0.012..0.025 rows=4 loops=1)      ← Executor actual
--                                   ^
--                          4 rows actually found

-- Executor did:
-- 1. Traversed index for customer_id = 123
-- 2. Fetched 4 matching rows from heap
-- 3. Checked visibility (MVCC)
-- 4. Returned results
```

**Separation benefits:**

```sql
-- 1. EXPLAIN shows plan without execution
EXPLAIN SELECT * FROM huge_table;
-- Fast (just planning, no execution)

-- 2. EXPLAIN ANALYZE executes and shows actual performance
EXPLAIN ANALYZE SELECT * FROM huge_table;
-- Slow (actual execution + timing)

-- 3. Prepared statements: Plan once, execute many times
PREPARE stmt AS SELECT * FROM orders WHERE customer_id = $1;
EXECUTE stmt(123);  -- Uses cached plan
EXECUTE stmt(456);  -- Reuses plan
EXECUTE stmt(789);  -- Reuses plan
```

**Why estimates differ from actuals:**

```sql
-- Planner estimate: rows=1000
-- Executor actual: rows=10

-- Causes:
-- 1. Outdated statistics
ANALYZE orders;  -- Updates statistics

-- 2. Correlation between columns not known
-- WHERE category = 'A' AND status = 'active'
-- Planner assumes independence
-- Reality: All category 'A' are 'active' (correlated)

-- 3. Data skew
-- Most rows have customer_id = 1
-- Planner uses average distribution

-- 4. Functional dependencies
-- WHERE state = 'CA' → Most have city = 'LA'
-- Planner doesn't know this dependency
```

---

**Q3: How can I see which phase of query execution is slow?**

**A:**

**MySQL - SHOW PROFILE:**

```sql
-- Enable profiling
SET profiling = 1;

-- Run query
SELECT o.*, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date >= '2024-01-01';

-- View profiles
SHOW PROFILES;
-- +----------+------------+------------------+
-- | Query_ID | Duration   | Query            |
-- +----------+------------+------------------+
-- |        1 | 0.50234000 | SELECT o.*, ...  |
-- +----------+------------+------------------+

-- Detailed breakdown
SHOW PROFILE FOR QUERY 1;
-- +----------------------+----------+
-- | Status               | Duration |
-- +----------------------+----------+
-- | starting             | 0.000070 |
-- | checking permissions | 0.000010 |
-- | Opening tables       | 0.000025 |
-- | init                 | 0.000035 |
-- | System lock          | 0.000012 |
-- | optimizing           | 0.000015 |
-- | statistics           | 0.000025 |
-- | preparing            | 0.000020 |
-- | executing            | 0.000005 |
-- | Sending data         | 0.501000 |  ← Bottleneck here!
-- | end                  | 0.000010 |
-- | query end            | 0.000008 |
-- | closing tables       | 0.000012 |
-- | freeing items        | 0.000020 |
-- | cleaning up          | 0.000015 |
-- +----------------------+----------+

-- CPU and I/O details
SHOW PROFILE CPU, BLOCK IO FOR QUERY 1;
-- Shows:
-- - CPU_user, CPU_system
-- - Block_ops_in, Block_ops_out
```

**Interpretation:**

```sql
-- "Sending data" high:
-- - Query is executing (fetching/joining)
-- - May need index
-- - May need query optimization

-- "Opening tables" high:
-- - Too many tables
-- - Table cache too small
-- - File system slow

-- "statistics" high:
-- - Optimizer spending time analyzing
-- - Many possible plans to evaluate
-- - Complex query

-- "optimizing" high:
-- - Complex query structure
-- - Many joins or subqueries
-- - Simplify query

-- "Creating tmp table" appears:
-- - Temporary table needed
-- - GROUP BY without index
-- - Complex aggregation
-- - Increase tmp_table_size
```

**PostgreSQL - EXPLAIN ANALYZE with timing:**

```sql
-- Detailed timing per node
EXPLAIN (ANALYZE, TIMING, BUFFERS)
SELECT o.*, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_date >= '2024-01-01';

-- Output shows timing for each node:
-- Hash Join  (actual time=0.250..125.430 rows=1000)
--             ^^^^^^        ^^^^^^^
--            startup      total time
--   Hash Cond: (o.customer_id = c.customer_id)
--   Buffers: shared hit=450 read=50
--   ->  Seq Scan on orders o (actual time=0.010..100.250 rows=5000)
--                                          ^^^^^^^^^^
--                                     Bottleneck: 100ms
--         Filter: (order_date >= '2024-01-01')
--         Rows Removed by Filter: 45000
--         Buffers: shared hit=400 read=40
--   ->  Hash  (actual time=0.150..0.150 rows=1000)
--         Buckets: 1024  Batches: 1  Memory Usage: 65kB
--         Buffers: shared hit=50 read=10
--         ->  Seq Scan on customers c (actual time=0.005..0.120 rows=1000)
--               Buffers: shared hit=50 read=10

-- Planning Time: 0.450 ms
-- Execution Time: 125.750 ms
```

**Interpretation:**

```sql
-- High "actual time" on node:
-- This node is the bottleneck

-- "Rows Removed by Filter" high:
-- Filter is inefficient
-- Add index on filtered column

-- "Buffers: read" high:
-- Disk I/O (not cached)
-- Increase shared_buffers
-- Or data doesn't fit in cache

-- "Planning Time" high:
-- Complex query
-- Many tables/indexes
-- Consider simplification

-- "Execution Time >> sum of node times":
-- Overhead in result marshalling
-- Large result set
-- Network latency
```

**Using auto_explain for production monitoring:**

```sql
-- PostgreSQL: Log slow plans automatically
-- postgresql.conf
shared_preload_libraries = 'auto_explain'
auto_explain.log_min_duration = 1000  -- Log queries > 1 second
auto_explain.log_analyze = true
auto_explain.log_buffers = true
auto_explain.log_timing = true

-- Queries > 1 second automatically logged with EXPLAIN ANALYZE
-- Check PostgreSQL logs for execution plans
```

---

## Interview Questions

**Question 1: A query plan shows "actual rows=100000" but "estimated rows=10". What's the impact and how do you fix it?**

**Difficulty:** Senior

**Answer:**

**The Problem:**

```sql
EXPLAIN ANALYZE
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status = 'pending';

-- Nested Loop  (cost=0.29..85.45 rows=10 width=100)
--              (actual time=0.123..5000.234 rows=100000 loops=1)
--                            estimated: 10 ^^^     ^^^ actual: 100000

-- Planner thought only 10 rows → Chose nested loop
-- Reality: 100,000 rows → Nested loop is TERRIBLE choice
-- Should have chosen hash join!
```

**Impact:**

**1. Wrong join algorithm:**
```
Nested Loop with 100,000 outer rows:
- For each of 100,000 rows, lookup in customers
- 100,000 index lookups (even with index, expensive)
- Total time: ~5 seconds

Hash Join would be:
- Build hash table from customers once
- Probe once for each order
- Total time: ~0.5 seconds (10x faster!)
```

**2. Wrong index selection:**
```sql
-- Planner thinks: 10 rows → index scan efficient
-- Reality: 100,000 rows → sequential scan better
```

**3. Cascading errors:**
```sql
-- Underestimate propagates through plan
SELECT ... FROM (
    SELECT * FROM orders WHERE status = 'pending'  -- thinks 10 rows
) o
JOIN order_items oi ON o.order_id = oi.order_id;
-- Thinks joining 10 orders (wrong!)
-- Actually joining 100,000 orders (very different plan needed!)
```

**Root Causes:**

**1. Outdated statistics:**
```sql
-- Check statistics age
-- PostgreSQL:
SELECT 
    schemaname,
    relname,
    last_analyze,
    last_autoanalyze,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables
WHERE relname = 'orders';

-- last_analyze: 2023-01-01 (11 months ago!)
-- Data changed significantly since then

-- MySQL:
SELECT 
    TABLE_NAME,
    UPDATE_TIME,
    TABLE_ROWS
FROM information_schema.TABLES
WHERE TABLE_NAME = 'orders';
```

**2. Data skew:**
```sql
-- Statistics show average:
-- status 'pending': 0.1% of rows (100 out of 100,000)

-- Reality after Black Friday:
-- status 'pending': 99% of rows (100,000 out of 101,000)

-- Uniform distribution assumed, actual distribution highly skewed
```

**3. Correlated columns:**
```sql
WHERE status = 'pending' AND priority = 'high'

-- Planner assumes independence:
-- - 10% have status = 'pending'
-- - 5% have priority = 'high'
-- - Estimate: 10% × 5% = 0.5%

-- Reality: All pending orders are high priority (correlated)
-- - Actual: 10% (not 0.5%)
```

**Fixes:**

**1. Update statistics (immediate fix):**
```sql
-- PostgreSQL:
ANALYZE orders;

-- MySQL:
ANALYZE TABLE orders;

-- Re-run query, check new plan
EXPLAIN ANALYZE SELECT ...;
-- Should now estimate correctly
```

**2. Increase statistics target for problematic columns:**
```sql
-- PostgreSQL: More detailed statistics
ALTER TABLE orders ALTER COLUMN status SET STATISTICS 1000;
-- Default: 100
-- Higher: More detailed histogram, better estimates

ANALYZE orders;

-- MySQL: Persistent statistics
ALTER TABLE orders STATS_PERSISTENT=1;
ANALYZE TABLE orders;
```

**3. Use extended statistics (PostgreSQL 10+):**
```sql
-- For correlated columns
CREATE STATISTICS orders_status_priority_stats (dependencies)
ON status, priority FROM orders;

ANALYZE orders;

-- Planner now knows status and priority are correlated
```

**4. Query hints (temporary workaround):**
```sql
-- PostgreSQL: Disable nested loop to force hash join
SET enable_nestloop = off;
EXPLAIN ANALYZE SELECT ...;
SET enable_nestloop = on;

-- MySQL: Force join order
SELECT /*+ JOIN_ORDER(orders, customers) */ ...

-- ⚠️ Hints are workarounds, fix root cause instead
```

**5. Schedule regular ANALYZE:**
```sql
-- PostgreSQL: Enable autovacuum
autovacuum = on
autovacuum_analyze_threshold = 50
autovacuum_analyze_scale_factor = 0.1

-- Analyzes when > 50 + 10% of rows changed

-- MySQL: Manual scheduled job
-- Cron: Daily at 2 AM
0 2 * * * mysql -e "ANALYZE TABLE orders;"
```

**Monitoring for future occurrences:**

```sql
-- PostgreSQL: Log plans with bad estimates
-- Find in pg_stat_statements where:
-- mean_exec_time / mean_plan_time > 100
-- (Execution much slower than expected)

-- MySQL: Slow query log
-- Compare Query_time to rows examined
-- High Query_time with estimated rows << examined rows
```

**Why This Matters:** Incorrect cardinality estimates are the #1 cause of bad query plans. Understanding how to diagnose and fix them is essential for database performance.

**Follow-up Questions:**
* How would you identify queries with bad estimates in production?
* What's the trade-off between statistics detail and ANALYZE time?
* When would you use query hints vs fixing statistics?

---

## Key Takeaways

* **Query execution has distinct phases** - parsing, optimization, execution
* **MySQL uses thread-per-connection** - lower memory, less isolation
* **PostgreSQL uses process-per-connection** - higher memory, better isolation
* **Parse trees transform through pipeline** - syntax → semantics → optimized → executable
* **Query rewriter optimizes before planner** - view expansion, constant folding, subquery flattening
* **Planner generates multiple paths** - estimates costs, selects cheapest
* **Executor implements plan nodes** - each node type has specific execution logic
* **MVCC visibility checks happen during execution** - not during planning
* **Statistics drive optimizer decisions** - outdated stats cause bad plans
* **EXPLAIN shows plan, EXPLAIN ANALYZE shows reality** - always compare estimates to actuals

---

## 2.4 Query Optimizer Internals

The query optimizer is the brain of the database, deciding how to execute queries efficiently. Understanding optimizer behavior is crucial for writing performant SQL.

### Cost-Based Optimization

Both MySQL and PostgreSQL use cost-based optimization, estimating the "cost" of different execution strategies and choosing the cheapest.

#### Cost Model Fundamentals

**Cost units are arbitrary** - not seconds or bytes, but relative measures:

```
Basic cost components:
┌─────────────────────────────────────────┐
│  I/O Costs (Disk Access)                │
│  - Sequential page read: 1.0            │
│  - Random page read: 4.0 (default)      │
│  - Page write: 20.0                     │
└─────────────────────────────────────────┘
┌─────────────────────────────────────────┐
│  CPU Costs (Processing)                 │
│  - Tuple processing: 0.01               │
│  - Operator evaluation: 0.0025          │
│  - Index tuple processing: 0.005        │
└─────────────────────────────────────────┘
```

**Why costs matter:**
```sql
-- Sequential scan cost calculation:
-- Cost = (pages × seq_page_cost) + (rows × cpu_tuple_cost)

-- Example table: 1000 pages, 100,000 rows
-- Sequential scan cost:
-- = (1000 × 1.0) + (100,000 × 0.01)
-- = 1000 + 1000
-- = 2000

-- Index scan cost calculation:
-- Cost = (index_pages × random_page_cost) + 
--        (table_pages × random_page_cost) + 
--        (rows × cpu_tuple_cost)

-- Example: 10% selectivity (10,000 rows returned)
-- Index scan cost:
-- = (50 × 4.0) + (100 × 4.0) + (10,000 × 0.01)
-- = 200 + 400 + 100
-- = 700

-- Optimizer chooses index scan (700 < 2000)
```

### PostgreSQL Optimizer Deep Dive

#### Cost Parameters

```sql
-- View all cost parameters
SELECT name, setting, unit, short_desc
FROM pg_settings
WHERE name LIKE '%cost%' OR name LIKE '%page%';

-- Key parameters:
SHOW seq_page_cost;        -- 1.0 (sequential page read)
SHOW random_page_cost;     -- 4.0 (random page read)
SHOW cpu_tuple_cost;       -- 0.01 (process one row)
SHOW cpu_index_tuple_cost; -- 0.005 (process index entry)
SHOW cpu_operator_cost;    -- 0.0025 (evaluate operator like =, <)

-- For SSD storage (faster random access):
ALTER SYSTEM SET random_page_cost = 1.1;  -- Down from 4.0
SELECT pg_reload_conf();

-- For data warehouse (prefer scans):
ALTER SYSTEM SET random_page_cost = 2.0;
```

#### Statistics Collection

PostgreSQL collects detailed statistics for cost estimation:

```sql
-- Table-level statistics
SELECT 
    schemaname,
    tablename,
    n_live_tup,      -- Estimated live rows
    n_dead_tup,      -- Dead tuples (needs VACUUM)
    last_analyze,
    last_autoanalyze
FROM pg_stat_user_tables
WHERE tablename = 'orders';

-- Column-level statistics
SELECT 
    attname,
    null_frac,       -- Fraction of NULLs
    avg_width,       -- Average column width in bytes
    n_distinct,      -- Number of distinct values (-1 = unique)
    correlation      -- Physical vs logical order correlation
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'customer_id';

-- Most common values (MCV)
SELECT 
    attname,
    most_common_vals,
    most_common_freqs
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'status';

-- Example output:
-- most_common_vals:  {pending,shipped,delivered}
-- most_common_freqs: {0.60,0.30,0.10}
-- Meaning: 60% pending, 30% shipped, 10% delivered
```

#### Histogram Bounds

```sql
-- Distribution histogram for non-MCV values
SELECT 
    attname,
    histogram_bounds
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'total_amount';

-- Example output:
-- histogram_bounds: {10.00,25.50,45.00,75.00,100.00,250.00,500.00,1000.00}

-- Interpretation:
-- ~12.5% of values between 10.00 and 25.50
-- ~12.5% of values between 25.50 and 45.00
-- etc. (100 buckets by default)

-- Increase detail for important columns:
ALTER TABLE orders ALTER COLUMN total_amount SET STATISTICS 1000;
ANALYZE orders;
-- Now 1000 histogram buckets instead of 100
```

#### Selectivity Estimation

```sql
-- Selectivity = fraction of rows matching condition
-- Used to estimate result size

-- Example 1: Equality on indexed column
SELECT * FROM orders WHERE status = 'pending';

-- Optimizer checks pg_stats:
-- most_common_vals: {pending,shipped,delivered}
-- most_common_freqs: {0.60,0.30,0.10}
-- Selectivity for 'pending' = 0.60
-- Estimated rows = total_rows × 0.60 = 100,000 × 0.60 = 60,000

-- Example 2: Range condition
SELECT * FROM orders WHERE total_amount > 100;

-- Optimizer uses histogram_bounds:
-- Finds 100 in histogram
-- Calculates fraction above 100
-- Selectivity ≈ 0.25 (25% of values > 100)
-- Estimated rows = 100,000 × 0.25 = 25,000

-- Example 3: Multiple conditions (AND)
SELECT * FROM orders 
WHERE status = 'pending' AND total_amount > 100;

-- Default: Assumes independence
-- Selectivity = 0.60 × 0.25 = 0.15
-- Estimated rows = 100,000 × 0.15 = 15,000

-- If columns are correlated, estimate will be wrong!
```

#### Join Cardinality Estimation

```sql
-- Estimating join result size
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

-- Formula (simplified):
-- result_rows = (orders.rows × customers.rows) / max(distinct_orders.customer_id, distinct_customers.customer_id)

-- Example:
-- orders: 100,000 rows, 10,000 distinct customer_ids
-- customers: 10,000 rows, 10,000 distinct customer_ids
-- result_rows = (100,000 × 10,000) / max(10,000, 10,000)
--             = 1,000,000,000 / 10,000
--             = 100,000 rows

-- Assumes each customer has equal number of orders (uniform distribution)
-- Reality may differ!
```

#### Path Generation

PostgreSQL generates all possible paths for each table and operation:

```sql
-- For a simple query:
SELECT * FROM orders WHERE customer_id = 123;

-- Paths generated:
-- 1. Sequential Scan
--    - Read all pages
--    - Filter each row
--    - Cost: (pages × seq_page_cost) + (rows × cpu_tuple_cost)

-- 2. Index Scan on idx_customer_id
--    - Traverse B-tree
--    - Fetch matching heap tuples
--    - Cost: (index_pages × random_page_cost) + 
--            (heap_pages × random_page_cost) +
--            (rows × cpu_tuple_cost)

-- 3. Bitmap Index Scan + Bitmap Heap Scan
--    - Build bitmap from index
--    - Scan heap pages in order
--    - Cost: index_cost + heap_scan_cost

-- Optimizer calculates cost for each, chooses minimum
```

#### Join Order Selection

```sql
-- For N tables, there are (N-1)! possible join orders
-- 3 tables: 2! = 2 orderings
-- 4 tables: 3! = 6 orderings
-- 5 tables: 4! = 24 orderings
-- 10 tables: 9! = 362,880 orderings!

-- Example: 3-table join
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id;

-- Possible orderings:
-- 1. (orders ⋈ customers) ⋈ products
-- 2. (orders ⋈ products) ⋈ customers
-- 3. (customers ⋈ orders) ⋈ products
-- 4. (customers ⋈ products) ⋈ orders  -- Unlikely (no direct join)
-- 5. (products ⋈ orders) ⋈ customers
-- 6. (products ⋈ customers) ⋈ orders  -- Unlikely (no direct join)

-- Optimizer evaluates costs for valid orderings
-- Chooses lowest cost

-- Heuristics to reduce search space:
-- - Eliminate Cartesian products (no join condition)
-- - Prefer joins with filters on driving table
-- - Use greedy/dynamic programming for large joins
```

#### GEQO (Genetic Query Optimizer)

```sql
-- For very complex joins (many tables), exhaustive search is too expensive
-- PostgreSQL uses genetic algorithm

SHOW geqo;                    -- ON
SHOW geqo_threshold;          -- 12 (use GEQO when >= 12 tables)
SHOW geqo_effort;             -- 5 (1-10, higher = more iterations)
SHOW geqo_pool_size;          -- 0 (auto)
SHOW geqo_generations;        -- 0 (auto)

-- How GEQO works:
-- 1. Generate random join orders (population)
-- 2. Evaluate cost of each
-- 3. Select best performers
-- 4. Combine (crossover) to create new orders
-- 5. Mutate some randomly
-- 6. Repeat for N generations
-- 7. Return best order found

-- Trade-off: Speed vs optimality
-- GEQO finds "good enough" plan quickly
-- May not find absolute best plan

-- Disable for critical queries if needed:
SET geqo = off;  -- Forces exhaustive search
```

### MySQL Optimizer Deep Dive

#### Cost Model

MySQL 5.7+ uses a configurable cost model:

```sql
-- View cost constants
SELECT * FROM mysql.server_cost;
-- +------------------------------+------------+---------------------+
-- | cost_name                    | cost_value | last_update         |
-- +------------------------------+------------+---------------------+
-- | disk_temptable_create_cost   |       NULL | 2024-01-01 00:00:00 |
-- | disk_temptable_row_cost      |       NULL | 2024-01-01 00:00:00 |
-- | key_compare_cost             |       NULL | 2024-01-01 00:00:00 |
-- | memory_temptable_create_cost |       NULL | 2024-01-01 00:00:00 |
-- | memory_temptable_row_cost    |       NULL | 2024-01-01 00:00:00 |
-- | row_evaluate_cost            |       NULL | 2024-01-01 00:00:00 |
-- +------------------------------+------------+---------------------+

-- NULL means using default
-- Defaults (as of MySQL 8.0):
-- row_evaluate_cost: 0.1
-- key_compare_cost: 0.05
-- memory_temptable_create_cost: 1.0
-- memory_temptable_row_cost: 0.1
-- disk_temptable_create_cost: 20.0
-- disk_temptable_row_cost: 0.5

-- Engine-specific costs
SELECT * FROM mysql.engine_cost;
-- +-------------+-------------+--------------+------------+
-- | engine_name | device_type | cost_name    | cost_value |
-- +-------------+-------------+--------------+------------+
-- | default     |           0 | io_block_read_cost | NULL |
-- | default     |           0 | memory_block_read_cost | NULL |
-- +-------------+-------------+--------------+------------+

-- Defaults:
-- io_block_read_cost: 1.0 (read page from disk)
-- memory_block_read_cost: 0.25 (read page from buffer pool)

-- Tune for SSD:
UPDATE mysql.server_cost 
SET cost_value = 0.5 
WHERE cost_name = 'io_block_read_cost';

FLUSH OPTIMIZER_COSTS;  -- Apply changes
```

#### Index Statistics

```sql
-- Table statistics
SELECT 
    TABLE_NAME,
    TABLE_ROWS,
    AVG_ROW_LENGTH,
    DATA_LENGTH,
    INDEX_LENGTH
FROM information_schema.TABLES
WHERE TABLE_NAME = 'orders';

-- Index cardinality
SHOW INDEX FROM orders;
-- +--------+------------+------------------+--------------+
-- | Table  | Key_name   | Column_name      | Cardinality  |
-- +--------+------------+------------------+--------------+
-- | orders | PRIMARY    | order_id         |      100000  |
-- | orders | idx_cust   | customer_id      |       10000  |
-- | orders | idx_date   | order_date       |        1000  |
-- +--------+------------+------------------+--------------+

-- Cardinality = estimated distinct values
-- Higher cardinality = more selective = better for index

-- Update statistics
ANALYZE TABLE orders;

-- Persistent statistics (InnoDB)
ALTER TABLE orders STATS_PERSISTENT=1;
-- Stores statistics in mysql.innodb_table_stats and mysql.innodb_index_stats
```

#### Optimizer Trace

MySQL provides detailed optimizer trace:

```sql
-- Enable optimizer trace
SET optimizer_trace='enabled=on';

-- Run query
SELECT * FROM orders WHERE customer_id = 123;

-- View trace
SELECT * FROM information_schema.OPTIMIZER_TRACE\G

-- Output shows:
-- 1. Query parsing
-- 2. Possible paths considered
-- 3. Cost calculations for each path
-- 4. Path selected
-- 5. Reason for selection

-- Cleanup
SET optimizer_trace='enabled=off';
```

**Example trace analysis:**

```json
{
  "steps": [
    {
      "join_preparation": {
        "select_id": 1,
        "steps": [
          {
            "expanded_query": "SELECT * FROM orders WHERE customer_id = 123"
          }
        ]
      }
    },
    {
      "join_optimization": {
        "select_id": 1,
        "steps": [
          {
            "condition_processing": {
              "condition": "WHERE",
              "original_condition": "customer_id = 123",
              "steps": [
                {
                  "transformation": "equality_propagation",
                  "resulting_condition": "customer_id = 123"
                }
              ]
            }
          },
          {
            "table_dependencies": [
              {
                "table": "orders",
                "row_may_be_null": false,
                "map_bit": 0,
                "depends_on_map_bits": []
              }
            ]
          },
          {
            "rows_estimation": [
              {
                "table": "orders",
                "range_analysis": {
                  "table_scan": {
                    "rows": 100000,
                    "cost": 20500
                  },
                  "potential_range_indexes": [
                    {
                      "index": "idx_customer_id",
                      "usable": true,
                      "key_parts": ["customer_id"]
                    }
                  ],
                  "best_covering_index_scan": {
                    "index": "idx_customer_id",
                    "cost": 2050,
                    "chosen": true
                  },
                  "setup_range_conditions": [],
                  "group_index_range": {
                    "chosen": false,
                    "cause": "not_applicable"
                  },
                  "analyzing_range_alternatives": {
                    "range_scan_alternatives": [
                      {
                        "index": "idx_customer_id",
                        "ranges": ["123 <= customer_id <= 123"],
                        "index_dives_for_eq_ranges": true,
                        "rowid_ordered": false,
                        "using_mrr": false,
                        "index_only": false,
                        "rows": 10,
                        "cost": 12.01,
                        "chosen": true
                      }
                    ],
                    "analyzing_roworder_intersect": {
                      "cause": "too_few_roworder_scans"
                    }
                  },
                  "chosen_range_access_summary": {
                    "range_access_plan": {
                      "type": "range_scan",
                      "index": "idx_customer_id",
                      "rows": 10,
                      "ranges": ["123 <= customer_id <= 123"]
                    },
                    "rows_for_plan": 10,
                    "cost_for_plan": 12.01,
                    "chosen": true
                  }
                }
              }
            ]
          }
        ]
      }
    }
  ]
}
```

#### Join Optimization

```sql
-- MySQL join optimizer strategies
-- 1. Nested loop (always available)
-- 2. Block nested loop (buffered)
-- 3. Hash join (MySQL 8.0.18+)

-- Join order determination
SELECT * FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
WHERE o.order_date > '2024-01-01';

-- Optimizer considers:
-- 1. Table with most selective WHERE clause → Drive table
--    (orders with order_date filter)
-- 2. Join order based on costs
-- 3. Available indexes

-- View join order with EXPLAIN
EXPLAIN FORMAT=TREE
SELECT ...;

-- Force specific join order (if needed)
SELECT /*+ JOIN_ORDER(orders, customers, products) */ ...
```

#### Optimizer Hints

```sql
-- Index hints
SELECT * FROM orders USE INDEX (idx_customer_id)
WHERE customer_id = 123;

SELECT * FROM orders FORCE INDEX (idx_customer_id)
WHERE customer_id = 123;

SELECT * FROM orders IGNORE INDEX (idx_order_date)
WHERE order_date > '2024-01-01';

-- Join hints (MySQL 8.0+)
SELECT /*+ JOIN_ORDER(orders, customers) */
    o.*, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

-- Join algorithm hints
SELECT /*+ HASH_JOIN(orders, customers) */
    o.*, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

SELECT /*+ NO_HASH_JOIN(orders, customers) */
    o.*, c.customer_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id;

-- Subquery hints
SELECT /*+ SUBQUERY(sq) */ ...
FROM orders o
WHERE o.customer_id IN (
    SELECT /*+ QB_NAME(sq) */ customer_id 
    FROM customers 
    WHERE country = 'USA'
);

-- Index merge hints
SELECT /*+ NO_INDEX_MERGE(orders idx_date, idx_status) */
    *
FROM orders
WHERE order_date > '2024-01-01' OR status = 'pending';
```

### Optimizer Limitations and Workarounds

#### Problem 1: Correlated Column Statistics

```sql
-- Optimizer assumes column independence
SELECT * FROM products
WHERE category = 'Electronics' AND brand = 'Samsung';

-- Optimizer estimate:
-- 10% have category = 'Electronics'
-- 5% have brand = 'Samsung'
-- Estimate: 10% × 5% = 0.5%

-- Reality: All Samsung products are Electronics (100% correlation)
-- Actual: 5% (all Samsung products)

-- 10x underestimate!

-- Workaround 1: Extended statistics (PostgreSQL 10+)
CREATE STATISTICS products_category_brand_stats (dependencies)
ON category, brand FROM products;

ANALYZE products;

-- Workaround 2: Functional/expression index
CREATE INDEX idx_category_brand ON products(category, brand);
-- Cardinality of composite index shows true selectivity

-- Workaround 3: Denormalization
-- Add computed column with combined value
ALTER TABLE products ADD COLUMN category_brand VARCHAR(100)
    GENERATED ALWAYS AS (CONCAT(category, ':', brand)) STORED;
CREATE INDEX idx_category_brand_computed ON products(category_brand);
```

#### Problem 2: Skewed Data Distribution

```sql
-- Uniform distribution assumed
-- Table: users (1,000,000 rows)
-- Column: country
-- Most common value: 'USA' (900,000 rows - 90%)
-- Other values: 100 countries (100,000 rows - 10%)

SELECT * FROM users WHERE country = 'USA';
-- Optimizer estimate: 1,000,000 / 100 = 10,000 rows (assumes 100 distinct)
-- Reality: 900,000 rows

-- 90x underestimate!

-- Solution: Increase statistics_target (PostgreSQL)
ALTER TABLE users ALTER COLUMN country SET STATISTICS 1000;
ANALYZE users;
-- Now optimizer knows 'USA' is most common

-- MySQL: Persistent statistics capture skew automatically
ANALYZE TABLE users;
```

#### Problem 3: Function on Indexed Column

```sql
-- Index on created_at
CREATE INDEX idx_created ON orders(created_at);

-- This CANNOT use index:
SELECT * FROM orders WHERE YEAR(created_at) = 2024;

-- Optimizer sees function, can't use index
-- Full table scan

-- Solution 1: Rewrite without function
SELECT * FROM orders 
WHERE created_at >= '2024-01-01' 
  AND created_at < '2025-01-01';
-- Uses index!

-- Solution 2: Expression index (PostgreSQL)
CREATE INDEX idx_created_year ON orders((EXTRACT(YEAR FROM created_at)));
SELECT * FROM orders WHERE EXTRACT(YEAR FROM created_at) = 2024;
-- Uses expression index

-- Solution 3: Generated column (MySQL 5.7+)
ALTER TABLE orders ADD COLUMN created_year INT 
    GENERATED ALWAYS AS (YEAR(created_at)) STORED;
CREATE INDEX idx_created_year ON orders(created_year);
SELECT * FROM orders WHERE created_year = 2024;
-- Uses index on generated column
```

#### Problem 4: Implicit Type Conversion

```sql
-- Column: customer_id VARCHAR(50)
CREATE INDEX idx_customer_id ON orders(customer_id);

-- Query with integer
SELECT * FROM orders WHERE customer_id = 123;

-- MySQL: Converts customer_id to INT for comparison
-- Index on VARCHAR cannot be used efficiently
-- Table scan!

-- Solution: Use correct type
SELECT * FROM orders WHERE customer_id = '123';
-- Uses index

-- Better: Fix schema (if possible)
ALTER TABLE orders MODIFY customer_id BIGINT;
-- Now index works optimally
```

---

## Frequently Asked Questions

**Q1: Why does EXPLAIN show different costs for the same query on PostgreSQL vs MySQL?**

**A:** **Different cost models and units** - costs are not comparable across databases:

**PostgreSQL costs:**
```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;
-- Index Scan using idx_customer_id on orders  (cost=0.29..8.31 rows=10)
```

**MySQL costs:**
```sql
EXPLAIN SELECT * FROM orders WHERE customer_id = 123;
-- rows: 10, filtered: 100.00
-- (No cost shown in standard EXPLAIN, only in EXPLAIN FORMAT=JSON)
```

**Why different:**

1. **Different units:**
   - PostgreSQL: Arbitrary units based on page reads
   - MySQL: Different arbitrary units based on row reads

2. **Different components:**
   - PostgreSQL: Includes startup cost (0.29) and total cost (8.31)
   - MySQL: Doesn't show costs prominently in standard EXPLAIN

3. **Different calculations:**
   ```
   PostgreSQL:
   - seq_page_cost = 1.0
   - random_page_cost = 4.0
   - cpu_tuple_cost = 0.01
   
   MySQL:
   - io_block_read_cost = 1.0
   - memory_block_read_cost = 0.25
   - row_evaluate_cost = 0.1
   ```

4. **Only relative costs matter within same database:**
   ```sql
   -- PostgreSQL: Compare plans for SAME query
   EXPLAIN SELECT * FROM orders WHERE customer_id = 123;
   -- Plan A: cost=8.31
   
   EXPLAIN SELECT * FROM orders WHERE order_date = '2024-01-01';
   -- Plan B: cost=450.00
   
   -- Plan A is cheaper (8.31 < 450.00)
   ```

**Key point:** Focus on relative costs within the same database, not absolute values or cross-database comparisons.

---

**Q2: When should I use optimizer hints, and when should I fix the root cause?**

**A:**

**Use hints as temporary workarounds:**
```sql
-- Scenario: Production performance issue, need immediate fix
-- Query using wrong index
SELECT * FROM orders WHERE status = 'pending';
-- Using idx_order_date (wrong!)

-- Quick fix: Force correct index
SELECT /*+ INDEX(orders idx_status) */ 
    * 
FROM orders 
WHERE status = 'pending';
-- Now uses idx_status

-- But investigate WHY optimizer chose wrong index:
-- - Outdated statistics?
-- - Index not selective enough?
-- - Data skew?
```

**Fix root cause for permanent solution:**

1. **Outdated statistics:**
```sql
-- Problem: Statistics haven't been updated
-- Hint: SELECT /*+ INDEX(orders idx_status) */ ...

-- Root cause fix:
ANALYZE orders;  -- PostgreSQL
ANALYZE TABLE orders;  -- MySQL

-- Now optimizer chooses correct index without hint
SELECT * FROM orders WHERE status = 'pending';
```

2. **Poor index selectivity:**
```sql
-- Problem: status column has low cardinality (only 3 values)
-- Optimizer reluctant to use index

-- Hint: FORCE INDEX(idx_status)

-- Root cause fix: Composite index
CREATE INDEX idx_status_date ON orders(status, order_date);
-- More selective, optimizer uses it
```

3. **Wrong query structure:**
```sql
-- Problem: Subquery not being optimized
SELECT * FROM orders
WHERE customer_id IN (
    SELECT customer_id FROM customers WHERE country = 'USA'
);

-- Hint: /*+ SEMIJOIN(@subq1 FIRSTMATCH) */

-- Root cause fix: Rewrite query
SELECT o.* 
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE c.country = 'USA';
-- Clearer intent, easier to optimize
```

**Decision matrix:**

| Situation | Hint? | Fix Root Cause? |
|-----------|-------|-----------------|
| Production emergency | ✅ Yes | ⏰ Later |
| Optimizer bug (rare) | ✅ Yes | Report to vendor |
| Outdated statistics | ❌ No | ✅ ANALYZE |
| Poor schema design | ❌ No | ✅ Redesign |
| Query complexity | ❌ No | ✅ Rewrite |
| Testing different plans | ✅ Yes | n/a (exploration) |

**Hint risks:**
* Brittle - breaks when data changes
* Maintenance burden - what worked yesterday may not work tomorrow
* Hides real problems
* May become obsolete with optimizer improvements

**Best practice:**
```sql
-- If you use hints, document WHY
SELECT /*+ INDEX(orders idx_status) */ 
    * 
FROM orders 
WHERE status = 'pending';
-- TODO: Remove hint after ANALYZE scheduled
-- Reason: Statistics stale, optimizer chooses idx_order_date
-- Ticket: OPS-1234
```

---

**Q3: How do I know if my statistics are stale?**

**A:**

**PostgreSQL - Check statistics age:**

```sql
-- Table-level statistics age
SELECT 
    schemaname,
    relname,
    last_analyze,
    last_autoanalyze,
    n_live_tup,
    n_dead_tup,
    n_mod_since_analyze,
    CASE 
        WHEN last_analyze IS NULL THEN 'NEVER ANALYZED'
        WHEN last_analyze < NOW() - INTERVAL '7 days' THEN 'STALE (>7 days)'
        WHEN last_analyze < NOW() - INTERVAL '1 day' THEN 'OLD (>1 day)'
        ELSE 'RECENT'
    END AS status
FROM pg_stat_user_tables
ORDER BY n_mod_since_analyze DESC;

-- Example output:
-- relname  | last_analyze        | n_mod_since_analyze | status
-- orders   | 2023-12-01 10:00:00 | 1500000             | STALE (>7 days)
-- products | 2024-01-31 08:00:00 | 50                  | RECENT
```

**Red flags:**
* `n_mod_since_analyze` > 10% of table rows
* `last_analyze` > 1 week ago for frequently updated tables
* `last_analyze` is NULL (never analyzed)

**MySQL - Check statistics age:**

```sql
-- Table update time
SELECT 
    TABLE_NAME,
    TABLE_ROWS,
    UPDATE_TIME,
    TIMESTAMPDIFF(DAY, UPDATE_TIME, NOW()) AS days_since_update,
    CASE 
        WHEN UPDATE_TIME IS NULL THEN 'NEVER UPDATED'
        WHEN UPDATE_TIME < NOW() - INTERVAL 7 DAY THEN 'STALE (>7 days)'
        WHEN UPDATE_TIME < NOW() - INTERVAL 1 DAY THEN 'OLD (>1 day)'
        ELSE 'RECENT'
    END AS status
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'your_database'
ORDER BY UPDATE_TIME ASC;

-- Check if persistent stats enabled
SELECT 
    TABLE_NAME,
    CREATE_OPTIONS
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'your_database'
  AND TABLE_NAME = 'orders';
-- Should show: stats_persistent=1
```

**Detect stale statistics from query performance:**

```sql
-- Compare estimated vs actual rows (PostgreSQL)
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';

-- Look for large discrepancies:
-- (cost=... rows=100 ...)          ← Estimated
-- (actual time=... rows=50000 ...)  ← Actual

-- 500x difference! Statistics are stale
```

**Automated monitoring:**

```sql
-- PostgreSQL: Query for tables needing ANALYZE
SELECT 
    schemaname,
    relname,
    n_live_tup,
    n_mod_since_analyze,
    ROUND(100.0 * n_mod_since_analyze / NULLIF(n_live_tup, 0), 2) AS pct_modified
FROM pg_stat_user_tables
WHERE n_mod_since_analyze > GREATEST(50, n_live_tup * 0.1)  -- > 10% modified
ORDER BY n_mod_since_analyze DESC;

-- MySQL: Find tables not analyzed recently
SELECT 
    TABLE_NAME,
    TABLE_ROWS,
    UPDATE_TIME
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'your_database'
  AND (UPDATE_TIME IS NULL OR UPDATE_TIME < NOW() - INTERVAL 7 DAY)
  AND TABLE_ROWS > 1000
ORDER BY TABLE_ROWS DESC;
```

**Fix stale statistics:**

```sql
-- PostgreSQL: Manual analyze
ANALYZE orders;  -- Single table
ANALYZE;         -- All tables

-- Increase autovacuum frequency for busy tables
ALTER TABLE orders SET (autovacuum_analyze_scale_factor = 0.05);
-- Triggers when 5% of rows modified (default: 10%)

-- MySQL: Manual analyze
ANALYZE TABLE orders;

-- Enable persistent statistics
ALTER TABLE orders STATS_PERSISTENT=1, STATS_AUTO_RECALC=1;
```

---

## Interview Questions

**Question 1: Explain how the optimizer chooses between a sequential scan and index scan. Walk through the cost calculation.**

**Difficulty:** Senior

**Answer:**

**Cost comparison determines the choice:**

**Sequential Scan Cost:**

```sql
-- Formula:
-- cost = (pages × seq_page_cost) + (rows × cpu_tuple_cost)

-- Example table:
-- - Total pages: 1000
-- - Total rows: 100,000
-- - seq_page_cost: 1.0
-- - cpu_tuple_cost: 0.01

-- Sequential scan:
-- cost = (1000 × 1.0) + (100,000 × 0.01)
--      = 1000 + 1000
--      = 2000

-- Reads entire table sequentially
-- Processes every row
```

**Index Scan Cost:**

```sql
-- Formula:
-- cost = (index_pages_read × random_page_cost) +
--        (table_pages_read × random_page_cost) +
--        (rows_fetched × cpu_tuple_cost) +
--        (index_tuples_processed × cpu_index_tuple_cost)

-- Example with 10% selectivity (10,000 rows returned):
-- - Index depth: 3 levels
-- - Index pages to read: 50 (including B-tree traversal)
-- - Table pages to read: 100 (random access to heap)
-- - random_page_cost: 4.0
-- - cpu_tuple_cost: 0.01
-- - cpu_index_tuple_cost: 0.005

-- Index scan:
-- cost = (50 × 4.0) + (100 × 4.0) + (10,000 × 0.01) + (10,000 × 0.005)
--      = 200 + 400 + 100 + 50
--      = 750

-- Optimizer chooses index scan (750 < 2000)
```

**Break-even point:**

```sql
-- When does index scan become more expensive than seq scan?

-- Depends on selectivity:
-- Low selectivity (< 10-15%): Index scan cheaper
-- High selectivity (> 10-15%): Sequential scan cheaper

-- Example calculations:

-- 1% selectivity (1,000 rows):
-- Index: (10 × 4.0) + (10 × 4.0) + (1,000 × 0.01) + (1,000 × 0.005)
--      = 40 + 40 + 10 + 5 = 95
-- Sequential: 2000
-- Winner: Index (95 < 2000) ✓

-- 50% selectivity (50,000 rows):
-- Index: (50 × 4.0) + (500 × 4.0) + (50,000 × 0.01) + (50,000 × 0.005)
--      = 200 + 2000 + 500 + 250 = 2950
-- Sequential: 2000
-- Winner: Sequential (2000 < 2950) ✓

-- 10% selectivity (10,000 rows):
-- Index: 750 (calculated above)
-- Sequential: 2000
-- Winner: Index (750 < 2000) ✓

-- 20% selectivity (20,000 rows):
-- Index: (50 × 4.0) + (200 × 4.0) + (20,000 × 0.01) + (20,000 × 0.005)
--      = 200 + 800 + 200 + 100 = 1300
-- Sequential: 2000
-- Winner: Index (1300 < 2000) ✓

-- Break-even around 30-40% selectivity (depends on random_page_cost)
```

**Factors affecting the decision:**

**1. random_page_cost setting:**
```sql
-- HDD (slow random access):
-- random_page_cost = 4.0 (default)
-- Favors sequential scans

-- SSD (fast random access):
-- random_page_cost = 1.1
-- More willing to use index scans

-- Cost comparison with SSD:
-- Index scan (10% selectivity):
-- cost = (50 × 1.1) + (100 × 1.1) + (10,000 × 0.01) + (10,000 × 0.005)
--      = 55 + 110 + 100 + 50 = 315
-- Sequential: 2000
-- Even better for index! (315 < 750)
```

**2. Index correlation:**
```sql
-- Correlation: -1 to +1
-- +1: Perfect correlation (physical order = logical order)
-- 0: No correlation (random order)
-- -1: Perfect reverse correlation

-- High correlation (sequential I/O even with index):
-- Table physically sorted by indexed column
-- Index scan reads pages sequentially
-- Cheaper than random I/O

-- Low correlation (random I/O):
-- Table physically unsorted
-- Index scan jumps around
-- More expensive
```

**3. Cached pages:**
```sql
-- If table already in buffer cache:
-- Sequential scan: All pages in memory, very cheap
-- Index scan: Random access in memory, also cheap

-- Optimizer may prefer sequential scan for simplicity
-- Even if both would be cached
```

**Real-world example:**

```sql
-- Query
SELECT * FROM orders WHERE customer_id = 123;

-- Statistics:
-- orders: 1,000,000 rows, 10,000 pages
-- customer_id: 10,000 distinct values (average 100 rows per customer)
-- Index on customer_id: 3 levels, 100 pages

-- Expected selectivity: 100 / 1,000,000 = 0.01% (very selective!)

-- Index scan cost:
-- Index pages: ~10 (traverse + read entries)
-- Table pages: ~5 (100 rows likely on ~5 pages if clustered)
-- cost = (10 × 4.0) + (5 × 4.0) + (100 × 0.01) + (100 × 0.005)
--      = 40 + 20 + 1 + 0.5 = 61.5

-- Sequential scan cost:
-- cost = (10,000 × 1.0) + (1,000,000 × 0.01)
--      = 10,000 + 10,000 = 20,000

-- Optimizer chooses index scan (61.5 << 20,000)
-- 325x cheaper!
```

**Why This Matters:** Understanding cost calculations reveals why the optimizer makes certain choices and how to influence those choices through configuration, statistics, and query rewriting.

**Follow-up Questions:**
* How does the optimizer know how many table pages will be read for an index scan?
* What happens if the index scan estimate is wrong?
* How would you tune costs for cloud storage with different I/O characteristics?

---

**Question 2: A complex 5-table join is very slow. EXPLAIN shows the optimizer chose a bad join order. How do you diagnose why and fix it?**

**Difficulty:** Senior

**Answer:**

**Systematic diagnosis approach:**

**Step 1: Analyze the current plan**

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
JOIN suppliers s ON p.supplier_id = s.supplier_id
WHERE o.order_date >= '2024-01-01'
  AND c.country = 'USA'
  AND cat.category_name = 'Electronics';

-- Current plan (BAD):
-- Hash Join  (cost=... rows=1000000 ...)  (actual rows=10 ...)
--   -> Seq Scan on orders (rows=1000000)  (actual rows=1000000)
--   -> Hash Join
--        -> Seq Scan on products (rows=50000)  (actual rows=50000)
--        -> Hash Join
--             -> Seq Scan on categories (rows=100)  (actual rows=1)
--                   Filter: category_name = 'Electronics'
--                   Rows Removed by Filter: 99
--             -> Hash
--                  -> Seq Scan on suppliers
--
-- Execution Time: 5000 ms
```

**Problems identified:**

1. **Wrong join order**: Started with largest tables (orders, products)
2. **No early filtering**: categories filter applied last
3. **Cardinality misestimate**: Expected 1M rows, got 10

**Step 2: Check statistics**

```sql
-- Check last analyze time
SELECT 
    schemaname,
    relname,
    last_analyze,
    n_live_tup,
    n_mod_since_analyze
FROM pg_stat_user_tables
WHERE relname IN ('orders', 'customers', 'products', 'categories', 'suppliers');

-- Example output:
-- relname    | last_analyze        | n_live_tup | n_mod_since_analyze
-- orders     | 2023-06-01 00:00:00 | 1000000    | 500000   ← STALE!
-- categories | 2024-01-31 00:00:00 | 100        | 0        ← Fresh
-- customers  | 2024-01-15 00:00:00 | 100000     | 5000     ← Okay

-- Fix: Update statistics
ANALYZE orders;
ANALYZE customers;
ANALYZE products;
```

**Step 3: Check filter selectivity**

```sql
-- Test each filter's selectivity
SELECT COUNT(*) FROM orders WHERE order_date >= '2024-01-01';
-- Result: 50,000 rows (5% of table)

SELECT COUNT(*) FROM customers WHERE country = 'USA';
-- Result: 30,000 rows (30% of table)

SELECT COUNT(*) FROM categories WHERE category_name = 'Electronics';
-- Result: 1 row (1% of table) ← Most selective!

-- Most selective filter should drive the query
```

**Step 4: Verify index availability**

```sql
-- Check indexes
SELECT 
    tablename,
    indexname,
    indexdef
FROM pg_indexes
WHERE tablename IN ('orders', 'customers', 'products', 'categories');

-- Missing indexes:
-- ✗ No index on orders.order_date
-- ✗ No index on customers.country
-- ✗ No index on categories.category_name
-- ✓ Index on products.category_id (FK)
-- ✓ Index on products.supplier_id (FK)

-- Add missing indexes
CREATE INDEX idx_orders_date ON orders(order_date);
CREATE INDEX idx_customers_country ON customers(country);
CREATE INDEX idx_categories_name ON categories(category_name);
```

**Step 5: Re-run with updated statistics and indexes**

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN products p ON o.product_id = p.product_id
JOIN categories cat ON p.category_id = cat.category_id
JOIN suppliers s ON p.supplier_id = s.supplier_id
WHERE o.order_date >= '2024-01-01'
  AND c.country = 'USA'
  AND cat.category_name = 'Electronics';

-- Better plan:
-- Nested Loop  (cost=... rows=10 ...)  (actual rows=10 ...)
--   -> Index Scan on categories  (rows=1)  (actual rows=1)  ← Start here!
--         Index Cond: category_name = 'Electronics'
--   -> Nested Loop
--        -> Index Scan on products  (rows=500)  (actual rows=450)
--              Index Cond: category_id = cat.category_id
--        -> Nested Loop
--             -> Index Scan on orders  (rows=50000)  (actual rows=5000)
--                   Index Cond: order_date >= '2024-01-01'
--             -> Index Scan on customers  (rows=1)
--                   Index Cond: customer_id = o.customer_id
--                   Filter: country = 'USA'
--
-- Execution Time: 50 ms  ← 100x faster!
```

**Why the better plan is faster:**

```
Old plan:
1. Scan all orders (1M rows)
2. Scan all products (50K rows)
3. Join orders × products = huge intermediate result
4. Filter categories (too late!)
5. Total rows processed: 1M + 50K + huge cartesian product

New plan:
1. Find 'Electronics' category (1 row) ← Very selective
2. Find products in that category (450 rows) ← Still small
3. Find orders for those products after 2024-01-01 (5,000 rows)
4. Filter customers in USA
5. Total rows processed: 1 + 450 + 5,000 ≈ 5,451 rows
```

**Step 6: If optimizer still chooses badly, investigate why**

```sql
-- Possible reasons:

-- 1. Correlation underestimated
SELECT 
    attname,
    n_distinct,
    correlation
FROM pg_stats
WHERE tablename = 'products' AND attname = 'category_id';
-- If correlation is low but reality is high → bad estimate

-- 2. Functional dependency not recognized
-- All Electronics products from supplier_id = 5
-- Optimizer doesn't know category → supplier relationship

-- Solution: Extended statistics (PostgreSQL 10+)
CREATE STATISTICS products_category_supplier_stats (dependencies)
ON category_id, supplier_id FROM products;

ANALYZE products;

-- 3. Complex join graph confuses optimizer
-- Too many join paths to evaluate
-- May fall back to heuristic join order

-- Solution: Simplify query
-- Break into CTEs to guide optimizer:
WITH electronics_products AS (
    SELECT p.*
    FROM products p
    JOIN categories c ON p.category_id = c.category_id
    WHERE c.category_name = 'Electronics'
),
recent_orders AS (
    SELECT *
    FROM orders
    WHERE order_date >= '2024-01-01'
)
SELECT *
FROM recent_orders o
JOIN electronics_products p ON o.product_id = p.product_id
JOIN customers c ON o.customer_id = c.customer_id
JOIN suppliers s ON p.supplier_id = s.supplier_id
WHERE c.country = 'USA';
```

**Step 7: Last resort - query hints**

```sql
-- If all else fails, force join order
-- PostgreSQL: Use LATERAL or CTEs to force order
-- MySQL: Use JOIN_ORDER hint

SELECT /*+ JOIN_ORDER(categories, products, orders, customers, suppliers) */
    *
FROM categories cat
JOIN products p ON p.category_id = cat.category_id
JOIN orders o ON o.product_id = p.product_id
JOIN customers c ON o.customer_id = c.customer_id
JOIN suppliers s ON p.supplier_id = s.supplier_id
WHERE cat.category_name = 'Electronics'
  AND o.order_date >= '2024-01-01'
  AND c.country = 'USA';
```

**Summary checklist:**
- ✅ Update statistics (ANALYZE)
- ✅ Add indexes on filter columns
- ✅ Add indexes on join columns
- ✅ Verify selectivity estimates
- ✅ Check for correlated columns
- ✅ Simplify with CTEs if needed
- ⚠️ Use hints only as last resort

**Why This Matters:** Complex join optimization failures are common in production. Systematic diagnosis identifies root causes rather than masking with hints.

**Follow-up Questions:**
* How would you monitor for join order problems in production?
* When would you use materialized views instead of fixing the query?
* How do you balance index creation vs maintenance costs?

---

## Key Takeaways

* **Cost-based optimization** uses statistics and cost models to choose execution plans
* **Statistics are critical** - outdated stats cause bad plans (run ANALYZE regularly)
* **Costs are relative** - only compare within same database, not across databases
* **Selectivity drives join order** - most selective filters should execute first
* **Index vs scan decision** depends on selectivity, correlation, and storage type
* **Optimizer has limitations** - correlated columns, data skew, complex joins
* **Extended statistics** help with correlated columns (PostgreSQL 10+)
* **Hints are workarounds** - fix root cause (statistics, indexes, query structure)
* **Monitor estimate accuracy** - large differences between estimated/actual rows indicate problems
* **GEQO handles complex joins** - genetic algorithm for many-table joins

---

## 2.5 Index Internals

Indexes are the workhorses of database performance. Understanding their internal structure explains why they work and when they don't.

### B-Tree Index Structure

B-Tree (Balanced Tree) is the most common index type in both MySQL and PostgreSQL.

```
B-Tree Structure (simplified 3-level tree):

                    Root Node
                   ┌─────────┐
                   │  50 │100│
                   └──┬───┬──┘
           ┌──────────┘   └──────────┐
           │                          │
      Branch Node                Branch Node
     ┌────┬────┐                ┌────┬────┐
     │ 25 │ 35 │                │ 75 │ 90 │
     └─┬──┬──┬─┘                └─┬──┬──┬─┘
   ┌───┘  │  └───┐          ┌────┘  │  └────┐
   │      │      │          │       │       │
Leaf   Leaf   Leaf       Leaf    Leaf    Leaf
┌──┐   ┌──┐   ┌──┐      ┌──┐    ┌──┐    ┌──┐
│10│   │30│   │40│      │60│    │80│    │95│
│15│   │31│   │45│      │65│    │85│    │96│
│20│   │32│   │48│      │70│    │86│    │97│
└──┘   └──┘   └──┘      └──┘    └──┘    └──┘
 │      │      │         │       │       │
 ↓      ↓      ↓         ↓       ↓       ↓
Row    Row    Row       Row     Row     Row
Pointers
```

**Properties:**
- Balanced: All leaf nodes at same depth
- Sorted: Keys in order left to right
- Fast lookups: O(log N)
- Range scans: Follow leaf pointers
- Insert/Delete: Self-balancing

### InnoDB Clustered vs Secondary Indexes

**Clustered Index (Primary Key):**

```
InnoDB Clustered Index (Primary Key = user_id):

Root/Branch Pages contain: [PK values + page pointers]

Leaf Pages contain: [PK + ALL ROW DATA]
┌──────────────────────────────────────┐
│ Leaf Page 1                          │
│ PK:1 → [user_id:1, name:Alice, ...]  │
│ PK:2 → [user_id:2, name:Bob, ...]    │
│ PK:3 → [user_id:3, name:Carol, ...]  │
└──────────────────────────────────────┘
         ↓ (next leaf pointer)
┌──────────────────────────────────────┐
│ Leaf Page 2                          │
│ PK:4 → [user_id:4, name:Dave, ...]   │
│ PK:5 → [user_id:5, name:Eve, ...]    │
└──────────────────────────────────────┘
```

**Secondary Index:**

```
InnoDB Secondary Index (email):

Leaf Pages contain: [indexed column + PK value]
┌───────────────────────────────┐
│ Leaf Page 1                   │
│ alice@example.com → PK:1      │
│ bob@example.com → PK:2        │
│ carol@example.com → PK:3      │
└───────────────────────────────┘

Lookup process:
1. Search secondary index for email
2. Find PK value (e.g., PK:1)
3. Search clustered index for PK:1
4. Retrieve full row
```

**Covering index optimization:**

```sql
-- Without covering index (two lookups):
CREATE INDEX idx_email ON users(email);

SELECT user_id, name FROM users WHERE email = 'alice@example.com';
-- 1. idx_email: alice@example.com → PK:1
-- 2. Clustered index: PK:1 → Full row

-- With covering index (one lookup):
CREATE INDEX idx_email_covering ON users(email, user_id, name);

SELECT user_id, name FROM users WHERE email = 'alice@example.com';
-- 1. idx_email_covering: alice@example.com → (PK:1, name:Alice)
-- Done! No second lookup needed
```

### PostgreSQL Heap + Index Structure

```
PostgreSQL Index Structure:

Index (B-Tree on user_id):
┌─────────────────────┐
│ user_id: 1 → CTID(0,1) │  ← Points to heap tuple
│ user_id: 2 → CTID(0,2) │
│ user_id: 5 → CTID(1,1) │
└─────────────────────┘
         ↓
Heap Table (Physical storage):
┌─────────────────────────────────┐
│ Page 0                          │
│ (0,1): [user_id:1, name:Alice]  │
│ (0,2): [user_id:2, name:Bob]    │
│ (0,3): [user_id:3, name:Carol] (deleted)│
└─────────────────────────────────┘
┌─────────────────────────────────┐
│ Page 1                          │
│ (1,1): [user_id:5, name:Eve]    │
│ (1,2): [user_id:4, name:Dave]   │
└─────────────────────────────────┘
```

**Key differences from InnoDB:**
* All indexes store CTID (page, tuple offset)
* Heap table is unordered (insertion order)
* All indexes have same lookup cost (unlike InnoDB clustered vs secondary)
* Dead tuples remain until VACUUM

### Index Types

#### B-Tree Index (Default)

```sql
-- PostgreSQL
CREATE INDEX idx_btree ON users(user_id);

-- MySQL (InnoDB)
CREATE INDEX idx_btree ON users(user_id);

-- Best for:
-- - Equality (=)
-- - Ranges (<, >, BETWEEN)
-- - Sorting (ORDER BY)
-- - LIKE 'prefix%' (prefix search)

-- Examples:
SELECT * FROM users WHERE user_id = 5;           -- ✓ Uses index
SELECT * FROM users WHERE user_id > 100;         -- ✓ Uses index
SELECT * FROM users WHERE user_id BETWEEN 10 AND 20;  -- ✓ Uses index
SELECT * FROM users WHERE name LIKE 'Ali%';      -- ✓ Uses index (if indexed)
SELECT * FROM users WHERE name LIKE '%ice';      -- ✗ Cannot use index
```

#### Hash Index

```sql
-- PostgreSQL only (InnoDB doesn't support hash indexes)
CREATE INDEX idx_hash ON users USING HASH (email);

-- Best for:
-- - Exact equality only (=)
-- - Fast lookups O(1) average case

-- Cannot be used for:
-- - Ranges (<, >, BETWEEN)
-- - Sorting (ORDER BY)
-- - Prefix search (LIKE 'prefix%')

-- Examples:
SELECT * FROM users WHERE email = 'alice@example.com';  -- ✓ Uses hash index
SELECT * FROM users WHERE email > 'alice@example.com';  -- ✗ Cannot use hash index
```

#### GiST Index (Generalized Search Tree)

```sql
-- PostgreSQL only
CREATE INDEX idx_gist ON documents USING GIST (document_vector);

-- Best for:
-- - Full-text search
-- - Geometric data
-- - Custom data types
-- - Range types

-- Examples:
-- Full-text search
CREATE INDEX idx_fts ON documents USING GIST (to_tsvector('english', content));

SELECT * FROM documents 
WHERE to_tsvector('english', content) @@ to_tsquery('database & performance');

-- Geometric search
CREATE INDEX idx_location ON places USING GIST (location);

SELECT * FROM places 
WHERE location && circle(point(0,0), 10);  -- Within circle
```

#### GIN Index (Generalized Inverted Index)

```sql
-- PostgreSQL only
CREATE INDEX idx_gin ON documents USING GIN (to_tsvector('english', content));

-- Best for:
-- - Full-text search (better than GiST for static data)
-- - Array contains queries
-- - JSONB queries

-- Examples:
-- Full-text search
SELECT * FROM documents 
WHERE to_tsvector('english', content) @@ to_tsquery('database & performance');

-- Array contains
CREATE INDEX idx_tags ON posts USING GIN (tags);

SELECT * FROM posts WHERE tags @> ARRAY['postgresql', 'performance'];

-- JSONB queries
CREATE INDEX idx_json ON users USING GIN (metadata);

SELECT * FROM users WHERE metadata @> '{"country": "USA"}';
```

#### Partial Index

```sql
-- Index only subset of rows
-- Smaller index, faster lookups for common queries

-- PostgreSQL
CREATE INDEX idx_active_users ON users(last_login) 
WHERE status = 'active';

-- Index only active users (e.g., 10% of table)
-- 10x smaller index!

SELECT * FROM users 
WHERE status = 'active' AND last_login > '2024-01-01';
-- Uses partial index

SELECT * FROM users 
WHERE status = 'inactive' AND last_login > '2024-01-01';
-- Cannot use partial index (status != 'active')

-- MySQL (functional approximation with virtual column)
ALTER TABLE users ADD COLUMN active_last_login DATETIME 
    AS (CASE WHEN status = 'active' THEN last_login END) STORED;
    
CREATE INDEX idx_active_login ON users(active_last_login);
```

#### Expression Index

```sql
-- Index on computed expression
-- PostgreSQL
CREATE INDEX idx_lower_email ON users(LOWER(email));

SELECT * FROM users WHERE LOWER(email) = 'alice@example.com';
-- Uses expression index

-- MySQL (8.0+)
CREATE INDEX idx_lower_email ON users((LOWER(email)));

SELECT * FROM users WHERE LOWER(email) = 'alice@example.com';
-- Uses expression index
```

### Index Maintenance

#### Index Bloat

```sql
-- PostgreSQL: Check index bloat
CREATE EXTENSION pgstattuple;

SELECT 
    schemaname,
    tablename,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    pg_size_pretty(pg_relation_size(relid)) AS table_size,
    round(100.0 * pgstatindex(indexrelid)::numeric / NULLIF(pg_relation_size(indexrelid), 0), 2) AS bloat_pct
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY pg_relation_size(indexrelid) DESC;

-- High bloat_pct (> 20-30%) indicates bloat

-- Fix: REINDEX
REINDEX INDEX idx_bloated;  -- Single index
REINDEX TABLE users;         -- All table indexes
REINDEX DATABASE mydb;       -- All database indexes (CAUTION: locks tables)

-- MySQL: OPTIMIZE TABLE rebuilds indexes
OPTIMIZE TABLE users;
```

#### Duplicate/Redundant Indexes

```sql
-- Find duplicate indexes (PostgreSQL)
SELECT 
    pg_size_pretty(SUM(pg_relation_size(idx))::BIGINT) AS size,
    (array_agg(idx))[1] AS idx1,
    (array_agg(idx))[2] AS idx2,
    (array_agg(idx))[3] AS idx3,
    (array_agg(idx))[4] AS idx4
FROM (
    SELECT 
        indexrelid::regclass AS idx,
        (indrelid::text ||E'\n'|| indclass::text ||E'\n'|| indkey::text ||E'\n'||
         COALESCE(indexprs::text,'')||E'\n' || COALESCE(indpred::text,'')) AS key
    FROM pg_index
) sub
GROUP BY key 
HAVING COUNT(*) > 1
ORDER BY SUM(pg_relation_size(idx)) DESC;

-- Redundant indexes (covered by other indexes)
-- Example:
-- idx_a on (user_id)
-- idx_ab on (user_id, email)
-- idx_a is redundant! (idx_ab can handle user_id queries)

-- Drop redundant
DROP INDEX idx_a;
```

#### Unused Indexes

```sql
-- PostgreSQL: Find unused indexes
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE idx_scan = 0  -- Never used
  AND indexrelname !~ '^pg_'  -- Exclude system indexes
ORDER BY pg_relation_size(indexrelid) DESC;

-- MySQL: Find unused indexes
SELECT 
    OBJECT_SCHEMA,
    OBJECT_NAME,
    INDEX_NAME,
    COUNT_STAR,
    COUNT_READ,
    COUNT_FETCH
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE INDEX_NAME IS NOT NULL
  AND COUNT_STAR = 0  -- Never accessed
ORDER BY COUNT_STAR;

-- Drop unused (carefully!)
DROP INDEX idx_unused;
```

---

## 2.6 Transaction Management & MVCC

Transactions ensure data consistency. MVCC (Multi-Version Concurrency Control) allows high concurrency without locking readers.

### ACID Properties

```sql
-- Atomicity: All or nothing
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
COMMIT;  -- Both succeed or both fail

-- Consistency: Constraints maintained
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- If balance goes negative and CHECK constraint exists → ROLLBACK

-- Isolation: Concurrent transactions don't interfere
-- (See isolation levels below)

-- Durability: Committed changes survive crashes
COMMIT;  -- Changes written to WAL, survive crash
```

### MVCC Overview

```
Traditional Locking (No MVCC):
┌────────────────────────────────┐
│ Transaction 1: UPDATE row      │ ← Locks row
└────────────────────────────────┘
         ↓ (lock)
┌────────────────────────────────┐
│ Row: LOCKED                    │
└────────────────────────────────┘
         ↓ (blocked!)
┌────────────────────────────────┐
│ Transaction 2: SELECT row      │ ← Waits for lock
└────────────────────────────────┘

MVCC (Multi-Version):
┌────────────────────────────────┐
│ Transaction 1 (TXN 100): UPDATE│ ← Creates new version
└────────────────────────────────┘
         ↓
┌────────────────────────────────┐
│ Row v1 (created by TXN 90)     │ ← Old version
│ Row v2 (created by TXN 100)    │ ← New version
└────────────────────────────────┘
         ↓ (no blocking!)
┌────────────────────────────────┐
│ Transaction 2 (TXN 99): SELECT │ ← Reads old version (v1)
└────────────────────────────────┘
```

**Key principle:** Readers don't block writers, writers don't block readers

### PostgreSQL MVCC Implementation

**Tuple Headers:**

```
Each tuple (row) has metadata:
┌────────────────────────────────────┐
│ t_xmin: 100  (transaction that created this version) │
│ t_xmax: 0    (transaction that deleted/updated, 0=active) │
│ t_cid: 0     (command ID within transaction) │
│ t_ctid: (0,1) (physical location or next version) │
│ t_infomask: flags (committed, aborted, etc.) │
│ ... actual row data ...             │
└────────────────────────────────────┘
```

**Visibility Rules:**

```sql
-- Transaction 100 creates row:
INSERT INTO users VALUES (1, 'Alice');
-- Tuple: t_xmin=100, t_xmax=0

-- Transaction 200 updates row:
UPDATE users SET name = 'Alice Smith' WHERE user_id = 1;
-- Old tuple: t_xmin=100, t_xmax=200 (marked deleted by TXN 200)
-- New tuple: t_xmin=200, t_xmax=0

-- Transaction 150 (started after 100, before 200):
SELECT * FROM users WHERE user_id = 1;
-- Sees: t_xmin=100, t_xmax=200
-- Check: Is 100 committed and visible to 150? Yes
-- Check: Is 200 committed and visible to 150? No (200 > 150)
-- Result: Returns old version (Alice)

-- Transaction 250 (started after 200):
SELECT * FROM users WHERE user_id = 1;
-- Sees both versions
-- Old: t_xmin=100, t_xmax=200 (deleted by committed TXN 200)
-- New: t_xmin=200, t_xmax=0 (created by committed TXN 200)
-- Result: Returns new version (Alice Smith)
```

**Dead Tuples and VACUUM:**

```sql
-- Updates create dead tuples
UPDATE users SET name = 'New Name' WHERE user_id = 1;
-- Old tuple: Dead (not visible to any transaction)
-- New tuple: Live

-- Dead tuples accumulate
-- VACUUM removes them

-- Manual VACUUM
VACUUM users;  -- Remove dead tuples, update statistics
VACUUM FULL users;  -- Rebuild table, reclaim disk space (locks table!)

-- Autovacuum (automatic)
SHOW autovacuum;  -- ON (default)

-- Autovacuum triggers when:
-- dead_tuples > autovacuum_vacuum_threshold + 
--               (autovacuum_vacuum_scale_factor × table_rows)
-- Default: 50 + (0.2 × table_rows)

-- Monitor VACUUM
SELECT 
    schemaname,
    relname,
    n_dead_tup,
    n_live_tup,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;
```

### MySQL/InnoDB MVCC Implementation

**Row Format with Hidden Columns:**

```
Each row has hidden system columns:
┌─────────────────────────────────────┐
│ DB_TRX_ID: 100  (transaction ID that created/modified) │
│ DB_ROLL_PTR: pointer to undo log    │
│ DB_ROW_ID: auto row ID (if no PK)   │
│ ... actual row data ...              │
└─────────────────────────────────────┘
```

**Undo Logs:**

```
InnoDB maintains undo logs for old versions:

Current Row in B-Tree:
┌───────────────────────────────┐
│ user_id: 1                    │
│ name: 'Alice Smith'           │
│ DB_TRX_ID: 200                │
│ DB_ROLL_PTR: → Undo Log       │
└───────────────────────────────┘
                ↓
Undo Log (old versions):
┌───────────────────────────────┐
│ name: 'Alice'                 │
│ DB_TRX_ID: 100                │
│ DB_ROLL_PTR: → NULL (original)│
└───────────────────────────────┘
```

**Read View:**

```sql
-- Transaction starts
START TRANSACTION;

-- Creates Read View:
-- - trx_id: This transaction's ID (e.g., 150)
-- - up_limit_id: Oldest active transaction (e.g., 140)
-- - low_limit_id: Next transaction ID (e.g., 160)
-- - trx_ids: List of active transaction IDs

-- Visibility check for row with DB_TRX_ID=200:
-- Is 200 < 150? No → Row created after this transaction started
-- Check undo log for version visible to transaction 150

SELECT * FROM users WHERE user_id = 1;
-- Walks undo log chain to find version visible to TXN 150
```

**Purge:**

```sql
-- InnoDB purge thread removes old undo logs
-- When no transaction needs old versions

-- Monitor undo log growth
SHOW ENGINE INNODB STATUS\G

-- Look for:
-- History list length: 1000  (number of undo log entries)
-- High value indicates slow purge (long-running transactions blocking purge)

-- Find long-running transactions blocking purge
SELECT 
    trx_id,
    trx_started,
    TIMESTAMPDIFF(SECOND, trx_started, NOW()) AS runtime_seconds,
    trx_query
FROM information_schema.INNODB_TRX
ORDER BY trx_started;

-- Kill long transaction (carefully!)
KILL <thread_id>;
```

### Isolation Levels

```sql
-- Set isolation level
-- PostgreSQL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN;
...
COMMIT;

-- MySQL
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
...
COMMIT;

-- Or set globally
-- PostgreSQL
ALTER DATABASE mydb SET default_transaction_isolation = 'read committed';

-- MySQL
SET GLOBAL transaction_isolation = 'READ-COMMITTED';
```

#### READ UNCOMMITTED (Dirty Reads)

```sql
-- Transaction 1
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- Not yet committed

-- Transaction 2 (READ UNCOMMITTED)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 1;
-- Reads uncommitted change! (Dirty read)

-- Transaction 1
ROLLBACK;  -- Oops, rollback

-- Transaction 2 saw data that never existed!
-- ❌ Not safe for production
```

#### READ COMMITTED (Default in PostgreSQL)

```sql
-- Transaction 1
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;

-- Transaction 2 (READ COMMITTED)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 1;
-- Reads old committed value (before Transaction 1's update)

-- Transaction 1
COMMIT;

-- Transaction 2
SELECT balance FROM accounts WHERE account_id = 1;
-- Now reads new committed value
-- ⚠️ Non-repeatable read! (same query, different results)
```

#### REPEATABLE READ (Default in MySQL)

```sql
-- Transaction 1
START TRANSACTION;  -- TXN ID: 100
SELECT balance FROM accounts WHERE account_id = 1;
-- balance: 1000

-- Transaction 2
START TRANSACTION;
UPDATE accounts SET balance = 500 WHERE account_id = 1;
COMMIT;

-- Transaction 1
SELECT balance FROM accounts WHERE account_id = 1;
-- Still reads: 1000 (repeatable read!)
-- Reads snapshot from when transaction started

-- ⚠️ Phantom reads possible in PostgreSQL REPEATABLE READ
-- (Not in MySQL due to next-key locking)
```

#### SERIALIZABLE

```sql
-- Strictest isolation
-- Behaves as if transactions executed serially

-- PostgreSQL: Uses SSI (Serializable Snapshot Isolation)
-- Detects conflicts and aborts transactions

-- Transaction 1
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
SELECT SUM(balance) FROM accounts;
-- sum: 10000

-- Transaction 2
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
INSERT INTO accounts VALUES (100, 500);
COMMIT;

-- Transaction 1
SELECT SUM(balance) FROM accounts;
-- Still: 10000 (snapshot isolation)
COMMIT;  -- May fail with serialization error if conflict detected

-- MySQL: Uses locking (stricter, more blocking)
```

---

## 2.7 Locking & Concurrency Control

Locks coordinate concurrent access. Understanding locking prevents deadlocks and performance issues.

### PostgreSQL Locking

**Table-Level Locks:**

```sql
-- ACCESS SHARE (Lightest)
-- Acquired by: SELECT
-- Conflicts with: ACCESS EXCLUSIVE
SELECT * FROM users;

-- ROW SHARE
-- Acquired by: SELECT FOR UPDATE, SELECT FOR SHARE
-- Conflicts with: EXCLUSIVE, ACCESS EXCLUSIVE
SELECT * FROM users WHERE user_id = 1 FOR UPDATE;

-- ROW EXCLUSIVE
-- Acquired by: INSERT, UPDATE, DELETE
-- Conflicts with: SHARE, SHARE ROW EXCLUSIVE, EXCLUSIVE, ACCESS EXCLUSIVE
UPDATE users SET name = 'Alice' WHERE user_id = 1;

-- SHARE UPDATE EXCLUSIVE
-- Acquired by: VACUUM, ANALYZE, CREATE INDEX CONCURRENTLY
-- Conflicts with: SHARE UPDATE EXCLUSIVE, SHARE, EXCLUSIVE, ACCESS EXCLUSIVE
VACUUM users;

-- SHARE
-- Acquired by: CREATE INDEX (non-concurrent)
-- Conflicts with: ROW EXCLUSIVE, SHARE UPDATE EXCLUSIVE, SHARE, EXCLUSIVE, ACCESS EXCLUSIVE
CREATE INDEX idx_name ON users(name);

-- SHARE ROW EXCLUSIVE
-- Acquired by: CREATE TRIGGER
-- Conflicts with: ROW EXCLUSIVE, SHARE UPDATE EXCLUSIVE, SHARE, SHARE ROW EXCLUSIVE, EXCLUSIVE, ACCESS EXCLUSIVE

-- EXCLUSIVE
-- Acquired by: REFRESH MATERIALIZED VIEW CONCURRENTLY
-- Conflicts with: ROW SHARE, ROW EXCLUSIVE, SHARE UPDATE EXCLUSIVE, SHARE, SHARE ROW EXCLUSIVE, EXCLUSIVE, ACCESS EXCLUSIVE

-- ACCESS EXCLUSIVE (Heaviest)
-- Acquired by: DROP TABLE, TRUNCATE, REINDEX, VACUUM FULL
-- Conflicts with: ALL other locks
DROP TABLE users;
```

**Row-Level Locks:**

```sql
-- FOR UPDATE (Exclusive row lock)
BEGIN;
SELECT * FROM users WHERE user_id = 1 FOR UPDATE;
-- Locks row for update
-- Other transactions cannot UPDATE/DELETE this row
-- Other transactions can still SELECT (non-locking)

UPDATE users SET balance = balance - 100 WHERE user_id = 1;
COMMIT;

-- FOR NO KEY UPDATE (Weaker exclusive lock)
SELECT * FROM users WHERE user_id = 1 FOR NO KEY UPDATE;
-- Locks row but allows updates to foreign keys referencing this row

-- FOR SHARE (Shared row lock)
BEGIN;
SELECT * FROM users WHERE user_id = 1 FOR SHARE;
-- Multiple transactions can hold FOR SHARE
-- Prevents UPDATE/DELETE until all shared locks released

SELECT * FROM users WHERE user_id = 1;  -- Works (no lock)
UPDATE users SET name = 'Bob' WHERE user_id = 1;  -- Blocks!
COMMIT;

-- FOR KEY SHARE (Weakest)
SELECT * FROM users WHERE user_id = 1 FOR KEY SHARE;
-- Locks only the key (for foreign key checks)
-- Allows updates to non-key columns
```

**Advisory Locks:**

```sql
-- Application-level locking
-- Useful for distributed locking

-- Exclusive advisory lock
SELECT pg_advisory_lock(12345);  -- Blocks until lock acquired
-- ... do work ...
SELECT pg_advisory_unlock(12345);

-- Try lock (non-blocking)
SELECT pg_try_advisory_lock(12345);
-- Returns true if acquired, false if not

-- Use case: Ensure only one worker processes a job
BEGIN;
SELECT pg_try_advisory_lock(job_id);
-- If true: Process job
-- If false: Skip (another worker has it)
COMMIT;  -- Releases lock
```

**Monitoring Locks:**

```sql
-- Current locks
SELECT 
    locktype,
    database,
    relation::regclass,
    page,
    tuple,
    transactionid,
    mode,
    granted
FROM pg_locks
WHERE NOT granted  -- Show blocked locks
ORDER BY relation;

-- Blocking queries
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocked_activity.usename AS blocked_user,
    blocking_locks.pid AS blocking_pid,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_statement,
    blocking_activity.query AS blocking_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

### MySQL/InnoDB Locking

**Row-Level Locks:**

```sql
-- Shared lock (S)
SELECT * FROM users WHERE user_id = 1 LOCK IN SHARE MODE;
-- or (MySQL 8.0+)
SELECT * FROM users WHERE user_id = 1 FOR SHARE;

-- Exclusive lock (X)
SELECT * FROM users WHERE user_id = 1 FOR UPDATE;

-- Gap locks (prevent phantom reads)
-- Lock range of index values
SELECT * FROM users WHERE user_id BETWEEN 10 AND 20 FOR UPDATE;
-- Locks rows 10-20 AND gaps between them
-- Prevents INSERT of user_id = 15

-- Next-key locks (gap lock + record lock)
-- Default in REPEATABLE READ
-- Combination of record lock and gap lock before it
```

**Intention Locks:**

```
Table-level locks indicating intent to lock rows:
- IS (Intention Shared): Intent to acquire S locks on rows
- IX (Intention Exclusive): Intent to acquire X locks on rows

Lock compatibility:
       IS    IX    S     X
IS    ✓     ✓     ✓     ✗
IX    ✓     ✓     ✗     ✗
S     ✓     ✗     ✓     ✗
X     ✗     ✗     ✗     ✗
```

**Auto-increment Locks:**

```sql
-- innodb_autoinc_lock_mode settings
SHOW VARIABLES LIKE 'innodb_autoinc_lock_mode';

-- 0 (traditional): Table-level lock for all INSERTs
-- 1 (consecutive): Lightweight mutex for simple INSERTs
-- 2 (interleaved): No locks, fastest but non-consecutive IDs in mixed statements
```

**Monitoring Locks:**

```sql
-- Current locks
SELECT 
    ENGINE_LOCK_ID,
    ENGINE_TRANSACTION_ID,
    OBJECT_SCHEMA,
    OBJECT_NAME,
    LOCK_TYPE,
    LOCK_MODE,
    LOCK_STATUS,
    LOCK_DATA
FROM performance_schema.data_locks;

-- Blocking locks
SELECT 
    waiting_trx.trx_id AS waiting_trx_id,
    waiting_trx.trx_mysql_thread_id AS waiting_thread,
    waiting_trx.trx_query AS waiting_query,
    blocking_trx.trx_id AS blocking_trx_id,
    blocking_trx.trx_mysql_thread_id AS blocking_thread,
    blocking_trx.trx_query AS blocking_query
FROM information_schema.INNODB_LOCK_WAITS w
JOIN information_schema.INNODB_TRX waiting_trx 
    ON w.requesting_trx_id = waiting_trx.trx_id
JOIN information_schema.INNODB_TRX blocking_trx 
    ON w.blocking_trx_id = blocking_trx.trx_id;
```

### Deadlock Detection

**Deadlock Example:**

```sql
-- Transaction 1
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- Holds lock on account 1

-- Transaction 2
START TRANSACTION;
UPDATE accounts SET balance = balance + 50 WHERE account_id = 2;
-- Holds lock on account 2

-- Transaction 1
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
-- Waits for lock on account 2 (held by Transaction 2)

-- Transaction 2
UPDATE accounts SET balance = balance - 50 WHERE account_id = 1;
-- Waits for lock on account 1 (held by Transaction 1)

-- DEADLOCK! Both waiting for each other

-- Database detects deadlock, aborts one transaction:
-- ERROR: deadlock detected
```

**Deadlock Prevention:**

```sql
-- 1. Consistent lock order
-- Always acquire locks in same order (e.g., ascending account_id)

-- Transaction 1
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;  -- Lock 1 first
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;  -- Then lock 2

-- Transaction 2
UPDATE accounts SET balance = balance - 50 WHERE account_id = 1;   -- Lock 1 first
UPDATE accounts SET balance = balance + 50 WHERE account_id = 2;   -- Then lock 2
-- No deadlock! Both acquire locks in same order

-- 2. Use shorter transactions
BEGIN;
-- Quick operations only
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
COMMIT;  -- Release locks ASAP

-- 3. Use appropriate isolation level
-- READ COMMITTED may reduce deadlocks vs REPEATABLE READ
```

---

## 2.8 Write-Ahead Logging & Crash Recovery

WAL (Write-Ahead Logging) ensures durability and enables crash recovery.

### WAL Principle

```
Write-Ahead Logging Rule:
1. Changes written to WAL BEFORE data pages
2. WAL written to disk BEFORE commit returns
3. If crash: Replay WAL to recover

Timeline:
1. Transaction modifies row in buffer pool (NOT on disk yet)
2. Change logged to WAL (written to disk)
3. COMMIT issued
4. WAL flushed to disk (fsync)
5. COMMIT returns to client ✓
6. Data pages written to disk later (background)

Crash during step 6:
- WAL has all changes → Replay WAL → Recover
```

### PostgreSQL WAL

**WAL Files:**

```bash
# WAL files location
/var/lib/postgresql/data/pg_wal/

# Files:
000000010000000000000001
000000010000000000000002
000000010000000000000003

# Each file: 16MB (default)
# Sequential writes (fast)
```

**WAL Records:**

```
WAL Record Structure:
┌──────────────────────────────┐
│ LSN (Log Sequence Number)    │  ← Unique position in WAL
│ Transaction ID                │
│ Resource Manager (RM)         │  ← Type of operation
│ Operation (INSERT/UPDATE/etc) │
│ Table/Block info              │
│ Old data (for undo)           │
│ New data (for redo)           │
└──────────────────────────────┘
```

**Configuration:**

```sql
-- WAL level
SHOW wal_level;
-- minimal: Crash recovery only
-- replica: Supports replication
-- logical: Supports logical decoding

-- WAL buffer size
SHOW wal_buffers;  -- Default: 16MB
-- Increase for write-heavy workload

-- Checkpoint settings
SHOW checkpoint_timeout;  -- Default: 5min
SHOW max_wal_size;        -- Default: 1GB

-- Synchronous commit
SHOW synchronous_commit;  -- on/off/local/remote_write/remote_apply
-- on: Wait for WAL to disk (safest, slower)
-- off: Don't wait (faster, risk data loss on crash)
```

**WAL Archiving:**

```sql
-- Enable WAL archiving for point-in-time recovery
-- postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /var/lib/postgresql/wal_archive/%f'

-- %p = path to WAL file
-- %f = filename

-- Archived WAL files allow recovery to any point in time
```

**Crash Recovery Process:**

```
1. Database starts after crash
2. Reads last checkpoint from pg_control
3. Replays WAL from checkpoint to end
   - REDO: Apply committed changes
   - UNDO: Rollback uncommitted changes
4. Database ready for connections
```

### MySQL/InnoDB WAL (Redo Logs)

**Redo Log Files:**

```bash
# Redo log files location
/var/lib/mysql/

# Files:
ib_logfile0
ib_logfile1

# Circular: ib_logfile0 → ib_logfile1 → ib_logfile0 ...
```

**Configuration:**

```sql
-- Redo log file size
SHOW VARIABLES LIKE 'innodb_log_file_size';
-- Default: 48MB (MySQL 5.7), 2GB (MySQL 8.0)
-- Larger = fewer flushes, longer recovery

-- Redo log files
SHOW VARIABLES LIKE 'innodb_log_files_in_group';
-- Default: 2

-- Flush behavior
SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';
-- 0: Flush every second (fast, risk 1 second data loss)
-- 1: Flush at each commit (safe, slower)
-- 2: Flush to OS cache, OS flushes per second (middle ground)

-- For maximum durability:
SET GLOBAL innodb_flush_log_at_trx_commit = 1;

-- For maximum performance (at risk):
SET GLOBAL innodb_flush_log_at_trx_commit = 2;
```

**Crash Recovery:**

```
1. InnoDB starts after crash
2. Reads redo logs from last checkpoint
3. Replays redo logs
   - Applies committed transactions
   - Rolls back uncommitted transactions (using undo logs)
4. Database ready

Recovery time depends on:
- Redo log size
- Amount of data to replay
- innodb_log_file_size
```

**Monitoring:**

```sql
-- Redo log usage
SHOW ENGINE INNODB STATUS\G

-- Look for:
-- Log sequence number: 1234567890
-- Log flushed up to: 1234567800
-- Last checkpoint at: 1234560000

-- Checkpoint age = Log sequence number - Last checkpoint
-- High checkpoint age → Need larger redo logs
```

### Checkpoints

**Purpose:** Mark point where all changes before this are on disk

```
Without checkpoint:
Crash → Replay entire WAL from beginning (slow!)

With checkpoint:
Crash → Replay WAL from last checkpoint only (faster!)

Checkpoint Process:
1. Flush all dirty pages to disk
2. Write checkpoint record to WAL
3. Update control file with checkpoint LSN
```

**PostgreSQL Checkpoints:**

```sql
-- Trigger checkpoint manually
CHECKPOINT;

-- Checkpoint settings
SHOW checkpoint_timeout;           -- Max time between checkpoints
SHOW checkpoint_completion_target; -- Spread checkpoint I/O over this fraction
SHOW checkpoint_warning;           -- Warn if checkpoints too frequent

-- Example:
-- checkpoint_timeout = 5min
-- checkpoint_completion_target = 0.9
-- Checkpoint spread over: 5 × 0.9 = 4.5 minutes

-- Monitor checkpoints
SELECT * FROM pg_stat_bgwriter;
-- checkpoint_timed: Checkpoints due to timeout (good)
-- checkpoint_req: Checkpoints due to WAL size (may need tuning)
```

**MySQL Checkpoints:**

```sql
-- InnoDB does fuzzy checkpointing (continuous, not blocking)
-- No manual checkpoint command

-- Checkpoint rate controlled by:
SHOW VARIABLES LIKE 'innodb_max_dirty_pages_pct';  -- Trigger when this % dirty
SHOW VARIABLES LIKE 'innodb_io_capacity';          -- I/O capacity for flushing

-- Increase for SSD:
SET GLOBAL innodb_io_capacity = 2000;
SET GLOBAL innodb_io_capacity_max = 4000;
```

---

## Key Takeaways

* **B-Tree indexes** provide O(log N) lookups and ordered scans
* **InnoDB clustered indexes** store data in index leaf pages (primary key order)
* **PostgreSQL indexes** store CTID pointers to heap (all indexes equal cost)
* **MVCC enables high concurrency** - readers don't block writers
* **Isolation levels** trade consistency for performance
* **VACUUM (PostgreSQL) removes dead tuples** - critical for performance
* **Purge (InnoDB) removes old undo logs** - long transactions block purge
* **Locks coordinate concurrent access** - understand table and row locks
* **Deadlocks occur with circular waits** - prevent with consistent lock order
* **WAL ensures durability** - changes logged before commit
* **Checkpoints reduce recovery time** - mark consistent database state

---

*This concludes Part 2: Database Internals & System Architecture*
