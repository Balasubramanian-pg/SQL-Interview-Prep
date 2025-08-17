**2. Customers with more than 5 orders in the last 30 days**
We assume table `Orders(order_id, customer_id, order_date)`.

```sql
SELECT customer_id, COUNT(*) AS order_count
FROM Orders
WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
GROUP BY customer_id
HAVING COUNT(*) > 5;
```

We filter to last 30 days, group by customer, and keep only those with >5 orders.
