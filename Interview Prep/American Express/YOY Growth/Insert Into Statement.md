# INSERT INTO Query for Year-on-Year Revenue Growth Calculation

To test the Year-on-Year (YoY) Revenue Growth calculation, we need to create a sample `Transactions` table with realistic data across multiple years. Here's how to set this up:

## Option 1: Basic Sample Data

```sql
-- Create the Transactions table if it doesn't exist
CREATE TABLE IF NOT EXISTS Transactions (
    customer_id INT,
    transaction_date DATE,
    amount DECIMAL(10, 2)
);

-- Insert sample transaction data for 3 consecutive years
INSERT INTO Transactions (customer_id, transaction_date, amount)
VALUES
    -- Year 2022
    (1, '2022-01-15', 100.00),
    (2, '2022-03-22', 150.00),
    (3, '2022-06-10', 200.00),
    (1, '2022-09-05', 120.00),
    (2, '2022-12-18', 180.00),
    -- Total for 2022: 750.00
    
    -- Year 2023 (10% growth)
    (1, '2023-02-10', 110.00),
    (2, '2023-04-15', 165.00),
    (3, '2023-07-20', 220.00),
    (4, '2023-08-12', 130.00), -- New customer
    (1, '2023-10-05', 132.00),
    (2, '2023-11-25', 198.00),
    -- Total for 2023: 825.00 (10% growth)
    
    -- Year 2024 (20% growth)
    (1, '2024-01-20', 132.00),
    (2, '2024-03-15', 198.00),
    (3, '2024-05-10', 264.00),
    (4, '2024-06-22', 156.00),
    (5, '2024-08-05', 225.00), -- New customer
    (1, '2024-09-18', 158.40),
    (2, '2024-11-30', 237.60),
    -- Total for 2024: 990.00 (20% growth)
    
    -- Edge case: Zero revenue year
    -- Year 2025 (business closure)
    -- Total for 2025: 0.00
    
    -- Year 2026 (recovery year)
    (3, '2026-02-14', 300.00),
    (5, '2026-05-20', 450.00),
    -- Total for 2026: 750.00
;
```

## Option 2: Randomized Bulk Data

```sql
-- Create the Transactions table if it doesn't exist
CREATE TABLE IF NOT EXISTS Transactions (
    customer_id INT,
    transaction_date DATE,
    amount DECIMAL(10, 2)
);

-- Insert randomized data for more realistic testing
-- This generates 50-100 transactions per year from 2020-2024
WITH years AS (
    SELECT 2020 AS year UNION ALL SELECT 2021 UNION ALL 
    SELECT 2022 UNION ALL SELECT 2023 UNION ALL SELECT 2024
),
customers AS (
    SELECT 1 AS customer_id UNION ALL SELECT 2 UNION ALL 
    SELECT 3 UNION ALL SELECT 4 UNION ALL SELECT 5
)
INSERT INTO Transactions (customer_id, transaction_date, amount)
SELECT 
    c.customer_id,
    DATEADD(day, 
            CAST(RAND(CHECKSUM(NEWID())) * 365 AS INT), 
            DATEFROMPARTS(y.year, 1, 1)) AS transaction_date,
    ROUND(50 + (RAND(CHECKSUM(NEWID())) * 450, 2) AS amount -- Random amounts between 50-500
FROM 
    years y
CROSS JOIN 
    customers c
CROSS APPLY (
    SELECT TOP (10 + CAST(RAND(CHECKSUM(NEWID())) * 40 AS INT) n 
    FROM master.dbo.spt_values 
    WHERE type = 'P' AND number < 100
) AS transactions_per_customer;
```

## Verifying the Solution

After inserting the data, you can run the YoY calculation query:

```sql
WITH yearly_revenue AS (
    SELECT
        EXTRACT(YEAR FROM transaction_date) AS year,
        SUM(amount) AS total_revenue
    FROM
        Transactions
    GROUP BY
        year
)
SELECT
    cur.year,
    cur.total_revenue AS current_year_revenue,
    pre.total_revenue AS previous_year_revenue,
    ROUND(
        (cur.total_revenue - pre.total_revenue) * 100.0 / NULLIF(pre.total_revenue, 0),
        2
    ) AS yoy_growth_percentage
FROM
    yearly_revenue cur
LEFT JOIN
    yearly_revenue pre ON cur.year = pre.year + 1
ORDER BY
    cur.year;
```

This will show you the yearly revenue with growth percentages, including edge cases like:
- The first year (with no previous year)
- A year with zero revenue (handled by NULLIF)
- Recovery after a bad year

You can modify the sample data to test different scenarios like:
- Negative growth years
- Seasonal businesses
- Rapid growth startups
- Businesses with intermittent zero-revenue periods
