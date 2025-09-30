Here's a **CROSS JOIN** query for your sales and quantity tables:

## CROSS JOIN Query

```sql
SELECT 
    s.sk_id AS sales_sk_id,
    s.cust_id AS sales_cust_id,
    s.sales_amount,
    q.sk_id AS qty_sk_id,
    q.cust_id AS qty_cust_id,
    q.quantity
FROM sales s
CROSS JOIN qty q;
```

## Expected Output:

With your data (3 sales rows × 2 quantity rows = 6 total rows):

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

## Alternative CROSS JOIN Syntax:

```sql
-- Using comma separator (older syntax)
SELECT 
    s.sk_id AS sales_sk_id,
    s.cust_id AS sales_cust_id,
    s.sales_amount,
    q.sk_id AS qty_sk_id,
    q.cust_id AS qty_cust_id,
    q.quantity
FROM sales s, qty q;
```

## Practical Use Case Example:

```sql
-- Calculate potential revenue if all quantities were sold at each sales price
SELECT 
    s.cust_id AS sales_customer,
    q.cust_id AS qty_customer,
    s.sales_amount,
    q.quantity,
    (s.sales_amount * q.quantity) AS potential_revenue
FROM sales s
CROSS JOIN qty q
ORDER BY potential_revenue DESC;
```

**CROSS JOIN** creates a Cartesian product - every row from the first table is combined with every row from the second table. This is useful for scenarios where you need to consider all possible combinations between two datasets.
