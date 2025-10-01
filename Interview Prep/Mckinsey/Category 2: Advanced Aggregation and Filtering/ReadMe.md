# Advanced Aggregation & Filtering

*   **Multiple Aggregation Levels**
> [!IMPORTANT]
> Combine different aggregation levels in single queries using advanced techniques.
```sql
-- Department averages vs company averages
SELECT 
    department,
    COUNT(*) AS employee_count,
    AVG(salary) AS dept_avg_salary,
    (SELECT AVG(salary) FROM employees) AS company_avg_salary,
    AVG(salary) - (SELECT AVG(salary) FROM employees) AS diff_from_company,
    CASE 
        WHEN AVG(salary) > (SELECT AVG(salary) FROM employees) 
        THEN 'Above Average'
        ELSE 'Below Average'
    END AS comparison_status
FROM employees
GROUP BY department
HAVING COUNT(*) >= 5;
```

**Advanced FILTER Clause**

*   **Conditional Aggregation with FILTER**
> [!TIP]
> Use FILTER for cleaner conditional aggregation instead of CASE in aggregates.
```sql
-- Multiple conditional aggregates with FILTER
SELECT 
    department,
    -- Standard aggregates
    COUNT(*) AS total_employees,
    AVG(salary) AS overall_avg_salary,
    
    -- Conditional aggregates with FILTER
    COUNT(*) FILTER (WHERE tenure_years > 5) AS experienced_count,
    AVG(salary) FILTER (WHERE tenure_years > 5) AS experienced_avg_salary,
    AVG(salary) FILTER (WHERE job_level = 'Senior') AS senior_avg_salary,
    AVG(salary) FILTER (WHERE job_level = 'Junior') AS junior_avg_salary,
    
    -- Multiple conditions
    COUNT(*) FILTER (
        WHERE tenure_years > 3 AND performance_rating >= 4
    ) AS high_performers
FROM employees
GROUP BY department
HAVING COUNT(*) FILTER (WHERE tenure_years > 5) >= 3;
```

**Window Functions with Aggregation**

*   **Rolling Calculations & Percentages**
```sql
-- Advanced window functions with aggregation
SELECT 
    department,
    job_level,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary,
    -- Window functions over aggregated results
    SUM(COUNT(*)) OVER () AS total_company_employees,
    COUNT(*) * 100.0 / SUM(COUNT(*)) OVER () AS pct_of_company,
    AVG(AVG(salary)) OVER () AS company_wide_avg,
    AVG(salary) - AVG(AVG(salary)) OVER () AS diff_from_company_avg,
    -- Ranking within departments
    RANK() OVER (ORDER BY AVG(salary) DESC) AS salary_rank
FROM employees
WHERE active = true
GROUP BY department, job_level
HAVING COUNT(*) >= 2;
```

**Complex HAVING Conditions**

*   **Multiple Aggregate Conditions**
> [!NOTE]
> HAVING can reference multiple aggregates and include complex logic.
```sql
-- Complex HAVING with multiple aggregate conditions
SELECT 
    store_id,
    product_category,
    COUNT(DISTINCT customer_id) AS unique_customers,
    SUM(sale_amount) AS total_sales,
    AVG(sale_amount) AS avg_sale_size,
    COUNT(*) AS transaction_count
FROM sales
WHERE sale_date BETWEEN '2024-01-01' AND '2024-03-31'
GROUP BY store_id, product_category
HAVING 
    -- Multiple aggregate conditions
    COUNT(DISTINCT customer_id) >= 50
    AND SUM(sale_amount) > 10000
    AND AVG(sale_amount) BETWEEN 50 AND 500
    AND COUNT(*) >= 100
    -- Ratio conditions
    AND SUM(sale_amount) / COUNT(DISTINCT customer_id) > 200  -- Avg spend per customer
    AND COUNT(*) * 1.0 / COUNT(DISTINCT customer_id) > 1.5   -- Transactions per customer
ORDER BY total_sales DESC;
```

**Multi-Level Aggregation**

*   **Nested Aggregation with Subqueries**
```sql
-- Department-level aggregates with employee-level details
SELECT 
    d.department_name,
    dept_stats.employee_count,
    dept_stats.avg_salary,
    dept_stats.max_salary,
    -- Correlated subquery for top earner details
    (SELECT employee_name 
     FROM employees e2 
     WHERE e2.department_id = d.department_id 
     AND e2.salary = dept_stats.max_salary
     LIMIT 1) AS top_earner_name,
    -- Subquery for salary distribution
    (SELECT COUNT(*) 
     FROM employees e3 
     WHERE e3.department_id = d.department_id 
     AND e3.salary > dept_stats.avg_salary * 1.2) AS above_120_pct_count
FROM departments d
INNER JOIN (
    -- Department-level aggregation subquery
    SELECT 
        department_id,
        COUNT(*) AS employee_count,
        AVG(salary) AS avg_salary,
        MAX(salary) AS max_salary,
        MIN(salary) AS min_salary
    FROM employees
    GROUP BY department_id
    HAVING COUNT(*) >= 3
) dept_stats ON d.department_id = dept_stats.department_id
WHERE d.active = true;
```

**Statistical Aggregation**

*   **Advanced Statistical Calculations**
```sql
-- Statistical aggregates and distributions
SELECT 
    department,
    COUNT(*) AS n,
    AVG(salary) AS mean_salary,
    MIN(salary) AS min_salary,
    MAX(salary) AS max_salary,
    -- Statistical functions
    STDDEV(salary) AS salary_stddev,
    VARIANCE(salary) AS salary_variance,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY salary) AS q1_salary,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY salary) AS q3_salary,
    -- Custom calculations
    (MAX(salary) - MIN(salary)) AS salary_range,
    STDDEV(salary) / AVG(salary) AS coefficient_of_variation,
    -- Count by salary bands
    COUNT(*) FILTER (WHERE salary < AVG(salary) - STDDEV(salary)) AS below_1_stddev,
    COUNT(*) FILTER (WHERE salary BETWEEN AVG(salary) - STDDEV(salary) 
                                     AND AVG(salary) + STDDEV(salary)) AS within_1_stddev,
    COUNT(*) FILTER (WHERE salary > AVG(salary) + STDDEV(salary)) AS above_1_stddev
FROM employees
GROUP BY department
HAVING COUNT(*) >= 10  -- Significant sample size
   AND STDDEV(salary) > 0  -- Prevent division by zero
ORDER BY AVG(salary) DESC;
```

**Time-Based Aggregation**

*   **Rolling Time Windows**
```sql
-- Complex time-based aggregation with multiple windows
SELECT 
    customer_id,
    -- Last 30 days
    COUNT(*) FILTER (WHERE sale_date >= CURRENT_DATE - 30) AS transactions_30d,
    SUM(sale_amount) FILTER (WHERE sale_date >= CURRENT_DATE - 30) AS sales_30d,
    AVG(sale_amount) FILTER (WHERE sale_date >= CURRENT_DATE - 30) AS avg_sale_30d,
    
    -- Last 90 days  
    COUNT(*) FILTER (WHERE sale_date >= CURRENT_DATE - 90) AS transactions_90d,
    SUM(sale_amount) FILTER (WHERE sale_date >= CURRENT_DATE - 90) AS sales_90d,
    
    -- Previous period comparisons
    COUNT(*) FILTER (WHERE sale_date BETWEEN CURRENT_DATE - 60 AND CURRENT_DATE - 31) 
        AS transactions_prev_30d,
        
    -- Growth calculations
    (SUM(sale_amount) FILTER (WHERE sale_date >= CURRENT_DATE - 30) -
     SUM(sale_amount) FILTER (WHERE sale_date BETWEEN CURRENT_DATE - 60 AND CURRENT_DATE - 31))
    / NULLIF(SUM(sale_amount) FILTER (WHERE sale_date BETWEEN CURRENT_DATE - 60 AND CURRENT_DATE - 31), 0)
    * 100 AS sales_growth_pct,
    
    -- Customer segmentation logic
    CASE 
        WHEN SUM(sale_amount) FILTER (WHERE sale_date >= CURRENT_DATE - 90) > 5000 
            THEN 'VIP'
        WHEN SUM(sale_amount) FILTER (WHERE sale_date >= CURRENT_DATE - 90) > 1000 
            THEN 'Regular'
        ELSE 'Occasional'
    END AS customer_segment
FROM sales
GROUP BY customer_id
HAVING 
    COUNT(*) FILTER (WHERE sale_date >= CURRENT_DATE - 90) >= 3
    AND SUM(sale_amount) FILTER (WHERE sale_date >= CURRENT_DATE - 90) > 100;
```

**Advanced GROUP BY Extensions**

*   **GROUPING SETS & CUBE**
```sql
-- Multiple grouping levels in single query
SELECT 
    COALESCE(department, 'All Departments') AS department,
    COALESCE(job_level, 'All Levels') AS job_level,
    COALESCE(region, 'All Regions') AS region,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary,
    GROUPING(department) AS is_department_total,
    GROUPING(job_level) AS is_level_total,
    GROUPING(region) AS is_region_total
FROM employees
GROUP BY GROUPING SETS (
    (department, job_level, region),  -- Detailed level
    (department, job_level),          -- Department by level
    (department, region),             -- Department by region  
    (department),                     -- Department total
    (job_level, region),              -- Level by region
    (job_level),                      -- Level total
    (region),                         -- Region total
    ()                                -- Grand total
)
HAVING COUNT(*) >= 5 
   AND (GROUPING(department) = 0 OR GROUPING(job_level) = 0)
ORDER BY GROUPING(department), GROUPING(job_level), GROUPING(region),
         department, job_level, region;
```

**Performance Optimization for Complex Aggregation**

*   **Efficient Large-Scale Aggregation**
> [!CAUTION]
> Complex aggregations on large datasets require careful optimization.
```sql
-- Optimized aggregation with pre-filtering and indexing
WITH filtered_data AS (
    SELECT 
        department_id,
        job_level,
        salary,
        tenure_years,
        performance_rating
    FROM employees
    WHERE active = true
        AND hire_date >= '2018-01-01'
        AND department_id IN (SELECT department_id FROM departments WHERE active = true)
),
department_aggregates AS (
    SELECT 
        department_id,
        COUNT(*) AS total_count,
        AVG(salary) AS avg_salary,
        COUNT(*) FILTER (WHERE tenure_years > 3) AS experienced_count
    FROM filtered_data
    GROUP BY department_id
    HAVING COUNT(*) >= 10
)
SELECT 
    d.department_name,
    da.total_count,
    da.avg_salary,
    da.experienced_count,
    da.experienced_count * 100.0 / da.total_count AS experienced_pct
FROM department_aggregates da
INNER JOIN departments d ON da.department_id = d.department_id
WHERE da.avg_salary > 50000
ORDER BY da.avg_salary DESC;
```
