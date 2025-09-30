I'll create INSERT statements for your sales and quantity tables with the specified structure and data requirements.

## Table Creation and INSERT Statements

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

-- Insert into Sales Table (3 rows)
INSERT INTO sales (sk_id, cust_id, sales_amount) VALUES
(1, 101, 1500.00),
(2, 102, 2750.50),
(3, 103, 3200.75);

-- Insert into Quantity Table (2 rows)
INSERT INTO qty (sk_id, cust_id, quantity) VALUES
(1, 101, 25),
(2, 102, 40);
```

## Verification Queries

```sql
-- Check Sales Table data
SELECT * FROM sales;

-- Check Quantity Table data
SELECT * FROM qty;
```

## Sample Output:

**Sales Table:**
```
sk_id | cust_id | sales_amount
------|---------|-------------
1     | 101     | 1500.00
2     | 102     | 2750.50
3     | 103     | 3200.75
```

**Quantity Table:**
```
sk_id | cust_id | quantity
------|---------|---------
1     | 101     | 25
2     | 102     | 40
```

## Join Query to see combined data:

```sql
-- Join both tables to see customer data
SELECT 
    s.sk_id,
    s.cust_id,
    s.sales_amount,
    q.quantity
FROM sales s
LEFT JOIN qty q ON s.cust_id = q.cust_id;
```

This will show you which customers exist in both tables and their respective sales and quantity data.

Let me know if you need any modifications or additional queries!
