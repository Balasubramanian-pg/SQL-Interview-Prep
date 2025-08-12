---
Difficulty: Easy
---
# SQL Interview Question: Finding Missing Order IDs

## Problem Statement

Given an orders table with:

- `order_id` (sequential but with gaps)
- `order_date`

We need to:

1. Identify all missing order IDs in the sequence
2. Return just the missing IDs (4, 8, 12, 17 in this case)

## Solution Code

```SQL
WITH sequence AS (
    SELECT generate_series(
        (SELECT MIN(order_id) FROM orders),
        (SELECT MAX(order_id) FROM orders)
    ) AS order_id
)
SELECT
    s.order_id
FROM
    sequence s
LEFT JOIN
    orders o ON s.order_id = o.order_id
WHERE
    o.order_id IS NULL;
```

## Explanation

1. **Generate Series**:
    - Creates a complete sequence from min to max order ID
    - Uses PostgreSQL's `generate_series()` function
2. **Left Join**:
    - Joins the complete sequence with actual orders
    - Missing orders will have NULL values in the right table
3. **Filtering**:
    - `WHERE o.order_id IS NULL` identifies gaps
    - Returns only IDs that exist in sequence but not in orders

## Key Concepts

1. **Sequence Generation**:
    - Essential for gap analysis in sequential data
    - Different databases have different functions (e.g., Oracle uses CONNECT BY)
2. **Left Join with NULL Check**:
    - Common pattern for finding missing records
    - Works because unmatched left rows get NULL right values
3. **Performance Considerations**:
    - Efficient for moderate ranges (1-20 in this case)
    - For large ranges, consider limiting the series generation

## Alternative Solution (Using Window Functions)

```SQL
WITH numbered_orders AS (
    SELECT
        order_id,
        LEAD(order_id) OVER (ORDER BY order_id) AS next_id
    FROM orders
)
SELECT
    order_id + 1 AS missing_start,
    next_id - 1 AS missing_end
FROM
    numbered_orders
WHERE
    next_id - order_id > 1;
```

This alternative identifies ranges of missing IDs rather than individual values, which can be more efficient for large datasets with consecutive gaps.
