SQL offers several powerful methods for running calculations across rows, often required for financial, statistical, or cumulative reporting. The primary techniques involve using **Window Functions** for cumulative and moving calculations, or simple **Aggregations** with **`GROUP BY`** for static totals.

## 1\. Window Functions (Calculations Across Related Rows) 

Window functions perform calculations across a set of table rows that are somehow related to the current row. They **do not collapse rows** like standard aggregation (e.g., `SUM` in a `GROUP BY`).

### A. Cumulative Sum (Running Total)

This calculates a running sum of a column, adding the current row's value to the sum of all preceding rows.

**Use Case:** Tracking total revenue as it accumulates over time.

```sql
SELECT
    sale_date,
    daily_revenue,
    SUM(daily_revenue) OVER (
        ORDER BY sale_date -- Defines the order of accumulation
    ) AS running_total_revenue
FROM
    DailySales;
```

### B. Moving/Rolling Average

This calculates an aggregate over a defined "window" of preceding and/or following rows (e.g., the last 7 days).

**Use Case:** Smoothing out daily volatility to see a trend.

```sql
SELECT
    sale_date,
    daily_revenue,
    AVG(daily_revenue) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW -- Defines the 7-day window (current day + 6 preceding days)
    ) AS rolling_7_day_average
FROM
    DailySales;
```

### C. Difference from Previous Row (Delta)

Using **`LAG()`** to compare the current value to the value in the previous row.

**Use Case:** Calculating the day-over-day change in a metric.

```sql
SELECT
    metric_date,
    daily_metric_value,
    daily_metric_value - LAG(daily_metric_value) OVER (
        ORDER BY metric_date
    ) AS day_over_day_change
FROM
    Metrics;
```

## 2\. Aggregation Functions (Calculations Per Group) 

Standard aggregate functions (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`) collapse the rows within a group to produce a single value for that group, requiring a `GROUP BY` clause.

**Use Case:** Calculating totals, averages, or counts for fixed categories (e.g., region, department, month).

```sql
SELECT
    department_id,
    -- Total cost for the department
    SUM(salary) AS total_payroll, 
    -- Average salary in the department
    AVG(salary) AS average_salary,
    -- Count of employees in the department
    COUNT(*) AS total_employees
FROM
    Employees
GROUP BY
    department_id
HAVING
    COUNT(*) > 10; -- Filter groups (departments) that meet a criterion
```

## 3\. Subqueries and Joins (Calculations Relative to a Total)

Sometimes, a calculation must reference a global total or average *before* filtering or grouping. This requires nesting queries or using CTEs.

**Use Case:** Calculating the percentage contribution of each department to the global total payroll.

```sql
WITH GlobalTotal AS (
    -- Calculate the total payroll for the entire company
    SELECT SUM(salary) AS global_payroll FROM Employees
)
SELECT
    e.department_id,
    SUM(e.salary) AS dept_payroll,
    -- Calculate percentage: Dept Total / Global Total
    SUM(e.salary) * 100.0 / gt.global_payroll AS percentage_of_total
FROM
    Employees AS e
CROSS JOIN -- Use CROSS JOIN to attach the single global total row to every department row
    GlobalTotal AS gt
GROUP BY
    e.department_id, gt.global_payroll
ORDER BY
    percentage_of_total DESC;
```
