**Question:**
How do I create a column called *performance* in my sales table that shows “High” if `sales_amount > 20` and “Low” otherwise?

**Answer:**
Use a `CASE` expression in your SQL query:

```sql
SELECT 
    sk_id,
    cust_id,
    sales_amount,
    CASE 
        WHEN sales_amount > 20 THEN 'High'
        ELSE 'Low'
    END AS performance
FROM sales_table;
```

This will return a result set with the new *performance* column.
