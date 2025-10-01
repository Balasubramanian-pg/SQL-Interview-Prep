# Table Creation & Data Insertion for Sales Analysis

*   **Database Schema Setup**
> [!IMPORTANT]
> Create the sales and quantity tables with proper primary keys and data types.

## Table Creation Statements

```sql
-- Create Sales Table
CREATE TABLE sales (
    sk_id INT PRIMARY KEY,
    cust_id INT,
    sales_amount DECIMAL(10,2)
);

-- Create Quantity Table  
CREATE TABLE qty (
    sk_id INT PRIMARY KEY,
    cust_id INT,
    quantity INT
);
```

## Data Insertion

*   **Sales Data Insert**
> [!TIP]
> Insert 3 rows with varying sales amounts and customer IDs.
```sql
-- Insert into Sales Table (3 rows)
INSERT INTO sales (sk_id, cust_id, sales_amount) VALUES
(1, 101, 1500.00),
(2, 102, 2750.50), 
(3, 103, 3200.75);
```

*   **Quantity Data Insert**
> [!NOTE]
> Insert 2 rows to demonstrate CROSS JOIN behavior with mismatched customer counts.
```sql
-- Insert into Quantity Table (2 rows)
INSERT INTO qty (sk_id, cust_id, quantity) VALUES
(1, 101, 25),
(2, 102, 40);
```

## Data Verification Queries

*   **Verify Sales Table**
```sql
-- Check Sales Table data
SELECT * FROM sales;
```

**Expected Sales Output:**
```
sk_id | cust_id | sales_amount
------|---------|-------------
1     | 101     | 1500.00
2     | 102     | 2750.50
3     | 103     | 3200.75
```

*   **Verify Quantity Table**
```sql
-- Check Quantity Table data  
SELECT * FROM qty;
```

**Expected Quantity Output:**
```
sk_id | cust_id | quantity
------|---------|---------
1     | 101     | 25
2     | 102     | 40
```

## Data Analysis Queries

*   **INNER JOIN Analysis**
> [!NOTE]
> Shows only customers present in both tables (cust_id 101 and 102).
```sql
-- Customers with both sales and quantity data
SELECT 
    s.sk_id AS sales_id,
    s.cust_id,
    s.sales_amount,
    q.quantity
FROM sales s
INNER JOIN qty q ON s.cust_id = q.cust_id;
```

*   **LEFT JOIN Analysis**
> [!TIP]
> Shows all sales customers, with NULL for missing quantity data (cust_id 103).
```sql
-- All sales customers with their quantity data (if available)
SELECT 
    s.sk_id,
    s.cust_id,
    s.sales_amount,
    q.quantity
FROM sales s
LEFT JOIN qty q ON s.cust_id = q.cust_id;
```

*   **CROSS JOIN Demonstration**
> [!CAUTION]
> Creates Cartesian product - all possible combinations regardless of customer matching.
```sql
-- CROSS JOIN: All sales × all quantity combinations
SELECT 
    s.sk_id AS sales_sk_id,
    s.cust_id AS sales_cust_id,
    s.sales_amount,
    q.sk_id AS qty_sk_id,
    q.cust_id AS qty_cust_id, 
    q.quantity
FROM sales s
CROSS JOIN qty q
ORDER BY s.sk_id, q.sk_id;
```

## Expected CROSS JOIN Output

**6 rows total (3 sales × 2 quantity = 6 combinations)**

```
sales_sk_id | sales_cust_id | sales_amount | qty_sk_id | qty_cust_id | quantity
------------|---------------|--------------|-----------|-------------|---------
1           | 101           | 1500.00      | 1         | 101         | 25
1           | 101           | 1500.00      | 2         | 102         | 40
2           | 102           | 2750.50      | 1         | 101         | 25
2           | 102           | 2750.50      | 2         | 102         | 40  
3           | 103           | 3200.75      | 1         | 101         | 25
3           | 103           | 3200.75      | 2         | 102         | 40
```

## Data Quality Check

*   **Missing Data Analysis**
```sql
-- Find sales customers without quantity data
SELECT 
    s.cust_id,
    s.sales_amount
FROM sales s
LEFT JOIN qty q ON s.cust_id = q.cust_id
WHERE q.cust_id IS NULL;
```

*   **Data Summary Statistics**
```sql
-- Quick data overview
SELECT 
    'Sales' AS table_name,
    COUNT(*) AS row_count,
    COUNT(DISTINCT cust_id) AS unique_customers
FROM sales

UNION ALL

SELECT 
    'Quantity' AS table_name, 
    COUNT(*) AS row_count,
    COUNT(DISTINCT cust_id) AS unique_customers
FROM qty;
```

## Table Modification Examples

*   **Adding Indexes for Performance**
```sql
-- Add indexes for better join performance
CREATE INDEX idx_sales_cust_id ON sales(cust_id);
CREATE INDEX idx_qty_cust_id ON qty(cust_id);
```

*   **Adding Additional Columns**
```sql
-- Example: Add date columns if needed
ALTER TABLE sales ADD COLUMN sale_date DATE;
ALTER TABLE qty ADD COLUMN quantity_date DATE;
```

The data is now ready for CROSS JOIN analysis and other SQL operations!
