---
layout: post
title: "Database Indexing: Stop Guessing, Start Measuring"
date: 2025-10-15 10:45:00 +0800
categories: database performance postgresql
author: Casey Liu
---

Our queries were slow. We added indexes. They got slower. Here's what we learned about database indexing the hard way.

## The Problem

Query taking 8 seconds on a table with 10M rows:

```sql
SELECT * FROM orders
WHERE user_id = 123
  AND status = 'pending'
  AND created_at > NOW() - INTERVAL '30 days';
```

**First instinct**: Add indexes on everything!

```sql
CREATE INDEX idx_user_id ON orders(user_id);
CREATE INDEX idx_status ON orders(status);
CREATE INDEX idx_created_at ON orders(created_at);
```

**Result**: Query still slow. Disk usage up 40%. Writes slower.

## What We Got Wrong

### Mistake 1: Single-Column Indexes for Multi-Column Queries

PostgreSQL can only efficiently use ONE index per table scan (usually).

The query above has 3 conditions. PostgreSQL picks the most selective index, scans it, then filters the rest in memory.

**Fix**: Composite index

```sql
CREATE INDEX idx_user_status_date ON orders(user_id, status, created_at);
```

Query time: 8s → 45ms

### Mistake 2: Wrong Column Order

Column order in composite indexes **matters**.

**Rule**: Most selective column first? **Wrong**.

**Correct rule**: Match your query patterns.

```sql
-- Query pattern: Always filter by user_id, sometimes by status
SELECT * FROM orders WHERE user_id = 123;
SELECT * FROM orders WHERE user_id = 123 AND status = 'pending';

-- Index should be:
CREATE INDEX idx_user_status ON orders(user_id, status);

-- NOT:
CREATE INDEX idx_status_user ON orders(status, user_id);  -- Won't help first query
```

Composite index can be used for leading columns:
- `(user_id, status)` helps: `WHERE user_id = ?` ✅
- `(user_id, status)` helps: `WHERE user_id = ? AND status = ?` ✅
- `(user_id, status)` **doesn't help**: `WHERE status = ?` ❌

### Mistake 3: Over-Indexing

Every index has a cost:
- Disk space
- Write performance (every INSERT/UPDATE/DELETE must update indexes)
- Query planner overhead (more options = slower planning)

We had 47 indexes on a 15-column table. Insane.

**Fix**: Remove unused indexes.

Find them:
```sql
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexname NOT LIKE '%pkey%';
```

We dropped 28 indexes. Write performance improved 3x.

## When to Use Different Index Types

### B-tree (Default)

Use for: Equality and range queries

```sql
CREATE INDEX idx_price ON products(price);

-- Good queries:
WHERE price = 99.99
WHERE price > 50 AND price < 100
ORDER BY price
```

### Hash

Use for: Equality only (slightly faster, less space)

```sql
CREATE INDEX idx_hash_user_id ON orders USING HASH (user_id);

-- Good:
WHERE user_id = 123

-- Bad (won't use index):
WHERE user_id > 100
```

### GIN (Generalized Inverted Index)

Use for: Arrays, JSONB, full-text search

```sql
CREATE INDEX idx_tags ON articles USING GIN (tags);

-- Good:
WHERE tags @> ARRAY['postgresql', 'indexing']  -- Contains
WHERE tags && ARRAY['performance']              -- Overlaps
```

### BRIN (Block Range Index)

Use for: Very large tables with natural ordering (e.g., timestamps)

```sql
CREATE INDEX idx_brin_created ON logs USING BRIN (created_at);
```

Tiny index size (100x smaller than B-tree), but only efficient if data is physically ordered on disk.

## Partial Indexes (Game Changer)

Only index rows you actually query:

```sql
-- We only query active users
CREATE INDEX idx_active_users ON users(email)
WHERE status = 'active';
```

Benefits:
- Smaller index (faster scans, less disk)
- Faster writes (inactive users don't update index)

## Covering Indexes (INCLUDE)

Avoid table lookups by including extra columns:

```sql
CREATE INDEX idx_user_id_covering ON orders(user_id)
INCLUDE (status, total, created_at);

-- This query uses ONLY the index (no table access):
SELECT status, total, created_at
FROM orders
WHERE user_id = 123;
```

Index-only scans are fast.

## Our Indexing Workflow

### 1. Identify Slow Queries

Enable slow query log:
```sql
ALTER SYSTEM SET log_min_duration_statement = 1000;  -- Log queries > 1s
```

### 2. Analyze Execution Plan

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 123 AND status = 'pending';
```

Look for:
- **Seq Scan** (table scan) - bad for large tables
- **Index Scan** - good
- **Index Only Scan** - excellent

### 3. Test Index in Isolation

```sql
BEGIN;
CREATE INDEX CONCURRENTLY idx_test ON orders(user_id, status);
EXPLAIN ANALYZE [your query];
ROLLBACK;  -- Don't commit yet
```

### 4. Measure Impact

- Query performance
- Index size (`pg_relation_size('idx_test')`)
- Write performance (run benchmark inserts)

### 5. Deploy with CONCURRENTLY

```sql
CREATE INDEX CONCURRENTLY idx_user_status ON orders(user_id, status);
```

Doesn't block writes (critical for production).

## Common Pitfalls

### 1. Function Calls in WHERE

```sql
-- Won't use index on email:
WHERE LOWER(email) = 'test@example.com'

-- Fix: Functional index
CREATE INDEX idx_lower_email ON users(LOWER(email));
```

### 2. OR Conditions

```sql
-- Hard to optimize:
WHERE user_id = 123 OR status = 'pending'

-- Fix: UNION or separate queries
```

### 3. Wildcard Prefix

```sql
-- Won't use index:
WHERE name LIKE '%son'

-- Will use index:
WHERE name LIKE 'John%'

-- For full-text: Use GIN + tsvector
```

## Results

**Before optimization**:
- Average query time: 3.2s
- Indexes: 47
- Disk usage: 180GB

**After**:
- Average query time: 120ms (96% improvement)
- Indexes: 19
- Disk usage: 95GB

## Tools We Use

- **pg_stat_statements**: Track query performance
- **EXPLAIN ANALYZE**: Understand execution plans
- **pg_stat_user_indexes**: Find unused indexes
- **pgAdmin / DataGrip**: Visualize explain plans

Indexing is not guesswork. Measure, test, validate.

---

*Database performance questions? Fire away!*
