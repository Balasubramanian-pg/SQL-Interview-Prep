# Calculate Percentage Sales Variance

## 1. Overview
This document explains how to solve a common SQL interview question that involves calculating a time-series metric—specifically, the day-over-day percentage variance in sales. The solution requires filtering the results to show only periods of positive growth (profit). This problem tests your understanding of window functions like `LAG()`, Common Table Expressions (CTEs) for structuring complex queries, and conditional filtering.

## 2. Problem Statement

### 2.1. The Scenario
You are given a `sales_data` table that contains daily sales records.

**sales_data:**
| sale_date  | sales_amount |
|------------|--------------|
| 2023-01-03 | 10           |
| 2023-01-04 | 20           |
| 2023-01-05 | 60           |
| 2023-01-06 | 30           |
| 2023-01-07 | 45           |

### 2.2. The Requirement
Write a SQL query to calculate the percentage variance in sales compared to the previous day. The formula for percentage variance is:
`(Current Day Sales - Previous Day Sales) / Previous Day Sales * 100`

The final output should only include records where there was a positive sales variance (i.e., a profit compared to the previous day).

**Expected Output:**
| sale_date  | sales_amount | percentage_variance |
|------------|--------------|-----------------------|
| 2023-01-04 | 20           | 100.00                |
| 2023-01-05 | 60           | 200.00                |
| 2023-01-07 | 45           | 50.00                 |

## 3. Setup Script
You can use the following T-SQL script to create the table and populate it with the sample data.

```sql
-- Create the table
CREATE TABLE sales_data (
    sale_date DATE,
    sales_amount INT
);
GO

-- Insert sample data
INSERT INTO sales_data (sale_date, sales_amount) VALUES
('2023-01-03', 10),
('2023-01-04', 20),
('2023-01-05', 60),
('2023-01-06', 30),
('2023-01-07', 45);
GO

-- Verify the initial data
SELECT * FROM sales_data;
```

## 4. Solution and Explanation
The problem is best solved in sequential steps using Common Table Expressions (CTEs).

### 4.1. Step 1: Get the Previous Day's Sales Using `LAG()`
First, we need to retrieve the previous day's sales amount for each row. The `LAG()` window function is perfect for this.

-   **Logic**: `LAG(sales_amount, 1, 0) OVER (ORDER BY sale_date)` looks back one row (`1`) based on the `sale_date` ordering and retrieves the `sales_amount`. We provide a default value of `0` to handle the first row, which has no preceding row.
-   **Query Snippet**:
    ```sql
    WITH PreviousSales AS (
        SELECT
            sale_date,
            sales_amount,
            LAG(sales_amount, 1, 0) OVER (ORDER BY sale_date) AS previous_day_sales
        FROM sales_data
    )
    SELECT * FROM PreviousSales;
    ```

### 4.2. Step 2: Calculate the Percentage Variance
With the current and previous sales amounts on the same row, we can now apply the variance formula.

> [!CAUTION]
> To avoid integer division, which can result in `0`, it's crucial to cast one of the numbers in the formula to a decimal or float data type before performing the calculation.

-   **Logic**: We build on the previous CTE and add a new column for the variance calculation. The formula is applied using the `sales_amount` and `previous_day_sales` columns.
-   **Query Snippet**:
    ```sql
    WITH PreviousSales AS (...),
    CalculatedVariance AS (
        SELECT
            sale_date,
            sales_amount,
            (sales_amount - previous_day_sales) * 100.0 / previous_day_sales AS percentage_variance
        FROM PreviousSales
        WHERE previous_day_sales > 0 -- Avoid division by zero
    )
    SELECT * FROM CalculatedVariance;
    ```

### 4.3. Step 3: Filter for Positive Variance
The final requirement is to only show records representing a profit. This is a simple filtering step on our calculated `percentage_variance`.

-   **Logic**: We add a `WHERE` clause to the final `SELECT` statement to filter for rows where `percentage_variance` is greater than or equal to zero.

## 5. The Complete Solution (Using CTEs)
This final query combines all the steps into a single, readable, and maintainable block.

```sql
WITH PreviousSales AS (
    -- Step 1: Get the previous day's sales for each record
    SELECT
        sale_date,
        sales_amount,
        LAG(sales_amount, 1, 0) OVER (ORDER BY sale_date) AS previous_day_sales
    FROM
        sales_data
),
CalculatedVariance AS (
    -- Step 2: Calculate the percentage variance, avoiding division by zero
    SELECT
        sale_date,
        sales_amount,
        CASE
            WHEN previous_day_sales > 0 THEN
                (CAST(sales_amount AS FLOAT) - previous_day_sales) * 100.0 / previous_day_sales
            ELSE 0
        END AS percentage_variance
    FROM
        PreviousSales
)
-- Step 3: Select the final columns and filter for profit margins only
SELECT
    sale_date,
    sales_amount,
    percentage_variance
FROM
    CalculatedVariance
WHERE
    percentage_variance > 0;
```
