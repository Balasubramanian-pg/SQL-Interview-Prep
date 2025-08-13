# Year-on-Year (YoY) Revenue Growth Calculation in SQL  

## 1. **Overview**  
This document explains how to calculate the total revenue for each year and its year-on-year (YoY) growth percentage from a table of transactions. The solution uses SQL with a Common Table Expression (CTE) and self-join to align current and previous year data.  

> [!NOTE]  
> The solution assumes the input table `Transactions` has columns: `customer_id`, `transaction_date`, and `amount`.  

---

## 2. **Desired Output**  
The output table `Results` includes the following columns:  
- `year`: The transaction year.  
- `current_year_revenue`: Total revenue for the current year.  
- `previous_year_revenue`: Total revenue for the previous year.  
- `yoy_growth_percentage`: Year-on-year growth percentage, rounded to two decimal places.  

---

## 3. **Logical Components**  
### 3.1 **Extract Year from Date**  
Use a date function to extract the year from `transaction_date`.  

### 3.2 **Calculate Yearly Revenue**  
Group transactions by year and sum the `amount` to get total revenue per year.  

### 3.3 **Use Common Table Expression (CTE)**  
Store the yearly revenue summary in a CTE for easier manipulation.  

### 3.4 **Self-Join for Previous Year Data**  
Join the CTE to itself to align each year's data with the previous year's data using the condition `current_year.year = previous_year.year + 1`.  

### 3.5 **Calculate YoY Growth Percentage**  
Use the formula:  
\[
\text{YoY Growth} = \left( \frac{\text{current\_year\_revenue} - \text{previous\_year\_revenue}}{\text{previous\_year\_revenue}} \right) \times 100
\]

---

## 4. **SQL Solution**  
```sql
WITH yearly_revenue AS (
    -- Step 1: Calculate total revenue for each year
    SELECT
        EXTRACT(YEAR FROM transaction_date) AS year,
        SUM(amount) AS total_revenue
    FROM
        Transactions
    GROUP BY
        year -- Group by the extracted year
)
-- Step 2: Self-join to align current and previous year revenue
SELECT
    cur.year,
    cur.total_revenue AS current_year_revenue,
    pre.total_revenue AS previous_year_revenue,
    -- Step 3: Calculate Year-on-Year growth and round it
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

> [!TIP]  
> The `NULLIF` function is used to handle cases where `previous_year_revenue` is `0`, avoiding division by zero errors.  

> [!WARNING]  
> Ensure the `transaction_date` column is of a date or datetime data type for the `EXTRACT` function to work correctly.  

---

## 5. **How It Works**  
### 5.1 **CTE `yearly_revenue`**  
Extracts the year from `transaction_date` and calculates the total revenue for each year using `SUM(amount)`.  

### 5.2 **Self-Join**  
The CTE is joined to itself with the condition `cur.year = pre.year + 1` to align current and previous year data. A `LEFT JOIN` ensures the first year (with no previous year) is included.  

### 5.3 **Growth Calculation**  
The YoY growth percentage is calculated and rounded to two decimal places.  
