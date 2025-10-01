# Set Operations & Unconventional Thinking

*   **Set Operation Fundamentals**
> [!IMPORTANT]
> Set operations combine results from multiple queries vertically (not horizontally like JOINs).
*   **UNION** - Distinct rows from both queries
*   **UNION ALL** - All rows from both queries (faster)
*   **INTERSECT** - Rows common to both queries
*   **EXCEPT** - Rows in first query but not in second

**Creative Set Operation Patterns**

*   **Data Quality Checks**
> [!TIP]
> Use EXCEPT to find data mismatches between tables.
```sql
-- Find customers in source but not in target (data sync issues)
SELECT customer_id, email FROM source_customers
EXCEPT
SELECT customer_id, email FROM target_customers;

-- Find overlapping records (potential duplicates)
SELECT customer_id, email FROM current_customers
INTERSECT
SELECT customer_id, email FROM archived_customers;
```

*   **Progressive Data Validation**
```sql
-- Multi-stage data validation using set operations
WITH valid_records AS (
    SELECT id, name, email FROM raw_data WHERE email LIKE '%@%'
),
unique_records AS (
    SELECT id, name, email FROM valid_records
    EXCEPT
    SELECT id, name, email FROM blacklist
),
final_records AS (
    SELECT id, name, email FROM unique_records
    INTERSECT
    SELECT id, name, email FROM approved_domains
)
SELECT COUNT(*) AS clean_records FROM final_records;
```

**Unconventional Problem Solving**

*   **Gap and Island Problems**
> [!NOTE]
> Use window functions creatively to find contiguous ranges.
```sql
-- Find date ranges without sales (gaps)
WITH sales_dates AS (
    SELECT DISTINCT sale_date FROM sales
),
date_series AS (
    SELECT generate_series(
        '2024-01-01'::date, 
        '2024-12-31'::date, 
        '1 day'::interval
    ) AS date_day
),
gaps AS (
    SELECT 
        date_day,
        CASE WHEN s.sale_date IS NULL THEN 1 ELSE 0 END AS is_gap,
        SUM(CASE WHEN s.sale_date IS NULL THEN 1 ELSE 0 END) 
            OVER (ORDER BY date_day) AS gap_group
    FROM date_series ds
    LEFT JOIN sales_dates s ON ds.date_day = s.sale_date
)
SELECT 
    MIN(date_day) AS gap_start,
    MAX(date_day) AS gap_end
FROM gaps
WHERE is_gap = 1
GROUP BY gap_group
HAVING COUNT(*) > 1;  -- Only gaps of 2+ days
```

*   **Pivot Without PIVOT Function**
```sql
-- Dynamic pivoting using set operations and aggregation
SELECT 
    department,
    SUM(CASE WHEN quarter = 'Q1' THEN sales END) AS q1_sales,
    SUM(CASE WHEN quarter = 'Q2' THEN sales END) AS q2_sales,
    SUM(CASE WHEN quarter = 'Q3' THEN sales END) AS q3_sales,
    SUM(CASE WHEN quarter = 'Q4' THEN sales END) AS q4_sales
FROM (
    -- Simulate pivoted data using UNION ALL
    SELECT department, 'Q1' AS quarter, q1_sales AS sales FROM sales_data
    UNION ALL
    SELECT department, 'Q2' AS quarter, q2_sales AS sales FROM sales_data  
    UNION ALL
    SELECT department, 'Q3' AS quarter, q3_sales AS sales FROM sales_data
    UNION ALL
    SELECT department, 'Q4' AS quarter, q4_sales AS sales FROM sales_data
) pivoted
GROUP BY department;
```

**Advanced Set Theory Applications**

*   **Relational Division Problem**
> [!CAUTION]
> Find records that match ALL items in a set - classic interview question.
```sql
-- Find customers who purchased ALL products in a specific category
SELECT customer_id
FROM (
    SELECT DISTINCT customer_id, product_id 
    FROM orders 
    WHERE product_id IN (SELECT product_id FROM products WHERE category = 'Electronics')
) customer_products
GROUP BY customer_id
HAVING COUNT(DISTINCT product_id) = (
    SELECT COUNT(DISTINCT product_id) 
    FROM products 
    WHERE category = 'Electronics'
);

-- Alternative using EXCEPT (often more efficient)
SELECT DISTINCT customer_id
FROM orders o1
WHERE NOT EXISTS (
    SELECT product_id FROM products WHERE category = 'Electronics'
    EXCEPT
    SELECT product_id FROM orders o2 
    WHERE o2.customer_id = o1.customer_id
);
```

*   **Finding Symmetric Relationships**
```sql
-- Find mutual relationships (A follows B AND B follows A)
SELECT 
    LEAST(f1.follower_id, f1.following_id) AS user1,
    GREATEST(f1.follower_id, f1.following_id) AS user2
FROM follows f1
INTERSECT
SELECT 
    LEAST(f2.following_id, f2.follower_id) AS user1, 
    GREATEST(f2.following_id, f2.follower_id) AS user2
FROM follows f2
WHERE f2.follower_id = f1.following_id 
  AND f2.following_id = f1.follower_id;
```

**Creative Data Generation**

*   **Generating Test Data with Set Operations**
```sql
-- Create comprehensive test matrix using CROSS JOIN-like patterns
WITH numbers AS (
    SELECT 1 AS n UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5
),
letters AS (
    SELECT 'A' AS l UNION SELECT 'B' UNION SELECT 'C' UNION SELECT 'D'
),
conditions AS (
    SELECT 'Active' AS status UNION SELECT 'Inactive' UNION SELECT 'Pending'
)
SELECT n.n, l.l, c.status
FROM numbers n
CROSS JOIN letters l
CROSS JOIN conditions c
ORDER BY n, l, c.status;
```

*   **Calendar Generation with Gaps**
```sql
-- Generate business calendar excluding weekends and holidays
WITH date_series AS (
    SELECT generate_series(
        '2024-01-01'::date,
        '2024-12-31'::date,
        '1 day'::interval
    ) AS calendar_date
),
holidays AS (
    SELECT holiday_date FROM company_holidays 
    WHERE year = 2024
),
business_days AS (
    SELECT calendar_date
    FROM date_series
    WHERE EXTRACT(DOW FROM calendar_date) NOT IN (0, 6)  -- No weekends
    EXCEPT
    SELECT holiday_date FROM holidays  -- No holidays
)
SELECT 
    calendar_date,
    ROW_NUMBER() OVER (ORDER BY calendar_date) AS business_day_number
FROM business_days;
```

**Unconventional JOIN Alternatives**

*   **Using UNION Instead of Complex CASE Logic**
```sql
-- Instead of complex conditional aggregation:
SELECT 
    department,
    COUNT(CASE WHEN status = 'Active' THEN 1 END) AS active_count,
    COUNT(CASE WHEN status = 'Inactive' THEN 1 END) AS inactive_count
FROM employees
GROUP BY department;

-- Use UNION ALL for clearer logic:
SELECT department, 'Active' AS status_type, COUNT(*) AS count
FROM employees WHERE status = 'Active'
GROUP BY department
UNION ALL
SELECT department, 'Inactive' AS status_type, COUNT(*) AS count  
FROM employees WHERE status = 'Inactive'
GROUP BY department;
```

*   **Set Operations for Data Reconciliation**
```sql
-- Reconcile data between two systems
WITH source_data AS (
    SELECT id, name, amount FROM system_a_transactions
    WHERE transaction_date = '2024-01-15'
),
target_data AS (
    SELECT id, name, amount FROM system_b_transactions  
    WHERE transaction_date = '2024-01-15'
),
missing_in_target AS (
    SELECT 'Missing in Target' AS issue_type, id, name, amount
    FROM source_data
    EXCEPT
    SELECT 'Missing in Target' AS issue_type, id, name, amount  
    FROM target_data
),
missing_in_source AS (
    SELECT 'Missing in Source' AS issue_type, id, name, amount
    FROM target_data
    EXCEPT
    SELECT 'Missing in Source' AS issue_type, id, name, amount
    FROM source_data  
),
amount_mismatch AS (
    SELECT 'Amount Mismatch' AS issue_type, s.id, s.name, 
           s.amount AS source_amount, t.amount AS target_amount
    FROM source_data s
    JOIN target_data t ON s.id = t.id
    WHERE s.amount != t.amount
)
SELECT * FROM missing_in_target
UNION ALL
SELECT * FROM missing_in_source  
UNION ALL
SELECT * FROM amount_mismatch;
```

**Advanced Pattern Matching**

*   **Finding Sequences and Patterns**
```sql
-- Find customers with specific purchase sequences
WITH customer_sequences AS (
    SELECT 
        customer_id,
        ARRAY_AGG(product_category ORDER BY order_date) AS purchase_sequence
    FROM orders
    GROUP BY customer_id
),
pattern_matches AS (
    SELECT 
        customer_id,
        purchase_sequence,
        -- Check if sequence contains Electronics -> Furniture -> Electronics
        EXISTS(
            SELECT 1 FROM unnest(purchase_sequence) WITH ORDINALITY AS seq(item, pos)
            WHERE item = 'Electronics'
            AND EXISTS(
                SELECT 1 FROM unnest(purchase_sequence) WITH ORDINALITY AS seq2(item2, pos2)
                WHERE item2 = 'Furniture' AND pos2 > pos
                AND EXISTS(
                    SELECT 1 FROM unnest(purchase_sequence) WITH ORDINALITY AS seq3(item3, pos3)
                    WHERE item3 = 'Electronics' AND pos3 > pos2
                )
            )
        ) AS has_pattern
    FROM customer_sequences
)
SELECT customer_id, purchase_sequence
FROM pattern_matches
WHERE has_pattern = true;
```

**Mathematical Set Applications**

*   **Venn Diagram Operations in SQL**
```sql
-- Implement full Venn diagram logic
WITH set_a AS (SELECT id FROM table_a),
     set_b AS (SELECT id FROM table_b),
     set_c AS (SELECT id FROM table_c)
     
-- A only
SELECT 'A only' AS region, id FROM set_a
EXCEPT (SELECT id FROM set_b UNION SELECT id FROM set_c)

UNION ALL

-- B only  
SELECT 'B only' AS region, id FROM set_b
EXCEPT (SELECT id FROM set_a UNION SELECT id FROM set_c)

UNION ALL

-- A ∩ B only (intersection of A and B but not C)
SELECT 'A ∩ B only' AS region, id FROM (
    SELECT id FROM set_a INTERSECT SELECT id FROM set_b
) ab
EXCEPT SELECT id FROM set_c;
```

**Performance Tricks with Set Operations**

*   **Early Filtering in Complex Queries**
```sql
-- Instead of filtering at the end, filter in each branch:
SELECT * FROM (
    SELECT id, name, 'TypeA' AS source FROM large_table_a WHERE active = true
    UNION ALL
    SELECT id, name, 'TypeB' AS source FROM large_table_b WHERE status = 'approved'
    UNION ALL  
    SELECT id, name, 'TypeC' AS source FROM large_table_c WHERE valid = 1
) combined
-- Much faster than UNION first, then WHERE
```

**Creative Problem-Solving Mindset**

*   **Think in Sets, Not Loops**
> [!IMPORTANT]
> SQL is declarative - describe WHAT you want, not HOW to get it.
```sql
-- Instead of thinking "for each customer, check if..."
-- Think "find the set of customers where..."

-- Instead of procedural logic:
-- "Loop through orders, sum amounts by month"
-- Use set-based:
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    SUM(amount) AS monthly_total
FROM orders
GROUP BY DATE_TRUNC('month', order_date);
```

*   **Leverage Boolean Logic**
```sql
-- Convert complex business rules to set operations
WITH eligible_customers AS (
    SELECT customer_id FROM customers WHERE status = 'Active'
    INTERSECT
    SELECT customer_id FROM orders WHERE order_date >= CURRENT_DATE - 30
    EXCEPT 
    SELECT customer_id FROM returns WHERE return_date >= CURRENT_DATE - 90
)
SELECT * FROM eligible_customers;
```

**Unconventional Interview Solutions**

*   **Solve Problems Backwards**
```sql
-- Sometimes easier to find what you DON'T want
SELECT * FROM universe
EXCEPT
SELECT * FROM invalid_cases;

-- Find employees without certain characteristics
SELECT * FROM employees
EXCEPT
SELECT e.* FROM employees e
JOIN performance_issues p ON e.id = p.employee_id
WHERE p.severity = 'High';
```
