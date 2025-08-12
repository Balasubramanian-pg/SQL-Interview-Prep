# Find First and Last Exchange Rates of the Month

## 1. Overview
This document explains how to solve a common SQL interview question that requires finding the first and last recorded values within a specific time period (a month) for different groups (currency codes). This problem is an excellent test of window functions (`ROW_NUMBER`) and conditional aggregation (`CASE` within an aggregate function) to pivot data.

## 2. Problem Statement

### 2.1. The Scenario
You are given a table named `exchange_rates` that contains daily currency exchange rates against a base currency.

**exchange_rates:**
| currency_code | rate_date  | exchange_rate |
|---------------|------------|---------------|
| EUR           | 2023-06-01 | 1.20          |
| EUR           | 2023-06-15 | 1.22          |
| EUR           | 2023-06-30 | 1.23          |
| EUR           | 2023-07-01 | 1.25          |
| EUR           | 2023-07-31 | 1.27          |
| IND           | 2023-06-01 | 1.40          |
| IND           | 2023-06-30 | 1.45          |

### 2.2. The Requirement
Write a SQL query that returns the exchange rate at the beginning of the month and at the end of the month for each currency code and for each month present in the data.

**Expected Output:**
| currency_year_month | beginning_of_month_rate | end_of_month_rate |
|---------------------|---------------------------|-------------------|
| EUR_2023_6          | 1.20                      | 1.23              |
| EUR_2023_7          | 1.25                      | 1.27              |
| IND_2023_6          | 1.40                      | 1.45              |

## 3. Setup Script
You can use the following T-SQL script to create the table and populate it with sample data.

```sql
-- Create the table
CREATE TABLE exchange_rates (
    currency_code VARCHAR(10),
    rate_date DATE,
    exchange_rate FLOAT
);
GO

-- Insert sample data
INSERT INTO exchange_rates (currency_code, rate_date, exchange_rate) VALUES
('EUR', '2023-06-01', 1.20),
('EUR', '2023-06-15', 1.22),
('EUR', '2023-06-30', 1.23),
('EUR', '2023-07-01', 1.25),
('EUR', '2023-07-31', 1.27),
('IND', '2023-06-01', 1.40),
('IND', '2023-06-15', 1.42),
('IND', '2023-06-30', 1.45);
GO

-- Verify the initial data
SELECT * FROM exchange_rates;
```

## 4. Solution and Explanation
The solution involves two main steps: first, ranking the records within each month, and second, using this ranking to pivot the first and last rates into columns.

### 4.1. Step 1: Rank Records Within Each Month using `ROW_NUMBER`
To identify the first and last day of the month for each currency, we can use the `ROW_NUMBER()` window function. We need two separate rankings:
1.  **Ascending Order (`rn_asc`)**: Orders dates from earliest to latest. The row with `rn_asc = 1` is the first record of the month.
2.  **Descending Order (`rn_desc`)**: Orders dates from latest to earliest. The row with `rn_desc = 1` is the last record of the month.

> [!IMPORTANT]
> The `PARTITION BY` clause is crucial here. It resets the `ROW_NUMBER()` count for each new `currency_code` and `month`, ensuring we find the first and last day for each distinct group.

-   **Query for Ranking:**
    ```sql
    SELECT
        currency_code,
        rate_date,
        exchange_rate,
        ROW_NUMBER() OVER(PARTITION BY currency_code, YEAR(rate_date), MONTH(rate_date) ORDER BY rate_date ASC) as rn_asc,
        ROW_NUMBER() OVER(PARTITION BY currency_code, YEAR(rate_date), MONTH(rate_date) ORDER BY rate_date DESC) as rn_desc
    FROM exchange_rates;
    ```

### 4.2. Step 2: Aggregate and Pivot Using `GROUP BY` and `CASE`
Now that we have identified the first and last rows, we can group the data by currency and month. We then use **conditional aggregation** to pivot the rates into their respective columns.

-   **Logic:**
    -   `GROUP BY currency_code, YEAR(rate_date), MONTH(rate_date)` collapses all rows for a specific currency-month into a single row.
    -   `MAX(CASE WHEN rn_asc = 1 THEN exchange_rate END)`: For each group, this expression checks which row has `rn_asc = 1`. For that row, it returns the `exchange_rate`; for all other rows, it returns `NULL`. The `MAX()` function then picks the single non-`NULL` value from the group.
    -   The same logic applies to `rn_desc = 1` to get the end-of-month rate.

## 5. The Complete Solution (Using CTEs)
Using a Common Table Expression (CTE) makes the multi-step logic clean and easy to follow.

```sql
WITH RankedRates AS (
    -- Step 1: Rank each record within its currency-month group in both ascending and descending order
    SELECT
        currency_code,
        YEAR(rate_date) AS rate_year,
        MONTH(rate_date) AS rate_month,
        exchange_rate,
        ROW_NUMBER() OVER(PARTITION BY currency_code, YEAR(rate_date), MONTH(rate_date) ORDER BY rate_date ASC) as rn_asc,
        ROW_NUMBER() OVER(PARTITION BY currency_code, YEAR(rate_date), MONTH(rate_date) ORDER BY rate_date DESC) as rn_desc
    FROM
        exchange_rates
)
-- Step 2: Group by month and use conditional aggregation to pivot the first and last rates
SELECT
    CONCAT(currency_code, '_', rate_year, '_', rate_month) AS currency_year_month,
    MAX(CASE WHEN rn_asc = 1 THEN exchange_rate END) AS beginning_of_month_rate,
    MAX(CASE WHEN rn_desc = 1 THEN exchange_rate END) AS end_of_month_rate
FROM
    RankedRates
GROUP BY
    currency_code, rate_year, rate_month
ORDER BY
    currency_year_month;
```
