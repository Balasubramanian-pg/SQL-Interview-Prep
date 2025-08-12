# Product Revenue and Unordered Products  

## 1. **Problem 1: Retrieve Total Revenue from Each Product**  

### 1.1 **Solution**  
```sql
SELECT
    p.product_name,
    SUM(o.quantity * p.product_price) AS total_revenue
FROM
    products p
JOIN
    orders o ON p.product_id = o.product_id
GROUP BY
    p.product_name;
```  

### 1.2 **Explanation**  
1. **JOIN**: Connects products with their orders (using INNER JOIN by default).  
2. **SUM(o.quantity * p.product_price)**: Calculates revenue for each product.  
3. **GROUP BY product_name**: Aggregates results by product.  
4. Returns product names with their total revenue.  

> [!TIP]  
> Ensure the `product_price` is correctly multiplied by `quantity` to calculate revenue accurately.  

---

## 2. **Problem 2: Find Products Not Ordered At All**  

### 2.1 **Solution**  
```sql
SELECT
    p.product_name
FROM
    products p
LEFT JOIN
    orders o ON p.product_id = o.product_id
WHERE
    o.order_id IS NULL;
```  

### 2.2 **Explanation**  
1. **LEFT JOIN**: Includes all products regardless of orders.  
2. **WHERE o.order_id IS NULL**: Filters for products with no matching orders.  
3. Returns only product names that haven't been ordered.  

> [!NOTE]  
> This query identifies "orphaned" products that exist in the inventory but have never been sold.  

---

## 3. **Key Concepts Demonstrated**  

### 3.1 **Aggregation with GROUP BY**  
- Calculating sums (revenue) grouped by product.  
- Essential for financial reporting queries.  

### 3.2 **JOIN Types**  
- **INNER JOIN** (default): For matching records only.  
- **LEFT JOIN**: To include all records from the left table.  

### 3.3 **NULL Checking**  
- Identifying missing relationships in data.  
- Common pattern for finding "orphaned" records.  

### 3.4 **Correct Calculation**  
- Revenue = `quantity × price` (not `quantity + price`).  
- Important to verify mathematical operations.  

> [!WARNING]  
> Incorrect calculations (e.g., using `+` instead of `*`) can lead to inaccurate revenue figures.  

---

These solutions demonstrate fundamental SQL skills needed for product analytics and inventory management scenarios commonly tested in interviews.  

> [!IMPORTANT]  
> Practice these queries with sample datasets to reinforce your understanding of JOINs, aggregation, and NULL handling.
