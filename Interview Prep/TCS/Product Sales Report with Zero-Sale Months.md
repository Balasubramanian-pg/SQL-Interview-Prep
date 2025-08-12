# Product Sales Report with Zero-Sale Months  

## 1. **Overview**  
This document explains how to solve an advanced SQL interview question that involves creating a comprehensive sales report. The core challenge is not just to aggregate existing sales data but to create a complete data "scaffold" that includes all products and all months within a given period, even when no sales occurred. This problem tests your ability to use `LEFT JOIN`, `CROSS JOIN`, `GROUP BY`, and Common Table Expressions (CTEs) to build a complete report from sparse data.  

> [!NOTE]  
> This solution is applicable in retail analytics, financial reporting, and any scenario requiring complete time-series data.  

---

## 2. **Problem Statement**  

### 2.1 **The Scenario**  
You are given two tables: `product` and `transactions`.  

**product:**  
| product_id | product_name | unit_price |  
|------------|--------------|------------|  
| 1          | Product A    | 10.00      |  
| 2          | Product B    | 25.00      |  
| 3          | Product C    | 15.00      |  

**transactions:**  
| product_id | so_date    | amount |  
|------------|------------|--------|  
| 1          | 2024-02-10 | 100    |  
| 1          | 2024-03-15 | 150    |  
| 3          | 2024-04-05 | 200    |  
| 3          | 2024-05-20 | 250    |  

### 2.2 **The Requirement**  
Write a SQL query to generate a report showing the total sales amount for **every product** for **every month** of the year 2024.  

- The report must include all products from the `product` table, even if a product had no sales at all (like Product B).  
- For each product, the report must display a row for all 12 months of the year. If a product had no sales in a particular month, the `total_sales` for that month should be `0`.  

**Expected Output (Partial):**  
| product_id | product_name | year | month | total_sales |  
|------------|--------------|------|-------|-------------|  
| 1          | Product A    | 2024 | 1     | 0           |  
| 1          | Product A    | 2024 | 2     | 100         |  
| 1          | Product A    | 2024 | 3     | 150         |  
| 1          | Product A    | 2024 | 4     | 0           |  
| ...        | ...          | ...  | ...   | ...         |  
| 2          | Product B    | 2024 | 1     | 0           |  
| 2          | Product B    | 2024 | 2     | 0           |  
| ...        | ...          | ...  | ...   | ...         |  

> [!IMPORTANT]  
> The solution must handle missing data gracefully, ensuring all product-month combinations are included.  

---

## 3. **Setup Script**  
You can use the following T-SQL script to create the tables and populate them with the sample data.  

```sql
-- Create Product Table
CREATE TABLE product (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(50),
    unit_price DECIMAL(10, 2)
);
GO

-- Create Transactions Table
CREATE TABLE transactions (
    transaction_id INT PRIMARY KEY IDENTITY(1,1),
    product_id INT,
    so_date DATE,
    amount DECIMAL(10, 2)
);
GO

-- Insert Data
INSERT INTO product (product_id, product_name, unit_price) VALUES
(1, 'Product A', 10.00),
(2, 'Product B', 25.00),
(3, 'Product C', 15.00);
GO

INSERT INTO transactions (product_id, so_date, amount) VALUES
(1, '2024-02-10', 100.00),
(1, '2024-03-15', 150.00),
(3, '2024-04-05', 200.00),
(3, '2024-05-20', 250.00);
GO
```  

---

## 4. **Solution and Explanation**  
The solution requires a multi-step approach to first build a complete grid of all products for all months and then join the actual sales data onto this grid.  

### 4.1 **Step 1: Generate a Complete Calendar Dimension**  
First, we need a helper table or CTE that contains all the months and years we want in our report. A `CROSS JOIN` is perfect for creating every combination.  

> [!NOTE]  
> The `VALUES` clause is a handy T-SQL feature to create a temporary, ad-hoc table of literal values.  

```sql
-- This CTE creates a row for every month of the year 2024
WITH AllDates AS (
    SELECT y.year_val, m.month_val
    FROM (VALUES (2024)) AS y(year_val)
    CROSS JOIN (VALUES (1), (2), (3), (4), (5), (6), (7), (8), (9), (10), (11), (12)) AS m(month_val)
)
SELECT * FROM AllDates;
```  

### 4.2 **Step 2: Create the Base Report Grid (All Products for All Months)**  
Next, we `CROSS JOIN` our complete calendar with the `product` table. This creates our "scaffold"—a row for every possible product-month combination.  

```sql
-- This builds on the previous step to create the base grid
...
SELECT p.product_id, p.product_name, d.year_val, d.month_val
FROM product AS p
CROSS JOIN AllDates AS d;
```  

> [!TIP]  
> The `CROSS JOIN` ensures all combinations are generated, even if there are no corresponding sales.  

### 4.3 **Step 3: Calculate Actual Sales per Month**  
Separately, we need to aggregate the actual sales from the `transactions` table.  

```sql
-- This CTE summarizes actual sales that occurred
WITH ActualSales AS (
    SELECT
        product_id,
        YEAR(so_date) AS sale_year,
        MONTH(so_date) AS sale_month,
        SUM(amount) AS total_sales
    FROM transactions
    GROUP BY product_id, YEAR(so_date), MONTH(so_date)
)
SELECT * FROM ActualSales;
```  

### 4.4 **Step 4: Combine the Grid with Actual Sales**  
Finally, we `LEFT JOIN` our complete grid (from Step 2) with the actual sales data (from Step 3).  

- The **Grid** is on the left side of the join to ensure all product-month combinations are kept.  
- The **Actual Sales** data is on the right.  
- We use `ISNULL` (or `COALESCE`) to replace any `NULL` `total_sales` with `0`.  

> [!IMPORTANT]  
> The `LEFT JOIN` ensures that even products or months with no sales are included in the final report.  

---

## 5. **The Complete Solution (Using CTEs)**  
Combining all the steps into a single query with Common Table Expressions (CTEs) provides a clean and readable solution.  

```sql
-- CTE for all months in the desired year(s)
WITH AllDates AS (
    SELECT y.year_val, m.month_val
    FROM (VALUES (2024)) AS y(year_val)
    CROSS JOIN (VALUES (1), (2), (3), (4), (5), (6), (7), (8), (9), (10), (11), (12)) AS m(month_val)
),
-- CTE for the complete grid of all products for all months
ProductCalendarGrid AS (
    SELECT p.product_id, p.product_name, d.year_val, d.month_val
    FROM product AS p
    CROSS JOIN AllDates AS d
),
-- CTE for actual aggregated sales data
ActualSales AS (
    SELECT
        product_id,
        YEAR(so_date) AS sale_year,
        MONTH(so_date) AS sale_month,
        SUM(amount) AS total_sales
    FROM transactions
    GROUP BY product_id, YEAR(so_date), MONTH(so_date)
)
-- Final SELECT statement to join the grid with the sales data
SELECT
    grid.product_id,
    grid.product_name,
    grid.year_val AS year,
    grid.month_val AS month,
    ISNULL(sales.total_sales, 0) AS total_sales
FROM
    ProductCalendarGrid AS grid
LEFT JOIN
    ActualSales AS sales
ON
    grid.product_id = sales.product_id
    AND grid.year_val = sales.sale_year
    AND grid.month_val = sales.sale_month
ORDER BY
    grid.product_id,
    grid.month_val;
```  

---

This solution demonstrates advanced SQL techniques, including `CROSS JOIN`, `LEFT JOIN`, CTEs, and conditional aggregation, making it a valuable addition to your SQL toolkit.  

> [!TIP]  
> Practice this pattern with different datasets and time intervals to strengthen your ability to handle sparse data and complete reporting.  
