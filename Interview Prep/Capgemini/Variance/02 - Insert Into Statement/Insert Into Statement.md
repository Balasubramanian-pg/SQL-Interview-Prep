Here's the `INSERT INTO` query to populate the `sales_data` table for the "Calculate Percentage Sales Variance" problem, including additional test cases:

```sql
-- Create the table if it doesn't exist
CREATE TABLE IF NOT EXISTS sales_data (
    sale_date DATE,
    sales_amount DECIMAL(10,2)
);

-- Clear existing data (if any)
TRUNCATE TABLE sales_data;

-- Insert sample data with various scenarios:
-- 1. Basic increasing/decreasing pattern
-- 2. Zero sales day
-- 3. Negative sales (returns/refunds)
-- 4. Consecutive same sales amounts
-- 5. Large variance cases
-- 6. First day with no previous data
INSERT INTO sales_data (sale_date, sales_amount) VALUES
-- Basic pattern
('2023-01-03', 10.00),  -- First day (no previous)
('2023-01-04', 20.00),  -- +100%
('2023-01-05', 60.00),  -- +200%
('2023-01-06', 30.00),  -- -50% (should be filtered out)
('2023-01-07', 45.00),  -- +50%
('2023-01-08', 45.00),  -- 0% (should be filtered out)

-- Edge cases
('2023-01-09', 0.00),   -- Zero sales
('2023-01-10', 15.00),  -- From 0 to 15 (infinite% but handled by NULLIF)
('2023-01-11', -5.00),  -- Negative sales
('2023-01-12', 10.00),  -- From -5 to 10 (+300%)

-- Same sales multiple days
('2023-01-13', 25.00),
('2023-01-14', 25.00),  -- 0% change
('2023-01-15', 25.00),  -- 0% change
('2023-01-16', 50.00),  -- +100%

-- Large variance
('2023-01-17', 100.00),
('2023-01-18', 10.00),  -- -90%
('2023-01-19', 1000.00); -- +9900%
```

To verify the solution works correctly, you can run:

```sql
WITH PreviousSales AS (
    SELECT
        sale_date,
        sales_amount,
        LAG(sales_amount, 1, NULL) OVER (ORDER BY sale_date) AS previous_day_sales
    FROM sales_data
),
CalculatedVariance AS (
    SELECT
        sale_date,
        sales_amount,
        CASE
            WHEN previous_day_sales IS NULL THEN NULL
            WHEN previous_day_sales = 0 THEN NULL
            ELSE ROUND((sales_amount - previous_day_sales) * 100.0 / previous_day_sales, 2)
        END AS percentage_variance
    FROM PreviousSales
)
SELECT
    sale_date,
    sales_amount,
    percentage_variance
FROM
    CalculatedVariance
WHERE
    percentage_variance > 0
ORDER BY
    sale_date;
```

Expected results:
1. Only rows with positive percentage variance should appear
2. The first day (with no previous data) should be excluded
3. Days with zero or negative variance should be filtered out
4. The calculation should handle edge cases like:
   - Division by zero (when previous day was 0)
   - Negative sales amounts
   - Infinite growth (from 0 to positive)
   - No change between days

This test data covers:
- Standard growth patterns
- Edge cases (zero sales, negative sales)
- Boundary conditions
- Various growth percentages
- Consecutive same values

The solution should correctly calculate the percentage change while avoiding division by zero and only showing positive growth periods.
