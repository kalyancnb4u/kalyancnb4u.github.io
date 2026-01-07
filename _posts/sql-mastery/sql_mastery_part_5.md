---
title: "Complete SQL Mastery Part 5: Production Operations & Advanced Features"
date: 2024-02-06 00:00:00 +0530
categories: [SQL, Mastery]
tags: [SQL, Database, PostgreSQL, MySQL, Production, Operations, Replication, Backup, Monitoring, Security, High-availability]
---

# Complete SQL Mastery Part 5: Production Operations & Advanced Features

## Introduction

Running databases in production requires more than SQL knowledge. You need expertise in reliability, security, performance monitoring, and disaster recovery. A database that's fast in development can fail catastrophically in production without proper operational practices.

In Parts 1-4, we covered SQL fundamentals, internals, advanced techniques, and optimization. Now we focus on production operations—the skills that separate junior developers from senior engineers.

This part covers:
* **Replication and high availability** - ensuring uptime and data redundancy
* **Backup and recovery strategies** - protecting against data loss
* **Monitoring and alerting** - detecting issues before users notice
* **Security and access control** - protecting sensitive data
* **Scaling strategies** - handling growth from thousands to millions of users

This knowledge is critical because:
* **Downtime is expensive**: Every minute offline costs money and trust
* **Data loss is catastrophic**: Backups are worthless if you can't restore
* **Security breaches have consequences**: Regulatory, financial, reputational
* **Performance degrades over time**: Without monitoring, problems accumulate
* **Career advancement**: Production expertise commands premium salaries

By the end of this part, you'll have the operational skills to run databases reliably at scale.

---

## 5.1 Replication & High Availability

Replication creates copies of your database for redundancy, scalability, and disaster recovery. Understanding replication architectures is essential for production systems.

### Replication Fundamentals

**Why replicate?**
* **High availability**: Failover to replica if primary fails
* **Read scaling**: Distribute read queries across replicas
* **Disaster recovery**: Geographic redundancy
* **Backup isolation**: Run backups on replica without affecting primary
* **Analytics isolation**: Heavy analytical queries on replica

**Replication types:**
* **Synchronous**: Primary waits for replica confirmation (consistent, slower)
* **Asynchronous**: Primary doesn't wait (faster, potential data loss)
* **Semi-synchronous**: Hybrid approach

### PostgreSQL Replication

#### Streaming Replication (Built-in)

```sql
-- PRIMARY SERVER SETUP

-- 1. Edit postgresql.conf
wal_level = replica                  -- Enable WAL for replication
max_wal_senders = 10                 -- Max concurrent connections from replicas
wal_keep_size = 1GB                  -- Keep WAL segments for replicas
hot_standby = on                     -- Allow reads on standby

-- 2. Edit pg_hba.conf (allow replica connections)
-- TYPE  DATABASE        USER            ADDRESS                 METHOD
host    replication     replicator      192.168.1.100/32        md5

-- 3. Create replication user
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'secure_password';

-- 4. Restart PostgreSQL
sudo systemctl restart postgresql

-- STANDBY SERVER SETUP

-- 1. Stop PostgreSQL on standby
sudo systemctl stop postgresql

-- 2. Remove existing data directory
sudo rm -rf /var/lib/postgresql/14/main/*

-- 3. Base backup from primary
pg_basebackup -h 192.168.1.10 -D /var/lib/postgresql/14/main \
    -U replicator -P -v -R -X stream -C -S standby1

-- -h: Primary server host
-- -D: Data directory
-- -U: Replication user
-- -P: Show progress
-- -v: Verbose
-- -R: Create standby.signal and minimal recovery configuration
-- -X stream: Stream WAL during backup
-- -C: Create replication slot
-- -S: Replication slot name

-- 4. Start standby
sudo systemctl start postgresql

-- VERIFICATION

-- On primary:
SELECT * FROM pg_stat_replication;
-- Shows connected replicas, lag, state

-- application_name | state     | sent_lsn   | write_lsn  | flush_lsn  | replay_lsn | sync_state
-- standby1        | streaming | 0/3000000  | 0/3000000  | 0/3000000  | 0/2FFFFFF  | async

-- On standby:
SELECT pg_is_in_recovery();
-- Returns: true (standby mode)

SELECT pg_last_wal_receive_lsn(), pg_last_wal_replay_lsn();
-- Shows replication progress
```

#### Logical Replication

```sql
-- Replicates specific tables, not entire database
-- Use cases: Partial replication, cross-version replication, selective sync

-- PRIMARY (PUBLISHER)

-- 1. Edit postgresql.conf
wal_level = logical

-- 2. Create publication
CREATE PUBLICATION my_publication FOR ALL TABLES;
-- Or specific tables:
CREATE PUBLICATION orders_pub FOR TABLE orders, order_items;

-- 3. Check publication
SELECT * FROM pg_publication;

-- STANDBY (SUBSCRIBER)

-- 1. Create same table structure
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2)
);

-- 2. Create subscription
CREATE SUBSCRIPTION my_subscription
    CONNECTION 'host=192.168.1.10 port=5432 dbname=mydb user=replicator password=secure_password'
    PUBLICATION my_publication;

-- 3. Verify
SELECT * FROM pg_subscription;
SELECT * FROM pg_stat_subscription;

-- MONITORING

-- On primary:
SELECT * FROM pg_stat_replication;

-- On subscriber:
SELECT subname, pid, received_lsn, latest_end_lsn, last_msg_send_time
FROM pg_stat_subscription;
```

**Logical vs Physical Replication:**

| Feature | Physical (Streaming) | Logical |
|---------|---------------------|---------|
| **Granularity** | Entire cluster | Specific tables |
| **Cross-version** | Same major version | Different versions OK |
| **Performance** | Faster (binary) | Slower (row-level) |
| **Standby queries** | Read-only | Read-write |
| **Use case** | HA, DR | Selective sync, upgrades |

### MySQL Replication

#### Asynchronous Replication

```sql
-- PRIMARY SERVER SETUP

-- 1. Edit my.cnf
[mysqld]
server-id = 1                        -- Unique server ID
log_bin = /var/log/mysql/mysql-bin   -- Binary log location
binlog_format = ROW                  -- ROW, STATEMENT, or MIXED
binlog_do_db = mydb                  -- Optional: specific database
max_binlog_size = 100M               -- Max binlog file size
expire_logs_days = 7                 -- Auto-purge old binlogs

-- 2. Restart MySQL
sudo systemctl restart mysql

-- 3. Create replication user
CREATE USER 'replicator'@'%' IDENTIFIED BY 'secure_password';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';
FLUSH PRIVILEGES;

-- 4. Get binary log position
FLUSH TABLES WITH READ LOCK;
SHOW MASTER STATUS;
-- +------------------+----------+--------------+------------------+
-- | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
-- +------------------+----------+--------------+------------------+
-- | mysql-bin.000003 |      120 | mydb         |                  |
-- +------------------+----------+--------------+------------------+
-- Record this information!

-- 5. Backup database
mysqldump -u root -p --all-databases --master-data=2 > backup.sql

-- 6. Unlock tables
UNLOCK TABLES;

-- REPLICA SERVER SETUP

-- 1. Edit my.cnf
[mysqld]
server-id = 2                        -- Different from primary
relay-log = /var/log/mysql/relay-bin
read_only = 1                        -- Prevent writes on replica
log_bin = /var/log/mysql/mysql-bin   -- For chaining

-- 2. Restart MySQL
sudo systemctl restart mysql

-- 3. Restore backup
mysql -u root -p < backup.sql

-- 4. Configure replication
CHANGE MASTER TO
    MASTER_HOST='192.168.1.10',
    MASTER_USER='replicator',
    MASTER_PASSWORD='secure_password',
    MASTER_LOG_FILE='mysql-bin.000003',
    MASTER_LOG_POS=120;

-- 5. Start replication
START SLAVE;

-- 6. Verify
SHOW SLAVE STATUS\G

-- Key fields:
-- Slave_IO_Running: Yes
-- Slave_SQL_Running: Yes
-- Seconds_Behind_Master: 0 (no lag)
-- Last_Error: (should be empty)
```

#### Semi-Synchronous Replication

```sql
-- Balances consistency and performance
-- Primary waits for at least one replica to confirm

-- PRIMARY SETUP

-- 1. Install plugin
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';

-- 2. Enable
SET GLOBAL rpl_semi_sync_master_enabled = 1;
SET GLOBAL rpl_semi_sync_master_timeout = 1000;  -- 1 second timeout

-- Add to my.cnf for persistence:
[mysqld]
rpl_semi_sync_master_enabled = 1
rpl_semi_sync_master_timeout = 1000

-- 3. Verify
SHOW STATUS LIKE 'Rpl_semi_sync_master_status';
-- Should show: ON

-- REPLICA SETUP

-- 1. Install plugin
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

-- 2. Enable
SET GLOBAL rpl_semi_sync_slave_enabled = 1;

-- Add to my.cnf:
[mysqld]
rpl_semi_sync_slave_enabled = 1

-- 3. Restart slave
STOP SLAVE IO_THREAD;
START SLAVE IO_THREAD;

-- MONITORING

-- On primary:
SHOW STATUS LIKE 'Rpl_semi_sync%';
-- Rpl_semi_sync_master_clients: 1 (connected replicas)
-- Rpl_semi_sync_master_status: ON
-- Rpl_semi_sync_master_yes_tx: 1234 (confirmed transactions)
-- Rpl_semi_sync_master_no_tx: 0 (fallback to async)
```

#### Group Replication (MySQL 8.0+)

```sql
-- Multi-primary or single-primary setup
-- Automatic failover, conflict detection

-- This is complex; here's a simplified overview:

-- 1. Configure group replication
[mysqld]
server_id = 1
gtid_mode = ON
enforce_gtid_consistency = ON
binlog_checksum = NONE
log_slave_updates = ON
binlog_format = ROW
transaction_write_set_extraction = XXHASH64

plugin_load_add = 'group_replication.so'
group_replication_group_name = "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa"
group_replication_start_on_boot = OFF
group_replication_local_address = "192.168.1.10:33061"
group_replication_group_seeds = "192.168.1.10:33061,192.168.1.11:33061,192.168.1.12:33061"

-- 2. Bootstrap group (first node only)
SET GLOBAL group_replication_bootstrap_group = ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group = OFF;

-- 3. Join group (other nodes)
START GROUP_REPLICATION;

-- 4. Verify
SELECT * FROM performance_schema.replication_group_members;
-- Shows all group members and their status
```

### Failover and Promotion

#### PostgreSQL Failover

```sql
-- MANUAL FAILOVER

-- 1. On standby, promote to primary:
pg_ctl promote -D /var/lib/postgresql/14/main

-- Or trigger file method:
touch /var/lib/postgresql/14/main/promote

-- 2. Verify promotion:
SELECT pg_is_in_recovery();
-- Should return: false (now primary)

-- 3. Update application connection strings to new primary

-- 4. Optional: Convert old primary to standby
-- (After fixing issues that caused failover)

-- AUTOMATIC FAILOVER (using tools)

-- Patroni (recommended for production)
-- - Monitors PostgreSQL health
-- - Automatic failover
-- - Distributed consensus (etcd/Consul/Zookeeper)

-- Installation (simplified):
-- 1. Install Patroni
pip install patroni[etcd]

-- 2. Configure patroni.yml
scope: postgres-cluster
name: node1

restapi:
  listen: 0.0.0.0:8008
  connect_address: 192.168.1.10:8008

etcd:
  host: 192.168.1.100:2379

bootstrap:
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576
    postgresql:
      use_pg_rewind: true
      parameters:
        wal_level: replica
        hot_standby: on
        max_wal_senders: 10

postgresql:
  listen: 0.0.0.0:5432
  connect_address: 192.168.1.10:5432
  data_dir: /var/lib/postgresql/14/main
  bin_dir: /usr/lib/postgresql/14/bin
  authentication:
    replication:
      username: replicator
      password: secure_password
    superuser:
      username: postgres
      password: secure_password

-- 3. Start Patroni
patroni /etc/patroni/patroni.yml

-- Patroni handles:
-- - Health checks
-- - Automatic failover
-- - Preventing split-brain
-- - Coordinating cluster state
```

#### MySQL Failover

```sql
-- MANUAL FAILOVER

-- 1. Stop writes on old primary (if possible)
SET GLOBAL read_only = 1;
SET GLOBAL super_read_only = 1;

-- 2. Ensure replica is caught up
SHOW SLAVE STATUS\G
-- Seconds_Behind_Master should be 0

-- 3. Promote replica to primary
STOP SLAVE;
RESET SLAVE ALL;
SET GLOBAL read_only = 0;
SET GLOBAL super_read_only = 0;

-- 4. Update application to new primary

-- 5. Reconfigure other replicas
STOP SLAVE;
CHANGE MASTER TO MASTER_HOST='new_primary_ip';
START SLAVE;

-- AUTOMATIC FAILOVER

-- MySQL InnoDB Cluster (MySQL 8.0+)
-- Uses Group Replication + MySQL Router + MySQL Shell

-- Setup (simplified):
mysqlsh

-- Configure instances:
dba.configureInstance('root@192.168.1.10:3306')
dba.configureInstance('root@192.168.1.11:3306')
dba.configureInstance('root@192.168.1.12:3306')

-- Create cluster:
cluster = dba.createCluster('myCluster')

-- Add instances:
cluster.addInstance('root@192.168.1.11:3306')
cluster.addInstance('root@192.168.1.12:3306')

-- Check status:
cluster.status()

-- MySQL Router provides automatic failover:
-- Applications connect to Router (not directly to MySQL)
-- Router redirects to current primary
```

### Monitoring Replication

```sql
-- POSTGRESQL MONITORING

-- Replication lag (bytes)
SELECT 
    client_addr,
    state,
    pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn) AS lag_bytes,
    replay_lag
FROM pg_stat_replication;

-- Replication lag (time)
SELECT 
    now() - pg_last_xact_replay_timestamp() AS replication_lag
FROM pg_stat_replication;

-- Alert if lag > 100MB or > 60 seconds

-- MYSQL MONITORING

-- Replication status
SHOW SLAVE STATUS\G

-- Key metrics:
-- Seconds_Behind_Master: Lag in seconds
-- Slave_IO_Running: YES (receiving binlog)
-- Slave_SQL_Running: YES (applying binlog)
-- Last_Error: Should be empty

-- Automated monitoring query:
SELECT 
    IF(Slave_IO_Running = 'Yes' AND Slave_SQL_Running = 'Yes', 'OK', 'ERROR') AS status,
    Seconds_Behind_Master AS lag_seconds,
    Last_Error AS error_message
FROM (
    SHOW SLAVE STATUS
) s;
```

### Split-Brain Prevention

```sql
-- Split-brain: Multiple nodes think they're primary
-- DISASTER: Data divergence, conflicts

-- Prevention strategies:

-- 1. Quorum-based systems
-- PostgreSQL + Patroni + etcd/Consul
-- Requires majority consensus for promotion
-- 3 nodes: Need 2 to agree (prevents split-brain)

-- 2. Fencing
-- Isolate failed primary before promoting standby
-- STONITH (Shoot The Other Node In The Head)

-- 3. Application-level checks
-- Health endpoint that verifies database role:

-- PostgreSQL:
CREATE FUNCTION is_primary() RETURNS boolean AS $$
BEGIN
    RETURN NOT pg_is_in_recovery();
END;
$$ LANGUAGE plpgsql;

-- Application checks before write:
SELECT is_primary();
-- If false, redirect to primary

-- MySQL:
SHOW VARIABLES LIKE 'read_only';
-- If ON, this is a replica

-- 4. Network partitions
-- Use multiple network paths
-- Monitor network connectivity
-- Quick detection and response
```

---

## 5.2 Backup & Recovery Strategies

Backups are insurance. The quality of your backup strategy determines whether data loss is recoverable or catastrophic.

### Backup Types

#### Full Backup

```sql
-- POSTGRESQL

-- 1. pg_dump (logical backup)
pg_dump -U postgres -h localhost mydb > mydb_backup.sql

-- With compression:
pg_dump -U postgres mydb | gzip > mydb_backup.sql.gz

-- All databases:
pg_dumpall -U postgres > all_databases.sql

-- Custom format (parallel restore, selective restore):
pg_dump -U postgres -F c -f mydb_backup.dump mydb

-- Parallel dump (faster):
pg_dump -U postgres -F d -j 4 -f mydb_backup_dir mydb
-- -j 4: Use 4 parallel jobs

-- 2. pg_basebackup (physical backup)
pg_basebackup -D /backup/base -F tar -z -P
-- -D: Output directory
-- -F tar: Tar format
-- -z: Compress
-- -P: Show progress

-- MYSQL

-- 1. mysqldump (logical backup)
mysqldump -u root -p mydb > mydb_backup.sql

-- With compression:
mysqldump -u root -p mydb | gzip > mydb_backup.sql.gz

-- All databases:
mysqldump -u root -p --all-databases > all_databases.sql

-- With locks (consistent backup):
mysqldump -u root -p --single-transaction --master-data=2 mydb > backup.sql
-- --single-transaction: Consistent snapshot (InnoDB)
-- --master-data=2: Include binlog position (for PITR)

-- Parallel backup:
mysqldump -u root -p --single-transaction \
    --routines --triggers --events \
    mydb | gzip > mydb_backup.sql.gz

-- 2. Physical backup (filesystem copy)
-- Stop MySQL first (or use Percona XtraBackup for hot backup)
sudo systemctl stop mysql
cp -r /var/lib/mysql /backup/mysql_$(date +%Y%m%d)
sudo systemctl start mysql

-- 3. Percona XtraBackup (hot backup)
xtrabackup --backup --target-dir=/backup/full_backup
-- No downtime, consistent backup
```

#### Incremental Backup

```sql
-- POSTGRESQL

-- WAL archiving enables point-in-time recovery
-- 1. Configure WAL archiving (postgresql.conf)
wal_level = replica
archive_mode = on
archive_command = 'cp %p /backup/wal_archive/%f'

-- %p: Full path of WAL file
-- %f: Filename only

-- 2. Take base backup
pg_basebackup -D /backup/base -F tar -z -X fetch

-- 3. WAL files are continuously archived
-- Incremental data in WAL archive

-- Recovery:
-- 1. Restore base backup
-- 2. Replay WAL files from archive

-- MYSQL

-- Binary log enables incremental backup
-- 1. Ensure binary logging enabled (my.cnf)
log_bin = /var/log/mysql/mysql-bin
binlog_format = ROW
expire_logs_days = 7

-- 2. Full backup with binlog position
mysqldump --single-transaction --master-data=2 \
    --flush-logs mydb > full_backup.sql

-- 3. Flush logs to rotate binlog
FLUSH BINARY LOGS;

-- 4. Backup binlog files
cp /var/log/mysql/mysql-bin.* /backup/binlogs/

-- Recovery:
-- 1. Restore full backup
-- 2. Apply binary logs:
mysqlbinlog /backup/binlogs/mysql-bin.000001 | mysql -u root -p
```

#### Continuous Backup

```sql
-- POSTGRESQL WAL-E / WAL-G (S3 backup)

-- Install WAL-G
wget https://github.com/wal-g/wal-g/releases/download/v2.0.1/wal-g-pg-ubuntu-20.04-amd64.tar.gz
tar -xzf wal-g-pg-ubuntu-20.04-amd64.tar.gz
sudo mv wal-g-pg-ubuntu-20.04-amd64 /usr/local/bin/wal-g

-- Configure environment variables
export WALG_S3_PREFIX="s3://my-bucket/postgres-backups"
export AWS_ACCESS_KEY_ID="your_key"
export AWS_SECRET_ACCESS_KEY="your_secret"
export AWS_REGION="us-east-1"

-- Configure PostgreSQL (postgresql.conf)
archive_mode = on
archive_command = 'wal-g wal-push %p'
archive_timeout = 60  -- Force WAL switch every 60 seconds

-- Create base backup
wal-g backup-push /var/lib/postgresql/14/main

-- List backups
wal-g backup-list

-- WAL files are continuously pushed to S3

-- MYSQL Percona XtraBackup + Streaming

-- Continuous streaming to S3
xtrabackup --backup --stream=xbstream --target-dir=/tmp | \
    gzip | aws s3 cp - s3://my-bucket/mysql-backup-$(date +%Y%m%d-%H%M%S).xbstream.gz

-- Automated with cron
-- 0 2 * * * /usr/local/bin/backup-to-s3.sh
```

### Point-in-Time Recovery (PITR)

```sql
-- POSTGRESQL PITR

-- Scenario: Disaster at 2024-01-15 14:30
-- Last base backup: 2024-01-15 02:00
-- Need to recover to 2024-01-15 14:25 (5 minutes before disaster)

-- 1. Stop PostgreSQL
sudo systemctl stop postgresql

-- 2. Move/backup current data directory
mv /var/lib/postgresql/14/main /var/lib/postgresql/14/main.old

-- 3. Restore base backup
tar -xzf /backup/base_20240115_0200.tar.gz -C /var/lib/postgresql/14/main

-- 4. Create recovery.conf (or recovery.signal in PG 12+)
-- PostgreSQL 12+: Create recovery.signal file
touch /var/lib/postgresql/14/main/recovery.signal

-- Edit postgresql.conf:
restore_command = 'cp /backup/wal_archive/%f %p'
recovery_target_time = '2024-01-15 14:25:00'
recovery_target_action = 'promote'

-- 5. Start PostgreSQL
sudo systemctl start postgresql

-- PostgreSQL will:
-- 1. Restore from base backup
-- 2. Replay WAL files up to 14:25:00
-- 3. Stop and promote to primary

-- 6. Verify recovery
SELECT now(), pg_is_in_recovery();

-- MYSQL PITR

-- Scenario: Same as above

-- 1. Restore full backup
mysql -u root -p < /backup/full_20240115_0200.sql

-- 2. Find binlog position for recovery end time
mysqlbinlog --start-datetime="2024-01-15 02:00:00" \
            --stop-datetime="2024-01-15 14:25:00" \
            /backup/binlogs/mysql-bin.* \
            > incremental_recovery.sql

-- 3. Review SQL (optional but recommended)
less incremental_recovery.sql
-- Look for the bad transaction, exclude if found

-- 4. Apply binlog
mysql -u root -p < incremental_recovery.sql

-- 5. Verify
SELECT NOW(), COUNT(*) FROM critical_table;
```

### Backup Testing

```sql
-- Untested backups are useless!

-- Automated backup testing script (pseudo-code):

-- 1. Create test database instance
-- 2. Restore latest backup
-- 3. Run integrity checks
-- 4. Verify critical tables exist and have data
-- 5. Run sample queries
-- 6. Alert if any step fails
-- 7. Destroy test instance

-- PostgreSQL integrity check:
SELECT 
    schemaname,
    tablename,
    n_live_tup,
    n_dead_tup,
    last_vacuum,
    last_autovacuum
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC;

-- Should match production row counts (within reason)

-- MySQL integrity check:
SHOW TABLE STATUS;

-- Check for corruption:
CHECK TABLE orders;
-- Should return: OK

-- Sample data verification:
SELECT 
    COUNT(*) AS total_orders,
    MAX(order_date) AS latest_order,
    SUM(total_amount) AS total_revenue
FROM orders;

-- Compare with production metrics
```

### Backup Retention Policy

```sql
-- Example policy:
-- - Daily backups: Keep 7 days
-- - Weekly backups: Keep 4 weeks
-- - Monthly backups: Keep 12 months
-- - Yearly backups: Keep 7 years (compliance)

-- Automated cleanup script (pseudo-code):

-- PostgreSQL (using WAL-G)
-- Delete backups older than 7 days
wal-g delete before FIND_FULL 7

-- Keep at least 4 full backups
wal-g delete retain 4

-- MySQL (using find command)
-- Delete daily backups older than 7 days
find /backup/daily -name "*.sql.gz" -mtime +7 -delete

-- Delete weekly backups older than 28 days
find /backup/weekly -name "*.sql.gz" -mtime +28 -delete

-- Delete monthly backups older than 365 days
find /backup/monthly -name "*.sql.gz" -mtime +365 -delete
```

### Backup Security

```sql
-- Encrypt backups at rest

-- PostgreSQL with GPG
pg_dump mydb | gzip | gpg --encrypt --recipient backup@company.com > backup.sql.gz.gpg

-- Decrypt and restore:
gpg --decrypt backup.sql.gz.gpg | gunzip | psql mydb

-- MySQL with GPG
mysqldump mydb | gzip | gpg --encrypt --recipient backup@company.com > backup.sql.gz.gpg

-- S3 server-side encryption
aws s3 cp backup.sql.gz s3://my-bucket/ --sse AES256

-- Or client-side encryption before upload
openssl enc -aes-256-cbc -salt -in backup.sql.gz -out backup.sql.gz.enc
aws s3 cp backup.sql.gz.enc s3://my-bucket/

-- Access control
-- - Separate AWS account for backups
-- - Immutable backups (S3 Object Lock)
-- - Audit logging (CloudTrail)
-- - Least privilege (IAM policies)
```

---

## 5.3 Monitoring & Performance Tuning

Monitoring detects problems before they become outages. Good monitoring answers: "Is the database healthy?" and "What's degrading?"

### Key Metrics to Monitor

#### System Metrics

```sql
-- CPU Usage
-- Target: < 70% average, < 90% peak
-- High CPU indicates:
-- - Missing indexes
-- - Inefficient queries
-- - Too many connections

-- Memory Usage
-- Target: 
-- - PostgreSQL: ~75% total RAM used (buffer cache + OS cache)
-- - MySQL: ~70% for buffer pool
-- High memory indicates:
-- - Buffer pool too large
-- - Memory leaks
-- - Connection pooling issues

-- Disk I/O
-- Target: < 80% utilization
-- High I/O indicates:
-- - Insufficient buffer pool
-- - Missing indexes
-- - Large sequential scans

-- Network
-- Target: < 50% bandwidth
-- High network indicates:
-- - SELECT * queries
-- - Large result sets
-- - Missing indexes (returning too many rows)
```

#### Database Metrics

```sql
-- POSTGRESQL MONITORING

-- 1. Connection count
SELECT 
    COUNT(*) AS total_connections,
    COUNT(*) FILTER (WHERE state = 'active') AS active,
    COUNT(*) FILTER (WHERE state = 'idle') AS idle,
    COUNT(*) FILTER (WHERE state = 'idle in transaction') AS idle_in_transaction
FROM pg_stat_activity;

-- Alert if idle_in_transaction > 10 (indicates connection pooling issues)

-- 2. Database size growth
SELECT 
    datname,
    pg_size_pretty(pg_database_size(datname)) AS size,
    pg_size_pretty(pg_database_size(datname) - 
        LAG(pg_database_size(datname)) OVER (ORDER BY datname)) AS growth
FROM pg_database
WHERE datname NOT IN ('template0', 'template1');

-- 3. Table bloat
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
    n_dead_tup,
    n_live_tup,
    round(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct
FROM pg_stat_user_tables
WHERE n_dead_tup > 10000
ORDER BY n_dead_tup DESC;

-- Alert if dead_pct > 20%

-- 4. Long-running queries
SELECT 
    pid,
    now() - query_start AS duration,
    state,
    query
FROM pg_stat_activity
WHERE state != 'idle'
  AND query_start < now() - interval '5 minutes'
ORDER BY duration DESC;

-- 5. Lock contention
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
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;

-- 6. Cache hit ratio
SELECT 
    'index hit rate' AS metric,
    round(100.0 * sum(idx_blks_hit) / NULLIF(sum(idx_blks_hit + idx_blks_read), 0), 2) AS ratio
FROM pg_statio_user_indexes
UNION ALL
SELECT 
    'table hit rate',
    round(100.0 * sum(heap_blks_hit) / NULLIF(sum(heap_blks_hit + heap_blks_read), 0), 2)
FROM pg_statio_user_tables;

-- Target: > 99%

-- 7. Checkpoint frequency
SELECT 
    checkpoints_timed,
    checkpoints_req,
    round(100.0 * checkpoints_req / NULLIF(checkpoints_timed + checkpoints_req, 0), 2) AS checkpoint_req_pct
FROM pg_stat_bgwriter;

-- If checkpoint_req_pct > 10%, increase max_wal_size

-- MYSQL MONITORING

-- 1. Connection usage
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
SHOW VARIABLES LIKE 'max_connections';

-- Usage: Threads_connected / max_connections
-- Target: < 80%

-- 2. Buffer pool hit ratio
SHOW STATUS LIKE 'Innodb_buffer_pool_read_requests';
SHOW STATUS LIKE 'Innodb_buffer_pool_reads';

-- Hit ratio = (read_requests - reads) / read_requests × 100
-- Target: > 99%

-- 3. Slow queries
SHOW STATUS LIKE 'Slow_queries';

-- Enable slow query log:
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;  -- Queries > 2 seconds
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';

-- 4. Table locks
SHOW STATUS LIKE 'Table_locks_waited';
SHOW STATUS LIKE 'Table_locks_immediate';

-- Lock wait ratio = waited / (waited + immediate)
-- Target: < 1%

-- 5. InnoDB row operations
SHOW STATUS LIKE 'Innodb_rows%';
-- Innodb_rows_read: Rows read
-- Innodb_rows_inserted: Rows inserted
-- Innodb_rows_updated: Rows updated
-- Innodb_rows_deleted: Rows deleted

-- 6. Temp table usage
SHOW STATUS LIKE 'Created_tmp_disk_tables';
SHOW STATUS LIKE 'Created_tmp_tables';

-- Disk temp table ratio = disk / total
-- Target: < 10%
-- If high, increase tmp_table_size and max_heap_table_size
```

### Query Performance Monitoring

```sql
-- POSTGRESQL pg_stat_statements

-- 1. Enable extension
CREATE EXTENSION pg_stat_statements;

-- 2. Configure (postgresql.conf)
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.track = all
pg_stat_statements.max = 10000

-- 3. Query slow queries
SELECT 
    substring(query, 1, 50) AS short_query,
    round(total_exec_time::numeric, 2) AS total_time_ms,
    calls,
    round((total_exec_time/calls)::numeric, 2) AS mean_time_ms,
    round((100 * total_exec_time / sum(total_exec_time) OVER ())::numeric, 2) AS pct_total_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;

-- Identifies:
-- - Most time-consuming queries
-- - Most frequent queries
-- - Queries with high variance

-- 4. Reset stats (after optimization)
SELECT pg_stat_statements_reset();

-- MYSQL Performance Schema

-- 1. Enable Performance Schema (my.cnf)
[mysqld]
performance_schema = ON
performance-schema-instrument = 'statement/%=ON'
performance-schema-consumer-statements-digest = ON

-- 2. Query slow queries
SELECT 
    SUBSTRING(DIGEST_TEXT, 1, 64) AS query_snippet,
    COUNT_STAR AS exec_count,
    ROUND(AVG_TIMER_WAIT / 1000000000, 2) AS avg_time_ms,
    ROUND(SUM_TIMER_WAIT / 1000000000, 2) AS total_time_ms,
    ROUND(100 * SUM_TIMER_WAIT / (SELECT SUM(SUM_TIMER_WAIT) 
        FROM performance_schema.events_statements_summary_by_digest), 2) AS pct_total
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 20;

-- 3. Query specific table access
SELECT 
    OBJECT_SCHEMA,
    OBJECT_NAME,
    COUNT_READ,
    COUNT_WRITE,
    COUNT_FETCH,
    SUM_TIMER_WAIT / 1000000000 AS total_time_ms
FROM performance_schema.table_io_waits_summary_by_table
WHERE OBJECT_SCHEMA NOT IN ('mysql', 'performance_schema', 'information_schema')
ORDER BY SUM_TIMER_WAIT DESC;
```

### Alerting Strategy

```sql
-- Critical alerts (page on-call):
-- - Database down
-- - Replication stopped
-- - Disk > 90% full
-- - Connection pool exhausted
-- - Long-running blocking queries (> 5 min)

-- Warning alerts (email/Slack):
-- - Slow query log has new queries
-- - Cache hit ratio < 95%
-- - Checkpoint frequency high
-- - Table bloat > 30%
-- - Replication lag > 60 seconds

-- Info alerts (dashboard):
-- - Database size growth trends
-- - Query performance trends
-- - Connection count trends

-- Example alerting rules (Prometheus/Grafana):

-- PostgreSQL connection utilization
(pg_stat_database_numbackends{datname="production"} / pg_settings_max_connections) > 0.8
-- Alert: Connection pool > 80%

-- Replication lag
pg_replication_lag_seconds > 60
-- Alert: Replication lag > 60 seconds

-- Cache hit ratio
(sum(pg_stat_database_blks_hit{datname="production"}) / 
 sum(pg_stat_database_blks_hit{datname="production"} + pg_stat_database_blks_read{datname="production"})) < 0.95
-- Alert: Cache hit ratio < 95%

-- MySQL Slow queries
mysql_global_status_slow_queries rate(5m) > 10
-- Alert: > 10 slow queries per 5 minutes
```

### Performance Tuning Workflow

```sql
-- 1. Identify bottleneck
-- - Monitor metrics
-- - Profile queries
-- - Analyze execution plans

-- 2. Reproduce in dev/staging
-- - Same data volume
-- - Same query pattern
-- - Same configuration

-- 3. Measure baseline
EXPLAIN ANALYZE <query>;
-- Record execution time and plan

-- 4. Apply optimization
-- - Add index
-- - Rewrite query
-- - Adjust configuration

-- 5. Measure improvement
EXPLAIN ANALYZE <query>;
-- Compare to baseline

-- 6. Deploy to production (during low-traffic period)
-- - Create index with CONCURRENTLY (PostgreSQL)
-- - Monitor impact
-- - Have rollback plan

-- 7. Verify in production
-- - Monitor metrics
-- - Check query performance
-- - Watch for unexpected side effects

-- Example optimization session:

-- Baseline:
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 123;
-- Seq Scan on orders (cost=0.00..25000.00 rows=100 width=50)
-- Execution Time: 250.123 ms

-- Add index:
CREATE INDEX CONCURRENTLY idx_customer ON orders(customer_id);

-- After optimization:
EXPLAIN ANALYZE
SELECT * FROM orders WHERE customer_id = 123;
-- Index Scan using idx_customer (cost=0.29..8.31 rows=100 width=50)
-- Execution Time: 0.543 ms

-- Improvement: 460x faster!
```

---

## Frequently Asked Questions

**Q1: How do I choose between synchronous and asynchronous replication?**

**A:**

**Synchronous replication:**

✓ **Use when:**
- Zero data loss acceptable (RPO = 0)
- Financial transactions
- Critical data (health records, legal)
- Replicas in same datacenter (low latency)

❌ **Don't use when:**
- Replicas across continents (high latency)
- Performance more important than consistency
- Read scalability is the goal

**Trade-offs:**
```sql
-- Synchronous (PostgreSQL)
synchronous_commit = on
synchronous_standby_names = 'standby1'

-- Impact:
-- - Writes wait for replica confirmation
-- - Latency: +5-50ms (LAN) or +100-500ms (WAN)
-- - Throughput: ~20-30% reduction
-- - Guarantee: No data loss if primary fails

-- Asynchronous (default)
synchronous_commit = off

-- Impact:
-- - Writes don't wait for replica
-- - Latency: Same as single server
-- - Throughput: No impact
-- - Risk: Data loss if primary fails before replication
```

**Decision matrix:**

| Scenario | Recommendation |
|----------|----------------|
| Banking transactions | Synchronous |
| E-commerce orders | Synchronous |
| Social media posts | Asynchronous |
| Analytics logs | Asynchronous |
| Same datacenter | Synchronous OK |
| Cross-continent | Asynchronous |

**Hybrid approach:**
```sql
-- PostgreSQL: Synchronous for local, async for remote
synchronous_standby_names = 'FIRST 1 (local_standby, remote_standby)'
-- Waits for local_standby (fast)
-- remote_standby is async (no wait)
```

---

**Q2: How often should I run VACUUM/ANALYZE?**

**A:**

**PostgreSQL autovacuum (enabled by default):**

```sql
-- Check autovacuum settings
SHOW autovacuum;  -- Should be 'on'

-- Autovacuum triggers:
-- VACUUM when: dead_tuples > threshold + (scale_factor × live_tuples)
-- Default: 50 + (0.2 × live_tuples)

-- For 1M row table:
-- Triggers when dead_tuples > 50 + (0.2 × 1,000,000) = 200,050

-- Check autovacuum activity:
SELECT 
    schemaname,
    relname,
    last_vacuum,
    last_autovacuum,
    n_dead_tup,
    n_live_tup,
    autovacuum_count
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;
```

**When to adjust autovacuum:**

```sql
-- High-write tables: More aggressive autovacuum
ALTER TABLE high_traffic_table SET (
    autovacuum_vacuum_scale_factor = 0.05,  -- 5% instead of 20%
    autovacuum_vacuum_threshold = 1000,      -- 1000 instead of 50
    autovacuum_analyze_scale_factor = 0.02   -- Analyze more frequently
);

-- Now triggers when:
-- dead_tuples > 1000 + (0.05 × live_tuples)
-- For 1M rows: 1000 + 50,000 = 51,000 (4x more frequent)

-- Large tables: Less aggressive (avoid I/O spikes)
ALTER TABLE huge_table SET (
    autovacuum_vacuum_scale_factor = 0.1,   -- 10%
    autovacuum_vacuum_cost_delay = 20,      -- Slower vacuum
    autovacuum_vacuum_cost_limit = 200      -- Lower I/O limit
);
```

**Manual VACUUM schedule:**

```sql
-- Critical tables: Daily manual VACUUM
-- Cron: 2 AM daily
0 2 * * * psql -c "VACUUM ANALYZE orders;"

-- All tables: Weekly VACUUM FULL (careful - locks table!)
-- Cron: Sunday 2 AM
0 2 * * 0 psql -c "VACUUM FULL;"

-- Or REINDEX (often better than VACUUM FULL)
0 2 * * 0 psql -c "REINDEX DATABASE mydb;"
```

**MySQL OPTIMIZE TABLE:**

```sql
-- InnoDB: No autovacuum equivalent
-- Manual optimization needed

-- Monthly optimize for large tables:
-- Cron: First day of month, 2 AM
0 2 1 * * mysql -e "OPTIMIZE TABLE orders;"

-- Or use pt-online-schema-change (Percona Toolkit)
-- Rebuilds table without blocking writes
pt-online-schema-change --alter "ENGINE=InnoDB" D=mydb,t=orders --execute
```

**When autovacuum isn't enough:**

```sql
-- Signs autovacuum is struggling:
-- 1. Table bloat > 30%
SELECT 
    tablename,
    round(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS bloat_pct
FROM pg_stat_user_tables
WHERE n_dead_tup > 100000;

-- 2. Long-running autovacuum
SELECT 
    pid,
    now() - query_start AS duration,
    query
FROM pg_stat_activity
WHERE query LIKE '%autovacuum%';

-- Solution: Increase autovacuum resources
-- postgresql.conf:
autovacuum_max_workers = 6           -- Default: 3
autovacuum_work_mem = 1GB            -- Default: -1 (use maintenance_work_mem)
autovacuum_vacuum_cost_limit = 2000  -- Default: 200 (higher = faster)
```

---

**Q3: My backup takes 6 hours and impacts production. How do I fix this?**

**A:**

**Problem:** Slow backup causes:
- CPU/IO contention during backup
- Long backup window
- Recovery time equally long
- Difficulty meeting backup schedule

**Solutions:**

**1. Backup from replica (best for read-heavy workloads):**

```sql
-- PostgreSQL: pg_basebackup from standby
pg_basebackup -h standby_server -D /backup/base -F tar -z -X stream

-- MySQL: mysqldump from replica
mysqldump -h replica_server -u root -p --single-transaction mydb > backup.sql

-- Pros:
-- - Zero impact on primary
-- - Backup can take as long as needed

-- Cons:
-- - Requires replica
-- - Replica must be in sync
```

**2. Parallel backup:**

```sql
-- PostgreSQL: Parallel pg_dump
pg_dump -j 8 -F d -f /backup/parallel_dump mydb
-- Uses 8 parallel workers
-- 4-8x faster on multi-core systems

-- Parallel restore also faster:
pg_restore -j 8 -d mydb /backup/parallel_dump

-- MySQL: Use mydumper (parallel mysqldump)
mydumper --database mydb --outputdir /backup --threads 8 --compress
-- Much faster than mysqldump

-- Restore with myloader:
myloader --directory=/backup --threads=8 --overwrite-tables
```

**3. Incremental/continuous backup:**

```sql
-- PostgreSQL: WAL-G continuous backup
-- Base backup once:
wal-g backup-push /var/lib/postgresql/14/main
-- Takes 6 hours once

-- Continuous WAL archiving (fast, always running):
archive_command = 'wal-g wal-push %p'
-- Each WAL file: ~16MB, pushed in seconds

-- Recovery:
-- Base backup (6 hours ago) + WAL files (last 6 hours) = full recovery

-- MySQL: Binary log archiving
-- Full backup weekly:
mysqldump --single-transaction mydb > weekly_backup.sql
-- Takes 6 hours once per week

-- Binary logs continuously archived:
-- Each binlog: ~100MB, rotated hourly
-- Archive binlogs to S3/backup server

-- Recovery:
-- Weekly backup + binlogs = full recovery
```

**4. Snapshot-based backup (cloud/SAN):**

```sql
-- AWS RDS: Automated snapshots
-- - Snapshot time: ~5 minutes (regardless of size)
-- - No I/O impact (uses EBS snapshots)
-- - Point-in-time recovery to any second

-- AWS EC2 + EBS:
-- 1. Freeze filesystem
-- 2. Take EBS snapshot
-- 3. Unfreeze

-- PostgreSQL:
SELECT pg_start_backup('snapshot_backup', false, false);
-- Take EBS/SAN snapshot here
SELECT pg_stop_backup();

-- Total freeze time: < 1 minute
```

**5. Optimize backup process:**

```sql
-- Exclude unnecessary data:
pg_dump --exclude-table=logs --exclude-table=temp_data mydb > backup.sql

-- Compress during backup:
pg_dump mydb | pigz -p 8 > backup.sql.gz
-- pigz: Parallel gzip (8 threads)

-- Custom format (best for large databases):
pg_dump -F c -f backup.dump mydb
-- - Compressed by default
-- - Parallel restore supported
-- - Selective table restore
```

**6. Schedule during low-traffic periods:**

```sql
-- Run full backup during maintenance window:
-- - Nightly: 2-4 AM (low traffic)
-- - Weekend: Sunday 2 AM

-- Cron schedule:
0 2 * * 0 /usr/local/bin/full_backup.sh  # Sunday 2 AM
```

**Recommended architecture for large databases:**

```
Primary DB
    ↓ (replication)
Standby DB
    ↓ (backup from here)
Continuous WAL/binlog backup to S3
    +
Weekly full backup from standby
    =
< 1 minute backup time on primary!
```

---

## Key Takeaways

* **Replication provides HA and read scaling** - understand sync vs async trade-offs
* **Automatic failover requires coordination** - Patroni, InnoDB Cluster prevent split-brain
* **Monitor replication lag** - alert on lag > 60 seconds or > 100MB
* **Backups are insurance** - test restores regularly, encryption at rest
* **PITR requires continuous archiving** - WAL/binlog archiving is critical
* **Retention policies balance cost and compliance** - automate cleanup
* **Monitor proactively** - pg_stat_statements, Performance Schema track query performance
* **Alert on leading indicators** - catch problems before outages
* **Autovacuum needs tuning** - aggressive settings for high-write tables
* **Backup from replica** - zero impact on primary performance

---

## 5.4 Security & Access Control

Database security protects against unauthorized access, data breaches, and compliance violations. A single security lapse can destroy a company.

### Authentication Methods

#### PostgreSQL Authentication

```sql
-- pg_hba.conf controls who can connect and how

-- Format:
-- TYPE  DATABASE  USER  ADDRESS      METHOD

-- Local connections (Unix socket)
local   all       all                  peer
-- peer: Use OS username (trusted)

-- TCP/IP connections
host    all       all   127.0.0.1/32   md5
-- md5: Password authentication (encrypted)

host    all       all   10.0.0.0/8     scram-sha-256
-- scram-sha-256: Strong password auth (recommended)

-- SSL-required connections
hostssl all       all   0.0.0.0/0      scram-sha-256

-- Reject specific IP
host    all       all   192.168.1.100/32  reject

-- Common configurations:

-- Development (trust local, password remote):
local   all       all                  trust
host    all       all   127.0.0.1/32   md5

-- Production (strong auth everywhere):
local   all       all                  scram-sha-256
hostssl all       all   0.0.0.0/0      scram-sha-256

-- After editing pg_hba.conf:
SELECT pg_reload_conf();
```

#### MySQL Authentication

```sql
-- User authentication
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'secure_password';
CREATE USER 'app_user'@'10.0.%.%' IDENTIFIED BY 'secure_password';
-- % is wildcard

-- Strong password plugin (MySQL 8.0+)
CREATE USER 'admin'@'localhost' 
    IDENTIFIED WITH caching_sha2_password BY 'VerySecure123!';

-- Require SSL
CREATE USER 'secure_user'@'%' 
    IDENTIFIED BY 'password'
    REQUIRE SSL;

-- Client certificate authentication
CREATE USER 'cert_user'@'%'
    IDENTIFIED BY 'password'
    REQUIRE X509;

-- Specific certificate
CREATE USER 'specific_cert_user'@'%'
    REQUIRE SUBJECT '/C=US/ST=CA/L=SF/O=Company/CN=appserver'
    AND ISSUER '/C=US/ST=CA/L=SF/O=CA/CN=RootCA';
```

### Role-Based Access Control (RBAC)

```sql
-- POSTGRESQL RBAC

-- 1. Create roles (groups)
CREATE ROLE readonly;
CREATE ROLE readwrite;
CREATE ROLE admin;

-- 2. Grant permissions to roles
-- Read-only role
GRANT CONNECT ON DATABASE mydb TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
    GRANT SELECT ON TABLES TO readonly;

-- Read-write role
GRANT CONNECT ON DATABASE mydb TO readwrite;
GRANT USAGE ON SCHEMA public TO readwrite;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO readwrite;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA public TO readwrite;
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
    GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO readwrite;
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
    GRANT USAGE, SELECT ON SEQUENCES TO readwrite;

-- Admin role
GRANT ALL PRIVILEGES ON DATABASE mydb TO admin;
ALTER DEFAULT PRIVILEGES IN SCHEMA public 
    GRANT ALL ON TABLES TO admin;

-- 3. Create users and assign roles
CREATE USER app_readonly WITH PASSWORD 'secure_pass';
GRANT readonly TO app_readonly;

CREATE USER app_readwrite WITH PASSWORD 'secure_pass';
GRANT readwrite TO app_readwrite;

CREATE USER dba_admin WITH PASSWORD 'very_secure_pass';
GRANT admin TO dba_admin;

-- 4. Grant on specific tables
GRANT SELECT, UPDATE ON orders TO app_user;
GRANT SELECT ON sensitive_data TO CURRENT_USER;  -- Only current user

-- 5. Revoke permissions
REVOKE INSERT, UPDATE, DELETE ON orders FROM app_readonly;

-- 6. Row-level security (PostgreSQL 9.5+)
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their own orders
CREATE POLICY orders_user_policy ON orders
    FOR SELECT
    USING (user_id = current_user);

-- Policy: Admins can see all orders
CREATE POLICY orders_admin_policy ON orders
    FOR ALL
    TO admin
    USING (true);

-- MYSQL RBAC

-- 1. Create roles (MySQL 8.0+)
CREATE ROLE 'app_readonly';
CREATE ROLE 'app_readwrite';
CREATE ROLE 'app_admin';

-- 2. Grant privileges to roles
GRANT SELECT ON mydb.* TO 'app_readonly';

GRANT SELECT, INSERT, UPDATE, DELETE ON mydb.* TO 'app_readwrite';

GRANT ALL PRIVILEGES ON mydb.* TO 'app_admin';

-- 3. Create users and assign roles
CREATE USER 'readonly_user'@'%' IDENTIFIED BY 'password';
GRANT 'app_readonly' TO 'readonly_user'@'%';
SET DEFAULT ROLE 'app_readonly' TO 'readonly_user'@'%';

-- 4. Grant specific privileges
GRANT SELECT (customer_id, customer_name) ON customers TO 'limited_user'@'%';
-- Column-level permissions

-- 5. Verify permissions
SHOW GRANTS FOR 'readonly_user'@'%';
SHOW GRANTS FOR CURRENT_USER;
```

### Data Encryption

#### Encryption at Rest

```sql
-- POSTGRESQL

-- 1. Transparent Data Encryption (TDE)
-- Not built-in; use filesystem encryption (LUKS, dm-crypt)
-- Or enterprise extensions (pgcrypto)

-- pgcrypto for column-level encryption:
CREATE EXTENSION pgcrypto;

-- Encrypt data
INSERT INTO users (username, ssn)
VALUES ('alice', pgp_sym_encrypt('123-45-6789', 'encryption_key'));

-- Decrypt data
SELECT username, pgp_sym_decrypt(ssn::bytea, 'encryption_key') AS ssn
FROM users;

-- Better: Use environment variable for key
SELECT pgp_sym_decrypt(ssn::bytea, current_setting('app.encryption_key'));

-- 2. Filesystem encryption (recommended)
-- LUKS (Linux Unified Key Setup)
cryptsetup luksFormat /dev/sdb
cryptsetup luksOpen /dev/sdb pgdata
mkfs.ext4 /dev/mapper/pgdata
mount /dev/mapper/pgdata /var/lib/postgresql

-- All data automatically encrypted at rest

-- MYSQL

-- 1. InnoDB encryption (MySQL 5.7+)
-- Generate master key
mkdir /var/lib/mysql-keyring
chown mysql:mysql /var/lib/mysql-keyring

-- my.cnf:
[mysqld]
early-plugin-load=keyring_file.so
keyring_file_data=/var/lib/mysql-keyring/keyring

-- Create encrypted table
CREATE TABLE sensitive_data (
    id INT PRIMARY KEY,
    credit_card VARCHAR(16)
) ENCRYPTION='Y';

-- Encrypt existing table
ALTER TABLE customers ENCRYPTION='Y';

-- 2. Binary log and relay log encryption
binlog_encryption = ON

-- 3. Undo log and redo log encryption (MySQL 8.0.16+)
innodb_undo_log_encrypt = ON
innodb_redo_log_encrypt = ON
```

#### Encryption in Transit (SSL/TLS)

```sql
-- POSTGRESQL SSL

-- 1. Generate certificates
openssl req -new -x509 -days 365 -nodes -text \
    -out server.crt -keyout server.key \
    -subj "/CN=dbserver.example.com"

-- Set permissions
chmod 600 server.key
chown postgres:postgres server.key server.crt

-- 2. Configure postgresql.conf
ssl = on
ssl_cert_file = 'server.crt'
ssl_key_file = 'server.key'
ssl_prefer_server_ciphers = on
ssl_ciphers = 'HIGH:MEDIUM:+3DES:!aNULL'

-- 3. Require SSL in pg_hba.conf
hostssl  all  all  0.0.0.0/0  scram-sha-256

-- 4. Client connection
psql "host=dbserver.example.com sslmode=require user=myuser dbname=mydb"

-- sslmode options:
-- disable: No SSL
-- allow: Try SSL, fallback to non-SSL
-- prefer: Prefer SSL, fallback to non-SSL
-- require: Require SSL, no certificate validation
-- verify-ca: Require SSL, verify CA
-- verify-full: Require SSL, verify CA and hostname

-- MYSQL SSL

-- 1. Generate certificates
mysql_ssl_rsa_setup --datadir=/var/lib/mysql

-- 2. Configure my.cnf
[mysqld]
require_secure_transport = ON
ssl-ca=/var/lib/mysql/ca.pem
ssl-cert=/var/lib/mysql/server-cert.pem
ssl-key=/var/lib/mysql/server-key.pem

-- 3. Require SSL for user
ALTER USER 'secure_user'@'%' REQUIRE SSL;

-- 4. Client connection
mysql --ssl-mode=REQUIRED -h dbserver.example.com -u myuser -p

-- Verify SSL connection:
SHOW STATUS LIKE 'Ssl_cipher';
-- Should show cipher in use
```

### Auditing and Compliance

```sql
-- POSTGRESQL AUDITING

-- 1. Enable logging (postgresql.conf)
logging_collector = on
log_directory = 'pg_log'
log_filename = 'postgresql-%Y-%m-%d_%H%M%S.log'
log_statement = 'all'  -- Log all statements (or 'ddl', 'mod', 'none')
log_connections = on
log_disconnections = on
log_duration = on
log_line_prefix = '%t [%p]: [%l-1] user=%u,db=%d,app=%a,client=%h '

-- 2. pgaudit extension (detailed auditing)
CREATE EXTENSION pgaudit;

-- Configure pgaudit (postgresql.conf)
pgaudit.log = 'read, write, ddl, role'
pgaudit.log_catalog = off
pgaudit.log_parameter = on

-- Audit specific objects
ALTER TABLE sensitive_data SET (pgaudit.log = 'read, write');

-- 3. Parse logs for compliance reports
-- Example: Find all failed login attempts
grep "FATAL:  password authentication failed" /var/log/postgresql/postgresql-*.log

-- MYSQL AUDIT

-- 1. Audit plugin (MySQL Enterprise or Percona)
INSTALL PLUGIN audit_log SONAME 'audit_log.so';

-- Configure (my.cnf)
[mysqld]
audit_log_format = JSON
audit_log_file = /var/log/mysql/audit.log
audit_log_policy = ALL  -- Log everything

-- 2. General query log (not recommended for production)
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/general.log';

-- 3. Binary log auditing
-- Binary logs record all changes
-- Can be parsed for compliance:
mysqlbinlog --start-datetime="2024-01-01 00:00:00" \
            --stop-datetime="2024-01-31 23:59:59" \
            /var/log/mysql/mysql-bin.* > january_changes.sql
```

### Security Hardening Checklist

```sql
-- POSTGRESQL

-- 1. Remove superuser access from application
-- Never connect as postgres user from application

-- 2. Limit connection sources
-- pg_hba.conf: Only allow specific IPs
host  all  all  10.0.1.0/24  scram-sha-256

-- 3. Disable trust authentication
-- Remove all "trust" entries from pg_hba.conf

-- 4. Use strong passwords
-- Set password policy:
ALTER SYSTEM SET password_encryption = 'scram-sha-256';

-- 5. Restrict function execution
REVOKE EXECUTE ON FUNCTION pg_read_file FROM PUBLIC;
REVOKE EXECUTE ON FUNCTION pg_ls_dir FROM PUBLIC;

-- 6. Limit concurrent connections per user
ALTER ROLE app_user CONNECTION LIMIT 100;

-- 7. Enable statement timeout
ALTER DATABASE mydb SET statement_timeout = '60s';

-- 8. Disable unused extensions
DROP EXTENSION IF EXISTS plpythonu;  -- Can execute arbitrary code

-- 9. Regular security updates
apt update && apt upgrade postgresql-14

-- MYSQL

-- 1. Disable remote root access
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
FLUSH PRIVILEGES;

-- 2. Remove anonymous users
DELETE FROM mysql.user WHERE User='';
FLUSH PRIVILEGES;

-- 3. Remove test database
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';
FLUSH PRIVILEGES;

-- 4. Validate password strength (my.cnf)
[mysqld]
validate_password.policy=STRONG
validate_password.length=12
validate_password.mixed_case_count=1
validate_password.number_count=1
validate_password.special_char_count=1

-- 5. Limit connection attempts
max_connect_errors = 10

-- 6. Disable LOAD DATA LOCAL INFILE
local-infile = 0

-- 7. Run mysql_secure_installation
mysql_secure_installation

-- 8. Bind to specific IP
bind-address = 10.0.1.10
-- Don't bind to 0.0.0.0 unless necessary

-- 9. Disable symbolic links
symbolic-links = 0

-- 10. File permissions
-- Ensure MySQL data directory is owned by mysql user
chown -R mysql:mysql /var/lib/mysql
chmod 750 /var/lib/mysql
```

### SQL Injection Prevention

```sql
-- Never construct SQL with string concatenation!

-- ❌ VULNERABLE (DON'T DO THIS!)
-- Application code (pseudo):
username = request.get('username')
password = request.get('password')
query = "SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'"

-- Attack:
-- username: admin' --
-- password: (anything)
-- Query becomes:
-- SELECT * FROM users WHERE username = 'admin' --' AND password = 'anything'
-- Comment (--) ignores rest of query → logs in as admin!

-- ✅ SAFE: Use parameterized queries / prepared statements

-- Python (psycopg2):
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", 
               (username, password))

-- Python (mysql-connector):
cursor.execute("SELECT * FROM users WHERE username = %s AND password = %s", 
               (username, password))

-- Node.js (pg):
client.query("SELECT * FROM users WHERE username = $1 AND password = $2", 
             [username, password])

-- Node.js (mysql2):
connection.execute("SELECT * FROM users WHERE username = ? AND password = ?", 
                   [username, password])

-- Java (JDBC):
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE username = ? AND password = ?");
stmt.setString(1, username);
stmt.setString(2, password);

-- PHP (PDO):
$stmt = $pdo->prepare("SELECT * FROM users WHERE username = ? AND password = ?");
$stmt->execute([$username, $password]);

-- Prepared statements:
-- 1. Query is parsed once
-- 2. Parameters are bound safely (escaped)
-- 3. SQL injection impossible
-- 4. Better performance (query plan cached)
```

---

## 5.5 Scaling Strategies

As data and traffic grow, single-server databases hit limits. Scaling strategies extend capacity while maintaining performance.

### Vertical Scaling (Scale Up)

```sql
-- Increase server resources (CPU, RAM, disk)

-- Advantages:
-- - Simple (no code changes)
-- - No sharding complexity
-- - ACID guarantees preserved

-- Disadvantages:
-- - Limited (hardware limits)
-- - Expensive (exponential cost curve)
-- - Downtime for upgrades

-- When to scale up:
-- - Database < 1TB
-- - Transactions < 10,000/sec
-- - Simple to implement
-- - Budget allows

-- Hardware recommendations:

-- Small database (< 100GB, < 1000 QPS):
-- - 8 cores
-- - 32 GB RAM
-- - 500 GB SSD

-- Medium database (100GB - 1TB, 1000-10,000 QPS):
-- - 16-32 cores
-- - 128-256 GB RAM
-- - 2-4 TB NVMe SSD
-- - 10 Gbps network

-- Large database (1TB+, 10,000+ QPS):
-- - 64-128 cores
-- - 512 GB - 2 TB RAM
-- - 10+ TB NVMe SSD RAID
-- - 25+ Gbps network

-- Configuration for large server:

-- PostgreSQL (512 GB RAM):
shared_buffers = 128GB                    -- 25% of RAM
effective_cache_size = 384GB              -- 75% of RAM
work_mem = 128MB                          -- Per operation
maintenance_work_mem = 8GB                -- For VACUUM, CREATE INDEX
max_connections = 400
max_parallel_workers_per_gather = 8

-- MySQL (512 GB RAM):
innodb_buffer_pool_size = 384GB           -- 75% of RAM
innodb_log_file_size = 8GB
innodb_flush_log_at_trx_commit = 2        -- Trade durability for performance
innodb_buffer_pool_instances = 16
max_connections = 1000
```

### Horizontal Scaling (Scale Out)

#### Read Replicas

```sql
-- Distribute read queries across replicas

-- Architecture:
-- Primary (writes) ← Application (writes)
--    ↓ replication
-- Replica 1 (reads) ← Application (reads)
-- Replica 2 (reads) ← Application (reads)
-- Replica 3 (reads) ← Application (reads)

-- Capacity:
-- 1 primary + 3 replicas = 4x read capacity

-- Application changes:
-- - Write queries → Primary
-- - Read queries → Replicas (round-robin or load balancer)

-- Example (Python):
primary_conn = psycopg2.connect("host=primary-db")
replica_pool = [
    psycopg2.connect("host=replica1-db"),
    psycopg2.connect("host=replica2-db"),
    psycopg2.connect("host=replica3-db")
]

def write_query(sql, params):
    cursor = primary_conn.cursor()
    cursor.execute(sql, params)
    primary_conn.commit()

def read_query(sql, params):
    replica = random.choice(replica_pool)
    cursor = replica.cursor()
    cursor.execute(sql, params)
    return cursor.fetchall()

-- Limitations:
-- - Replication lag (eventual consistency)
-- - Read-after-write consistency issues:
write_query("INSERT INTO orders VALUES (%s, %s)", (order_id, data))
result = read_query("SELECT * FROM orders WHERE order_id = %s", (order_id,))
-- May not find the order yet (replication lag!)

-- Solutions:
-- 1. Read from primary for critical reads
-- 2. Session affinity (sticky sessions to one replica)
-- 3. Read from primary for N seconds after write
-- 4. Application-level caching
```

#### Connection Pooling

```sql
-- Reduces connection overhead, increases capacity

-- POSTGRESQL: PgBouncer

-- Install:
apt install pgbouncer

-- Configure /etc/pgbouncer/pgbouncer.ini:
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
listen_addr = *
listen_port = 6432
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt
pool_mode = transaction
max_client_conn = 10000
default_pool_size = 25
reserve_pool_size = 5
reserve_pool_timeout = 3

-- pool_mode options:
-- - session: Connection held for entire session (safest)
-- - transaction: Connection held for transaction (recommended)
-- - statement: Connection held for single statement (most aggressive)

-- Create userlist.txt:
"myuser" "md5<hashed_password>"

-- Start PgBouncer:
systemctl start pgbouncer

-- Application connects to PgBouncer instead of PostgreSQL:
psql -h localhost -p 6432 -U myuser mydb

-- Capacity increase:
-- Without pooling: 1000 concurrent connections = 1000 PostgreSQL processes
-- With pooling: 10,000 concurrent connections = 100 PostgreSQL processes
-- 100x more connections with same resources!

-- MYSQL: ProxySQL

-- Install:
apt install proxysql

-- Configure /etc/proxysql.cnf:
mysql_servers =
(
    { address="primary-db" , port=3306 , hostgroup=0 , max_connections=200 },
    { address="replica1-db", port=3306 , hostgroup=1 , max_connections=200 },
    { address="replica2-db", port=3306 , hostgroup=1 , max_connections=200 }
)

mysql_users =
(
    { username = "app_user" , password = "password" , default_hostgroup = 0 , active = 1 }
)

mysql_query_rules =
(
    {
        rule_id=1
        active=1
        match_pattern="^SELECT"
        destination_hostgroup=1  -- Send SELECTs to replicas
        apply=1
    },
    {
        rule_id=2
        active=1
        match_pattern=".*"
        destination_hostgroup=0  -- Send everything else to primary
        apply=1
    }
)

-- Application connects to ProxySQL:
mysql -h proxysql-host -P 6033 -u app_user -p
```

#### Partitioning (Sharding)

```sql
-- Split data across multiple databases

-- Horizontal partitioning (sharding):
-- Partition by customer_id, user_id, region, etc.

-- Example: Shard by customer_id
-- Shard 1: customer_id 0 - 999,999
-- Shard 2: customer_id 1,000,000 - 1,999,999
-- Shard 3: customer_id 2,000,000 - 2,999,999

-- Routing logic (application layer):
def get_shard(customer_id):
    shard_num = customer_id // 1_000_000
    return shard_connections[shard_num]

def insert_order(customer_id, order_data):
    shard = get_shard(customer_id)
    shard.execute("INSERT INTO orders VALUES (%s, %s)", 
                  (customer_id, order_data))

def get_orders(customer_id):
    shard = get_shard(customer_id)
    return shard.execute("SELECT * FROM orders WHERE customer_id = %s", 
                         (customer_id,))

-- Challenges:

-- 1. Cross-shard queries (difficult/expensive):
-- Query: "Get total sales across all customers"
-- Must query all shards and aggregate

-- 2. Cross-shard transactions (not supported):
-- Transaction spanning customer_id 500,000 and 1,500,000
-- Different shards → can't use database transactions
-- Must use distributed transaction (2PC, Saga pattern)

-- 3. Rebalancing when adding shards:
-- Adding Shard 4 requires moving data from Shards 1-3

-- 4. Hotspots:
-- If customer_id 500,000 has 1M orders, Shard 1 is overwhelmed
-- Need to re-shard by (customer_id, order_date) or similar

-- Sharding strategies:

-- 1. Range-based (above example)
-- Pros: Simple, range queries work
-- Cons: Unbalanced (if data skewed)

-- 2. Hash-based
shard_num = hash(customer_id) % num_shards

-- Pros: Evenly distributed
-- Cons: Range queries require all shards

-- 3. Directory-based
-- Lookup table maps customer_id → shard
-- Pros: Flexible, can rebalance without moving data
-- Cons: Extra lookup, single point of failure

-- 4. Geography-based
-- Shard by region (US-East, US-West, EU, Asia)
-- Pros: Data locality, compliance
-- Cons: Unbalanced, cross-region queries expensive

-- Sharding frameworks:
-- - Vitess (MySQL)
-- - Citus (PostgreSQL extension)
-- - Application-level (custom)
```

#### Caching Layer

```sql
-- Cache frequently accessed data in Redis/Memcached

-- Cache hit ratio target: > 80%

-- Example (Python + Redis):
import redis
import psycopg2

redis_client = redis.Redis(host='redis-server')
db_conn = psycopg2.connect("host=db-server")

def get_user(user_id):
    # Try cache first
    cached = redis_client.get(f"user:{user_id}")
    if cached:
        return json.loads(cached)
    
    # Cache miss - query database
    cursor = db_conn.cursor()
    cursor.execute("SELECT * FROM users WHERE user_id = %s", (user_id,))
    user = cursor.fetchone()
    
    # Store in cache (TTL = 1 hour)
    redis_client.setex(f"user:{user_id}", 3600, json.dumps(user))
    
    return user

-- Invalidation strategies:

-- 1. Time-based (TTL)
-- Pros: Simple, automatic
-- Cons: Stale data possible

-- 2. Write-through
-- Update cache on every write
-- Pros: Always fresh
-- Cons: More complex, write overhead

-- 3. Write-invalidate
-- Delete from cache on write
-- Pros: Simple, no stale data
-- Cons: Cache misses after writes

def update_user(user_id, data):
    # Update database
    cursor = db_conn.cursor()
    cursor.execute("UPDATE users SET ... WHERE user_id = %s", (user_id,))
    db_conn.commit()
    
    # Invalidate cache
    redis_client.delete(f"user:{user_id}")

-- Cache patterns:

-- 1. Cache-aside (lazy loading)
-- Check cache → If miss, load from DB → Store in cache

-- 2. Read-through
-- Cache automatically loads from DB on miss

-- 3. Write-through
-- Writes go to cache and DB simultaneously

-- 4. Write-behind
-- Writes go to cache, async flush to DB
-- Risk: Data loss if cache crashes
```

### Multi-Tenant Architectures

```sql
-- Separate databases per tenant (highest isolation):
CREATE DATABASE tenant_1;
CREATE DATABASE tenant_2;
CREATE DATABASE tenant_3;

-- Pros:
-- - Complete isolation (security, performance)
-- - Easy backup/restore per tenant
-- - Easy to move tenant to different server

-- Cons:
-- - Management overhead (1000s of databases)
-- - Schema changes must apply to all databases
-- - Cross-tenant queries impossible

-- Shared database, separate schemas per tenant:
CREATE SCHEMA tenant_1;
CREATE SCHEMA tenant_2;

CREATE TABLE tenant_1.orders (...);
CREATE TABLE tenant_2.orders (...);

-- Pros:
-- - Better resource utilization
-- - Easier management than separate databases

-- Cons:
-- - Must set search_path per query
-- - Less isolation

-- Shared tables with tenant_id column:
CREATE TABLE orders (
    order_id BIGINT,
    tenant_id INT,  -- Discriminator
    customer_id INT,
    ...
    PRIMARY KEY (tenant_id, order_id)
);

CREATE INDEX idx_tenant ON orders(tenant_id);

SELECT * FROM orders WHERE tenant_id = 1;

-- Pros:
-- - Simple, efficient
-- - Easy cross-tenant analytics

-- Cons:
-- - Lowest isolation
-- - Must ensure tenant_id in ALL queries (security risk!)

-- Row-level security (RLS) for safety:
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON orders
    USING (tenant_id = current_setting('app.current_tenant')::int);

-- Application sets tenant:
SET app.current_tenant = 1;
SELECT * FROM orders;  -- Only sees tenant 1's data
```

---

## Key Takeaways

* **SSL/TLS is mandatory for production** - encrypt data in transit
* **Use RBAC for access control** - principle of least privilege
* **Row-level security** adds fine-grained control (PostgreSQL)
* **Prepared statements prevent SQL injection** - never concatenate user input
* **Audit logging tracks compliance** - who did what, when
* **Encryption at rest** protects against disk theft
* **Regular security updates** prevent known vulnerabilities
* **Vertical scaling is simple** but has limits (hardware, cost)
* **Read replicas** scale read workload (4-10x capacity increase)
* **Connection pooling** multiplies capacity (10-100x more connections)
* **Sharding** enables massive scale but adds complexity
* **Caching** reduces database load (80%+ cache hit ratio possible)

---

*This concludes Part 5: Production Operations & Advanced Features*

