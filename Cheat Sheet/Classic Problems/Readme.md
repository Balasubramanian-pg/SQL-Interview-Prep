# Cheat Sheet: Classic SQL Interview Problems

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
|---|---|---|---|
| **Find duplicate records** | Basic GROUP BY + HAVING | `GROUP BY columns HAVING COUNT(*) > 1` | Count occurrences |
| **Remove duplicates, keep one** | ROW_NUMBER with DELETE/WHERE | `ROW_NUMBER() OVER(PARTITION BY dup_cols ORDER BY id)` then filter `rn = 1` | Window function deduplication |
| **Find gaps in sequences** | Self-join or NOT EXISTS | `WHERE NOT EXISTS (SELECT 1 WHERE id = t1.id + 1)` | Check for missing next value |
| **Find consecutive events** | LAG/LEAD or self-join | `WHERE date = LAG(date) OVER(ORDER BY date) + 1` | Compare adjacent rows |
| **Top N per group** | ROW_NUMBER + PARTITION BY | `ROW_NUMBER() OVER(PARTITION BY group_col ORDER BY value DESC)` filter `<= N` | Ranking within partitions |
| **Running totals** | SUM() OVER with frame | `SUM(amount) OVER(ORDER BY date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` | Cumulative window function |
| **Moving averages** | AVG() OVER with frame | `AVG(value) OVER(ORDER BY date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)` | Sliding window calculation |
| **First/Last in group** | FIRST_VALUE / LAST_VALUE | `FIRST_VALUE(col) OVER(PARTITION BY group ORDER BY date)` | Get boundary values |
| **Compare to previous row** | LAG function | `value - LAG(value) OVER(ORDER BY date)` | Access previous row |
| **Employee-Manager hierarchy** | Self-join | `FROM employees e LEFT JOIN employees m ON e.manager_id = m.id` | Join table to itself |
| **N-level recursion** (org chart depth) | Recursive CTE | `WITH RECURSIVE cte AS (base UNION ALL recursive)` | Build hierarchy levels |
| **Find records with no match** | LEFT JOIN + IS NULL or NOT EXISTS | `LEFT JOIN table2 ON ... WHERE table2.id IS NULL` | Anti-join pattern |
| **Pivot rows to columns** | CASE + GROUP BY or PIVOT | `SUM(CASE WHEN category = 'X' THEN amount END)` | Conditional aggregation |
| **Unpivot columns to rows** | UNION ALL or UNPIVOT | `SELECT 'col1', col1 UNION ALL SELECT 'col2', col2` | Stack columns as rows |
| **Date range overlaps** | WHERE with date comparisons | `WHERE start1 <= end2 AND end1 >= start2` | Interval intersection logic |
| **Same value in ALL related records** | NOT EXISTS with negation | `WHERE NOT EXISTS (SELECT 1 WHERE condition != expected)` | Check universal condition |
| **Cumulative distinct count** | COUNT(DISTINCT) with window | Complex—usually requires self-join or specialized function | Running unique count |

## Pattern 1: Finding Duplicates

**Basic Duplicates (show duplicate values):**
```sql
-- Find emails that appear more than once
SELECT email, COUNT(*) as duplicate_count
FROM users
GROUP BY email
HAVING COUNT(*) > 1
```

**Show All Duplicate Rows (including details):**
```sql
-- Method 1: Subquery
SELECT *
FROM users
WHERE email IN (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
)

-- Method 2: Window function (better performance)
SELECT *
FROM (
    SELECT *, COUNT(*) OVER(PARTITION BY email) as cnt
    FROM users
) sub
WHERE cnt > 1
```

**Find Duplicates Based on Multiple Columns:**
```sql
SELECT first_name, last_name, COUNT(*) as dup_count
FROM users
GROUP BY first_name, last_name
HAVING COUNT(*) > 1
```

## Pattern 2: Removing Duplicates

**Keep First Occurrence (by ID):**
```sql
-- Using ROW_NUMBER (most common)
DELETE FROM users
WHERE id IN (
    SELECT id
    FROM (
        SELECT id, ROW_NUMBER() OVER(PARTITION BY email ORDER BY id) as rn
        FROM users
    ) sub
    WHERE rn > 1
)

-- Or SELECT to see what would be kept
SELECT *
FROM (
    SELECT *, ROW_NUMBER() OVER(PARTITION BY email ORDER BY id) as rn
    FROM users
) sub
WHERE rn = 1
```

**Keep Most Recent (by date):**
```sql
WITH ranked AS (
    SELECT *,
           ROW_NUMBER() OVER(PARTITION BY email ORDER BY created_at DESC) as rn
    FROM users
)
SELECT * FROM ranked WHERE rn = 1
```

**Interview Trick:** Mention that `DELETE` with subquery may need a temp table in some databases (like MySQL).

## Pattern 3: Finding Gaps in Sequences

**Missing IDs in Sequence:**
```sql
-- Method 1: Self-join (classic)
SELECT t1.id + 1 as missing_id
FROM orders t1
LEFT JOIN orders t2 ON t2.id = t1.id + 1
WHERE t2.id IS NULL
  AND t1.id < (SELECT MAX(id) FROM orders)

-- Method 2: NOT EXISTS (cleaner)
SELECT t1.id + 1 as gap_start
FROM orders t1
WHERE NOT EXISTS (
    SELECT 1 FROM orders t2 WHERE t2.id = t1.id + 1
)
AND t1.id < (SELECT MAX(id) FROM orders)

-- Method 3: Window function (modern)
SELECT id + 1 as gap_start,
       LEAD(id) OVER(ORDER BY id) - 1 as gap_end
FROM orders
WHERE LEAD(id) OVER(ORDER BY id) - id > 1
```

**Missing Dates in Date Range:**
```sql
-- Generate all dates and find missing ones
WITH RECURSIVE date_range AS (
    SELECT MIN(order_date) as dt FROM orders
    UNION ALL
    SELECT dt + INTERVAL 1 DAY
    FROM date_range
    WHERE dt < (SELECT MAX(order_date) FROM orders)
)
SELECT dr.dt as missing_date
FROM date_range dr
LEFT JOIN orders o ON o.order_date = dr.dt
WHERE o.order_date IS NULL
```

## Pattern 4: Consecutive Events

**Find Consecutive Days with Activity:**
```sql
-- Using LAG to compare adjacent rows
WITH flagged AS (
    SELECT 
        user_id,
        activity_date,
        LAG(activity_date) OVER(PARTITION BY user_id ORDER BY activity_date) as prev_date
    FROM user_activity
)
SELECT 
    user_id,
    activity_date,
    CASE 
        WHEN activity_date = prev_date + INTERVAL 1 DAY THEN 'Consecutive'
        ELSE 'Gap'
    END as streak_status
FROM flagged
```

**Count Consecutive Streak Length:**
```sql
-- Identify groups of consecutive dates
WITH groups AS (
    SELECT 
        user_id,
        activity_date,
        activity_date - ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY activity_date) as grp
    FROM user_activity
)
SELECT 
    user_id,
    MIN(activity_date) as streak_start,
    MAX(activity_date) as streak_end,
    COUNT(*) as streak_length
FROM groups
GROUP BY user_id, grp
```

**Interview Gold:** The `date - ROW_NUMBER()` trick creates constant values for consecutive sequences!

## Pattern 5: Top N Per Group

**Top 3 Salaries Per Department:**
```sql
-- Method 1: ROW_NUMBER (one per rank)
SELECT *
FROM (
    SELECT 
        dept_id,
        employee_name,
        salary,
        ROW_NUMBER() OVER(PARTITION BY dept_id ORDER BY salary DESC) as rn
    FROM employees
) ranked
WHERE rn <= 3

-- Method 2: DENSE_RANK (includes ties)
SELECT *
FROM (
    SELECT 
        dept_id,
        employee_name,
        salary,
        DENSE_RANK() OVER(PARTITION BY dept_id ORDER BY salary DESC) as rnk
    FROM employees
) ranked
WHERE rnk <= 3
```

**Latest Record Per Customer:**
```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date DESC) as rn
    FROM orders
) latest
WHERE rn = 1
```

## Pattern 6: Running Totals & Cumulative Sums

**Basic Running Total:**
```sql
SELECT 
    order_date,
    amount,
    SUM(amount) OVER(ORDER BY order_date) as running_total
FROM orders
```

**Running Total Per Group:**
```sql
SELECT 
    dept_id,
    expense_date,
    amount,
    SUM(amount) OVER(PARTITION BY dept_id ORDER BY expense_date) as dept_running_total
FROM expenses
```

**Running Total with Frame (explicit):**
```sql
SELECT 
    order_date,
    amount,
    SUM(amount) OVER(
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_total
FROM orders
```

**Interview Trap:** Without `ROWS BETWEEN`, behavior can be unexpected with duplicate ORDER BY values!

## Pattern 7: Moving Averages

**Last 3 Rows Average:**
```sql
SELECT 
    date,
    value,
    AVG(value) OVER(
        ORDER BY date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) as moving_avg_3
FROM metrics
```

**Centered Moving Average (1 before, current, 1 after):**
```sql
SELECT 
    date,
    value,
    AVG(value) OVER(
        ORDER BY date
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) as centered_avg
FROM metrics
```

**Moving Average Per Group:**
```sql
SELECT 
    product_id,
    sale_date,
    quantity,
    AVG(quantity) OVER(
        PARTITION BY product_id
        ORDER BY sale_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as weekly_avg
FROM sales
```

## Pattern 8: Compare to Previous/Next Row

**Difference from Previous Row:**
```sql
SELECT 
    date,
    value,
    value - LAG(value) OVER(ORDER BY date) as change_from_prev
FROM metrics
```

**Percent Change:**
```sql
SELECT 
    date,
    value,
    ROUND(
        (value - LAG(value) OVER(ORDER BY date)) * 100.0 / 
        LAG(value) OVER(ORDER BY date),
        2
    ) as pct_change
FROM metrics
```

**Compare Next 3 Rows:**
```sql
SELECT 
    employee_id,
    salary,
    LEAD(salary, 1) OVER(ORDER BY salary) as next_salary,
    LEAD(salary, 2) OVER(ORDER BY salary) as salary_plus_2,
    LEAD(salary, 3) OVER(ORDER BY salary) as salary_plus_3
FROM employees
```

## Pattern 9: First/Last Value in Group

**First and Last Salary Per Department:**
```sql
SELECT DISTINCT
    dept_id,
    FIRST_VALUE(salary) OVER(
        PARTITION BY dept_id 
        ORDER BY hire_date
    ) as first_hire_salary,
    LAST_VALUE(salary) OVER(
        PARTITION BY dept_id 
        ORDER BY hire_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) as last_hire_salary
FROM employees
```

**Interview Trap:** `LAST_VALUE` needs `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` or it only looks at current row!

## Pattern 10: Self-Join (Employee-Manager)

**Get Employee with Manager Name:**
```sql
SELECT 
    e.employee_name,
    e.salary,
    m.employee_name as manager_name,
    m.salary as manager_salary
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id
```

**Find Employees Earning More Than Manager:**
```sql
SELECT 
    e.employee_name,
    e.salary,
    m.employee_name as manager_name,
    m.salary as manager_salary
FROM employees e
INNER JOIN employees m ON e.manager_id = m.employee_id
WHERE e.salary > m.salary
```

## Pattern 11: Recursive CTE (Hierarchy Depth)

**Find All Subordinates Under a Manager:**
```sql
WITH RECURSIVE subordinates AS (
    -- Base case: direct reports
    SELECT employee_id, employee_name, manager_id, 1 as level
    FROM employees
    WHERE manager_id = 5  -- Starting manager
    
    UNION ALL
    
    -- Recursive case: reports of reports
    SELECT e.employee_id, e.employee_name, e.manager_id, s.level + 1
    FROM employees e
    INNER JOIN subordinates s ON e.manager_id = s.employee_id
)
SELECT * FROM subordinates
```

**Full Org Chart with Depth:**
```sql
WITH RECURSIVE org_chart AS (
    -- Base: top-level (CEO)
    SELECT employee_id, employee_name, manager_id, 1 as level,
           CAST(employee_name AS VARCHAR(1000)) as path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: add children
    SELECT e.employee_id, e.employee_name, e.manager_id, oc.level + 1,
           CAST(oc.path || ' > ' || e.employee_name AS VARCHAR(1000))
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.employee_id
)
SELECT * FROM org_chart ORDER BY path
```

## Pattern 12: Anti-Join (Records with No Match)

**Customers Who Never Ordered:**
```sql
-- Method 1: LEFT JOIN + IS NULL
SELECT c.*
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL

-- Method 2: NOT EXISTS (often faster)
SELECT *
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
)

-- Method 3: NOT IN (dangerous with NULLs!)
SELECT *
FROM customers
WHERE customer_id NOT IN (
    SELECT customer_id FROM orders WHERE customer_id IS NOT NULL
)
```

## Pattern 13: Pivot (Rows to Columns)

**Manual Pivot with CASE:**
```sql
SELECT 
    product_id,
    SUM(CASE WHEN month = 'Jan' THEN sales ELSE 0 END) as Jan,
    SUM(CASE WHEN month = 'Feb' THEN sales ELSE 0 END) as Feb,
    SUM(CASE WHEN month = 'Mar' THEN sales ELSE 0 END) as Mar
FROM monthly_sales
GROUP BY product_id
```

**Using PIVOT (SQL Server):**
```sql
SELECT product_id, [Jan], [Feb], [Mar]
FROM monthly_sales
PIVOT (
    SUM(sales)
    FOR month IN ([Jan], [Feb], [Mar])
) as pvt
```

## Pattern 14: Unpivot (Columns to Rows)

**Manual Unpivot with UNION ALL:**
```sql
SELECT product_id, 'Jan' as month, Jan as sales FROM sales_table
UNION ALL
SELECT product_id, 'Feb' as month, Feb as sales FROM sales_table
UNION ALL
SELECT product_id, 'Mar' as month, Mar as sales FROM sales_table
```

## Pattern 15: Date Range Overlaps

**Find Overlapping Reservations:**
```sql
-- Two ranges overlap if: start1 <= end2 AND end1 >= start2
SELECT 
    r1.room_id,
    r1.guest_name as guest1,
    r2.guest_name as guest2,
    r1.check_in as check_in1,
    r2.check_in as check_in2
FROM reservations r1
INNER JOIN reservations r2 
    ON r1.room_id = r2.room_id
    AND r1.reservation_id < r2.reservation_id  -- Avoid duplicate pairs
    AND r1.check_in <= r2.check_out
    AND r1.check_out >= r2.check_in
```

## Pattern 16: ALL Related Records Meet Condition

**Students Who Passed ALL Exams:**
```sql
-- Using NOT EXISTS (check if any failed)
SELECT student_id, student_name
FROM students s
WHERE NOT EXISTS (
    SELECT 1 
    FROM exam_results e 
    WHERE e.student_id = s.student_id 
    AND e.score < 60
)

-- Using GROUP BY + HAVING
SELECT student_id
FROM exam_results
GROUP BY student_id
HAVING MIN(score) >= 60  -- All scores must be >= 60
```

**Customers Who Bought ALL Product Categories:**
```sql
SELECT customer_id
FROM purchases
GROUP BY customer_id
HAVING COUNT(DISTINCT category) = (
    SELECT COUNT(DISTINCT category) FROM products
)
```

## Interview Power Moves

When you encounter these patterns, mention:

1. **"This is a classic N-per-group problem—I'll use ROW_NUMBER with PARTITION BY"** (shows pattern recognition)

2. **"For consecutive sequences, the date minus ROW_NUMBER trick creates constant group identifiers"** (shows you know advanced techniques)

3. **"I'd use NOT EXISTS instead of NOT IN here to avoid the NULL trap"** (shows edge case awareness)

4. **"LAST_VALUE needs UNBOUNDED FOLLOWING in the frame, or it only sees up to current row"** (shows window function depth)

5. **"For removing duplicates, I'd check if we can use DISTINCT or if we need ROW_NUMBER with DELETE"** (shows you think about multiple approaches)

6. **"Recursive CTEs can hit max recursion limits—I'd check if we need to set recursion depth"** (shows production awareness)
