**8. Most frequently sold product in each category**
Assume `OrderDetails(order_id, product_id, quantity)` and `Products(product_id, category_id)`.

```sql
SELECT category_id, product_id, total_sold
FROM (
    SELECT p.category_id,
           od.product_id,
           SUM(od.quantity) AS total_sold,
           RANK() OVER(PARTITION BY p.category_id ORDER BY SUM(od.quantity) DESC) AS rnk
    FROM OrderDetails od
    JOIN Products p ON od.product_id = p.product_id
    GROUP BY p.category_id, od.product_id
) t
WHERE rnk = 1;
```

Ranking per category and picking only rank = 1 gives the top-selling product.
