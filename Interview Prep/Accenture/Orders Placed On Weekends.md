**4. Orders placed on weekends**

```sql
SELECT order_id, customer_id, order_date
FROM Orders
WHERE EXTRACT(DOW FROM order_date) IN (0, 6);  
-- 0 = Sunday, 6 = Saturday in most SQL dialects
```

This checks the day of week. Some SQLs use `DATEPART(WEEKDAY, order_date)`.
