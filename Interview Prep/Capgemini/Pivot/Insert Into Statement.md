Here's the complete `INSERT INTO` query to populate the `sales_data` table for testing pivot operations, along with comprehensive test cases:

```sql
-- Create sales_data table if it doesn't exist
CREATE TABLE IF NOT EXISTS sales_data (
    month VARCHAR(7),  -- Format: 'YYYY-MM'
    category VARCHAR(20),
    amount DECIMAL(10,2)
);

-- Clear existing data (if any)
TRUNCATE TABLE sales_data;

-- Insert sample data with various scenarios:
-- 1. Regular months with both categories
-- 2. Months missing one category
-- 3. Multiple entries per category per month
-- 4. Edge cases (zero amounts, NULL values)
-- 5. Additional categories for testing scalability
INSERT INTO sales_data (month, category, amount) VALUES
-- Q1 - Complete data
('2023-01', 'electronics', 15000.00),
('2023-01', 'clothing', 8000.00),
('2023-02', 'electronics', 18000.00),
('2023-02', 'clothing', 9500.00),
('2023-03', 'electronics', 22000.00),
('2023-03', 'clothing', 7500.00),

-- Q2 - Missing categories
('2023-04', 'electronics', 19000.00),  -- No clothing
('2023-05', 'clothing', 11000.00),     -- No electronics
('2023-06', 'electronics', 25000.00),
('2023-06', 'clothing', 12000.00),

-- Q3 - Multiple entries per category
('2023-07', 'electronics', 10000.00),
('2023-07', 'electronics', 8000.00),   -- Should sum to 18000
('2023-07', 'clothing', 6000.00),
('2023-07', 'clothing', 4000.00),      -- Should sum to 10000
('2023-08', 'electronics', 21000.00),
('2023-08', 'clothing', 9000.00),
('2023-09', 'electronics', 23000.00),
('2023-09', 'clothing', 8500.00),

-- Q4 - Edge cases
('2023-10', 'electronics', 0.00),      -- Zero amount
('2023-10', 'clothing', NULL),         -- NULL value
('2023-11', 'electronics', 17000.00),
('2023-11', 'clothing', 7000.00),
('2023-12', 'electronics', 28000.00),
('2023-12', 'clothing', 15000.00),

-- Additional categories for testing
('2023-01', 'furniture', 5000.00),
('2023-02', 'furniture', 6000.00),
('2023-03', 'furniture', 4500.00);
```

To test the pivot solutions:

1. **PostgreSQL CROSSTAB** (requires tablefunc extension):
```sql
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT * FROM CROSSTAB(
    'SELECT
         month,
         category,
         SUM(amount) as amount
      FROM sales_data
      GROUP BY month, category
      ORDER BY 1, 2'
) AS ct (
    month VARCHAR,
    clothing NUMERIC,
    electronics NUMERIC,
    furniture NUMERIC
);
```

2. **SQL Server PIVOT**:
```sql
SELECT
    month,
    ISNULL([clothing], 0) AS clothing,
    ISNULL([electronics], 0) AS electronics,
    ISNULL([furniture], 0) AS furniture
FROM
    (SELECT month, category, amount FROM sales_data) AS SourceTable
PIVOT
(
    SUM(amount)
    FOR category IN ([clothing], [electronics], [furniture])
) AS PivotTable
ORDER BY month;
```

3. **Standard SQL (CASE approach)**:
```sql
SELECT
    month,
    SUM(CASE WHEN category = 'clothing' THEN amount ELSE 0 END) AS clothing,
    SUM(CASE WHEN category = 'electronics' THEN amount ELSE 0 END) AS electronics,
    SUM(CASE WHEN category = 'furniture' THEN amount ELSE 0 END) AS furniture
FROM
    sales_data
GROUP BY
    month
ORDER BY
    month;
```

Key features of this test data:
- Complete year of monthly data
- Realistic sales patterns (seasonality, growth)
- Missing data scenarios
- Multiple transactions per category per month
- Edge cases (zero amounts, NULL values)
- Additional category to demonstrate scalability

The solutions should:
- Correctly aggregate amounts when multiple entries exist
- Handle months with missing categories (show NULL/0 appropriately)
- Maintain proper sorting by month
- Scale to additional categories if needed
- Handle edge cases gracefully

For PostgreSQL, remember to enable the tablefunc extension first if you haven't already. The CASE approach is the most portable across database systems.
