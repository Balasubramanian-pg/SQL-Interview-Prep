# SQL Performance & Deep Understanding

*   **Query Execution Order**
> [!IMPORTANT]
> SQL processes clauses in this order, NOT how you write them:
1.  **FROM/JOIN** - Load and join tables
2.  **WHERE** - Filter rows
3.  **GROUP BY** - Aggregate data
4.  **HAVING** - Filter groups
5.  **SELECT** - Choose columns
6.  **ORDER BY** - Sort results
7.  **LIMIT/OFFSET** - Final row limits

**Indexing Deep Dive**

*   **How Indexes Actually Work**
> [!TIP]
> Indexes are B-trees that allow 3-4x faster lookups vs full table scans.
```sql
-- Composite index example (order matters!)
CREATE INDEX idx_employees_dept_date 
ON employees (department_id, hire_date);

-- This uses the index (leftmost prefix)
SELECT * FROM employees 
WHERE department_id = 5 AND hire_date > '2020-01-01';

-- This may not use index efficiently
SELECT * FROM employees 
WHERE hire_date > '2020-01-01'; -- department_id missing
```

*   **Index Selection Rules**
```sql
-- Good index candidates:
-- 1. WHERE clause columns
-- 2. JOIN conditions  
-- 3. ORDER BY columns
-- 4. Columns in covering indexes

-- Covering index (includes all needed columns)
CREATE INDEX idx_cov_orders 
ON orders (customer_id, order_date) 
INCLUDE (total_amount, status);
```

**Execution Plan Analysis**

*   **Key Plan Operators to Understand**
> [!CAUTION]
> Watch for these expensive operations in EXPLAIN plans:
```sql
EXPLAIN (ANALYZE, BUFFERS) 
SELECT * FROM large_table WHERE category = 'A';

-- RED FLAGS:
-- ❌ "Seq Scan" on large tables
-- ❌ "Hash Join" without indexes
-- ❌ "Sort" without index
-- ❌ "Nested Loop" with large tables

-- GREEN FLAGS:
-- ✅ "Index Scan"/"Index Only Scan"
-- ✅ "Bitmap Heap Scan" 
-- ✅ "Merge Join" with indexes
```

**Query Performance Patterns**

*   **The N+1 Query Problem**
> [!WARNING]
> Avoid multiple round trips - use JOINs instead.
```sql
-- ❌ BAD: N+1 queries
-- 1 query to get customers
-- N queries to get orders per customer

-- ✅ GOOD: Single query with JOIN
SELECT c.customer_id, c.name, o.order_date, o.amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.signup_date > '2024-01-01';
```

*   **Correlated Subquery Optimization**
```sql
-- ❌ SLOW: Correlated subquery
SELECT employee_id, salary,
    (SELECT AVG(salary) FROM employees e2 
     WHERE e2.department = e1.department) AS dept_avg
FROM employees e1;

-- ✅ FAST: Window function
SELECT employee_id, salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg
FROM employees;
```

**Advanced JOIN Strategies**

*   **JOIN Order Matters**
> [!NOTE]
> Smaller tables first, filtered tables first.
```sql
-- ✅ Better: Filter early, smaller table first
SELECT *
FROM (SELECT * FROM small_table WHERE active = true) small
JOIN large_table large ON small.id = large.small_id;

-- ❌ Worse: Large table first
SELECT *
FROM large_table large
JOIN small_table small ON large.small_id = small.id
WHERE small.active = true;
```

**Statistics and Cardinality**

*   **How the Optimizer Thinks**
```sql
-- Check table statistics
ANALYZE employees; -- Update statistics

-- View statistics
SELECT tablename, attname, n_distinct, most_common_vals
FROM pg_stats 
WHERE tablename = 'employees';

-- Cardinality estimation examples:
-- WHERE id = 5        --> 1 row expected
-- WHERE status = 'A'   --> (n_distinct) rows expected  
-- WHERE date > '2024' --> 30% of table expected
```

**Temporary Tables vs CTEs**

*   **Performance Characteristics**
```sql
-- CTE: Optimizer may inline or materialize
WITH recent_orders AS (
    SELECT * FROM orders WHERE order_date > '2024-01-01'
)
SELECT * FROM recent_orders;

-- Temp Table: Explicit materialization
CREATE TEMP TABLE recent_orders AS
SELECT * FROM orders WHERE order_date > '2024-01-01';

SELECT * FROM recent_orders; -- Faster for multiple uses
```

**Parameter Sniffing Problem**

*   **Cached Plan Issues**
```sql
-- Problem: First execution with parameter = 1 (10 rows)
-- Subsequent executions with parameter = 2 (1M rows) use wrong plan

-- Solutions:
-- 1. Use OPTION (RECOMPILE)
-- 2. Use local variables
-- 3. Use query hints
```

**Locking and Concurrency**

*   **Understanding Isolation Levels**
```sql
-- Common issues:
-- ❌ Lost updates
-- ❌ Dirty reads  
-- ❌ Phantom reads
-- ❌ Deadlocks

-- Set isolation level
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Use row-level locking
SELECT * FROM accounts 
WHERE id = 1 
FOR UPDATE; -- Exclusive lock
```

**Advanced Aggregation Performance**

*   **GROUP BY Optimization**
```sql
-- ❌ Slow: Grouping unindexed columns
SELECT category, COUNT(*) 
FROM products 
GROUP BY category; -- No index on category

-- ✅ Fast: Grouping indexed columns  
SELECT category, COUNT(*)
FROM products 
GROUP BY category; -- With index on category

-- ✅ Better: Filter before grouping
SELECT category, COUNT(*)
FROM products
WHERE price > 100  -- Reduce dataset first
GROUP BY category;
```

**Window Function Performance**

*   **Efficient Window Patterns**
```sql
-- ❌ Expensive: Multiple window definitions
SELECT 
    AVG(salary) OVER (PARTITION BY dept),
    MAX(salary) OVER (PARTITION BY dept),
    MIN(salary) OVER (PARTITION BY dept)
FROM employees;

-- ✅ Better: Single window definition
SELECT 
    AVG(salary) OVER w,
    MAX(salary) OVER w, 
    MIN(salary) OVER w
FROM employees
WINDOW w AS (PARTITION BY dept);
```

**Materialized Views**

*   **Pre-computed Results**
```sql
-- For expensive aggregations
CREATE MATERIALIZED VIEW sales_summary AS
SELECT 
    product_id,
    SUM(quantity) as total_sold,
    AVG(price) as avg_price,
    COUNT(*) as transaction_count
FROM sales
GROUP BY product_id;

-- Refresh periodically
REFRESH MATERIALIZED VIEW sales_summary;
```

**Connection Pooling Impact**

*   **Architecture Considerations**
```sql
-- Each connection has costs:
-- ❌ Memory per connection
-- ❌ Locks and transactions
-- ❌ Temp table space

-- Best practices:
-- ✅ Use connection pooling
-- ✅ Keep transactions short
-- ✅ Release connections quickly
```

**Hardware Awareness**

*   **IO and Memory Patterns**
```sql
-- Memory-intensive operations:
-- ❌ Large sorts (ORDER BY without index)
-- ❌ Hash joins on large tables
-- ❌ Window functions with large partitions

-- IO-intensive operations:
-- ❌ Full table scans
-- ❌ Large index scans
-- ❌ Temp table spills to disk
```

**Query Hints (Use Sparingly)**

*   **Override Optimizer**
```sql
-- PostgreSQL examples
SELECT * FROM employees 
WHERE department_id = 5
ORDER BY hire_date;

-- Force index usage
SET enable_seqscan = off;

-- Force join order
SET join_collapse_limit = 1;
```

**Monitoring and Diagnostics**

*   **Find Slow Queries**
```sql
-- PostgreSQL: Check pg_stat_statements
SELECT query, calls, total_time, mean_time, rows
FROM pg_stat_statements 
ORDER BY mean_time DESC 
LIMIT 10;

-- Find missing indexes
SELECT schemaname, tablename, seq_scan, seq_tup_read,
       seq_tup_read / seq_scan as avg_tuples_per_scan
FROM pg_stat_user_tables 
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC;
```

**Deep Understanding Checks**

*   **Test Your Knowledge**
```sql
-- What's the performance difference?
SELECT COUNT(*) FROM users; -- Fast
SELECT COUNT(id) FROM users; -- Fast  
SELECT COUNT(email) FROM users; -- Slow if email has NULLs

-- What executes first?
SELECT COUNT(*) 
FROM orders 
WHERE customer_id IN (
    SELECT customer_id FROM customers WHERE active = true
);
-- Answer: Subquery executes first, then main query
```

**Common Performance Anti-patterns**

*   **What to Avoid**
```sql
-- ❌ SELECT * (especially with JOINs)
-- ❌ Functions on indexed columns in WHERE
-- ❌ OR conditions that prevent index usage  
-- ❌ Implicit type conversions
-- ❌ Cursors for set-based operations
-- ❌ Over-normalization requiring excessive JOINs
```
