# Complex Joins & Advanced SQL Logic

*   **Multiple JOIN Patterns**
> [!IMPORTANT]
> JOIN order and type dramatically change result sets.
```sql
-- Complex multi-table relationship
SELECT 
    e.employee_name,
    d.department_name,
    m.manager_name,
    p.project_name,
    c.client_name
FROM employees e
    INNER JOIN departments d ON e.dept_id = d.dept_id
    LEFT JOIN employees m ON e.manager_id = m.employee_id
    INNER JOIN project_assignments pa ON e.employee_id = pa.employee_id
    INNER JOIN projects p ON pa.project_id = p.project_id
    LEFT JOIN clients c ON p.client_id = c.client_id
WHERE e.hire_date > '2020-01-01'
    AND d.location = 'New York';
```

**Advanced JOIN Scenarios**

*   **Self-Joins for Hierarchical Data**
> [!TIP]
> Use self-joins to represent organizational charts or tree structures.
```sql
-- Employee management hierarchy
SELECT 
    emp.employee_name AS Employee,
    mgr.employee_name AS Manager,
    mgr2.employee_name AS 'Manager\'s Manager'
FROM employees emp
    LEFT JOIN employees mgr ON emp.manager_id = mgr.employee_id
    LEFT JOIN employees mgr2 ON mgr.manager_id = mgr2.employee_id
WHERE emp.department = 'Engineering';
```

*   **Multiple Conditions in JOINs**
```sql
-- Complex business logic in JOIN conditions
SELECT 
    o.order_id,
    c.customer_name,
    p.product_name
FROM orders o
    INNER JOIN customers c ON o.customer_id = c.customer_id 
        AND c.status = 'Active'
        AND c.credit_rating > 650
    INNER JOIN order_items oi ON o.order_id = oi.order_id
    INNER JOIN products p ON oi.product_id = p.product_id
        AND p.discontinued = FALSE
        AND (p.category = 'Electronics' OR p.price > 1000);
```

**Complex WHERE Logic**

*   **Multiple Conditional Logic**
> [!NOTE]
> Parentheses are crucial for complex boolean logic.
```sql
-- Complex filtering with AND/OR combinations
SELECT *
FROM sales 
WHERE (product_category = 'Electronics' AND sale_amount > 1000)
   OR (product_category = 'Furniture' AND sale_amount > 5000)
   OR (sale_date BETWEEN '2024-01-01' AND '2024-01-31' 
       AND customer_segment IN ('VIP', 'Corporate'))
   AND return_flag = FALSE
   AND (payment_status = 'Completed' OR credit_approved = TRUE);
```

*   **Correlated Subqueries in WHERE**
```sql
-- Find employees earning more than their department average
SELECT e1.employee_name, e1.salary, e1.department
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department = e1.department  -- Correlation
    AND e2.hire_date > '2019-01-01'
);
```

**Advanced Analytic Patterns**

*   **Window Functions with Complex Logic**
```sql
-- Running totals with partitions and ordering
SELECT 
    employee_id,
    sale_date,
    sale_amount,
    SUM(sale_amount) OVER (
        PARTITION BY employee_id 
        ORDER BY sale_date 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total,
    AVG(sale_amount) OVER (
        PARTITION BY employee_id, DATE_TRUNC('month', sale_date)
    ) AS monthly_avg,
    RANK() OVER (
        PARTITION BY department_id 
        ORDER BY sale_amount DESC
    ) AS dept_rank
FROM sales
WHERE sale_date >= '2024-01-01';
```

*   **Recursive CTEs for Complex Hierarchies**
> [!CAUTION]
> Recursive queries require careful termination conditions.
```sql
-- Organizational hierarchy to unlimited depth
WITH RECURSIVE org_chart AS (
    -- Anchor: Top-level managers (no manager)
    SELECT 
        employee_id,
        employee_name,
        manager_id,
        1 AS level,
        employee_name AS management_chain
    FROM employees 
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive: Subordinates of previous level
    SELECT 
        e.employee_id,
        e.employee_name,
        e.manager_id,
        oc.level + 1,
        oc.management_chain || ' -> ' || e.employee_name
    FROM employees e
    INNER JOIN org_chart oc ON e.manager_id = oc.employee_id
    WHERE level < 10  -- Prevent infinite recursion
)
SELECT * FROM org_chart
ORDER BY level, employee_name;
```

**Complex Aggregation Scenarios**

*   **Multiple GROUP BY with HAVING**
```sql
-- Complex aggregation with multiple conditions
SELECT 
    department_id,
    job_title,
    COUNT(*) AS employee_count,
    AVG(salary) AS avg_salary,
    MAX(salary) AS max_salary,
    SUM(CASE WHEN tenure_years > 5 THEN 1 ELSE 0 END) AS experienced_count
FROM employees
WHERE active_status = TRUE
    AND hire_date >= '2018-01-01'
GROUP BY department_id, job_title
HAVING COUNT(*) >= 5
    AND AVG(salary) BETWEEN 50000 AND 150000
    AND SUM(CASE WHEN tenure_years > 5 THEN 1 ELSE 0 END) >= 2
ORDER BY department_id, avg_salary DESC;
```

*   **Pivoting with Conditional Aggregation**
```sql
-- Dynamic pivoting using CASE statements
SELECT 
    department,
    COUNT(*) AS total_employees,
    SUM(CASE WHEN job_level = 'Junior' THEN 1 ELSE 0 END) AS juniors,
    SUM(CASE WHEN job_level = 'Mid' THEN 1 ELSE 0 END) AS mids,
    SUM(CASE WHEN job_level = 'Senior' THEN 1 ELSE 0 END) AS seniors,
    SUM(CASE WHEN job_level = 'Lead' THEN 1 ELSE 0 END) AS leads,
    AVG(CASE WHEN job_level = 'Senior' THEN salary END) AS avg_senior_salary
FROM employees
GROUP BY department
HAVING COUNT(*) > 10
    AND SUM(CASE WHEN job_level = 'Senior' THEN 1 ELSE 0 END) >= 3;
```

**Complex Business Logic Implementation**

*   **Multi-Table Business Rules**
```sql
-- Complex business validation across multiple tables
SELECT 
    c.customer_id,
    c.customer_name,
    COUNT(o.order_id) AS total_orders,
    SUM(o.order_amount) AS lifetime_value,
    MAX(o.order_date) AS last_order_date,
    CASE 
        WHEN COUNT(o.order_id) >= 10 AND SUM(o.order_amount) > 10000 
            THEN 'Platinum'
        WHEN COUNT(o.order_id) >= 5 AND SUM(o.order_amount) > 5000 
            THEN 'Gold'
        WHEN COUNT(o.order_id) >= 1 AND SUM(o.order_amount) > 1000 
            THEN 'Silver'
        ELSE 'Bronze'
    END AS customer_tier,
    -- Complex eligibility check
    CASE 
        WHEN c.account_status = 'Active'
            AND (SELECT COUNT(*) FROM returns r WHERE r.customer_id = c.customer_id) < 3
            AND DATEDIFF(CURRENT_DATE, MAX(o.order_date)) <= 90
        THEN 'Eligible for Promotion'
        ELSE 'Not Eligible'
    END AS promotion_eligibility
FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    LEFT JOIN returns r ON c.customer_id = r.customer_id
WHERE c.registration_date >= '2020-01-01'
GROUP BY c.customer_id, c.customer_name, c.account_status
HAVING COUNT(o.order_id) > 0
ORDER BY lifetime_value DESC;
```

**Performance Considerations for Complex Queries**

*   **Query Optimization Tips**
> [!TIP]
> Break down complex queries and use CTEs for readability and performance.
```sql
-- Using CTEs to simplify complex logic
WITH department_stats AS (
    SELECT 
        department_id,
        AVG(salary) AS avg_salary,
        COUNT(*) AS employee_count
    FROM employees
    GROUP BY department_id
    HAVING COUNT(*) >= 5
),
high_earners AS (
    SELECT 
        e.*,
        ds.avg_salary
    FROM employees e
    INNER JOIN department_stats ds ON e.department_id = ds.department_id
    WHERE e.salary > ds.avg_salary * 1.2
)
SELECT 
    he.employee_name,
    he.salary,
    d.department_name,
    he.avg_salary AS department_average
FROM high_earners he
INNER JOIN departments d ON he.department_id = d.department_id
ORDER BY he.salary DESC;
```
