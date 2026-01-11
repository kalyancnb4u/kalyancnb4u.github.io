---
title: "SQL Mastery Part 1: SQL Fundamentals & Command Categories"
date: 2024-02-02 00:00:00 +0530
categories: [SQL, SQL Mastery]
tags: [SQL, Database, PostgreSQL, MySQL, Performance, Optimization, Internals, Indexing, Transactions, DDL, DML, DCL, TCL]
---

# Complete SQL Mastery Part 1: SQL Fundamentals & Command Categories

## Introduction

Structured Query Language (SQL) is the universal language of relational databases. Whether you're building web applications, analyzing data, or managing enterprise systems, SQL is the foundation that connects your application code to persistent data storage. This comprehensive guide will take you from fundamental concepts through production-grade expertise, covering both MySQL and PostgreSQL—the two most popular open-source relational database management systems.

This is Part 1 of a six-part series designed to transform you into a database expert. In this post, we'll cover SQL fundamentals and all command categories: DDL (Data Definition Language), DML (Data Manipulation Language), DQL (Data Query Language), DCL (Data Control Language), and TCL (Transaction Control Language). By the end, you'll understand not just the syntax, but the underlying mechanics and best practices that separate junior developers from senior database architects.

---

## 1.1 Introduction to SQL and Databases

### What is SQL and Why Does It Exist?

SQL (Structured Query Language) is a declarative language designed specifically for managing and querying relational databases. Unlike imperative programming languages where you specify *how* to accomplish a task step-by-step, SQL allows you to specify *what* you want, and the database engine figures out the optimal way to retrieve or manipulate the data.

**Why SQL exists:**

1. **Data Abstraction**: SQL provides a high-level interface that hides the complexity of physical data storage, indexing, and retrieval
2. **Standardization**: The SQL standard (ANSI/ISO SQL) ensures that core concepts work across different database systems
3. **Optimization**: Database engines can optimize SQL queries automatically based on statistics, indexes, and query patterns
4. **Data Integrity**: SQL enforces constraints, transactions, and relationships to maintain data consistency
5. **Concurrent Access**: SQL databases handle multiple users accessing and modifying data simultaneously

### The Relational Database Model

The relational model, introduced by Edgar F. Codd in 1970, organizes data into tables (relations) consisting of rows (tuples) and columns (attributes). This model provides several critical advantages:

**Key Concepts:**

* **Tables**: Named collections of related data organized in rows and columns
* **Rows**: Individual records containing data for a single entity
* **Columns**: Attributes or fields that describe properties of the entity
* **Primary Keys**: Unique identifiers for each row in a table
* **Foreign Keys**: References to primary keys in other tables, establishing relationships
* **Constraints**: Rules that enforce data integrity (NOT NULL, UNIQUE, CHECK, etc.)

**Example Table Structure:**

```sql
-- A simple users table demonstrating relational concepts
CREATE TABLE users (
    user_id INT PRIMARY KEY,              -- Unique identifier
    username VARCHAR(50) NOT NULL UNIQUE, -- Cannot be null, must be unique
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active'
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    user_id INT NOT NULL,                 -- Foreign key reference
    order_total DECIMAL(10,2) NOT NULL,
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
        ON DELETE CASCADE                 -- Cascading delete
        ON UPDATE CASCADE
);
```

### ACID Properties Explained

ACID is an acronym describing the four key properties that guarantee reliable transaction processing in database systems:

#### Atomicity

Transactions are "all-or-nothing" operations. Either all changes in a transaction are committed to the database, or none are. If any part of a transaction fails, the entire transaction is rolled back.

```sql
-- Example: Transfer money between accounts (atomic operation)
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- If either UPDATE fails, both are rolled back
COMMIT;
```

**Why it matters**: Prevents partial updates that could leave data in an inconsistent state. In the example above, you can't debit one account without crediting the other.

#### Consistency

Transactions move the database from one valid state to another, maintaining all defined rules, constraints, triggers, and cascades. The database must always be in a consistent state before and after a transaction.

```sql
-- Example: Constraint ensures consistency
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    stock_quantity INT NOT NULL CHECK (stock_quantity >= 0),
    price DECIMAL(10,2) NOT NULL CHECK (price > 0)
);

-- This transaction will fail because it violates the CHECK constraint
START TRANSACTION;
UPDATE products SET stock_quantity = -5 WHERE product_id = 1;
-- ERROR: Check constraint violated
ROLLBACK;
```

**Why it matters**: Ensures that business rules and data integrity constraints are never violated, even temporarily.

#### Isolation

Concurrent transactions are isolated from each other. Changes made by one transaction are not visible to other transactions until the transaction is committed. This prevents issues like dirty reads, non-repeatable reads, and phantom reads.

**Isolation Levels** (from least to most strict):
1. **READ UNCOMMITTED**: Can see uncommitted changes from other transactions (dirty reads possible)
2. **READ COMMITTED**: Only sees committed changes (PostgreSQL default)
3. **REPEATABLE READ**: Guarantees that reading the same row twice returns the same data (MySQL default)
4. **SERIALIZABLE**: Strictest isolation, transactions appear to execute serially

```sql
-- Example demonstrating isolation
-- Session 1
START TRANSACTION;
UPDATE products SET price = 99.99 WHERE product_id = 1;
-- Not yet committed

-- Session 2 (running concurrently)
SELECT price FROM products WHERE product_id = 1;
-- Under READ COMMITTED or stricter, this sees the old price
-- Under READ UNCOMMITTED, this might see 99.99 (dirty read)
```

**Why it matters**: Prevents race conditions and data corruption when multiple users access the same data simultaneously.

#### Durability

Once a transaction is committed, its changes are permanent, even in the event of system crashes, power failures, or other errors. Committed data is written to non-volatile storage.

**Implementation mechanisms:**
* **Write-Ahead Logging (WAL)**: Changes are first written to a log before being applied to data files
* **Checkpoints**: Periodic synchronization of in-memory data to disk
* **Transaction logs**: Permanent record of all committed transactions

```sql
-- Once this commits, the change survives system crashes
START TRANSACTION;
INSERT INTO critical_data (id, value) VALUES (1, 'important');
COMMIT;  -- Data is now durable
```

**Why it matters**: Guarantees that your data won't be lost after a transaction succeeds, providing reliability and trust in the database system.

### Database vs DBMS vs RDBMS

Understanding these terms is crucial for technical discussions:

| Term | Definition | Examples |
|------|------------|----------|
| **Database** | An organized collection of structured data | MySQL database, PostgreSQL database |
| **DBMS** (Database Management System) | Software that manages databases, providing storage, retrieval, and administration | MySQL Server, PostgreSQL Server, MongoDB |
| **RDBMS** (Relational DBMS) | A DBMS based on the relational model, using SQL | MySQL, PostgreSQL, Oracle, SQL Server |

**Key distinction**: Not all databases are relational (e.g., MongoDB is a document database, Redis is a key-value store), but all RDBMSs use SQL and the relational model.

### SQL Standards and Implementations

SQL has evolved through several standardization efforts by ANSI (American National Standards Institute) and ISO (International Organization for Standardization):

**Major SQL Standards:**
* **SQL-86**: First ANSI standard
* **SQL-92**: Major revision, widely implemented
* **SQL:1999**: Added recursive queries, triggers, object-oriented features
* **SQL:2003**: Window functions, XML support
* **SQL:2011**: Temporal data, enhanced analytics
* **SQL:2016**: JSON support, polymorphic tables

**Reality of SQL standards**: While standards exist, no database implements them 100%, and each adds proprietary extensions. This is why understanding both MySQL and PostgreSQL syntax variations is essential for production work.

### MySQL vs PostgreSQL: Philosophy and Architecture

Both MySQL and PostgreSQL are powerful, open-source RDBMS, but they evolved with different philosophies:

| Aspect | MySQL 8.0+ | PostgreSQL 13+ |
|--------|------------|----------------|
| **Philosophy** | Speed and ease of use; web application focus | Standards compliance and extensibility; enterprise focus |
| **ACID Compliance** | Full ACID with InnoDB storage engine | Full ACID by design |
| **Storage Engine** | Pluggable (InnoDB default, MyISAM legacy) | Single integrated storage system |
| **Replication** | Built-in binary log replication, group replication | Streaming replication, logical replication |
| **Licensing** | GPL (with commercial dual-license option) | PostgreSQL License (very permissive) |
| **Typical Use Cases** | Web applications, read-heavy workloads, simple schemas | Complex queries, analytics, data integrity critical systems |
| **JSON Support** | Native JSON type (since 5.7) | Native JSON and superior JSONB type |
| **Full-Text Search** | Built-in FULLTEXT indexes | Advanced text search with multiple language support |
| **Window Functions** | Added in MySQL 8.0 | Mature implementation since PostgreSQL 8.4 |
| **CTE Support** | Added in MySQL 8.0 (with limitations) | Full recursive CTE support |

**When to choose MySQL:**
* Building web applications with straightforward requirements
* Need master-slave replication for read scaling
* Team has existing MySQL expertise
* Require MariaDB compatibility

**When to choose PostgreSQL:**
* Complex queries with CTEs, window functions, advanced SQL
* Data integrity is paramount
* Need advanced data types (arrays, hstore, custom types)
* Require PostGIS for geospatial data
* Building analytics or reporting systems

> **Note**: This guide covers both systems throughout, noting differences where they exist. Understanding both makes you a more versatile database professional.

---

## 1.2 SQL Command Categories Overview

SQL commands are organized into five major categories based on their function. Understanding these categories helps you reason about how databases process commands and manage permissions.

### The Five Command Categories

```
SQL Commands
├── DDL (Data Definition Language)    - Defines database structure
├── DML (Data Manipulation Language)  - Manipulates data
├── DQL (Data Query Language)         - Queries data
├── DCL (Data Control Language)       - Controls access
└── TCL (Transaction Control Language) - Manages transactions
```

### DDL - Data Definition Language

**Purpose**: Define and modify the structure of database objects (tables, indexes, views, etc.)

**Key Commands**: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`, `COMMENT`

**Characteristics**:
* Auto-committed in MySQL (cannot be rolled back)
* Can be run within transactions in PostgreSQL
* Requires specific privileges (CREATE, ALTER, DROP)
* Changes schema metadata
* Can lock tables during execution

**Example**:
```sql
-- Creates permanent schema changes
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100) NOT NULL
);
```

### DML - Data Manipulation Language

**Purpose**: Insert, update, and delete data within tables

**Key Commands**: `INSERT`, `UPDATE`, `DELETE`, `MERGE`/`REPLACE`

**Characteristics**:
* NOT auto-committed (can be rolled back if in a transaction)
* Subject to constraints and triggers
* Requires INSERT, UPDATE, or DELETE privileges
* Logged in transaction logs
* Can be batched for performance

**Example**:
```sql
-- Can be rolled back if in a transaction
START TRANSACTION;
INSERT INTO employees (emp_id, emp_name) VALUES (1, 'John Doe');
UPDATE employees SET emp_name = 'Jane Doe' WHERE emp_id = 1;
DELETE FROM employees WHERE emp_id = 1;
ROLLBACK;  -- All changes undone
```

### DQL - Data Query Language

**Purpose**: Retrieve data from the database

**Key Commands**: `SELECT`

**Characteristics**:
* Read-only operations
* Requires SELECT privilege
* Can be complex with joins, subqueries, window functions
* Optimized by query planner
* Can use indexes for performance

**Example**:
```sql
-- Complex query with joins and aggregation
SELECT 
    d.dept_name,
    COUNT(e.emp_id) as employee_count,
    AVG(e.salary) as avg_salary
FROM departments d
LEFT JOIN employees e ON d.dept_id = e.dept_id
GROUP BY d.dept_name
HAVING COUNT(e.emp_id) > 5
ORDER BY avg_salary DESC;
```

### DCL - Data Control Language

**Purpose**: Control access to data and database objects

**Key Commands**: `GRANT`, `REVOKE`

**Characteristics**:
* Typically used by database administrators
* Auto-committed
* Affects privilege tables (mysql.user in MySQL, pg_authid in PostgreSQL)
* Security-critical operations
* Can cascade to revoke dependent privileges

**Example**:
```sql
-- Grant read access to a user
GRANT SELECT ON employees TO 'analyst_user'@'localhost';

-- Revoke write access
REVOKE INSERT, UPDATE, DELETE ON employees FROM 'analyst_user'@'localhost';
```

### TCL - Transaction Control Language

**Purpose**: Manage transactions and their properties

**Key Commands**: `BEGIN`/`START TRANSACTION`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `SET TRANSACTION`

**Characteristics**:
* Controls atomicity and isolation
* Essential for data integrity
* Can nest using savepoints
* Affects locking and concurrency
* Configure isolation levels

**Example**:
```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;

SAVEPOINT after_debit;

UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- Something went wrong with credit, rollback to savepoint
ROLLBACK TO after_debit;

-- Fix and commit
UPDATE accounts SET balance = balance + 100 WHERE account_id = 3;
COMMIT;
```

### How These Categories Interact

Understanding the interaction between command categories is crucial for building robust database applications:

```
Application Request
       ↓
   [DCL Check]  ← Does user have permission?
       ↓
  [TCL Start]   ← Begin transaction (if needed)
       ↓
   [DQL/DML]    ← Execute queries/modifications
       ↓
  [Constraints] ← Check DDL-defined rules
       ↓
   [TCL End]    ← Commit or rollback
```

**Example workflow**:

```sql
-- 1. DCL: User must have privileges
-- Assume user has SELECT, INSERT, UPDATE on orders table

-- 2. TCL: Start transaction
START TRANSACTION;

-- 3. DQL: Read current state
SELECT order_status FROM orders WHERE order_id = 12345;

-- 4. DML: Modify data
UPDATE orders 
SET order_status = 'shipped', shipped_at = CURRENT_TIMESTAMP
WHERE order_id = 12345;

-- 5. DDL-defined constraints are checked automatically
-- (e.g., CHECK constraints, foreign keys)

-- 6. TCL: Commit changes
COMMIT;
```

### Auto-Commit vs Explicit Transactions

Both MySQL and PostgreSQL support two transaction modes:

**Auto-Commit Mode** (default in MySQL):
* Each statement is automatically committed
* No explicit BEGIN/COMMIT needed
* Cannot rollback individual statements
* Suitable for single-statement operations

```sql
-- In auto-commit mode (MySQL default)
INSERT INTO logs (message) VALUES ('Event occurred');
-- Automatically committed, cannot rollback
```

**Explicit Transaction Mode**:
* Manual control over transaction boundaries
* Multiple statements can be grouped
* Can rollback before commit
* Required for complex multi-step operations

```sql
-- Explicit transaction (both MySQL and PostgreSQL)
START TRANSACTION;

INSERT INTO orders (user_id, total) VALUES (123, 99.99);
INSERT INTO order_items (order_id, product_id, quantity) 
VALUES (LAST_INSERT_ID(), 456, 2);

COMMIT;  -- Both inserts succeed together or fail together
```

**Best Practice**: Use explicit transactions for any operation that involves multiple related statements or when data consistency across statements is required.

---

## 1.3 DDL - Data Definition Language

Data Definition Language (DDL) commands define the structure and schema of your database. These are the commands that create, modify, and delete database objects. Understanding DDL is essential for database design, migration, and optimization.

### CREATE Commands

#### CREATE DATABASE

Creates a new database (schema in PostgreSQL terminology).

**MySQL Syntax:**
```sql
CREATE DATABASE company_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

-- With conditional creation
CREATE DATABASE IF NOT EXISTS company_db;

-- Show databases
SHOW DATABASES;
```

**PostgreSQL Syntax:**
```sql
CREATE DATABASE company_db
WITH 
    ENCODING = 'UTF8'
    LC_COLLATE = 'en_US.UTF-8'
    LC_CTYPE = 'en_US.UTF-8'
    TEMPLATE = template0;

-- List databases
\l  -- In psql
SELECT datname FROM pg_database;
```

**Key Differences:**

| Feature | MySQL | PostgreSQL |
|---------|-------|------------|
| Default Character Set | latin1 (older versions), utf8mb4 (modern) | UTF8 |
| Collation | Specified with COLLATE | Specified with LC_COLLATE |
| Template | N/A | Can clone from template databases |
| Owner | Current user | Specified with OWNER clause |

**Best Practices:**
* Always use UTF8/utf8mb4 encoding for international character support
* Use `IF NOT EXISTS` in deployment scripts to avoid errors
* Set appropriate collation for your language requirements
* In PostgreSQL, specify template0 to ensure clean encoding

#### CREATE TABLE

The most commonly used DDL command, defining the structure of data storage.

**Complete Example with All Column Types:**

```sql
-- MySQL Example
CREATE TABLE employees (
    -- Numeric types
    emp_id INT AUTO_INCREMENT PRIMARY KEY,
    department_id SMALLINT NOT NULL,
    salary DECIMAL(10, 2) NOT NULL,
    bonus_percentage FLOAT,
    
    -- String types
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    bio TEXT,
    status ENUM('active', 'inactive', 'suspended') DEFAULT 'active',
    
    -- Date/Time types
    hire_date DATE NOT NULL,
    last_login DATETIME,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    -- Binary types
    profile_picture BLOB,
    
    -- JSON type (MySQL 5.7+)
    metadata JSON,
    
    -- Constraints
    CONSTRAINT chk_salary CHECK (salary >= 0),
    CONSTRAINT fk_department FOREIGN KEY (department_id) 
        REFERENCES departments(dept_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE,
    
    INDEX idx_last_name (last_name),
    INDEX idx_email (email),
    INDEX idx_hire_date (hire_date)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**PostgreSQL Example:**

```sql
CREATE TABLE employees (
    -- Numeric types
    emp_id SERIAL PRIMARY KEY,  -- Auto-incrementing
    department_id SMALLINT NOT NULL,
    salary NUMERIC(10, 2) NOT NULL,
    bonus_percentage REAL,
    
    -- String types
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    bio TEXT,
    
    -- PostgreSQL doesn't have ENUM type directly, use custom type or CHECK
    status VARCHAR(20) DEFAULT 'active' 
        CHECK (status IN ('active', 'inactive', 'suspended')),
    
    -- Date/Time types
    hire_date DATE NOT NULL,
    last_login TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Binary types
    profile_picture BYTEA,
    
    -- JSON types (PostgreSQL has both JSON and JSONB)
    metadata JSONB,  -- JSONB is usually preferred (binary, indexable)
    
    -- Array type (PostgreSQL-specific)
    skills TEXT[],
    
    -- Constraints
    CONSTRAINT chk_salary CHECK (salary >= 0),
    CONSTRAINT fk_department FOREIGN KEY (department_id)
        REFERENCES departments(dept_id)
        ON DELETE RESTRICT
        ON UPDATE CASCADE
);

-- Indexes created separately in PostgreSQL
CREATE INDEX idx_last_name ON employees(last_name);
CREATE INDEX idx_email ON employees(email);
CREATE INDEX idx_hire_date ON employees(hire_date);

-- Trigger for updated_at (PostgreSQL doesn't have ON UPDATE)
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_employees_updated_at 
BEFORE UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION update_updated_at_column();
```

**Data Types Comparison:**

| Purpose | MySQL | PostgreSQL | Notes |
|---------|-------|------------|-------|
| Auto-increment | `AUTO_INCREMENT` | `SERIAL` or `GENERATED ALWAYS AS IDENTITY` | PostgreSQL SERIAL is shorthand for sequence |
| Fixed-point decimal | `DECIMAL(p,s)` | `NUMERIC(p,s)` or `DECIMAL(p,s)` | Identical behavior |
| Floating-point | `FLOAT`, `DOUBLE` | `REAL`, `DOUBLE PRECISION` | Similar, REAL is single-precision |
| Boolean | `TINYINT(1)` or `BOOLEAN` | `BOOLEAN` | PostgreSQL has true native boolean |
| String | `VARCHAR(n)`, `TEXT` | `VARCHAR(n)`, `TEXT` | TEXT has no length limit in either |
| Enumeration | `ENUM('a','b')` | Custom TYPE or CHECK constraint | PostgreSQL has CREATE TYPE |
| JSON | `JSON` | `JSON`, `JSONB` | JSONB is binary, faster, indexable |
| Binary | `BLOB`, `VARBINARY` | `BYTEA` | Different syntax, same purpose |
| Array | Not supported | `type[]` | PostgreSQL-specific feature |

**Temporary vs Permanent Tables:**

```sql
-- Temporary table (both MySQL and PostgreSQL)
-- Automatically dropped at session end
CREATE TEMPORARY TABLE temp_calculations (
    id INT,
    result DECIMAL(10,2)
);

-- MySQL: Temporary tables are session-specific
-- PostgreSQL: Temporary tables are also session-specific but in separate schema
```

**IF NOT EXISTS Clause:**

```sql
-- Both MySQL and PostgreSQL support this
CREATE TABLE IF NOT EXISTS employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100)
);

-- Prevents error if table already exists
-- Useful in deployment scripts and migrations
```

#### CREATE INDEX

Indexes dramatically improve query performance but come with trade-offs.

**MySQL Index Creation:**

```sql
-- Simple index
CREATE INDEX idx_last_name ON employees(last_name);

-- Unique index
CREATE UNIQUE INDEX idx_email ON employees(email);

-- Composite index (multi-column)
CREATE INDEX idx_name_dept ON employees(last_name, department_id);

-- Full-text index
CREATE FULLTEXT INDEX idx_bio ON employees(bio);

-- Spatial index (for GEOMETRY types)
CREATE SPATIAL INDEX idx_location ON stores(location);

-- Index with length prefix (for long VARCHAR/TEXT)
CREATE INDEX idx_bio_prefix ON employees(bio(100));

-- Descending index (MySQL 8.0+)
CREATE INDEX idx_salary_desc ON employees(salary DESC);
```

**PostgreSQL Index Creation:**

```sql
-- B-tree index (default)
CREATE INDEX idx_last_name ON employees(last_name);

-- Unique index
CREATE UNIQUE INDEX idx_email ON employees(email);

-- Composite index
CREATE INDEX idx_name_dept ON employees(last_name, department_id);

-- Partial index (PostgreSQL-specific, very powerful)
CREATE INDEX idx_active_employees ON employees(last_name)
WHERE status = 'active';

-- Expression index
CREATE INDEX idx_lower_email ON employees(LOWER(email));

-- Concurrent index creation (doesn't lock table)
CREATE INDEX CONCURRENTLY idx_hire_date ON employees(hire_date);

-- GIN index for JSONB (PostgreSQL-specific)
CREATE INDEX idx_metadata ON employees USING GIN(metadata);

-- Full-text search index
CREATE INDEX idx_bio_fts ON employees USING GIN(to_tsvector('english', bio));

-- Hash index (equality searches only)
CREATE INDEX idx_emp_id_hash ON employees USING HASH(emp_id);

-- BRIN index (for large tables with natural ordering)
CREATE INDEX idx_created_brin ON employees USING BRIN(created_at);
```

**Index Types Comparison:**

| Index Type | MySQL | PostgreSQL | Use Case |
|------------|-------|------------|----------|
| B-tree | Default | Default | General purpose, range queries |
| Hash | Available (InnoDB 8.0+) | Available | Equality searches only |
| Full-text | FULLTEXT | GIN with to_tsvector | Text searching |
| Spatial | SPATIAL | GiST | Geographic queries |
| Partial | Not supported | Supported | Indexing subset of rows |
| Expression | Not directly | Supported | Indexing computed values |
| Covering/Include | Covering possible | INCLUDE clause | Index-only scans |

**Best Practices:**
* Create indexes on columns used in WHERE, JOIN, ORDER BY clauses
* Use composite indexes for queries filtering on multiple columns
* Consider index selectivity (unique values / total rows)
* Monitor index usage and remove unused indexes
* In PostgreSQL, use CONCURRENTLY for production systems to avoid locks
* Don't over-index—each index slows down INSERT/UPDATE/DELETE

#### CREATE VIEW

Views are virtual tables based on query results.

**MySQL View Creation:**

```sql
-- Simple view
CREATE VIEW active_employees AS
SELECT emp_id, first_name, last_name, email
FROM employees
WHERE status = 'active';

-- View with joins
CREATE VIEW employee_details AS
SELECT 
    e.emp_id,
    e.first_name,
    e.last_name,
    d.dept_name,
    e.salary
FROM employees e
JOIN departments d ON e.department_id = d.dept_id;

-- Updatable view (simple, based on single table)
CREATE VIEW high_earners AS
SELECT emp_id, first_name, last_name, salary
FROM employees
WHERE salary > 100000
WITH CHECK OPTION;  -- Ensures updates maintain WHERE condition

-- Replace existing view
CREATE OR REPLACE VIEW active_employees AS
SELECT emp_id, first_name, last_name, email, hire_date
FROM employees
WHERE status = 'active';
```

**PostgreSQL View Creation:**

```sql
-- Same syntax as MySQL for basic views
CREATE VIEW active_employees AS
SELECT emp_id, first_name, last_name, email
FROM employees
WHERE status = 'active';

-- Materialized view (PostgreSQL-specific, stores results physically)
CREATE MATERIALIZED VIEW employee_summary AS
SELECT 
    department_id,
    COUNT(*) as employee_count,
    AVG(salary) as avg_salary,
    MAX(salary) as max_salary
FROM employees
GROUP BY department_id;

-- Refresh materialized view
REFRESH MATERIALIZED VIEW employee_summary;

-- Concurrent refresh (doesn't lock view)
REFRESH MATERIALIZED VIEW CONCURRENTLY employee_summary;
```

**View Updatability:**

MySQL and PostgreSQL both support updatable views under certain conditions:

**Updatable View Requirements:**
* Based on a single table
* No aggregate functions (COUNT, SUM, etc.)
* No DISTINCT, GROUP BY, HAVING
* No UNION, subqueries in SELECT
* All non-nullable columns without defaults included

```sql
-- Example of updatable view
CREATE VIEW simple_employees AS
SELECT emp_id, first_name, last_name, email
FROM employees;

-- This works (updates underlying table)
UPDATE simple_employees SET email = 'new@email.com' WHERE emp_id = 1;

-- Example of non-updatable view
CREATE VIEW salary_stats AS
SELECT department_id, AVG(salary) as avg_sal
FROM employees
GROUP BY department_id;

-- This fails (contains aggregate function)
-- UPDATE salary_stats SET avg_sal = 50000;  -- ERROR
```

#### CREATE STORED PROCEDURE / FUNCTION

Stored procedures and functions encapsulate business logic in the database.

**MySQL Stored Procedure:**

```sql
DELIMITER $$

CREATE PROCEDURE calculate_bonus(
    IN emp_id_param INT,
    IN bonus_pct DECIMAL(5,2),
    OUT bonus_amount DECIMAL(10,2)
)
BEGIN
    DECLARE emp_salary DECIMAL(10,2);
    
    -- Get employee salary
    SELECT salary INTO emp_salary
    FROM employees
    WHERE emp_id = emp_id_param;
    
    -- Calculate bonus
    SET bonus_amount = emp_salary * (bonus_pct / 100);
    
    -- Log calculation
    INSERT INTO bonus_log (emp_id, bonus_amount, calculated_at)
    VALUES (emp_id_param, bonus_amount, NOW());
END$$

DELIMITER ;

-- Call the procedure
CALL calculate_bonus(123, 10.5, @bonus);
SELECT @bonus;
```

**MySQL Function:**

```sql
DELIMITER $$

CREATE FUNCTION get_employee_tenure(emp_id_param INT)
RETURNS INT
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE years INT;
    
    SELECT TIMESTAMPDIFF(YEAR, hire_date, CURDATE())
    INTO years
    FROM employees
    WHERE emp_id = emp_id_param;
    
    RETURN years;
END$$

DELIMITER ;

-- Use the function
SELECT first_name, last_name, get_employee_tenure(emp_id) as tenure
FROM employees;
```

**PostgreSQL Function (PL/pgSQL):**

```sql
-- Procedure (PostgreSQL 11+)
CREATE OR REPLACE PROCEDURE calculate_bonus(
    emp_id_param INT,
    bonus_pct NUMERIC,
    INOUT bonus_amount NUMERIC DEFAULT NULL
)
LANGUAGE plpgsql
AS $$
DECLARE
    emp_salary NUMERIC;
BEGIN
    -- Get employee salary
    SELECT salary INTO emp_salary
    FROM employees
    WHERE emp_id = emp_id_param;
    
    -- Calculate bonus
    bonus_amount := emp_salary * (bonus_pct / 100);
    
    -- Log calculation
    INSERT INTO bonus_log (emp_id, bonus_amount, calculated_at)
    VALUES (emp_id_param, bonus_amount, CURRENT_TIMESTAMP);
END;
$$;

-- Call procedure
CALL calculate_bonus(123, 10.5);
```

**PostgreSQL Function:**

```sql
CREATE OR REPLACE FUNCTION get_employee_tenure(emp_id_param INT)
RETURNS INTEGER
LANGUAGE plpgsql
STABLE  -- Function doesn't modify database
AS $$
DECLARE
    years INTEGER;
BEGIN
    SELECT EXTRACT(YEAR FROM AGE(CURRENT_DATE, hire_date))
    INTO years
    FROM employees
    WHERE emp_id = emp_id_param;
    
    RETURN years;
END;
$$;

-- Use the function
SELECT first_name, last_name, get_employee_tenure(emp_id) as tenure
FROM employees;
```

**Procedures vs Functions:**

| Aspect | Procedures | Functions |
|--------|------------|-----------|
| Return Value | Can have OUT parameters | Must return a value |
| Call Syntax | `CALL procedure()` | Used in SELECT, WHERE, etc. |
| Transaction Control | Can contain COMMIT/ROLLBACK (PostgreSQL) | Cannot control transactions |
| Usage | Business logic, data manipulation | Calculations, transformations |

#### CREATE TRIGGER

Triggers automatically execute code in response to specific events.

**MySQL Trigger:**

```sql
-- Before INSERT trigger
DELIMITER $$

CREATE TRIGGER before_employee_insert
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    -- Validate email format
    IF NEW.email NOT REGEXP '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Invalid email format';
    END IF;
    
    -- Auto-generate username
    SET NEW.username = CONCAT(LOWER(NEW.first_name), '.', LOWER(NEW.last_name));
    
    -- Set timestamps
    SET NEW.created_at = NOW();
    SET NEW.updated_at = NOW();
END$$

DELIMITER ;

-- After UPDATE trigger for audit logging
DELIMITER $$

CREATE TRIGGER after_salary_update
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    IF OLD.salary != NEW.salary THEN
        INSERT INTO salary_audit (
            emp_id,
            old_salary,
            new_salary,
            changed_by,
            changed_at
        ) VALUES (
            NEW.emp_id,
            OLD.salary,
            NEW.salary,
            USER(),
            NOW()
        );
    END IF;
END$$

DELIMITER ;
```

**PostgreSQL Trigger:**

```sql
-- Trigger function
CREATE OR REPLACE FUNCTION validate_employee_insert()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Validate email format
    IF NEW.email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$' THEN
        RAISE EXCEPTION 'Invalid email format: %', NEW.email;
    END IF;
    
    -- Auto-generate username
    NEW.username := LOWER(NEW.first_name || '.' || NEW.last_name);
    
    -- Set timestamps
    NEW.created_at := CURRENT_TIMESTAMP;
    NEW.updated_at := CURRENT_TIMESTAMP;
    
    RETURN NEW;
END;
$$;

-- Create trigger
CREATE TRIGGER before_employee_insert
BEFORE INSERT ON employees
FOR EACH ROW
EXECUTE FUNCTION validate_employee_insert();

-- Audit trigger
CREATE OR REPLACE FUNCTION audit_salary_changes()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF OLD.salary != NEW.salary THEN
        INSERT INTO salary_audit (
            emp_id,
            old_salary,
            new_salary,
            changed_by,
            changed_at
        ) VALUES (
            NEW.emp_id,
            OLD.salary,
            NEW.salary,
            CURRENT_USER,
            CURRENT_TIMESTAMP
        );
    END IF;
    
    RETURN NEW;
END;
$$;

CREATE TRIGGER after_salary_update
AFTER UPDATE ON employees
FOR EACH ROW
WHEN (OLD.salary IS DISTINCT FROM NEW.salary)
EXECUTE FUNCTION audit_salary_changes();
```

**Trigger Types:**

| Timing | Event | Description |
|--------|-------|-------------|
| BEFORE | INSERT | Execute before row is inserted |
| BEFORE | UPDATE | Execute before row is updated |
| BEFORE | DELETE | Execute before row is deleted |
| AFTER | INSERT | Execute after row is inserted |
| AFTER | UPDATE | Execute after row is updated |
| AFTER | DELETE | Execute after row is deleted |

**Best Practices:**
* Keep trigger logic simple and fast
* Avoid recursive triggers
* Use BEFORE triggers for validation and AFTER for logging
* Document trigger behavior thoroughly
* Consider performance impact on DML operations

#### CREATE USER / ROLE

Database security starts with proper user management.

**MySQL User Creation:**

```sql
-- Create user with password
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'secure_password';

-- Create user for remote connection
CREATE USER 'app_user'@'192.168.1.%' IDENTIFIED BY 'secure_password';

-- Create user with specific authentication plugin
CREATE USER 'app_user'@'localhost' 
IDENTIFIED WITH mysql_native_password BY 'secure_password';

-- Create user with password expiration
CREATE USER 'temp_user'@'localhost' 
IDENTIFIED BY 'temp_password' 
PASSWORD EXPIRE INTERVAL 90 DAY;

-- Create user with resource limits
CREATE USER 'limited_user'@'localhost'
IDENTIFIED BY 'password'
WITH MAX_QUERIES_PER_HOUR 1000
     MAX_UPDATES_PER_HOUR 500
     MAX_CONNECTIONS_PER_HOUR 50;
```

**PostgreSQL Role Creation:**

```sql
-- Create role (PostgreSQL uses roles instead of users)
CREATE ROLE app_user WITH LOGIN PASSWORD 'secure_password';

-- Create role with specific attributes
CREATE ROLE admin_user WITH
    LOGIN
    PASSWORD 'admin_password'
    SUPERUSER
    CREATEDB
    CREATEROLE
    VALID UNTIL '2025-12-31';

-- Create role for group membership
CREATE ROLE developers;
CREATE ROLE john_dev WITH LOGIN PASSWORD 'password' IN ROLE developers;

-- Create role with connection limit
CREATE ROLE limited_user WITH
    LOGIN
    PASSWORD 'password'
    CONNECTION LIMIT 10;
```

**User vs Role (PostgreSQL concept):**

In PostgreSQL, users and roles are essentially the same, but:
* **Role**: Cannot login unless given LOGIN attribute
* **User**: Role with LOGIN attribute
* Roles can be members of other roles (role inheritance)

```sql
-- Role hierarchy in PostgreSQL
CREATE ROLE readonly_users;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_users;

CREATE ROLE user1 WITH LOGIN PASSWORD 'pass1' IN ROLE readonly_users;
CREATE ROLE user2 WITH LOGIN PASSWORD 'pass2' IN ROLE readonly_users;

-- user1 and user2 automatically have SELECT permissions
```

### ALTER Commands

ALTER commands modify existing database objects.

#### ALTER TABLE

The most commonly used ALTER command for schema evolution.

**Adding Columns:**

```sql
-- MySQL
ALTER TABLE employees
ADD COLUMN middle_name VARCHAR(50) AFTER first_name;

-- PostgreSQL (no AFTER clause)
ALTER TABLE employees
ADD COLUMN middle_name VARCHAR(50);

-- Add with default value
ALTER TABLE employees
ADD COLUMN employee_type VARCHAR(20) DEFAULT 'full-time' NOT NULL;

-- Add multiple columns
ALTER TABLE employees
ADD COLUMN start_date DATE,
ADD COLUMN end_date DATE;
```

**Modifying Columns:**

```sql
-- ❌ BAD: MySQL MODIFY syntax (not portable)
ALTER TABLE employees
MODIFY COLUMN salary DECIMAL(12,2) NOT NULL;

-- ✅ GOOD: PostgreSQL ALTER COLUMN syntax (more explicit)
ALTER TABLE employees
ALTER COLUMN salary TYPE NUMERIC(12,2),
ALTER COLUMN salary SET NOT NULL;

-- Change column name (MySQL)
ALTER TABLE employees
CHANGE COLUMN old_name new_name VARCHAR(100);

-- Change column name (PostgreSQL)
ALTER TABLE employees
RENAME COLUMN old_name TO new_name;

-- Change data type (both)
ALTER TABLE employees
MODIFY COLUMN department_id INT;  -- MySQL

ALTER TABLE employees
ALTER COLUMN department_id TYPE INTEGER USING department_id::INTEGER;  -- PostgreSQL
```

**Dropping Columns:**

```sql
-- Both MySQL and PostgreSQL
ALTER TABLE employees
DROP COLUMN middle_name;

-- Drop multiple columns
ALTER TABLE employees
DROP COLUMN start_date,
DROP COLUMN end_date;

-- PostgreSQL: Drop with CASCADE (removes dependent objects)
ALTER TABLE employees
DROP COLUMN bio CASCADE;
```

**Adding/Dropping Constraints:**

```sql
-- Add PRIMARY KEY
ALTER TABLE employees
ADD PRIMARY KEY (emp_id);

-- Add FOREIGN KEY
ALTER TABLE employees
ADD CONSTRAINT fk_dept FOREIGN KEY (department_id)
    REFERENCES departments(dept_id)
    ON DELETE CASCADE;

-- Add CHECK constraint
ALTER TABLE employees
ADD CONSTRAINT chk_salary CHECK (salary > 0);

-- Add UNIQUE constraint
ALTER TABLE employees
ADD CONSTRAINT uk_email UNIQUE (email);

-- Drop constraint (MySQL)
ALTER TABLE employees
DROP FOREIGN KEY fk_dept;

-- Drop constraint (PostgreSQL)
ALTER TABLE employees
DROP CONSTRAINT fk_dept;

-- PostgreSQL: Add NOT NULL
ALTER TABLE employees
ALTER COLUMN email SET NOT NULL;

-- PostgreSQL: Drop NOT NULL
ALTER TABLE employees
ALTER COLUMN phone DROP NOT NULL;
```

#### Online DDL (MySQL) vs Concurrent DDL (PostgreSQL)

**MySQL Online DDL (InnoDB):**

MySQL 5.6+ supports online DDL for many operations:

```sql
-- Add index without blocking reads/writes
ALTER TABLE employees
ADD INDEX idx_last_name (last_name),
ALGORITHM=INPLACE, LOCK=NONE;

-- ALGORITHM options:
-- - INPLACE: No table copy, but may require table metadata lock
-- - COPY: Creates temp table copy (blocks writes)
-- - DEFAULT: MySQL chooses

-- LOCK options:
-- - NONE: Allow reads and writes
-- - SHARED: Allow reads, block writes
-- - EXCLUSIVE: Block reads and writes
-- - DEFAULT: MySQL chooses minimum lock level

-- Add column online (MySQL 8.0+)
ALTER TABLE employees
ADD COLUMN new_column VARCHAR(50),
ALGORITHM=INSTANT;  -- Metadata-only change
```

**PostgreSQL Concurrent Operations:**

```sql
-- Add index without blocking writes
CREATE INDEX CONCURRENTLY idx_last_name ON employees(last_name);

-- Add column (always non-blocking for reads in PostgreSQL)
ALTER TABLE employees
ADD COLUMN new_column VARCHAR(50);

-- However, adding NOT NULL constraint blocks
-- Better approach:
ALTER TABLE employees
ADD COLUMN new_column VARCHAR(50);

-- Add default value (PostgreSQL 11+, instant)
ALTER TABLE employees
ALTER COLUMN new_column SET DEFAULT 'value';

-- Add NOT NULL in separate transaction after backfilling
```

**Schema Migration Best Practices:**

```sql
-- ❌ BAD: Blocking operation on large table
ALTER TABLE orders
ADD COLUMN tracking_number VARCHAR(50) NOT NULL DEFAULT '';

-- ✅ GOOD: Multi-step migration
-- Step 1: Add column as nullable
ALTER TABLE orders
ADD COLUMN tracking_number VARCHAR(50);

-- Step 2: Backfill in batches
UPDATE orders 
SET tracking_number = CONCAT('TRK-', order_id)
WHERE tracking_number IS NULL
LIMIT 1000;
-- Repeat until done

-- Step 3: Add NOT NULL constraint
ALTER TABLE orders
MODIFY COLUMN tracking_number VARCHAR(50) NOT NULL;  -- MySQL

ALTER TABLE orders
ALTER COLUMN tracking_number SET NOT NULL;  -- PostgreSQL
```

#### ALTER INDEX

```sql
-- MySQL: Rename index
ALTER TABLE employees
RENAME INDEX old_index_name TO new_index_name;

-- PostgreSQL: Rename index
ALTER INDEX old_index_name RENAME TO new_index_name;

-- PostgreSQL: Change index properties
ALTER INDEX idx_employees SET (fillfactor = 70);
```

#### ALTER DATABASE

```sql
-- MySQL: Change database character set
ALTER DATABASE company_db 
CHARACTER SET utf8mb4 
COLLATE utf8mb4_unicode_ci;

-- PostgreSQL: Rename database
ALTER DATABASE old_name RENAME TO new_name;

-- PostgreSQL: Change owner
ALTER DATABASE company_db OWNER TO new_owner;

-- PostgreSQL: Change connection limit
ALTER DATABASE company_db CONNECTION LIMIT 100;
```

### DROP Commands

DROP commands permanently delete database objects. **Use with extreme caution!**

#### DROP TABLE

```sql
-- Basic drop
DROP TABLE employees;

-- Conditional drop
DROP TABLE IF EXISTS employees;

-- MySQL: Drop multiple tables
DROP TABLE IF EXISTS employees, departments, orders;

-- PostgreSQL: CASCADE removes dependent objects
DROP TABLE employees CASCADE;
-- This also drops:
-- - Foreign keys referencing this table
-- - Views based on this table
-- - Triggers on this table

-- PostgreSQL: RESTRICT fails if dependencies exist (default)
DROP TABLE employees RESTRICT;
```

#### DROP DATABASE

```sql
-- Both MySQL and PostgreSQL
DROP DATABASE IF EXISTS old_database;

-- ⚠️ WARNING: This is permanent and irreversible!
-- Always backup before dropping databases
```

#### DROP INDEX

```sql
-- MySQL: Drop index from table
DROP INDEX idx_last_name ON employees;

-- Alternative MySQL syntax
ALTER TABLE employees DROP INDEX idx_last_name;

-- PostgreSQL: Drop index directly
DROP INDEX idx_last_name;

-- PostgreSQL: Concurrent drop (doesn't block table)
DROP INDEX CONCURRENTLY idx_last_name;
```

### Soft Deletes vs Hard Deletes

In production systems, consider soft deletes for important data:

```sql
-- Hard delete (permanent)
DELETE FROM employees WHERE emp_id = 123;

-- Soft delete pattern
ALTER TABLE employees
ADD COLUMN deleted_at TIMESTAMP NULL,
ADD COLUMN deleted_by VARCHAR(50) NULL;

-- "Delete" by marking
UPDATE employees
SET deleted_at = CURRENT_TIMESTAMP,
    deleted_by = CURRENT_USER
WHERE emp_id = 123;

-- Query active records
SELECT * FROM employees WHERE deleted_at IS NULL;

-- Create view for active employees
CREATE VIEW active_employees AS
SELECT * FROM employees WHERE deleted_at IS NULL;
```

### Recovery from Accidental Drops

**Prevention:**
* Always use transactions when available
* Enable binary logging (MySQL) or WAL archiving (PostgreSQL)
* Regular backups
* Implement permissions carefully
* Use safe-update mode in clients

**MySQL Recovery:**
```sql
-- If binary logging enabled
-- 1. Find position before DROP
-- 2. Restore from backup to that position
-- 3. Extract data using mysqlbinlog

-- Example recovery process
mysqlbinlog --start-datetime="2024-01-01 09:00:00" \
            --stop-datetime="2024-01-01 09:30:00" \
            mysql-bin.000001 > recovery.sql
mysql < recovery.sql
```

**PostgreSQL Point-in-Time Recovery:**
```bash
# If WAL archiving enabled
# 1. Stop database
# 2. Restore base backup
# 3. Configure recovery target time
# 4. Start recovery mode

# recovery.conf
recovery_target_time = '2024-01-01 09:25:00'
recovery_target_inclusive = true
```

### TRUNCATE Command

TRUNCATE is a fast way to delete all rows from a table.

**Syntax:**

```sql
-- Both MySQL and PostgreSQL
TRUNCATE TABLE employees;

-- MySQL: Truncate multiple tables
TRUNCATE TABLE employees, departments;

-- PostgreSQL: CASCADE truncates referencing tables
TRUNCATE TABLE employees CASCADE;

-- PostgreSQL: RESTART IDENTITY resets sequences
TRUNCATE TABLE employees RESTART IDENTITY;
```

**TRUNCATE vs DELETE:**

| Aspect | TRUNCATE | DELETE |
|--------|----------|--------|
| Speed | Very fast (drops and recreates) | Slow (row-by-row) |
| WHERE clause | Not supported | Supported |
| Rollback | Can rollback in transaction (PostgreSQL), auto-commits (MySQL) | Can rollback in transaction |
| Triggers | Doesn't fire DELETE triggers | Fires DELETE triggers |
| Foreign Keys | Can fail if referenced | Respects foreign keys |
| Auto-increment | Resets to initial value | Doesn't reset |
| Disk space | Immediately reclaimed | May need VACUUM (PostgreSQL) or OPTIMIZE (MySQL) |

**Example:**

```sql
-- ❌ BAD: Slow on large tables
DELETE FROM logs WHERE log_date < DATE_SUB(NOW(), INTERVAL 90 DAY);

-- ✅ GOOD: Use partitioning and drop old partitions
-- Or if deleting all rows:
TRUNCATE TABLE logs;
```

### RENAME Command

**MySQL:**

```sql
-- Rename table
RENAME TABLE old_table TO new_table;

-- Rename multiple tables atomically
RENAME TABLE 
    old_table1 TO new_table1,
    old_table2 TO new_table2;

-- Move table to different database
RENAME TABLE db1.table1 TO db2.table1;
```

**PostgreSQL:**

```sql
-- Rename table
ALTER TABLE old_table RENAME TO new_table;

-- Rename column
ALTER TABLE employees RENAME COLUMN old_name TO new_name;

-- Rename constraint
ALTER TABLE employees RENAME CONSTRAINT old_constraint TO new_constraint;
```

### COMMENT Command

Documentation is critical for maintainability:

**MySQL:**

```sql
-- Add table comment
ALTER TABLE employees COMMENT = 'Employee master data table';

-- Add column comment
ALTER TABLE employees
MODIFY COLUMN salary DECIMAL(10,2) COMMENT 'Annual salary in USD';

-- View comments
SHOW CREATE TABLE employees;
SHOW FULL COLUMNS FROM employees;
```

**PostgreSQL:**

```sql
-- Add table comment
COMMENT ON TABLE employees IS 'Employee master data table';

-- Add column comment
COMMENT ON COLUMN employees.salary IS 'Annual salary in USD';

-- Add database comment
COMMENT ON DATABASE company_db IS 'Production company database';

-- Add function comment
COMMENT ON FUNCTION calculate_bonus(INT, NUMERIC) IS 'Calculates employee bonus';

-- View comments
SELECT 
    cols.column_name,
    pg_catalog.col_description(c.oid, cols.ordinal_position::int)
FROM information_schema.columns cols
JOIN pg_catalog.pg_class c ON c.relname = cols.table_name
WHERE cols.table_name = 'employees';
```

### Common Pitfalls

1. **Not using IF EXISTS/IF NOT EXISTS**
   ```sql
   -- ❌ BAD: Fails if table doesn't exist
   DROP TABLE temp_table;
   
   -- ✅ GOOD: Safe for deployment scripts
   DROP TABLE IF EXISTS temp_table;
   ```

2. **Forgetting CASCADE implications**
   ```sql
   -- ❌ DANGEROUS: Might drop more than intended
   DROP TABLE departments CASCADE;
   
   -- ✅ BETTER: Check dependencies first
   -- MySQL
   SELECT TABLE_NAME, CONSTRAINT_NAME
   FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
   WHERE REFERENCED_TABLE_NAME = 'departments';
   
   -- PostgreSQL
   SELECT
       tc.table_name, 
       kcu.column_name
   FROM information_schema.table_constraints tc
   JOIN information_schema.key_column_usage kcu
       ON tc.constraint_name = kcu.constraint_name
   WHERE constraint_type = 'FOREIGN KEY'
       AND tc.table_name = 'departments';
   ```

3. **ALTER TABLE without considering locks**
   ```sql
   -- ❌ BAD: Locks table on large dataset
   ALTER TABLE huge_table ADD COLUMN new_col VARCHAR(50) NOT NULL DEFAULT '';
   
   -- ✅ GOOD: Phased approach
   ALTER TABLE huge_table ADD COLUMN new_col VARCHAR(50);
   -- Backfill in batches during low traffic
   -- Then add NOT NULL if needed
   ```

### Best Practices

✅ **Always backup before DDL operations on production**
✅ **Use transactions where possible (PostgreSQL)**
✅ **Test DDL changes on staging environment first**
✅ **Use IF EXISTS/IF NOT EXISTS in deployment scripts**
✅ **Document schema changes with COMMENTs**
✅ **Consider online/concurrent operations for large tables**
✅ **Review foreign key cascades carefully**
✅ **Plan for rollback procedures**
✅ **Use version control for schema migrations**
✅ **Monitor replication lag after DDL on replicas**

### Performance Implications

DDL operations can have significant performance impact:

**Table-Locking Operations:**
* ALTER TABLE (depending on change type)
* CREATE INDEX (without CONCURRENTLY in PostgreSQL)
* TRUNCATE TABLE
* DROP TABLE

**Metadata-Only Operations (Fast):**
* ADD COLUMN with default (PostgreSQL 11+, MySQL 8.0+ with ALGORITHM=INSTANT)
* RENAME TABLE/COLUMN
* DROP INDEX CONCURRENTLY (PostgreSQL)

**Long-Running Operations:**
* Adding indexes on large tables
* Adding NOT NULL to populated columns
* Changing column data types (requires table rebuild)

---

## 1.4 DML - Data Manipulation Language

Data Manipulation Language (DML) commands are used to insert, update, and delete data within database tables. Unlike DDL, DML operations can be rolled back if executed within a transaction.

### INSERT Statement

INSERT adds new rows to a table.

#### Single Row Insert

**Basic Syntax:**

```sql
-- Specify all columns
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date)
VALUES (1, 'John', 'Doe', 'john.doe@company.com', '2024-01-15');

-- Omit auto-increment column
INSERT INTO employees (first_name, last_name, email, hire_date)
VALUES ('Jane', 'Smith', 'jane.smith@company.com', '2024-01-16');

-- Use DEFAULT keyword
INSERT INTO employees (first_name, last_name, email, hire_date, status)
VALUES ('Bob', 'Johnson', 'bob.j@company.com', '2024-01-17', DEFAULT);

-- Omit column list (must provide all values in order)
-- ❌ BAD PRACTICE: Fragile if table structure changes
INSERT INTO employees
VALUES (3, 'Alice', 'Williams', 'alice.w@company.com', 'active', '2024-01-18', NOW(), NOW(), NULL);
```

#### Multi-Row Insert

Significantly more efficient than multiple single-row inserts:

```sql
-- Insert multiple rows in one statement
INSERT INTO employees (first_name, last_name, email, hire_date)
VALUES 
    ('John', 'Doe', 'john@company.com', '2024-01-01'),
    ('Jane', 'Smith', 'jane@company.com', '2024-01-02'),
    ('Bob', 'Johnson', 'bob@company.com', '2024-01-03'),
    ('Alice', 'Williams', 'alice@company.com', '2024-01-04');

-- Batch size considerations
-- MySQL: Can handle thousands of rows per INSERT
-- PostgreSQL: Similar, but watch out for statement timeout

-- ✅ GOOD: Batch in reasonable sizes (e.g., 1000 rows)
-- ❌ BAD: Single massive INSERT with millions of rows
```

#### INSERT ... SELECT

Copy data from one table to another:

```sql
-- Copy all columns
INSERT INTO employees_archive
SELECT * FROM employees
WHERE hire_date < '2020-01-01';

-- Copy specific columns with transformation
INSERT INTO employee_summary (emp_id, full_name, years_employed)
SELECT 
    emp_id,
    CONCAT(first_name, ' ', last_name),
    TIMESTAMPDIFF(YEAR, hire_date, CURDATE())  -- MySQL
    -- EXTRACT(YEAR FROM AGE(CURRENT_DATE, hire_date))  -- PostgreSQL
FROM employees
WHERE status = 'active';

-- Insert aggregated data
INSERT INTO daily_stats (stat_date, total_employees, avg_salary)
SELECT 
    CURDATE(),
    COUNT(*),
    AVG(salary)
FROM employees
WHERE status = 'active';
```

#### INSERT ... ON DUPLICATE KEY UPDATE (MySQL)

MySQL's UPSERT implementation:

```sql
-- If emp_id exists, update; otherwise, insert
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date)
VALUES (1, 'John', 'Doe', 'john.doe@company.com', '2024-01-15')
ON DUPLICATE KEY UPDATE
    first_name = VALUES(first_name),
    last_name = VALUES(last_name),
    email = VALUES(email),
    updated_at = NOW();

-- More complex update logic
INSERT INTO product_inventory (product_id, quantity, last_updated)
VALUES (101, 50, NOW())
ON DUPLICATE KEY UPDATE
    quantity = quantity + VALUES(quantity),
    last_updated = NOW();

-- Using actual values in UPDATE
INSERT INTO page_views (page_id, view_count, last_viewed)
VALUES (5, 1, NOW())
ON DUPLICATE KEY UPDATE
    view_count = view_count + 1,
    last_viewed = NOW();
```

#### INSERT ... ON CONFLICT (PostgreSQL UPSERT)

PostgreSQL's more flexible UPSERT:

```sql
-- Basic upsert
INSERT INTO employees (emp_id, first_name, last_name, email, hire_date)
VALUES (1, 'John', 'Doe', 'john.doe@company.com', '2024-01-15')
ON CONFLICT (emp_id) DO UPDATE SET
    first_name = EXCLUDED.first_name,
    last_name = EXCLUDED.last_name,
    email = EXCLUDED.email,
    updated_at = CURRENT_TIMESTAMP;

-- EXCLUDED refers to the row that would have been inserted

-- Conditional update
INSERT INTO products (product_id, name, price, stock)
VALUES (101, 'Widget', 19.99, 100)
ON CONFLICT (product_id) DO UPDATE SET
    price = EXCLUDED.price,
    stock = products.stock + EXCLUDED.stock
WHERE products.price != EXCLUDED.price;  -- Only update if price changed

-- Multiple conflict targets (unique constraint)
INSERT INTO users (username, email, created_at)
VALUES ('johndoe', 'john@example.com', NOW())
ON CONFLICT (email) DO UPDATE SET
    username = EXCLUDED.username,
    updated_at = NOW();

-- Do nothing on conflict
INSERT INTO logs (id, message, created_at)
VALUES (1, 'Log entry', NOW())
ON CONFLICT (id) DO NOTHING;

-- Upsert with WHERE clause (PostgreSQL 15+)
INSERT INTO cache (key, value, expires_at)
VALUES ('user:123', '{"name":"John"}', NOW() + INTERVAL '1 hour')
ON CONFLICT (key) DO UPDATE SET
    value = EXCLUDED.value,
    expires_at = EXCLUDED.expires_at
WHERE cache.expires_at < NOW();  -- Only update expired entries
```

**MySQL vs PostgreSQL Upsert:**

| Feature | MySQL | PostgreSQL |
|---------|-------|------------|
| Syntax | `ON DUPLICATE KEY UPDATE` | `ON CONFLICT ... DO UPDATE` |
| Conflict Target | Implicit (any unique key) | Explicit (specify column/constraint) |
| UPDATE WHERE | Not supported | Supported |
| DO NOTHING | Not directly supported | Supported |
| Excluded Values | `VALUES(column)` | `EXCLUDED.column` |
| Flexibility | Less flexible | More flexible |

#### INSERT ... RETURNING (PostgreSQL)

Get information about inserted rows:

```sql
-- Return inserted ID
INSERT INTO employees (first_name, last_name, email, hire_date)
VALUES ('John', 'Doe', 'john@example.com', '2024-01-15')
RETURNING emp_id;

-- Return multiple columns
INSERT INTO orders (user_id, total, order_date)
VALUES (123, 99.99, CURRENT_TIMESTAMP)
RETURNING order_id, order_date, total;

-- Return calculated values
INSERT INTO products (name, price, tax_rate)
VALUES ('Widget', 100.00, 0.08)
RETURNING product_id, price * (1 + tax_rate) AS total_price;

-- With bulk insert
INSERT INTO logs (level, message)
VALUES 
    ('INFO', 'Application started'),
    ('DEBUG', 'Configuration loaded'),
    ('INFO', 'Server listening')
RETURNING log_id, created_at;
```

**MySQL Alternative (LAST_INSERT_ID):**

```sql
-- Get last auto-increment value (only works for single row)
INSERT INTO employees (first_name, last_name, email)
VALUES ('John', 'Doe', 'john@example.com');

SELECT LAST_INSERT_ID();

-- For multiple rows, only returns first ID
INSERT INTO employees (first_name, last_name, email)
VALUES 
    ('Jane', 'Smith', 'jane@example.com'),
    ('Bob', 'Johnson', 'bob@example.com');

SELECT LAST_INSERT_ID();  -- Returns ID of first row only
```

#### Bulk Insert Optimization

```sql
-- ❌ SLOW: Individual inserts in a loop
INSERT INTO logs (message) VALUES ('Log 1');
INSERT INTO logs (message) VALUES ('Log 2');
INSERT INTO logs (message) VALUES ('Log 3');
-- ... thousands more

-- ✅ FAST: Batch insert
INSERT INTO logs (message)
VALUES 
    ('Log 1'),
    ('Log 2'),
    ('Log 3'),
    -- ... batch of 1000
    ('Log 1000');

-- Even faster with transaction control
START TRANSACTION;

INSERT INTO logs (message)
VALUES ('Log 1'), ('Log 2'), ..., ('Log 1000');

INSERT INTO logs (message)
VALUES ('Log 1001'), ('Log 1002'), ..., ('Log 2000');

-- Repeat for all batches

COMMIT;

-- ✅ BEST: Use LOAD DATA (MySQL) or COPY (PostgreSQL) for bulk loads
-- MySQL
LOAD DATA LOCAL INFILE '/path/to/data.csv'
INTO TABLE logs
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
(message);

-- PostgreSQL
COPY logs(message)
FROM '/path/to/data.csv'
DELIMITER ','
CSV HEADER;
```

**Performance Comparison:**

| Method | Rows/Second (Approximate) |
|--------|---------------------------|
| Individual INSERTs | 100-500 |
| Batched INSERTs (100 rows) | 5,000-10,000 |
| Batched INSERTs in transaction | 10,000-50,000 |
| LOAD DATA / COPY | 50,000-500,000+ |

#### Transaction Batching

```sql
-- Optimal batching strategy
START TRANSACTION;

-- Batch 1
INSERT INTO large_table (column1, column2, column3)
VALUES (/* 1000 rows */);

-- Batch 2
INSERT INTO large_table (column1, column2, column3)
VALUES (/* 1000 rows */);

-- Every N batches, commit and start new transaction
COMMIT;
START TRANSACTION;

-- Continue...
```

### UPDATE Statement

UPDATE modifies existing data in a table.

#### Basic UPDATE Syntax

```sql
-- Update single column
UPDATE employees
SET email = 'new.email@company.com'
WHERE emp_id = 123;

-- Update multiple columns
UPDATE employees
SET 
    first_name = 'John',
    last_name = 'Doe',
    updated_at = NOW()
WHERE emp_id = 123;

-- Update with expression
UPDATE products
SET price = price * 1.10  -- 10% increase
WHERE category = 'electronics';

-- Update with CASE
UPDATE employees
SET bonus = CASE
    WHEN performance_rating >= 4.5 THEN salary * 0.15
    WHEN performance_rating >= 3.5 THEN salary * 0.10
    WHEN performance_rating >= 2.5 THEN salary * 0.05
    ELSE 0
END
WHERE status = 'active';
```

#### UPDATE with JOIN

**MySQL Syntax:**

```sql
-- Update using JOIN
UPDATE employees e
JOIN departments d ON e.department_id = d.dept_id
SET e.location = d.location
WHERE d.dept_name = 'Engineering';

-- Update with multiple joins
UPDATE orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN address a ON c.address_id = a.address_id
SET o.shipping_region = a.region
WHERE o.order_date >= '2024-01-01';

-- Update with LEFT JOIN to handle missing matches
UPDATE products p
LEFT JOIN inventory i ON p.product_id = i.product_id
SET p.in_stock = COALESCE(i.quantity > 0, FALSE);
```

**PostgreSQL Syntax:**

```sql
-- PostgreSQL uses FROM clause
UPDATE employees e
SET location = d.location
FROM departments d
WHERE e.department_id = d.dept_id
    AND d.dept_name = 'Engineering';

-- Multiple tables in FROM
UPDATE orders o
SET shipping_region = a.region
FROM customers c
JOIN address a ON c.address_id = a.address_id
WHERE o.customer_id = c.customer_id
    AND o.order_date >= '2024-01-01';

-- Subquery alternative (works in both)
UPDATE employees
SET location = (
    SELECT location
    FROM departments
    WHERE departments.dept_id = employees.department_id
)
WHERE department_id IN (
    SELECT dept_id FROM departments WHERE dept_name = 'Engineering'
);
```

#### Multi-Table UPDATE

**MySQL:**

```sql
-- Update multiple tables in one statement
UPDATE employees e
JOIN salaries s ON e.emp_id = s.emp_id
SET 
    e.last_raise_date = NOW(),
    s.current_salary = s.current_salary * 1.05
WHERE e.performance_rating >= 4.0;
```

**PostgreSQL doesn't support multi-table UPDATE directly**:

```sql
-- Use WITH CTE and multiple UPDATE statements in transaction
BEGIN;

WITH updated_employees AS (
    UPDATE employees
    SET last_raise_date = CURRENT_TIMESTAMP
    WHERE performance_rating >= 4.0
    RETURNING emp_id
)
UPDATE salaries s
SET current_salary = current_salary * 1.05
FROM updated_employees e
WHERE s.emp_id = e.emp_id;

COMMIT;
```

#### UPDATE ... RETURNING (PostgreSQL)

```sql
-- Return updated values
UPDATE employees
SET salary = salary * 1.10
WHERE performance_rating >= 4.5
RETURNING emp_id, first_name, last_name, salary;

-- Use in CTE for further processing
WITH updated AS (
    UPDATE employees
    SET salary = salary * 1.10,
        last_raise_date = CURRENT_TIMESTAMP
    WHERE performance_rating >= 4.5
    RETURNING emp_id, salary
)
INSERT INTO salary_history (emp_id, new_salary, change_date)
SELECT emp_id, salary, CURRENT_TIMESTAMP
FROM updated;
```

#### Performance Considerations

```sql
-- ❌ BAD: Missing WHERE clause (updates ALL rows)
UPDATE employees SET updated_at = NOW();

-- ❌ BAD: Function on indexed column prevents index usage
UPDATE employees
SET status = 'inactive'
WHERE YEAR(hire_date) < 2000;

-- ✅ GOOD: Use index-friendly condition
UPDATE employees
SET status = 'inactive'
WHERE hire_date < '2000-01-01';

-- ❌ BAD: Correlated subquery in SET clause (runs for each row)
UPDATE products
SET avg_rating = (
    SELECT AVG(rating) FROM reviews WHERE product_id = products.product_id
);

-- ✅ BETTER: Use JOIN or temporary table
UPDATE products p
JOIN (
    SELECT product_id, AVG(rating) as avg_rating
    FROM reviews
    GROUP BY product_id
) r ON p.product_id = r.product_id
SET p.avg_rating = r.avg_rating;
```

#### Avoiding Full Table Updates

```sql
-- For large tables, batch updates
-- ❌ BAD: Single massive update
UPDATE huge_table
SET processed = TRUE
WHERE status = 'pending';  -- Might be millions of rows

-- ✅ GOOD: Batch processing
-- Create index if not exists
CREATE INDEX idx_status ON huge_table(status) WHERE status = 'pending';

-- Update in batches
UPDATE huge_table
SET processed = TRUE
WHERE status = 'pending'
LIMIT 10000;  -- MySQL

-- Repeat until no rows affected

-- PostgreSQL batching (using CTE)
WITH batch AS (
    SELECT id
    FROM huge_table
    WHERE status = 'pending'
    LIMIT 10000
)
UPDATE huge_table h
SET processed = TRUE
FROM batch b
WHERE h.id = b.id;
```

### DELETE Statement

DELETE removes rows from a table.

#### Basic DELETE

```sql
-- Delete specific rows
DELETE FROM employees WHERE emp_id = 123;

-- Delete with multiple conditions
DELETE FROM logs
WHERE log_level = 'DEBUG'
    AND created_at < DATE_SUB(NOW(), INTERVAL 30 DAY);  -- MySQL
    -- AND created_at < CURRENT_TIMESTAMP - INTERVAL '30 days'  -- PostgreSQL

-- Delete all rows (slow, use TRUNCATE instead)
-- ❌ BAD on large tables
DELETE FROM temp_table;

-- ✅ BETTER
TRUNCATE TABLE temp_table;
```

#### DELETE vs TRUNCATE vs DROP

| Aspect | DELETE | TRUNCATE | DROP |
|--------|--------|----------|------|
| Removes rows | Yes (with WHERE) | Yes (all rows) | Yes (and table structure) |
| Removes table | No | No | Yes |
| WHERE clause | Supported | Not supported | N/A |
| Speed | Slow (row-by-row) | Fast (deallocates pages) | Very fast |
| Rollback | Yes (in transaction) | Sometimes (PostgreSQL yes, MySQL no) | Sometimes |
| Triggers | Fires DELETE triggers | No triggers fired | Fires DROP triggers |
| Auto-increment | Not reset | Reset | N/A |
| Disk space | Needs VACUUM/OPTIMIZE | Immediately released | Immediately released |

#### DELETE with JOIN

**MySQL:**

```sql
-- Delete using JOIN
DELETE e
FROM employees e
JOIN departments d ON e.department_id = d.dept_id
WHERE d.dept_name = 'Closed Department';

-- Delete from multiple tables
DELETE e, d
FROM employees e
JOIN departments d ON e.department_id = d.dept_id
WHERE d.dept_name = 'Closed Department';

-- Delete with LEFT JOIN (orphaned records)
DELETE o
FROM orders o
LEFT JOIN customers c ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

**PostgreSQL:**

```sql
-- PostgreSQL uses USING clause
DELETE FROM employees e
USING departments d
WHERE e.department_id = d.dept_id
    AND d.dept_name = 'Closed Department';

-- Delete orphaned records
DELETE FROM orders o
USING customers c
WHERE o.customer_id = c.customer_id
    AND c.customer_id IS NULL;

-- Alternative with subquery (works in both)
DELETE FROM employees
WHERE department_id IN (
    SELECT dept_id FROM departments WHERE dept_name = 'Closed Department'
);
```

#### DELETE ... RETURNING (PostgreSQL)

```sql
-- Return deleted rows
DELETE FROM employees
WHERE status = 'terminated'
RETURNING emp_id, first_name, last_name, termination_date;

-- Archive before deleting
WITH deleted AS (
    DELETE FROM employees
    WHERE termination_date < CURRENT_DATE - INTERVAL '7 years'
    RETURNING *
)
INSERT INTO employees_archive
SELECT * FROM deleted;
```

#### Cascading Deletes

```sql
-- Define CASCADE in foreign key
CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(dept_id)
        ON DELETE CASCADE  -- Automatically delete employees when department is deleted
);

-- When you delete a department, all related employees are deleted
DELETE FROM departments WHERE dept_id = 5;
-- All employees with department_id = 5 are automatically deleted

-- Other CASCADE options:
-- ON DELETE SET NULL - Sets foreign key to NULL
-- ON DELETE SET DEFAULT - Sets foreign key to default value
-- ON DELETE RESTRICT - Prevents deletion if references exist (default)
-- ON DELETE NO ACTION - Same as RESTRICT
```

#### Soft Delete Pattern

Better practice for important data:

```sql
-- Add deletion tracking columns
ALTER TABLE employees
ADD COLUMN deleted_at TIMESTAMP NULL,
ADD COLUMN deleted_by VARCHAR(50) NULL;

-- Soft delete function (PostgreSQL)
CREATE OR REPLACE FUNCTION soft_delete_employee(emp_id_param INT)
RETURNS VOID AS $$
BEGIN
    UPDATE employees
    SET 
        deleted_at = CURRENT_TIMESTAMP,
        deleted_by = CURRENT_USER,
        status = 'deleted'
    WHERE emp_id = emp_id_param
        AND deleted_at IS NULL;  -- Prevent double-deletion
END;
$$ LANGUAGE plpgsql;

-- Query non-deleted records
SELECT * FROM employees WHERE deleted_at IS NULL;

-- Create view for convenience
CREATE VIEW active_employees AS
SELECT * FROM employees WHERE deleted_at IS NULL;

-- Permanent cleanup (hard delete after retention period)
DELETE FROM employees
WHERE deleted_at < CURRENT_TIMESTAMP - INTERVAL '7 years';
```

### REPLACE Statement (MySQL-Specific)

REPLACE is MySQL's alternative to UPSERT:

```sql
-- REPLACE is equivalent to DELETE + INSERT
-- If row exists (based on PRIMARY KEY or UNIQUE index), delete it, then insert
REPLACE INTO employees (emp_id, first_name, last_name, email)
VALUES (1, 'John', 'Doe', 'john.doe@company.com');

-- ⚠️ WARNING: REPLACE deletes the entire row and re-inserts
-- This means:
-- 1. Auto-increment values may change
-- 2. Triggers fire twice (DELETE then INSERT)
-- 3. Columns not specified are set to DEFAULT, not preserved

-- Example problem:
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100),
    created_at TIMESTAMP,
    login_count INT DEFAULT 0
);

INSERT INTO users VALUES (1, 'johndoe', 'john@example.com', NOW(), 10);

-- This DELETES the row and creates new one
REPLACE INTO users (user_id, username, email)
VALUES (1, 'johndoe', 'newemail@example.com');
-- Result: login_count is reset to 0, created_at is reset!

-- ✅ BETTER: Use INSERT ... ON DUPLICATE KEY UPDATE
INSERT INTO users (user_id, username, email)
VALUES (1, 'johndoe', 'newemail@example.com')
ON DUPLICATE KEY UPDATE
    email = VALUES(email);
-- Result: only email is updated, login_count and created_at preserved
```

**When to use REPLACE:**
* When you truly want to replace the entire row
* When column defaults are acceptable
* Never for tables where you want to preserve non-key columns

**Performance implications:**
* REPLACE is slower than INSERT ... ON DUPLICATE KEY UPDATE
* Triggers fire twice
* Row-level locking may be more complex

### MERGE Statement

MERGE is part of SQL standard but has limited support:

**SQL Server / Oracle:**
```sql
-- Not natively supported in MySQL or PostgreSQL
MERGE INTO target_table t
USING source_table s
ON t.id = s.id
WHEN MATCHED THEN
    UPDATE SET t.value = s.value
WHEN NOT MATCHED THEN
    INSERT (id, value) VALUES (s.id, s.value);
```

**MySQL Workaround:**
```sql
-- Use INSERT ... ON DUPLICATE KEY UPDATE
INSERT INTO target_table (id, value)
SELECT id, value FROM source_table
ON DUPLICATE KEY UPDATE value = VALUES(value);
```

**PostgreSQL Workaround (PostgreSQL 15+):**
```sql
-- PostgreSQL 15 added MERGE support
MERGE INTO target_table t
USING source_table s
ON t.id = s.id
WHEN MATCHED THEN
    UPDATE SET value = s.value
WHEN NOT MATCHED THEN
    INSERT (id, value) VALUES (s.id, s.value);

-- Pre-PostgreSQL 15, use INSERT ... ON CONFLICT
INSERT INTO target_table (id, value)
SELECT id, value FROM source_table
ON CONFLICT (id) DO UPDATE
SET value = EXCLUDED.value;
```

---

### Common Pitfalls

1. **Missing WHERE clause in UPDATE/DELETE**
   ```sql
   -- ❌ DISASTER: Updates ALL rows
   UPDATE employees SET salary = 100000;
   
   -- ✅ SAFE: Use WHERE clause
   UPDATE employees SET salary = 100000 WHERE emp_id = 123;
   
   -- ✅ SAFER: Enable safe-update mode in MySQL
   SET sql_safe_updates = 1;
   -- Now UPDATE/DELETE without WHERE or LIMIT will fail
   ```

2. **Not using transactions for related DML**
   ```sql
   -- ❌ BAD: Data inconsistency if second INSERT fails
   INSERT INTO orders (customer_id, total) VALUES (123, 99.99);
   INSERT INTO order_items (order_id, product_id) VALUES (LAST_INSERT_ID(), 456);
   
   -- ✅ GOOD: Atomic transaction
   START TRANSACTION;
   INSERT INTO orders (customer_id, total) VALUES (123, 99.99);
   INSERT INTO order_items (order_id, product_id) VALUES (LAST_INSERT_ID(), 456);
   COMMIT;
   ```

3. **Using REPLACE when INSERT ... ON DUPLICATE KEY UPDATE is better**
   ```sql
   -- ❌ BAD: Resets all non-specified columns
   REPLACE INTO cache (cache_key, cache_value)
   VALUES ('key1', 'new_value');
   
   -- ✅ GOOD: Only updates specified columns
   INSERT INTO cache (cache_key, cache_value, created_at)
   VALUES ('key1', 'new_value', NOW())
   ON DUPLICATE KEY UPDATE
       cache_value = VALUES(cache_value),
       updated_at = NOW();
   ```

### Best Practices

✅ **Always use WHERE in UPDATE/DELETE** unless intentionally modifying all rows
✅ **Use transactions** for multi-statement DML operations
✅ **Batch large operations** to reduce lock time
✅ **Prefer INSERT ... ON CONFLICT** (PostgreSQL) or INSERT ... ON DUPLICATE KEY UPDATE (MySQL) over REPLACE
✅ **Use RETURNING/LAST_INSERT_ID()** to get inserted IDs
✅ **Test DML changes on staging** before production
✅ **Monitor replication lag** after large DML operations
✅ **Use soft deletes** for important business data
✅ **Add indexes** to columns used in WHERE clauses of frequent UPDATEs/DELETEs
✅ **Consider partitioning** for very large tables with frequent deletes

### Performance Implications

**INSERT Performance:**
* Single-row INSERT: ~100-500 ops/sec
* Batched INSERT: ~10,000-50,000 ops/sec
* LOAD DATA/COPY: ~100,000+ ops/sec
* Disable indexes during bulk load, rebuild after
* Use transactions to batch commits

**UPDATE Performance:**
* Depends on index overhead (each index must be updated)
* Row-level locking (InnoDB) vs table-level locking
* Triggers and foreign key checks slow updates
* Batch updates to reduce lock contention

**DELETE Performance:**
* Slower than TRUNCATE (row-by-row vs deallocate pages)
* Leaves gaps in table (needs VACUUM in PostgreSQL, OPTIMIZE in MySQL)
* Cascading deletes can be very slow
* Consider soft deletes for frequently deleted data

---

## Frequently Asked Questions

**Q1: What's the difference between TRUNCATE and DELETE?**

**A:** TRUNCATE is a DDL command that quickly removes all rows by deallocating data pages, while DELETE is a DML command that removes rows one by one. Key differences:
* **Speed**: TRUNCATE is much faster on large tables
* **Rollback**: DELETE can be rolled back in a transaction, TRUNCATE auto-commits in MySQL (but is transaction-safe in PostgreSQL)
* **Triggers**: DELETE fires row-level triggers, TRUNCATE does not
* **WHERE clause**: DELETE supports WHERE, TRUNCATE deletes all rows
* **AUTO_INCREMENT**: TRUNCATE resets auto-increment counters, DELETE does not

**Related Concepts**: DDL vs DML, Transaction management

---

**Q2: How do I handle UPSERT operations in MySQL vs PostgreSQL?**

**A:** Both databases support UPSERT but with different syntax:

**MySQL:**
```sql
INSERT INTO table (id, value) VALUES (1, 'new')
ON DUPLICATE KEY UPDATE value = VALUES(value);
```

**PostgreSQL:**
```sql
INSERT INTO table (id, value) VALUES (1, 'new')
ON CONFLICT (id) DO UPDATE SET value = EXCLUDED.value;
```

PostgreSQL's syntax is more powerful—you can specify which conflict to handle, add WHERE clauses to the UPDATE, and use DO NOTHING for ignore behavior.

---

**Q3: Why is my multi-row INSERT slower than expected?**

**A:** Common causes:
1. **Too many indexes**: Each index must be updated for every row
2. **Foreign key checks**: Validation happens for each row
3. **Triggers**: Before/after insert triggers execute for each row
4. **No transaction**: Each INSERT auto-commits separately
5. **Too small batches**: Network overhead dominates

**Solutions**:
* Batch rows in groups of 1000-5000
* Wrap in explicit transaction
* Temporarily disable non-critical indexes
* Use LOAD DATA (MySQL) or COPY (PostgreSQL) for very large loads
* Consider disabling foreign key checks temporarily for bulk loads (with caution)

---

## Interview Questions

**Question 1: Explain the difference between REPLACE and INSERT ... ON DUPLICATE KEY UPDATE in MySQL.**

**Difficulty:** Mid-Level

**Answer:**

REPLACE is essentially a DELETE followed by INSERT. If a row with the same primary key or unique index exists, it deletes that row entirely and inserts a new one. This means:
* All columns are reset to their new values or defaults
* Auto-increment may change
* DELETE and INSERT triggers both fire

INSERT ... ON DUPLICATE KEY UPDATE is a true upsert—it only updates the specified columns when a duplicate key is found:
* Non-specified columns retain their original values
* Only INSERT and UPDATE triggers fire (no DELETE)
* Generally more efficient

```sql
-- Example demonstrating the difference
CREATE TABLE test (
    id INT PRIMARY KEY,
    value VARCHAR(50),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

INSERT INTO test VALUES (1, 'original', NOW(), NOW());

-- REPLACE: created_at will be reset
REPLACE INTO test (id, value) VALUES (1, 'replaced');

-- ON DUPLICATE KEY UPDATE: created_at preserved
INSERT INTO test (id, value) VALUES (1, 'updated')
ON DUPLICATE KEY UPDATE value = VALUES(value), updated_at = NOW();
```

**Why This Matters:** In production systems, preserving created timestamps, audit fields, and other metadata is often critical. Choosing the wrong approach can lead to data integrity issues.

**Follow-up Questions:**
* How would you implement soft deletes with REPLACE?
* What performance implications does REPLACE have vs ON DUPLICATE KEY UPDATE?

---

**Question 2: How would you safely delete millions of rows from a production table?**

**Difficulty:** Senior

**Answer:**

Deleting millions of rows at once can:
* Lock the table for extended periods
* Fill up transaction logs
* Block other queries
* Cause replication lag

**Safe approach - Batched deletes:**

```sql
-- 1. Add an index on delete condition if not exists
CREATE INDEX idx_delete_condition ON large_table(status, created_at);

-- 2. Delete in batches with delays
DELIMITER $$
CREATE PROCEDURE safe_delete_old_records()
BEGIN
    DECLARE deleted_rows INT DEFAULT 1;
    
    WHILE deleted_rows > 0 DO
        DELETE FROM large_table
        WHERE status = 'archived'
            AND created_at < DATE_SUB(NOW(), INTERVAL 1 YEAR)
        LIMIT 10000;
        
        SET deleted_rows = ROW_COUNT();
        
        -- Small delay to reduce load
        SELECT SLEEP(0.1);
    END WHILE;
END$$
DELIMITER ;

CALL safe_delete_old_records();
```

**PostgreSQL approach:**
```sql
DO $$
DECLARE
    deleted_count INTEGER;
BEGIN
    LOOP
        DELETE FROM large_table
        WHERE ctid IN (
            SELECT ctid FROM large_table
            WHERE status = 'archived'
                AND created_at < CURRENT_DATE - INTERVAL '1 year'
            LIMIT 10000
        );
        
        GET DIAGNOSTICS deleted_count = ROW_COUNT;
        EXIT WHEN deleted_count = 0;
        
        PERFORM pg_sleep(0.1);
        COMMIT;  -- Release locks between batches
    END LOOP;
END$$;
```

**Alternative - Partitioning:**
If possible, use table partitioning and simply drop old partitions:
```sql
-- Much faster than DELETE
ALTER TABLE large_table DROP PARTITION p_old_data;
```

**Why This Matters:** Large DELETE operations can bring down production systems. Understanding how to perform them safely is crucial for database reliability.

**Follow-up Questions:**
* How would you monitor progress of the deletion?
* What's the impact on replication?
* How would you handle this if the table has foreign key references?

---

**Question 3: Explain the behavior of AUTO_INCREMENT in INSERT ... ON DUPLICATE KEY UPDATE.**

**Difficulty:** Mid-Level

**Answer:**

When INSERT ... ON DUPLICATE KEY UPDATE encounters a duplicate key:
* MySQL allocates an AUTO_INCREMENT value
* MySQL performs the UPDATE instead of INSERT
* The AUTO_INCREMENT counter is **not** rolled back
* This creates gaps in the sequence

```sql
CREATE TABLE test (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    name VARCHAR(50)
);

INSERT INTO test (email, name) VALUES ('john@example.com', 'John');
-- id = 1

INSERT INTO test (email, name) VALUES ('john@example.com', 'John Doe')
ON DUPLICATE KEY UPDATE name = VALUES(name);
-- Update happens, but AUTO_INCREMENT advances to 2

INSERT INTO test (email, name) VALUES ('jane@example.com', 'Jane');
-- id = 3 (skipped 2!)

SELECT * FROM test;
-- 1, john@example.com, John Doe
-- 3, jane@example.com, Jane
```

This behavior is by design for performance reasons (to avoid locking the AUTO_INCREMENT counter during the check for duplicates).

**Implications:**
* Gaps in ID sequences are normal and acceptable
* Don't rely on consecutive IDs
* This is not a bug—it's expected behavior

**Why This Matters:** Understanding this prevents unnecessary debugging and alarm about "missing" IDs in production systems.

**Follow-up Questions:**
* How does PostgreSQL handle this with sequences?
* Is this behavior different with InnoDB vs MyISAM?
* How would you avoid gaps if they're truly problematic for your use case?

---

## Key Takeaways

* **DML commands (INSERT, UPDATE, DELETE) manipulate data** and can be rolled back within transactions, unlike DDL commands
* **Use batching** for bulk operations to dramatically improve performance
* **UPSERT syntax differs** between MySQL (ON DUPLICATE KEY UPDATE) and PostgreSQL (ON CONFLICT)
* **RETURNING clause** (PostgreSQL) is more powerful than LAST_INSERT_ID() (MySQL)
* **REPLACE is dangerous** and usually INSERT ... ON DUPLICATE KEY UPDATE is better
* **Always use WHERE clauses** in UPDATE/DELETE unless you truly intend to modify all rows
* **Soft deletes** are often better than hard deletes for business-critical data
* **Transactions are essential** for multi-step DML operations to maintain data integrity
* **Batch large operations** to avoid long locks and reduce replication lag
* **Monitor AUTO_INCREMENT behavior** in UPSERT operations to understand ID gaps

---

## 1.5 DQL - Data Query Language

Data Query Language (DQL) consists primarily of the SELECT statement—the most used SQL command. While we've covered SELECT basics in section 1.2, this section dives deep into advanced SELECT operations, including filtering, ordering, pagination, and complex expressions.

### SELECT Statement Deep Dive

The SELECT statement retrieves data from one or more tables. Understanding its full syntax and execution order is crucial for writing efficient queries.

**Complete SELECT Syntax:**

```sql
SELECT [DISTINCT] column_list
FROM table_name
[JOIN other_tables ON join_conditions]
[WHERE filter_conditions]
[GROUP BY grouping_columns]
[HAVING group_filter_conditions]
[ORDER BY sort_columns [ASC|DESC]]
[LIMIT count [OFFSET skip]];
```

#### Logical Processing Order

SQL processes SELECT statements in a specific logical order (different from the written order):

1. **FROM** - Identify source tables
2. **JOIN** - Combine tables based on conditions
3. **WHERE** - Filter individual rows
4. **GROUP BY** - Group rows with common values
5. **HAVING** - Filter groups
6. **SELECT** - Choose columns and apply expressions
7. **DISTINCT** - Remove duplicates
8. **ORDER BY** - Sort results
9. **LIMIT/OFFSET** - Restrict number of rows returned

**Example demonstrating the order:**

```sql
-- Written order
SELECT 
    department,
    COUNT(*) as emp_count,
    AVG(salary) as avg_salary
FROM employees
WHERE status = 'active'
GROUP BY department
HAVING AVG(salary) > 50000
ORDER BY emp_count DESC
LIMIT 10;

-- Logical execution order:
-- 1. FROM employees
-- 2. WHERE status = 'active' (filter rows first)
-- 3. GROUP BY department (create groups)
-- 4. HAVING AVG(salary) > 50000 (filter groups)
-- 5. SELECT department, COUNT(*), AVG(salary) (compute aggregates)
-- 6. ORDER BY emp_count DESC (sort results)
-- 7. LIMIT 10 (return top 10)
```

**Why this matters**: Understanding logical order helps you:
* Debug complex queries
* Understand why certain column aliases work in ORDER BY but not in WHERE
* Optimize query performance
* Reason about query correctness

### Column Selection and Aliasing

```sql
-- Select specific columns
SELECT first_name, last_name, email FROM employees;

-- Select all columns (avoid in production)
SELECT * FROM employees;

-- Column aliases
SELECT 
    first_name AS "First Name",  -- With spaces, use quotes
    last_name AS surname,         -- Without AS keyword (less readable)
    email "Email Address"         -- Without AS and with quotes
FROM employees;

-- Table aliases
SELECT 
    e.first_name,
    e.last_name,
    d.dept_name
FROM employees e  -- Alias for table
JOIN departments d ON e.department_id = d.dept_id;

-- Computed columns
SELECT 
    first_name,
    last_name,
    salary,
    salary * 1.10 AS salary_with_raise,
    CONCAT(first_name, ' ', last_name) AS full_name
FROM employees;
```

**Best Practices:**
* Always specify column names explicitly (avoid SELECT *)
* Use meaningful aliases
* Use AS keyword for clarity
* Use table aliases for multi-table queries

### DISTINCT and Duplicate Handling

```sql
-- Get unique values
SELECT DISTINCT department_id FROM employees;

-- DISTINCT on multiple columns (unique combinations)
SELECT DISTINCT department_id, job_title FROM employees;

-- Count unique values
SELECT COUNT(DISTINCT department_id) as unique_departments
FROM employees;

-- ❌ COMMON MISTAKE: DISTINCT with aggregate
-- This is usually wrong
SELECT DISTINCT department_id, COUNT(*) 
FROM employees 
GROUP BY department_id;

-- ✅ CORRECT: Remove DISTINCT
SELECT department_id, COUNT(*) 
FROM employees 
GROUP BY department_id;
```

**MySQL vs PostgreSQL DISTINCT:**

| Feature | MySQL | PostgreSQL |
|---------|-------|------------|
| Basic DISTINCT | Supported | Supported |
| DISTINCT ON | Not supported | Supported (PostgreSQL-specific) |
| Performance | Uses temporary table or index | Uses hash or sort |

**PostgreSQL DISTINCT ON (powerful feature):**

```sql
-- Get first employee in each department (by hire_date)
SELECT DISTINCT ON (department_id)
    department_id,
    first_name,
    last_name,
    hire_date
FROM employees
ORDER BY department_id, hire_date ASC;

-- DISTINCT ON must match leftmost ORDER BY columns
-- Returns one row per unique department_id (the earliest hired)
```

### Expression Evaluation and Computed Columns

SQL supports rich expressions in SELECT:

**String Functions:**

```sql
-- MySQL string functions
SELECT 
    CONCAT(first_name, ' ', last_name) AS full_name,
    UPPER(email) AS email_upper,
    LOWER(first_name) AS first_name_lower,
    LENGTH(last_name) AS name_length,
    SUBSTRING(email, 1, LOCATE('@', email) - 1) AS username,
    REPLACE(phone, '-', '') AS phone_digits,
    TRIM(first_name) AS trimmed_name,
    LEFT(last_name, 3) AS initials
FROM employees;

-- PostgreSQL string functions (mostly same, some differences)
SELECT 
    first_name || ' ' || last_name AS full_name,  -- Concatenation operator
    UPPER(email) AS email_upper,
    LOWER(first_name) AS first_name_lower,
    LENGTH(last_name) AS name_length,
    SUBSTRING(email, 1, POSITION('@' IN email) - 1) AS username,
    REPLACE(phone, '-', '') AS phone_digits,
    TRIM(first_name) AS trimmed_name,
    LEFT(last_name, 3) AS initials
FROM employees;
```

**Date Functions:**

```sql
-- MySQL date functions
SELECT 
    CURDATE() AS current_date,
    CURTIME() AS current_time,
    NOW() AS current_timestamp,
    DATE(created_at) AS creation_date,
    YEAR(hire_date) AS hire_year,
    MONTH(hire_date) AS hire_month,
    DAY(hire_date) AS hire_day,
    TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) AS years_employed,
    DATE_ADD(hire_date, INTERVAL 1 YEAR) AS first_anniversary,
    DATE_SUB(NOW(), INTERVAL 30 DAY) AS thirty_days_ago,
    DATE_FORMAT(created_at, '%Y-%m-%d %H:%i:%s') AS formatted_date
FROM employees;

-- PostgreSQL date functions
SELECT 
    CURRENT_DATE AS current_date,
    CURRENT_TIME AS current_time,
    CURRENT_TIMESTAMP AS current_timestamp,
    created_at::DATE AS creation_date,
    EXTRACT(YEAR FROM hire_date) AS hire_year,
    EXTRACT(MONTH FROM hire_date) AS hire_month,
    EXTRACT(DAY FROM hire_date) AS hire_day,
    EXTRACT(YEAR FROM AGE(CURRENT_DATE, hire_date)) AS years_employed,
    hire_date + INTERVAL '1 year' AS first_anniversary,
    CURRENT_TIMESTAMP - INTERVAL '30 days' AS thirty_days_ago,
    TO_CHAR(created_at, 'YYYY-MM-DD HH24:MI:SS') AS formatted_date
FROM employees;
```

**Mathematical Functions:**

```sql
-- Both MySQL and PostgreSQL
SELECT 
    salary,
    ROUND(salary, 2) AS rounded_salary,
    CEIL(salary) AS ceiling_salary,
    FLOOR(salary) AS floor_salary,
    ABS(-100) AS absolute_value,
    POWER(2, 3) AS two_cubed,
    SQRT(144) AS square_root,
    MOD(10, 3) AS modulo,
    salary * 0.15 AS tax_amount,
    salary - (salary * 0.15) AS after_tax_salary
FROM employees;
```

### Type Casting and Conversion

Converting between data types is common in queries:

**MySQL Type Casting:**

```sql
-- CAST function (standard SQL)
SELECT 
    CAST(salary AS CHAR) AS salary_string,
    CAST('123' AS SIGNED) AS number,
    CAST('2024-01-15' AS DATE) AS date_value,
    CAST(price AS DECIMAL(10,2)) AS precise_price
FROM employees;

-- CONVERT function (MySQL-specific)
SELECT 
    CONVERT(salary, CHAR) AS salary_string,
    CONVERT('123', SIGNED) AS number,
    CONVERT('2024-01-15', DATE) AS date_value
FROM employees;

-- Implicit conversion (be careful!)
SELECT * FROM employees WHERE emp_id = '123';  -- String auto-converted to INT
```

**PostgreSQL Type Casting:**

```sql
-- CAST function (standard SQL)
SELECT 
    CAST(salary AS TEXT) AS salary_string,
    CAST('123' AS INTEGER) AS number,
    CAST('2024-01-15' AS DATE) AS date_value,
    CAST(price AS NUMERIC(10,2)) AS precise_price
FROM employees;

-- PostgreSQL :: operator (preferred in PostgreSQL)
SELECT 
    salary::TEXT AS salary_string,
    '123'::INTEGER AS number,
    '2024-01-15'::DATE AS date_value,
    price::NUMERIC(10,2) AS precise_price
FROM employees;

-- String to JSON
SELECT 
    '{"name":"John","age":30}'::JSON AS json_data,
    '{"name":"John","age":30}'::JSONB AS jsonb_data
FROM employees;
```

**Performance Note**: Implicit type conversions can prevent index usage:

```sql
-- ❌ BAD: Function on indexed column
SELECT * FROM employees WHERE CAST(emp_id AS CHAR) = '123';

-- ✅ GOOD: Cast the literal instead
SELECT * FROM employees WHERE emp_id = CAST('123' AS SIGNED);
-- Or better yet, use correct type
SELECT * FROM employees WHERE emp_id = 123;
```

### NULL Handling

NULL represents missing or unknown data. NULL handling is critical for correct queries:

**IS NULL and IS NOT NULL:**

```sql
-- Check for NULL values
SELECT * FROM employees WHERE middle_name IS NULL;

-- Check for non-NULL values
SELECT * FROM employees WHERE middle_name IS NOT NULL;

-- ❌ WRONG: This doesn't work for NULL
SELECT * FROM employees WHERE middle_name = NULL;  -- Always returns 0 rows

-- ❌ WRONG: This also doesn't work
SELECT * FROM employees WHERE middle_name != NULL;  -- Always returns 0 rows
```

**COALESCE Function:**

Returns the first non-NULL value in the list:

```sql
-- Provide default value for NULL
SELECT 
    first_name,
    last_name,
    COALESCE(middle_name, '') AS middle_name,
    COALESCE(phone, 'No phone') AS phone,
    COALESCE(bonus, 0) AS bonus
FROM employees;

-- Multiple fallback values
SELECT 
    COALESCE(mobile_phone, work_phone, home_phone, 'No phone') AS contact_phone
FROM employees;

-- Use in calculations
SELECT 
    salary + COALESCE(bonus, 0) AS total_compensation
FROM employees;
```

**IFNULL (MySQL) vs COALESCE:**

```sql
-- MySQL IFNULL (only 2 arguments)
SELECT 
    IFNULL(middle_name, '') AS middle_name,
    IFNULL(bonus, 0) AS bonus
FROM employees;

-- COALESCE is standard SQL and works in both MySQL and PostgreSQL
-- Prefer COALESCE for portability
```

**NULLIF Function:**

Returns NULL if two values are equal, otherwise returns the first value:

```sql
-- Avoid division by zero
SELECT 
    total_sales,
    total_orders,
    total_sales / NULLIF(total_orders, 0) AS avg_order_value
FROM sales_summary;

-- Convert empty strings to NULL
SELECT 
    NULLIF(middle_name, '') AS middle_name
FROM employees;
```

**Three-Valued Logic (TRUE, FALSE, NULL):**

```sql
-- AND truth table
TRUE AND TRUE = TRUE
TRUE AND FALSE = FALSE
TRUE AND NULL = NULL
FALSE AND NULL = FALSE
NULL AND NULL = NULL

-- OR truth table
TRUE OR FALSE = TRUE
TRUE OR NULL = TRUE
FALSE OR NULL = NULL
NULL OR NULL = NULL

-- NOT
NOT TRUE = FALSE
NOT FALSE = TRUE
NOT NULL = NULL

-- Practical example
SELECT * FROM employees
WHERE (status = 'active' OR status IS NULL)
  AND (department_id = 5 OR department_id IS NULL);
```

### CASE Expressions

CASE expressions provide conditional logic in SQL:

**Simple CASE:**

```sql
-- Simple CASE (equality check)
SELECT 
    first_name,
    status,
    CASE status
        WHEN 'active' THEN 'Active Employee'
        WHEN 'inactive' THEN 'Inactive Employee'
        WHEN 'suspended' THEN 'Suspended Employee'
        ELSE 'Unknown Status'
    END AS status_description
FROM employees;
```

**Searched CASE (more flexible):**

```sql
-- Searched CASE (any boolean expression)
SELECT 
    first_name,
    salary,
    CASE
        WHEN salary >= 100000 THEN 'High'
        WHEN salary >= 50000 THEN 'Medium'
        WHEN salary >= 25000 THEN 'Low'
        ELSE 'Very Low'
    END AS salary_category,
    CASE
        WHEN TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) >= 10 THEN 'Veteran'
        WHEN TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) >= 5 THEN 'Senior'
        WHEN TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) >= 2 THEN 'Intermediate'
        ELSE 'Junior'
    END AS seniority_level
FROM employees;
```

**CASE in Aggregations:**

```sql
-- Conditional counting
SELECT 
    department_id,
    COUNT(*) AS total_employees,
    COUNT(CASE WHEN status = 'active' THEN 1 END) AS active_count,
    COUNT(CASE WHEN status = 'inactive' THEN 1 END) AS inactive_count,
    COUNT(CASE WHEN salary > 50000 THEN 1 END) AS high_earners
FROM employees
GROUP BY department_id;

-- Conditional sum
SELECT 
    SUM(CASE WHEN status = 'active' THEN salary ELSE 0 END) AS active_payroll,
    SUM(CASE WHEN status = 'inactive' THEN salary ELSE 0 END) AS inactive_payroll
FROM employees;
```

**Pivot Table with CASE:**

```sql
-- Transform rows to columns
SELECT 
    department_id,
    SUM(CASE WHEN YEAR(hire_date) = 2022 THEN 1 ELSE 0 END) AS hired_2022,
    SUM(CASE WHEN YEAR(hire_date) = 2023 THEN 1 ELSE 0 END) AS hired_2023,
    SUM(CASE WHEN YEAR(hire_date) = 2024 THEN 1 ELSE 0 END) AS hired_2024
FROM employees
GROUP BY department_id;
```

**CASE for Sorting:**

```sql
-- Custom sort order
SELECT first_name, status
FROM employees
ORDER BY 
    CASE status
        WHEN 'active' THEN 1
        WHEN 'suspended' THEN 2
        WHEN 'inactive' THEN 3
        ELSE 4
    END,
    first_name;
```

---

## 1.6 DCL - Data Control Language

Data Control Language (DCL) commands manage database security and access control. These commands are typically used by database administrators to control who can access and modify data.

### GRANT Command

GRANT gives privileges to users or roles.

**Privilege Types:**

* **Data privileges**: SELECT, INSERT, UPDATE, DELETE
* **Schema privileges**: CREATE, ALTER, DROP
* **Administrative privileges**: SUPER, RELOAD, SHUTDOWN, etc.

**MySQL GRANT Syntax:**

```sql
-- Grant specific privileges on a table
GRANT SELECT, INSERT ON database_name.table_name 
TO 'username'@'host';

-- Grant all privileges on a database
GRANT ALL PRIVILEGES ON database_name.* 
TO 'username'@'localhost';

-- Grant specific column privileges
GRANT SELECT (emp_id, first_name, last_name), 
      UPDATE (email) 
ON employees 
TO 'analyst'@'localhost';

-- Grant with ability to grant to others
GRANT SELECT ON employees TO 'manager'@'localhost' 
WITH GRANT OPTION;

-- Grant execute on stored procedure
GRANT EXECUTE ON PROCEDURE calculate_bonus 
TO 'app_user'@'%';

-- Common privilege combinations
-- Read-only user
GRANT SELECT ON company_db.* TO 'readonly_user'@'localhost';

-- Application user (read/write)
GRANT SELECT, INSERT, UPDATE, DELETE ON company_db.* 
TO 'app_user'@'localhost';

-- Full control (DBA)
GRANT ALL PRIVILEGES ON *.* TO 'dba_user'@'localhost' 
WITH GRANT OPTION;
```

**PostgreSQL GRANT Syntax:**

```sql
-- Grant privileges on table
GRANT SELECT, INSERT ON employees TO analyst_user;

-- Grant all privileges on table
GRANT ALL PRIVILEGES ON employees TO admin_user;

-- Grant on all tables in schema
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;

-- Grant for future tables
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO readonly_user;

-- Grant schema usage
GRANT USAGE ON SCHEMA public TO app_user;

-- Grant sequence privileges
GRANT USAGE, SELECT ON SEQUENCE employees_emp_id_seq TO app_user;

-- Grant execute on function
GRANT EXECUTE ON FUNCTION calculate_bonus(INT, NUMERIC) TO app_user;

-- Role-based grants
GRANT readonly_role TO user1, user2, user3;
```

**Object-Level vs Database-Level vs Global Permissions:**

```sql
-- MySQL hierarchy
-- Global (all databases)
GRANT SELECT ON *.* TO 'user'@'localhost';

-- Database level
GRANT SELECT ON database_name.* TO 'user'@'localhost';

-- Table level
GRANT SELECT ON database_name.table_name TO 'user'@'localhost';

-- Column level
GRANT SELECT (column1, column2) ON database_name.table_name TO 'user'@'localhost';

-- PostgreSQL hierarchy
-- Database level (via CONNECT)
GRANT CONNECT ON DATABASE database_name TO user;

-- Schema level
GRANT USAGE ON SCHEMA schema_name TO user;

-- Table level
GRANT SELECT ON schema_name.table_name TO user;

-- Column level
GRANT SELECT (column1, column2) ON schema_name.table_name TO user;
```

### REVOKE Command

REVOKE removes previously granted privileges:

**MySQL REVOKE:**

```sql
-- Revoke specific privileges
REVOKE SELECT, INSERT ON database_name.table_name 
FROM 'username'@'host';

-- Revoke all privileges
REVOKE ALL PRIVILEGES ON database_name.* 
FROM 'username'@'localhost';

-- Revoke grant option
REVOKE GRANT OPTION ON database_name.* 
FROM 'username'@'localhost';

-- Revoke cascade (removes dependent grants)
REVOKE SELECT ON employees FROM 'user1'@'localhost';
-- If user1 granted SELECT to user2 with GRANT OPTION,
-- user2's grant may also be revoked (implementation-dependent)
```

**PostgreSQL REVOKE:**

```sql
-- Revoke privileges
REVOKE SELECT, INSERT ON employees FROM analyst_user;

-- Revoke all
REVOKE ALL PRIVILEGES ON employees FROM analyst_user;

-- Revoke with cascade (removes dependent privileges)
REVOKE admin_role FROM user1 CASCADE;

-- Revoke grant option only
REVOKE GRANT OPTION FOR SELECT ON employees FROM user1;

-- Revoke from PUBLIC (all users)
REVOKE SELECT ON sensitive_table FROM PUBLIC;
```

### User Management

**MySQL User Management:**

```sql
-- Create user
CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'secure_password';

-- Create with specific authentication
CREATE USER 'newuser'@'localhost' 
IDENTIFIED WITH mysql_native_password BY 'password';

-- Alter user password
ALTER USER 'username'@'localhost' IDENTIFIED BY 'new_password';

-- Password expiration
ALTER USER 'username'@'localhost' PASSWORD EXPIRE INTERVAL 90 DAY;

-- Account locking
ALTER USER 'username'@'localhost' ACCOUNT LOCK;
ALTER USER 'username'@'localhost' ACCOUNT UNLOCK;

-- Resource limits
CREATE USER 'limited_user'@'localhost' 
IDENTIFIED BY 'password'
WITH MAX_QUERIES_PER_HOUR 1000
     MAX_UPDATES_PER_HOUR 500
     MAX_CONNECTIONS_PER_HOUR 50
     MAX_USER_CONNECTIONS 10;

-- Drop user
DROP USER 'username'@'localhost';

-- Drop if exists
DROP USER IF EXISTS 'username'@'localhost';
```

**PostgreSQL User Management:**

```sql
-- Create role (user)
CREATE ROLE newuser WITH LOGIN PASSWORD 'secure_password';

-- Create with attributes
CREATE ROLE admin_user WITH
    LOGIN
    SUPERUSER
    CREATEDB
    CREATEROLE
    PASSWORD 'admin_password'
    VALID UNTIL '2025-12-31'
    CONNECTION LIMIT 10;

-- Alter role
ALTER ROLE username WITH PASSWORD 'new_password';

-- Alter role attributes
ALTER ROLE username WITH SUPERUSER CREATEDB;
ALTER ROLE username WITH NOSUPERUSER NOCREATEDB;

-- Account locking (disable login)
ALTER ROLE username WITH NOLOGIN;
ALTER ROLE username WITH LOGIN;

-- Drop role
DROP ROLE username;

-- Drop if exists
DROP ROLE IF EXISTS username;

-- Rename role
ALTER ROLE old_name RENAME TO new_name;
```

### MySQL Privilege System

**Authentication Plugins:**

```sql
-- View authentication plugin
SELECT user, host, plugin FROM mysql.user;

-- Available plugins:
-- - mysql_native_password (traditional)
-- - caching_sha2_password (default in MySQL 8.0+)
-- - auth_socket (Unix socket authentication)
-- - sha256_password

-- Change authentication plugin
ALTER USER 'username'@'localhost' 
IDENTIFIED WITH caching_sha2_password BY 'password';
```

**mysql.user Table:**

```sql
-- View user privileges
SELECT * FROM mysql.user WHERE user = 'username'\G

-- Important columns:
-- - Host: Allowed connection host
-- - Select_priv, Insert_priv, Update_priv, Delete_priv: Data privileges
-- - Create_priv, Drop_priv, Alter_priv: DDL privileges
-- - Grant_priv: Can grant privileges to others
-- - max_questions, max_updates, max_connections: Resource limits
```

**FLUSH PRIVILEGES:**

```sql
-- Reload grant tables from mysql.user, mysql.db, etc.
-- Usually not needed after GRANT/REVOKE (they auto-flush)
-- Needed after direct manipulation of mysql.* tables
FLUSH PRIVILEGES;

-- Direct manipulation example (not recommended)
UPDATE mysql.user SET Select_priv = 'Y' 
WHERE user = 'username' AND host = 'localhost';
FLUSH PRIVILEGES;  -- Required!
```

**SHOW GRANTS:**

```sql
-- Show grants for current user
SHOW GRANTS;

-- Show grants for specific user
SHOW GRANTS FOR 'username'@'localhost';

-- Example output:
-- GRANT SELECT, INSERT ON company_db.* TO 'app_user'@'localhost'
-- GRANT EXECUTE ON PROCEDURE company_db.calculate_bonus TO 'app_user'@'localhost'
```

### PostgreSQL Privilege System

**Roles vs Users:**

In PostgreSQL, users and roles are the same object. The only difference is the LOGIN attribute:

```sql
-- These are equivalent
CREATE USER username WITH PASSWORD 'password';
CREATE ROLE username WITH LOGIN PASSWORD 'password';

-- Role without LOGIN (group role)
CREATE ROLE developers;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO developers;

-- Users can be members of roles
GRANT developers TO user1, user2, user3;

-- Users inherit permissions from their roles
SET ROLE developers;  -- Temporarily assume role
RESET ROLE;           -- Return to original role
```

**Role Inheritance:**

```sql
-- Create role hierarchy
CREATE ROLE readonly_users;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_users;

CREATE ROLE power_users WITH INHERIT;
GRANT readonly_users TO power_users;
GRANT INSERT, UPDATE ON specific_table TO power_users;

CREATE ROLE john WITH LOGIN PASSWORD 'pass' IN ROLE power_users;

-- john inherits:
-- - SELECT on all tables (from readonly_users via power_users)
-- - INSERT, UPDATE on specific_table (from power_users)
```

**pg_hba.conf Configuration:**

PostgreSQL uses pg_hba.conf to control client authentication:

```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
# Local connections
local   all             postgres                                peer
local   all             all                                     md5

# IPv4 local connections
host    all             all             127.0.0.1/32            md5

# IPv4 connections from specific network
host    company_db      app_user        192.168.1.0/24          md5

# SSL connections only
hostssl all             all             0.0.0.0/0               md5

# Reject connections from specific IP
host    all             all             10.0.0.100/32           reject

# Methods:
# - trust: Allow without password (dangerous!)
# - reject: Reject connection
# - md5: Password authentication
# - scram-sha-256: More secure password authentication
# - peer: Use OS username (Unix sockets only)
# - cert: SSL certificate authentication
```

**Row-Level Security (RLS):**

PostgreSQL supports row-level security policies:

```sql
-- Enable RLS on table
ALTER TABLE employees ENABLE ROW LEVEL SECURITY;

-- Create policy: users can only see their own records
CREATE POLICY user_own_records ON employees
FOR SELECT
TO app_users
USING (user_id = CURRENT_USER::INT);

-- Create policy: managers can see all in their department
CREATE POLICY manager_department_records ON employees
FOR SELECT
TO managers
USING (
    department_id IN (
        SELECT department_id FROM manager_assignments 
        WHERE manager = CURRENT_USER
    )
);

-- Policies for INSERT/UPDATE/DELETE
CREATE POLICY user_insert ON employees
FOR INSERT
TO app_users
WITH CHECK (user_id = CURRENT_USER::INT);

-- Disable RLS for superusers
ALTER TABLE employees FORCE ROW LEVEL SECURITY;  -- Even superusers follow policies

-- View policies
\d+ employees
SELECT * FROM pg_policies WHERE tablename = 'employees';
```

**Schema-Level Permissions:**

```sql
-- Grant schema usage
GRANT USAGE ON SCHEMA public TO readonly_user;

-- Grant create in schema
GRANT CREATE ON SCHEMA public TO developer_user;

-- Revoke public schema access
REVOKE ALL ON SCHEMA public FROM PUBLIC;

-- Grant all on schema
GRANT ALL ON SCHEMA public TO admin_user;
```

---

## Frequently Asked Questions

**Q1: What's the difference between GRANT and user creation?**

**A:** CREATE USER creates a user account, while GRANT assigns privileges to that account. In MySQL, you must create the user first, then grant privileges:

```sql
-- MySQL
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT, INSERT ON database.table TO 'app_user'@'localhost';
```

In PostgreSQL, CREATE ROLE can do both if you include LOGIN:

```sql
-- PostgreSQL - creates user with login ability
CREATE ROLE app_user WITH LOGIN PASSWORD 'password';
GRANT SELECT, INSERT ON table TO app_user;
```

---

**Q2: Why can't my user connect even after GRANT ALL PRIVILEGES?**

**A:** Common causes:

1. **Host mismatch** (MySQL):
```sql
-- This only allows localhost connections
GRANT ALL ON db.* TO 'user'@'localhost';

-- For remote connections, use:
GRANT ALL ON db.* TO 'user'@'%';  -- Any host
GRANT ALL ON db.* TO 'user'@'192.168.1.%';  -- Specific network
```

2. **Missing database CONNECT privilege** (PostgreSQL):
```sql
GRANT CONNECT ON DATABASE mydb TO username;
GRANT USAGE ON SCHEMA public TO username;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO username;
```

3. **pg_hba.conf restrictions** (PostgreSQL):
```
# Check pg_hba.conf allows connections from user's IP
host    mydb    username    192.168.1.0/24    md5
```

4. **Firewall blocking** database port (3306 for MySQL, 5432 for PostgreSQL)

---

**Q3: How do I grant permissions on future tables?**

**A:** 

**MySQL**: Not directly supported. You must grant on tables as they're created, or grant on the entire database:

```sql
-- Grant on all current and future tables in database
GRANT SELECT ON database_name.* TO 'user'@'localhost';
```

**PostgreSQL**: Use ALTER DEFAULT PRIVILEGES:

```sql
-- Grant SELECT on all future tables created by current role
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO readonly_user;

-- Grant for tables created by specific role
ALTER DEFAULT PRIVILEGES FOR ROLE admin_user IN SCHEMA public
GRANT SELECT ON TABLES TO readonly_user;
```

---

## Interview Questions

**Question 1: Explain the principle of least privilege and how you would implement it for a new application user.**

**Difficulty:** Mid-Level

**Answer:**

The principle of least privilege means granting users only the minimum permissions required to perform their job. For a new application user:

```sql
-- ❌ BAD: Too broad
GRANT ALL PRIVILEGES ON *.* TO 'app_user'@'%';

-- ✅ GOOD: Minimal necessary privileges
-- 1. Create user with strong password
CREATE USER 'app_user'@'192.168.1.%' 
IDENTIFIED WITH caching_sha2_password BY 'strong_random_password'
PASSWORD EXPIRE INTERVAL 90 DAY
MAX_USER_CONNECTIONS 20;

-- 2. Grant only needed privileges on specific tables
GRANT SELECT, INSERT, UPDATE ON app_db.users TO 'app_user'@'192.168.1.%';
GRANT SELECT, INSERT ON app_db.orders TO 'app_user'@'192.168.1.%';
GRANT SELECT ON app_db.products TO 'app_user'@'192.168.1.%';

-- 3. Grant execute on specific stored procedures only
GRANT EXECUTE ON PROCEDURE app_db.process_order TO 'app_user'@'192.168.1.%';

-- 4. Never grant FILE, SUPER, or other admin privileges
```

**PostgreSQL approach:**

```sql
-- Use roles for permission management
CREATE ROLE app_readonly;
GRANT CONNECT ON DATABASE app_db TO app_readonly;
GRANT USAGE ON SCHEMA public TO app_readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO app_readonly;

CREATE ROLE app_readwrite;
GRANT app_readonly TO app_readwrite;  -- Inherit readonly
GRANT INSERT, UPDATE ON users, orders TO app_readwrite;

CREATE USER app_user1 WITH LOGIN PASSWORD 'pass' IN ROLE app_readwrite;

-- Row-level security for multi-tenant apps
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
CREATE POLICY user_orders ON orders
FOR ALL TO app_readwrite
USING (user_id = CURRENT_USER::INT);
```

**Why This Matters:** Over-privileged accounts are a major security risk. If an application is compromised, attackers gain only the minimal privileges granted, limiting potential damage.

**Follow-up Questions:**
* How would you audit and review existing user permissions?
* What's your approach to managing permissions in development vs. production?

---

**Question 2: How do you handle user management in a microservices architecture where each service has its own database?**

**Difficulty:** Senior

**Answer:**

In microservices, each service should have its own database user with permissions scoped only to that service's schema:

```sql
-- MySQL approach
-- Service 1: User Service
CREATE USER 'user_service'@'%' IDENTIFIED BY 'strong_password_1';
GRANT SELECT, INSERT, UPDATE, DELETE ON user_db.* TO 'user_service'@'%';

-- Service 2: Order Service
CREATE USER 'order_service'@'%' IDENTIFIED BY 'strong_password_2';
GRANT SELECT, INSERT, UPDATE, DELETE ON order_db.* TO 'order_service'@'%';
-- Read-only access to user data (if needed)
GRANT SELECT ON user_db.users TO 'order_service'@'%';

-- Service 3: Analytics Service (read-only across databases)
CREATE USER 'analytics_service'@'%' IDENTIFIED BY 'strong_password_3';
GRANT SELECT ON user_db.* TO 'analytics_service'@'%';
GRANT SELECT ON order_db.* TO 'analytics_service'@'%';
GRANT SELECT ON product_db.* TO 'analytics_service'@'%';
```

**PostgreSQL approach with schemas:**

```sql
-- Single database, multiple schemas (one per service)
CREATE DATABASE company_db;

-- User service schema and role
CREATE SCHEMA user_service;
CREATE ROLE user_service_role;
GRANT USAGE ON SCHEMA user_service TO user_service_role;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA user_service TO user_service_role;
ALTER DEFAULT PRIVILEGES IN SCHEMA user_service GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO user_service_role;

CREATE USER user_service WITH LOGIN PASSWORD 'pass1' IN ROLE user_service_role;

-- Order service schema and role
CREATE SCHEMA order_service;
CREATE ROLE order_service_role;
GRANT USAGE ON SCHEMA order_service TO order_service_role;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA order_service TO order_service_role;
-- Cross-service read access
GRANT USAGE ON SCHEMA user_service TO order_service_role;
GRANT SELECT ON user_service.users TO order_service_role;

CREATE USER order_service WITH LOGIN PASSWORD 'pass2' IN ROLE order_service_role;
```

**Best Practices:**
* One user per service
* Never share credentials between services
* Use connection pooling to limit connections
* Implement service-to-service authentication (e.g., mTLS)
* Regular credential rotation
* Audit logs for all database access

**Why This Matters:** Proper isolation prevents one compromised service from accessing data from other services. It also provides clear audit trails and simplifies debugging.

**Follow-up Questions:**
* How do you handle schema migrations in a microservices environment?
* What's your strategy for read-replicas in microservices?
* How do you implement distributed transactions across services?

---

## Key Takeaways

* **DCL manages database security** through GRANT and REVOKE commands
* **Principle of least privilege** should guide all permission assignments
* **MySQL uses user@host model** for fine-grained access control
* **PostgreSQL uses roles** which can inherit permissions from other roles
* **Row-level security** (PostgreSQL) provides fine-grained data access control
* **Always use strong passwords** and password expiration policies
* **Regular permission audits** prevent privilege creep
* **Never grant ALL PRIVILEGES** unless absolutely necessary
* **Use roles/groups** to simplify permission management
* **Document all permission grants** for security audits

---

*This concludes Part 1 Section 1.6. Continue to Section 1.7: TCL - Transaction Control Language →*
