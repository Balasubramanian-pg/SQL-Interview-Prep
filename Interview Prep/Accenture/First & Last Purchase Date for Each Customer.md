**3. First and last purchase date for each customer**

```sql
SELECT customer_id,
       MIN(order_date) AS first_purchase,
       MAX(order_date) AS last_purchase
FROM Orders
GROUP BY customer_id;
```

The aggregate functions `MIN` and `MAX` directly give earliest and latest purchase.
