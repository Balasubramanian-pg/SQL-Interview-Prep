This cheat sheet focuses on two common window function applications: **Running Totals** and **Moving Averages**. These calculations are essential for time-series analysis and trend identification.

## Running Totals & Moving Averages Cheat Sheet Table

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Running/Cumulative Total** | Calculating a progressive sum of values up to the current row. | Use the `SUM()` function with an unbounded preceding window frame. | `$SUM(column) OVER (PARTITION BY group_col ORDER BY sort_col ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)$` |
| **Simple Moving Average (SMA)** | Calculating the average of a fixed number of preceding rows and the current row. | Use the `AVG()` function with a bounded preceding window frame. | `$AVG(column) OVER (PARTITION BY group_col ORDER BY sort_col ROWS BETWEEN N PRECEDING AND CURRENT ROW)$` |
| **Cumulative Average** | Calculating the average of **all** preceding rows and the current row. | Use the `AVG()` function with an unbounded preceding window frame. | `$AVG(column) OVER (PARTITION BY group_col ORDER BY sort_col ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)$` |
| **Rolling Sum/Total** | Calculating the sum of a fixed number of rows centered around the current row. | Use the `SUM()` function with a bounded window frame (preceding and following). | `$SUM(column) OVER (PARTITION BY group_col ORDER BY sort_col ROWS BETWEEN N PRECEDING AND M FOLLOWING)$` |

## Running Totals & Moving Averages Mini Playbook (SQL Snippets)

These snippets illustrate the common use cases for cumulative and rolling calculations.

### 1\. Running/Cumulative Total

Calculate the **cumulative sales total** over time, resetting for each store.

```sql
SELECT
    store_id,
    sale_date,
    daily_sales_amount,
    SUM(daily_sales_amount) OVER (
        PARTITION BY store_id
        ORDER BY sale_date
        -- Frame specification for Running Total
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_store_sales
FROM
    daily_sales;
```

### 2\. Simple Moving Average (SMA)

Calculate the **3-day Simple Moving Average** of a stock's closing price.

```sql
SELECT
    ticker,
    trade_date,
    closing_price,
    AVG(closing_price) OVER (
        PARTITION BY ticker
        ORDER BY trade_date
        -- Frame specification: current row and 2 preceding rows (3 days total)
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS three_day_moving_average
FROM
    stock_prices;
```

### 3\. Cumulative Average

Calculate the **average test score** for a student up to the current test.

```sql
SELECT
    student_id,
    test_number,
    score,
    AVG(score) OVER (
        PARTITION BY student_id
        ORDER BY test_number
        -- Frame specification for Cumulative Average
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_average_score
FROM
    student_scores;
```

### 4\. Rolling Sum/Total

Calculate the **total revenue** across a 5-day window: the current day, 2 days prior, and 2 days following (a **centered** 5-day rolling sum).

```sql
SELECT
    day_date,
    daily_revenue,
    SUM(daily_revenue) OVER (
        ORDER BY day_date
        -- Frame specification: 2 preceding rows, Current Row, 2 following rows
        ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING
    ) AS centered_5_day_rolling_sum
FROM
    revenue_data;
```
