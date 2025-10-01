# Cheat Sheet: Subquery Patterns in Interviews

| Interview Question Pattern | What They're Really Testing | Best Approach | Why |
|---|---|---|---|
| **Find employees earning more than average** | Basic subquery in WHERE | `WHERE salary > (SELECT AVG(salary) FROM employees)` | Scalar subquery returns single value |
| **Find employees in departments with >100 people** | Do you know IN vs EXISTS | `WHERE dept_id IN (SELECT dept_id FROM employees GROUP BY dept_id HAVING COUNT(*) > 100)` | IN when checking against a list |
| **Find employees who have at least one sale** | EXISTS vs IN performance awareness | `WHERE EXISTS (SELECT 1 FROM sales WHERE sales.emp_id = employees.id)` | EXISTS stops at first match (faster) |
| **Find departments with NO employees** | The NOT EXISTS pattern | `WHERE NOT EXISTS (SELECT 1 FROM employees WHERE employees.dept_id = departments.id)` | Most efficient way to find missing relationships |
| **Find products never ordered** | NOT IN trap awareness | `WHERE id NOT IN (SELECT product_id FROM orders WHERE product_id IS NOT NULL)` | NOT IN breaks with NULLs! Must filter them out |
| **Get employee name with their department name** | Scalar subquery in SELECT | `SELECT name, (SELECT dept_name FROM departments WHERE id = employees.dept_id) AS dept` | Can work, but JOIN usually better |
| **Find top 3 highest paid per department** | Correlated subquery knowledge | `WHERE (SELECT COUNT(*) FROM employees e2 WHERE e2.dept = e1.dept AND e2.salary > e1.salary) < 3` | Classic but slow—use window functions instead! |
| **Compare each sale to department average** | Correlated vs non-correlated | Correlated: runs for EACH row. Non-correlated: runs ONCE | Critical performance difference |
| **Create derived table for calculations** | Subquery in FROM clause | `FROM (SELECT dept_id, AVG(salary) as avg_sal FROM employees GROUP BY dept_id) dept_avg` | Treat subquery result as a table |
| **Filter based on aggregated data** | CTE vs subquery readability | `WITH high_earners AS (SELECT...) SELECT * FROM high_earners` | CTE is cleaner for complex queries |

## Quick Decision Tree

```
Need to check if rows exist?
├─ YES → Use EXISTS (fastest for existence checks)
└─ NO → Need to compare against values?
    ├─ Single value → Scalar subquery (=, >, <)
    ├─ List of values → IN (but watch for NULLs!)
    └─ Complex logic → JOIN might be better

Is subquery executed once or per row?
├─ Once → Non-correlated (better performance)
└─ Per row → Correlated (slower, but sometimes necessary)

Need readable, reusable query?
└─ YES → Use CTE instead of nested subqueries
```

## Correlated vs Non-Correlated Subqueries

**Non-Correlated (Independent):**
```sql
-- Runs ONCE, result is reused
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
```

**Correlated (Dependent):**
```sql
-- Runs for EVERY row in outer query!
SELECT e1.name, e1.salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary) 
    FROM employees e2 
    WHERE e2.dept_id = e1.dept_id  -- References outer query!
)
```

**Interview Gold:** Always mention that correlated subqueries are slower because they execute repeatedly!

## The EXISTS vs IN Battle

| Scenario | Best Choice | Reason |
|---|---|---|
| Checking existence (don't need values) | `EXISTS` | Stops at first match, doesn't return data |
| Small list of known values | `IN (1, 2, 3)` | Simple and efficient |
| Subquery returns large result set | `EXISTS` | Doesn't materialize full result set |
| Subquery might have NULLs | `EXISTS` | NULL-safe, IN is not! |
| Need to check non-existence | `NOT EXISTS` | Safest approach |
| Avoiding NULLs with NOT IN | `NOT IN + IS NOT NULL filter` | Must explicitly handle NULLs |

**The NULL Trap with IN/NOT IN:**
```sql
-- ❌ WRONG: Returns nothing if subquery has any NULL!
SELECT * FROM products
WHERE id NOT IN (SELECT product_id FROM orders)
-- If ANY product_id is NULL, entire result is empty!

-- ✅ CORRECT: Filter NULLs or use NOT EXISTS
SELECT * FROM products
WHERE id NOT IN (SELECT product_id FROM orders WHERE product_id IS NOT NULL)

-- ✅ BETTER: Use NOT EXISTS (NULL-safe)
SELECT * FROM products p
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.product_id = p.id)
```

**Interview Power Move:** Mention that `NOT IN` with NULLs is a classic SQL trap!

## Subquery Locations & Use Cases

**1. WHERE Clause (Most Common)**
```sql
-- Scalar comparison
WHERE salary > (SELECT AVG(salary) FROM employees)

-- List membership
WHERE dept_id IN (SELECT id FROM departments WHERE location = 'NYC')

-- Existence check
WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = customers.id)
```

**2. SELECT Clause (Scalar Subqueries)**
```sql
-- Must return exactly ONE value per row
SELECT 
    name,
    salary,
    (SELECT AVG(salary) FROM employees) as company_avg,
    (SELECT COUNT(*) FROM orders WHERE customer_id = customers.id) as order_count
FROM employees
```

**Trap:** If subquery returns multiple rows, query fails! Always ensure single value.

**3. FROM Clause (Derived Tables)**
```sql
-- Treat subquery result as a table
SELECT dept, avg_salary
FROM (
    SELECT dept_id as dept, AVG(salary) as avg_salary
    FROM employees
    GROUP BY dept_id
) dept_stats
WHERE avg_salary > 50000
```

**4. HAVING Clause (After Aggregation)**
```sql
SELECT dept_id, AVG(salary)
FROM employees
GROUP BY dept_id
HAVING AVG(salary) > (SELECT AVG(salary) FROM employees)
```

## CTE vs Subquery vs Temp Table

| Feature | CTE | Subquery | Temp Table |
|---|---|---|---|
| Readability | ⭐⭐⭐⭐⭐ Best | ⭐⭐ Nested = messy | ⭐⭐⭐ Good |
| Reusability in same query | ✅ Can reference multiple times | ❌ Must repeat | ✅ Can reference |
| Performance | Same as subquery | Same as CTE | Can be indexed! |
| Recursive queries | ✅ Yes | ❌ No | ❌ No |
| Scope | Single query | Single query | Entire session |
| Best for | Complex multi-step logic | Simple one-time use | Large intermediate results |

**When to Use Each:**

**CTE (WITH clause):**
```sql
-- ✅ Use when: Multiple references or complex logic
WITH high_earners AS (
    SELECT * FROM employees WHERE salary > 100000
),
top_depts AS (
    SELECT dept_id FROM high_earners GROUP BY dept_id HAVING COUNT(*) > 5
)
SELECT * FROM high_earners WHERE dept_id IN (SELECT dept_id FROM top_depts)
```

**Subquery:**
```sql
-- ✅ Use when: Simple, one-time filtering
SELECT * FROM employees
WHERE dept_id IN (SELECT id FROM departments WHERE region = 'West')
```

**Temp Table:**
```sql
-- ✅ Use when: Large intermediate result, need indexes, or multiple queries
CREATE TEMP TABLE high_earners AS
SELECT * FROM employees WHERE salary > 100000;

CREATE INDEX idx_dept ON high_earners(dept_id);

SELECT * FROM high_earners WHERE dept_id = 5;
SELECT AVG(salary) FROM high_earners;
```

## Performance Traps & Optimizations

**Trap 1: Correlated Subquery in SELECT**
```sql
-- ❌ SLOW: Runs subquery for EVERY row
SELECT 
    e.name,
    (SELECT COUNT(*) FROM sales WHERE emp_id = e.id) as sale_count
FROM employees e

-- ✅ FAST: Use JOIN or window function
SELECT e.name, COUNT(s.id) as sale_count
FROM employees e
LEFT JOIN sales s ON s.emp_id = e.id
GROUP BY e.name
```

**Trap 2: Subquery in WHERE with Large Result**
```sql
-- ❌ SLOW: IN materializes entire result set
WHERE id IN (SELECT customer_id FROM orders WHERE amount > 1000)

-- ✅ FASTER: EXISTS stops at first match
WHERE EXISTS (SELECT 1 FROM orders WHERE customer_id = customers.id AND amount > 1000)
```

**Trap 3: Multiple Uncorrelated Subqueries**
```sql
-- ❌ SLOW: Scans employees table 3 times
SELECT *
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
  AND dept_id IN (SELECT id FROM departments WHERE budget > (SELECT AVG(budget) FROM departments))

-- ✅ FASTER: Use CTE to compute once
WITH stats AS (
    SELECT AVG(salary) as avg_sal FROM employees
)
SELECT * FROM employees, stats
WHERE salary > stats.avg_sal
```

## Classic Interview Patterns

**Pattern 1: "Second Highest" (Correlated Subquery Way)**
```sql
-- Classic but SLOW approach
SELECT DISTINCT salary
FROM employees e1
WHERE 1 = (
    SELECT COUNT(DISTINCT salary)
    FROM employees e2
    WHERE e2.salary > e1.salary
)

-- Better: Use window functions (see ranking cheat sheet!)
```

**Pattern 2: Finding Gaps in Sequences**
```sql
-- Find missing order IDs
SELECT t1.id + 1 as missing_id
FROM orders t1
WHERE NOT EXISTS (
    SELECT 1 FROM orders t2 WHERE t2.id = t1.id + 1
)
AND t1.id < (SELECT MAX(id) FROM orders)
```

**Pattern 3: All or Nothing (Relational Division)**
```sql
-- Find customers who bought ALL product types
SELECT customer_id
FROM orders
GROUP BY customer_id
HAVING COUNT(DISTINCT product_type) = (
    SELECT COUNT(DISTINCT product_type) FROM products
)
```

## When Interviewer Asks "Why?"

| Question | Gold Answer |
|---|---|
| **"Why use EXISTS over IN?"** | EXISTS is usually faster—stops at first match and doesn't materialize full result set. Also NULL-safe. |
| **"When would you use a subquery in SELECT?"** | When you need a computed value per row, but mention it's often slower than JOIN or window functions. |
| **"CTE vs Subquery?"** | CTEs improve readability and can be referenced multiple times. Same performance as subquery. |
| **"What's a correlated subquery?"** | One that references columns from outer query—runs for every row, so slower than non-correlated. |
| **"Why avoid NOT IN with NULLs?"** | SQL uses three-valued logic. If subquery has NULL, NOT IN returns no results. Use NOT EXISTS instead. |

## Interview Power Moves

When discussing subqueries, mention:

1. **"Correlated subqueries run per row—I'd check if a window function or JOIN could replace it"** (shows performance awareness)

2. **"NOT IN with potential NULLs is a trap—I always use NOT EXISTS or filter NULLs explicitly"** (shows you know edge cases)

3. **"For complex queries, I prefer CTEs for readability—same performance, easier maintenance"** (shows you think about code quality)

4. **"EXISTS is typically faster than IN for large result sets because it short-circuits"** (shows optimization knowledge)

5. **"I'd check the execution plan to see if the optimizer is materializing the subquery"** (shows you understand database internals)
