# Regional Sales Analysis with Cumulative Percentages  

## 1. **Problem Statement**  
Given a `sales` table with:  
- `region`  
- `amount`  

We need to:  
1. Calculate the total sales for each region.  
2. Determine the percentage of each region's sales relative to the overall total.  
3. Compute the cumulative percentage of sales as regions are ordered by total sales in descending order.  

> [!NOTE]  
> This problem tests your ability to use window functions, aggregation, and cumulative calculations in SQL.  

## 2. **Solution 1: Using Window Functions**  
```sql
WITH region_sales AS (
    SELECT
        region,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY region
),
grand_total AS (
    SELECT SUM(amount) AS overall_total FROM sales
)
SELECT
    rs.region,
    rs.total_sales,
    rs.total_sales / gt.overall_total * 100 AS percentage_of_total,
    SUM(rs.total_sales) OVER (ORDER BY rs.total_sales DESC) / gt.overall_total * 100 AS cumulative_percentage
FROM region_sales rs
CROSS JOIN grand_total gt
ORDER BY rs.total_sales DESC;
```  

## 3. **Explanation**  

### 3.1 **CTE: `region_sales`**  
- **Purpose**: Calculates the total sales for each region.  
- **Aggregation**: Uses `SUM(amount)` grouped by `region`.  

> [!TIP]  
> This CTE simplifies the main query by precomputing regional totals.  

### 3.2 **CTE: `grand_total`**  
- **Purpose**: Computes the overall total sales across all regions.  
- **Aggregation**: Uses `SUM(amount)` without grouping.  

> [!IMPORTANT]  
> The grand total is essential for calculating percentages.  

### 3.3 **Main Query**  
- **Percentage Calculation**: Divides each region's total sales by the grand total and multiplies by 100.  
- **Cumulative Percentage**: Uses a window function (`SUM() OVER`) to accumulate sales in descending order and divides by the grand total.  

> [!NOTE]  
> The `CROSS JOIN` ensures the grand total is available for all rows in the result.  

### 3.4 **Window Function**  
- **Purpose**: Calculates the cumulative sum of sales ordered by total sales in descending order.  
- **Syntax**: `SUM(rs.total_sales) OVER (ORDER BY rs.total_sales DESC)`.  

> [!IMPORTANT]  
> The `ORDER BY` in the window function determines the sequence of accumulation.  

## 4. **Alternative Solution: Using `RATIO_TO_REPORT` (Oracle)**  
```sql
SELECT
    region,
    total_sales,
    percentage_of_total * 100 AS percentage,
    SUM(percentage_of_total) OVER (ORDER BY total_sales DESC) * 100 AS cumulative_percentage
FROM (
    SELECT
        region,
        SUM(amount) AS total_sales,
        RATIO_TO_REPORT(SUM(amount)) OVER () AS percentage_of_total
    FROM sales
    GROUP BY region
)
ORDER BY total_sales DESC;
```  

> [!TIP]  
> `RATIO_TO_REPORT()` is an Oracle-specific function that simplifies percentage calculations.  

## 5. **Key Concepts**  

### 5.1 **Window Functions**  
- **Purpose**: Perform calculations across a set of rows related to the current row.  
- **Example**: `SUM() OVER (ORDER BY ...)` for cumulative sums.  

> [!IMPORTANT]  
> Window functions are essential for analytical queries like cumulative percentages.  

### 5.2 **CTEs (Common Table Expressions)**  
- **Benefit**: Break down complex queries into reusable parts.  
- **Example**: `region_sales` and `grand_total` CTEs.  

> [!TIP]  
> Use CTEs to improve query readability and maintainability.  

### 5.3 **Percentage Calculations**  
- **Formula**: `(Part / Total) * 100`.  
- **Application**: Used to express regional sales as a percentage of the grand total.  

> [!NOTE]  
> Ensure the total is not zero to avoid division errors.  

### 5.4 **Cumulative Calculations**  
- **Purpose**: Accumulate values over a sorted dataset.  
- **Example**: Cumulative percentage of sales.  

> [!IMPORTANT]  
> The order of rows in the `ORDER BY` clause determines the accumulation sequence.  


## 6. **Additional Notes**  
- **Performance**: For large datasets, ensure the `sales` table is properly indexed on `region` and `amount`.  
- **Edge Cases**: Handle cases where the grand total is zero to avoid division by zero errors.  

> [!WARNING]  
> Test the query with sample data to ensure accurate results, especially for edge cases.  


This solution demonstrates how to calculate regional sales percentages and cumulative percentages using SQL. It leverages window functions, CTEs, and aggregation to provide insightful analytics.  

> [!TIP]  
> Practice this pattern with different datasets to reinforce your understanding of window functions and cumulative calculations in SQL.  
