---
Difficulty: Hard
---
  

Problem Statement  
How can we calculate the total revenue for each year and its year-on-year (YoY) growth percentage from a table of transactions?

**Input Table:**

- Transactions (customer_id, transaction_date, amount)

**Desired Output Table:**

- Results (year, current_year_revenue, previous_year_revenue, yoy_growth_percentage)

**Logical Components:**

- Extract the year from `transaction_date` using a date function.
- Use `GROUP BY` and `SUM()` to calculate total revenue per year.
- Store the yearly revenue in a Common Table Expression (CTE).
- Perform a self-join on the CTE to align each year's data with the previous year's data.
- The join condition will be `current_year.year = previous_year.year + 1`.
- Calculate the growth percentage using the formula: `(current - previous) / previous`.

  

1. First, calculate the total revenue for each year by summing the transaction amounts and grouping them by the transaction year.
2. Store this yearly revenue summary in a temporary result set using a Common Table Expression (CTE).
3. Join this CTE to itself, matching each year with the year that came before it (`year` = `year - 1`).
4. Using the current and previous year's revenue from the joined data, calculate the year-on-year growth percentage.
5. Select and format the final columns for the desired output.

  
To find the year-on-year growth, you first need to simplify the data into a yearly summary. This involves extracting the year from each transaction date and summing up all transaction amounts for that year.

Once you have a list of total revenue for each year, the next challenge is to get the current year's revenue and the previous year's revenue on the same row so you can perform a calculation. This is achieved by joining the yearly summary table to itself. By setting the join condition to match `current_year` with `previous_year + 1`, you effectively place the two adjacent years side-by-side.

Finally, with both values available on one row, you can apply the standard percentage growth formula and format the result.

### SQL Solution

```SQL
WITH yearly_revenue AS (
    -- Step 1: Calculate total revenue for each year
    SELECT
        EXTRACT(YEAR FROM transaction_date) AS year,
        SUM(amount) AS total_revenue
    FROM
        Transactions
    GROUP BY
        1 -- Group by the first column (year)
)
-- Step 2: Self-join to align current and previous year revenue
SELECT
    cur.year,
    cur.total_revenue AS current_year_revenue,
    pre.total_revenue AS previous_year_revenue,
    -- Step 3: Calculate Year-on-Year growth and round it
    ROUND(
        (cur.total_revenue - pre.total_revenue) * 100.0 / pre.total_revenue,
        2
    ) AS yoy_growth_percentage
FROM
    yearly_revenue cur
LEFT JOIN
    yearly_revenue pre ON cur.year = pre.year + 1
ORDER BY
    cur.year;
```

### How it Works:

1. **CTE** `**yearly_revenue**`: This part first extracts the `year` from the `transaction_date`. It then calculates the `SUM(amount)` for each year by using `GROUP BY year`. The result is a simple temporary table with two columns: `year` and `total_revenue`.
2. **Self** `**LEFT JOIN**`: The main query joins the `yearly_revenue` CTE to itself.
    - `cur` (current) is the alias for the left side of the join, representing the current year.
    - `pre` (previous) is the alias for the right side, representing the previous year.
    - The `ON cur.year = pre.year + 1` condition is the key: it matches each year in `cur` with the preceding year in `pre`.
    - A `LEFT JOIN` is used to ensure the earliest year in the dataset (which has no preceding year) is still included in the result. For that first year, `pre.total_revenue` will be `NULL`.
3. **Calculation**: The `SELECT` statement calculates the year-on-year growth. Multiplying by `100.0` ensures floating-point division to get an accurate percentage. The `ROUND()` function formats the result to two decimal places, as shown in the video.