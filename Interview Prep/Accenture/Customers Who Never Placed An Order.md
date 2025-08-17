**7. Customers who have never placed an order**
Assume `Customers(customer_id, name)` and `Orders(order_id, customer_id)`.

```sql
SELECT c.customer_id, c.name
FROM Customers c
LEFT JOIN Orders o ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

The `LEFT JOIN` keeps all customers, and filtering `NULL` gives those with no matching orders.
