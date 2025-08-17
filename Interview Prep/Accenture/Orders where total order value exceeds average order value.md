**10. Orders where total order value exceeds average order value**
Assume `OrderDetails(order_id, product_id, quantity, unit_price)`.

```sql
WITH OrderTotals AS (
    SELECT order_id, SUM(quantity * unit_price) AS total_value
    FROM OrderDetails
    GROUP BY order_id
)
SELECT *
FROM OrderTotals
WHERE total_value > (SELECT AVG(total_value) FROM OrderTotals);
```

We compute each order’s total, then filter those above the overall average.
