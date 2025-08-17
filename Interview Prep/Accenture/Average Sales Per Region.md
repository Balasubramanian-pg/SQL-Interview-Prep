**5. Average sales per region**
Assume `Sales(sale_id, region, amount)`.

```sql
SELECT region, AVG(amount) AS avg_sales
FROM Sales
GROUP BY region;
```

Grouping by region and applying `AVG` gives the average sales value per region.
