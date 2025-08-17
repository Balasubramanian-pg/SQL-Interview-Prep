**9. Profit margin per product, ranked within category**
Assume `Products(product_id, category_id, cost_price, selling_price)`.

```sql
SELECT category_id, 
       product_id,
       (selling_price - cost_price) / selling_price * 100 AS profit_margin,
       RANK() OVER(PARTITION BY category_id ORDER BY (selling_price - cost_price) / selling_price DESC) AS rnk
FROM Products;
```

We calculate margin and rank products within their category.
