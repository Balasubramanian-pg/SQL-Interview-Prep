---
Difficulty: Easy
---
# SQL Interview Questions: Product Revenue and Unordered Products

## Problem 1: Retrieve Total Revenue from Each Product

### Solution

```SQL
SELECT
    p.product_name,
    SUM(o.quantity * p.product_price) AS total_revenue
FROM
    products p
JOIN
    orders o ON p.product_id = o.product_id
GROUP BY
    p.product_name;
```

### Explanation

1. **JOIN** connects products with their orders (using INNER JOIN by default)
2. **SUM(o.quantity * p.product_price)** calculates revenue for each product
3. **GROUP BY product_name** aggregates results by product
4. Returns product names with their total revenue

## Problem 2: Find Products Not Ordered At All

### Solution

```SQL
SELECT
    p.product_name
FROM
    products p
LEFT JOIN
    orders o ON p.product_id = o.product_id
WHERE
    o.order_id IS NULL;
```

### Explanation

1. **LEFT JOIN** includes all products regardless of orders
2. **WHERE o.order_id IS NULL** filters for products with no matching orders
3. Returns only product names that haven't been ordered

## Key Concepts Demonstrated

1. **Aggregation with GROUP BY**:
    - Calculating sums (revenue) grouped by product
    - Essential for financial reporting queries
2. **JOIN Types**:
    - INNER JOIN (default) for matching records only
    - LEFT JOIN to include all records from left table
3. **NULL Checking**:
    - Identifying missing relationships in data
    - Common pattern for finding "orphaned" records
4. **Correct Calculation**:
    - Revenue = quantity × price (not quantity + price)
    - Important to verify mathematical operations

These solutions demonstrate fundamental SQL skills needed for product analytics and inventory management scenarios commonly tested in interviews.