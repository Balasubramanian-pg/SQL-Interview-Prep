# Window Functions Mastery

*   **Window Function Fundamentals**
> [!IMPORTANT]
> Window functions perform calculations across a set of table rows related to the current row.
*   Unlike GROUP BY, window functions don't collapse rows.
*   Each row maintains its identity while gaining access to related rows.
*   Essential for complex analytics without subqueries.

**Basic Window Function Syntax**

```sql
SELECT 
    employee_id,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary,
    SUM(salary) OVER (PARTITION BY department) AS dept_total_salary
FROM employees;
```

**Ranking Functions**

*   **ROW_NUMBER, RANK, DENSE_RANK**
> [!TIP]
> Understand the subtle differences in ranking functions.
```sql
SELECT 
    employee_name,
    department,
    salary,
    -- Unique sequential numbers
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num,
    -- Ranking with gaps for ties
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank,
    -- Ranking without gaps for ties  
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dense_rank,
    -- Percentile ranking
    PERCENT_RANK() OVER (PARTITION BY department ORDER BY salary) AS percentile_rank,
    -- NTILE for buckets
    NTILE(4) OVER (PARTITION BY department ORDER BY salary DESC) AS quartile
FROM employees
WHERE department IN ('Engineering', 'Sales');
```

**Advanced Ranking Scenarios**

```sql
-- Complex ranking with multiple criteria
SELECT 
    salesperson_id,
    region,
    sales_amount,
    sales_quantity,
    ROW_NUMBER() OVER (
        PARTITION BY region 
        ORDER BY sales_amount DESC, sales_quantity DESC
    ) AS sales_rank,
    DENSE_RANK() OVER (
        PARTITION BY region, department
        ORDER BY commission_earned DESC
    ) AS commission_rank
FROM sales_performance
WHERE quarter = 'Q1-2024';
```

**Aggregate Window Functions**

*   **Running Totals & Moving Averages**
> [!NOTE]
> Frame clauses control which rows are included in the window.
```sql
SELECT 
    employee_id,
    sale_date,
    daily_sales,
    -- Running total
    SUM(daily_sales) OVER (
        PARTITION BY employee_id 
        ORDER BY sale_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total,
    -- Moving average (7-day)
    AVG(daily_sales) OVER (
        PARTITION BY employee_id 
        ORDER BY sale_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7d,
    -- Cumulative average
    AVG(daily_sales) OVER (
        PARTITION BY employee_id 
        ORDER BY sale_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_avg
FROM daily_sales
WHERE sale_date >= '2024-01-01';
```

**Advanced Frame Specifications**

```sql
-- Complex window frames
SELECT 
    date,
    product_id,
    daily_revenue,
    -- Previous 3 days average (excluding current)
    AVG(daily_revenue) OVER (
        PARTITION BY product_id
        ORDER BY date
        ROWS BETWEEN 3 PRECEDING AND 1 PRECEDING
    ) AS prev_3day_avg,
    -- Next 2 days sum
    SUM(daily_revenue) OVER (
        PARTITION BY product_id
        ORDER BY date
        ROWS BETWEEN 1 FOLLOWING AND 2 FOLLOWING
    ) AS next_2day_sum,
    -- Centered moving average (2 before, 2 after)
    AVG(daily_revenue) OVER (
        PARTITION BY product_id
        ORDER BY date
        ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING
    ) AS centered_5day_avg
FROM product_revenue;
```

**Analytic Functions**

*   **LEAD, LAG for Time Series Analysis**
> [!IMPORTANT]
> LEAD/LAG provide access to subsequent/previous rows without self-joins.
```sql
SELECT 
    customer_id,
    order_date,
    order_amount,
    -- Previous order amount
    LAG(order_amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS prev_order_amount,
    -- Next order date
    LEAD(order_date) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS next_order_date,
    -- Difference from previous order
    order_amount - LAG(order_amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS amount_change,
    -- Percentage growth from previous
    (order_amount - LAG(order_amount) OVER (
        PARTITION BY customer_id ORDER BY order_date
    )) * 100.0 / LAG(order_amount) OVER (
        PARTITION BY customer_id ORDER BY order_date
    ) AS pct_growth
FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-03-31';
```

**Advanced LEAD/LAG Patterns**

```sql
-- Complex lead/lag with default values and offsets
SELECT 
    month,
    revenue,
    LAG(revenue, 1, 0) OVER (ORDER BY month) AS prev_month,
    LAG(revenue, 12, revenue) OVER (ORDER BY month) AS year_ago,
    LEAD(revenue, 1, revenue) OVER (ORDER BY month) AS next_month,
    -- YoY growth
    (revenue - LAG(revenue, 12) OVER (ORDER BY month)) 
    / LAG(revenue, 12) OVER (ORDER BY month) * 100 AS yoy_growth_pct,
    -- MoM growth with handling for zero/negative
    CASE 
        WHEN LAG(revenue) OVER (ORDER BY month) > 0 
        THEN (revenue - LAG(revenue) OVER (ORDER BY month)) 
             / LAG(revenue) OVER (ORDER BY month) * 100
        ELSE NULL
    END AS mom_growth_pct
FROM monthly_revenue;
```

**Statistical Window Functions**

*   **Percentiles & Distribution Analysis**
```sql
-- Advanced statistical functions
SELECT 
    employee_id,
    department,
    salary,
    -- Percentile calculations
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) 
        OVER (PARTITION BY department) AS median_salary,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY salary) 
        OVER (PARTITION BY department) AS q1_salary,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY salary) 
        OVER (PARTITION BY department) AS q3_salary,
    -- First and last values
    FIRST_VALUE(salary) OVER (
        PARTITION BY department 
        ORDER BY salary 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_salary,
    LAST_VALUE(salary) OVER (
        PARTITION BY department 
        ORDER BY salary 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS highest_salary,
    -- Standard deviation and variance
    STDDEV(salary) OVER (PARTITION BY department) AS salary_stddev,
    VARIANCE(salary) OVER (PARTITION BY department) AS salary_variance
FROM employees
WHERE department IS NOT NULL;
```

**Multiple Window Definitions**

*   **Different Windows in Same Query**
> [!CAUTION]
> Multiple window definitions can significantly impact performance.
```sql
SELECT 
    employee_id,
    department,
    salary,
    hire_date,
    -- Department-level aggregates
    AVG(salary) OVER dept_window AS dept_avg_salary,
    MAX(salary) OVER dept_window AS dept_max_salary,
    -- Company-level aggregates
    AVG(salary) OVER company_window AS company_avg_salary,
    -- Tenure-based ranking
    ROW_NUMBER() OVER tenure_window AS tenure_rank,
    -- Salary growth trajectory
    salary - LAG(salary) OVER career_window AS salary_increase,
    -- Career stage classification
    CASE 
        WHEN ROW_NUMBER() OVER career_window <= 3 THEN 'Early Career'
        WHEN ROW_NUMBER() OVER career_window <= 10 THEN 'Mid Career'
        ELSE 'Late Career'
    END AS career_stage
FROM employees
WINDOW 
    dept_window AS (PARTITION BY department),
    company_window AS (),
    tenure_window AS (PARTITION BY department ORDER BY hire_date DESC),
    career_window AS (PARTITION BY employee_id ORDER BY hire_date)
ORDER BY department, salary DESC;
```

**Advanced Business Use Cases**

*   **Customer Behavior Analysis**
```sql
-- Complex customer analytics with multiple windows
SELECT 
    customer_id,
    order_date,
    order_amount,
    -- Customer lifetime value running total
    SUM(order_amount) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_lifetime_value,
    -- Days between orders
    order_date - LAG(order_date) OVER (
        PARTITION BY customer_id 
        ORDER BY order_date
    ) AS days_since_last_order,
    -- Customer segment based on spending patterns
    CASE 
        WHEN AVG(order_amount) OVER (
            PARTITION BY customer_id
        ) > 1000 THEN 'High Value'
        WHEN COUNT(*) OVER (
            PARTITION BY customer_id
        ) >= 5 THEN 'Frequent'
        ELSE 'Occasional'
    END AS customer_segment,
    -- Order size percentile within customer's history
    PERCENT_RANK() OVER (
        PARTITION BY customer_id 
        ORDER BY order_amount
    ) AS order_size_percentile
FROM orders
WHERE order_date >= '2023-01-01';
```

**Performance Optimization**

*   **Efficient Window Function Queries**
> [!TIP]
> Use appropriate indexes and partition strategies for window functions.
```sql
-- Optimized window function query
WITH ranked_sales AS (
    SELECT 
        salesperson_id,
        region,
        sales_amount,
        ROW_NUMBER() OVER (
            PARTITION BY region 
            ORDER BY sales_amount DESC
        ) AS region_rank,
        SUM(sales_amount) OVER (
            PARTITION BY region
        ) AS region_total,
        sales_amount * 100.0 / SUM(sales_amount) OVER (
            PARTITION BY region
        ) AS pct_of_region
    FROM sales_performance
    WHERE quarter = 'Q1-2024'
        AND sales_amount > 0
)
SELECT *
FROM ranked_sales
WHERE region_rank <= 10  -- Top 10 per region
ORDER BY region, region_rank;
```

**Nested Window Functions**

*   **Window Functions in Derived Tables**
```sql
-- Multi-level window function analysis
SELECT 
    department,
    employee_name,
    salary,
    dept_rank,
    company_rank,
    salary_percentile,
    CASE 
        WHEN dept_rank = 1 THEN 'Department Top Earner'
        WHEN company_rank <= 10 THEN 'Company Top 10'
        WHEN salary_percentile >= 0.9 THEN 'Top 10%'
        ELSE 'Other'
    END AS compensation_tier
FROM (
    SELECT 
        department,
        employee_name,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank,
        ROW_NUMBER() OVER (ORDER BY salary DESC) AS company_rank,
        PERCENT_RANK() OVER (ORDER BY salary) AS salary_percentile
    FROM employees
    WHERE active = true
) ranked_employees
WHERE dept_rank <= 5 OR company_rank <= 50;
```

**Advanced Frame Clause Patterns**

```sql
-- Sophisticated window framing for financial analysis
SELECT 
    trading_date,
    stock_symbol,
    closing_price,
    -- Simple moving averages
    AVG(closing_price) OVER (
        PARTITION BY stock_symbol
        ORDER BY trading_date
        ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
    ) AS sma_5day,
    AVG(closing_price) OVER (
        PARTITION BY stock_symbol
        ORDER BY trading_date
        ROWS BETWEEN 19 PRECEDING AND CURRENT ROW
    ) AS sma_20day,
    -- Exponential moving average approximation
    AVG(closing_price) OVER (
        PARTITION BY stock_symbol
        ORDER BY trading_date
        ROWS BETWEEN 9 PRECEDING AND CURRENT ROW
    ) AS ema_10day,
    -- Volatility (standard deviation)
    STDDEV(closing_price) OVER (
        PARTITION BY stock_symbol
        ORDER BY trading_date
        ROWS BETWEEN 19 PRECEDING AND CURRENT ROW
    ) AS volatility_20day,
    -- High/Low range
    MAX(closing_price) OVER (
        PARTITION BY stock_symbol
        ORDER BY trading_date
        ROWS BETWEEN 19 PRECEDING AND CURRENT ROW
    ) AS high_20day,
    MIN(closing_price) OVER (
        PARTITION BY stock_symbol
        ORDER BY trading_date
        ROWS BETWEEN 19 PRECEDING AND CURRENT ROW
    ) AS low_20day
FROM stock_prices
WHERE trading_date >= '2024-01-01';
```
