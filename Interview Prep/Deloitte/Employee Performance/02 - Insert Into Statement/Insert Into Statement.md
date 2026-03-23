Here's the complete `INSERT INTO` query to populate the `employees` table for testing performance classification scenarios:

```sql
-- Create employees table if it doesn't exist
CREATE TABLE IF NOT EXISTS employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    performance_rating VARCHAR(20),
    salary DECIMAL(10,2),
    department VARCHAR(50)
);

-- Clear existing data (if any)
TRUNCATE TABLE employees;

-- Insert sample data with various performance and salary combinations
INSERT INTO employees (employee_id, name, performance_rating, salary, department) VALUES
-- Top Performers (excellent rating AND salary > 7000)
(1, 'John Smith', 'excellent', 7500.00, 'Engineering'),
(2, 'Sarah Johnson', 'excellent', 8000.00, 'Marketing'),
(3, 'Michael Brown', 'excellent', 9000.00, 'Finance'),

-- Good Performers (good rating AND salary > 5000)
(4, 'Emily Davis', 'good', 6000.00, 'HR'),
(5, 'Robert Wilson', 'good', 5500.00, 'Engineering'),
(6, 'Jennifer Lee', 'good', 5200.00, 'Operations'),

-- Edge cases
(7, 'David Miller', 'excellent', 6500.00, 'Sales'),  -- Excellent but salary <= 7000
(8, 'Jessica Taylor', 'good', 4800.00, 'Support'),   -- Good but salary <= 5000
(9, 'Daniel Anderson', 'average', 8500.00, 'IT'),    -- High salary but average rating
(10, 'Lisa Martinez', 'poor', 4000.00, 'HR'),        -- Low rating and salary

-- Special cases
(11, 'Kevin Robinson', 'excellent', 7000.00, 'Finance'),  -- Borderline salary
(12, 'Amanda White', 'good', 5000.00, 'Marketing'),       -- Borderline salary
(13, 'James Harris', NULL, 6000.00, 'Engineering'),       -- Missing rating
(14, 'Maria Garcia', 'excellent', NULL, 'Sales'),         -- Missing salary
(15, 'Thomas Martin', 'good', 3000.00, 'Support');        -- Low salary despite good rating
```

To test the performance classification query:

```sql
SELECT
    employee_id,
    name,
    performance_rating,
    salary,
    department,
    CASE
        WHEN performance_rating = 'excellent' AND salary > 7000 THEN 'Top Performer'
        WHEN performance_rating = 'good' AND salary > 5000 THEN 'Good Performer'
        ELSE 'Others'
    END AS performance_level,
    CASE
        WHEN performance_rating = 'excellent' AND salary > 7000 THEN 'Eligible for bonus'
        WHEN performance_rating = 'good' AND salary > 5000 THEN 'Development plan'
        ELSE 'Performance review needed'
    END AS action_item
FROM
    employees
ORDER BY
    CASE performance_level
        WHEN 'Top Performer' THEN 1
        WHEN 'Good Performer' THEN 2
        ELSE 3
    END,
    name;
```

Key features of this test data:
1. **Clear Top Performers** (meet both criteria)
2. **Clear Good Performers** (meet both criteria)
3. **Borderline cases** (salary exactly at threshold)
4. **Special cases**:
   - High salary with lower rating
   - Good rating with low salary
   - Missing performance ratings
   - Missing salary data
5. **Department information** for additional analysis

The solution should:
- Correctly classify employees based on both rating AND salary
- Handle NULL values appropriately (defaulting to 'Others')
- Maintain all original columns in the output
- Work with the additional sorting by performance level

Additional queries you might want to test:

```sql
-- Count employees by performance level and department
SELECT
    department,
    COUNT(CASE WHEN performance_rating = 'excellent' AND salary > 7000 THEN 1 END) AS top_performers,
    COUNT(CASE WHEN performance_rating = 'good' AND salary > 5000 THEN 1 END) AS good_performers,
    COUNT(*) - COUNT(CASE WHEN performance_rating = 'excellent' AND salary > 7000 THEN 1 END) 
             - COUNT(CASE WHEN performance_rating = 'good' AND salary > 5000 THEN 1 END) AS others,
    COUNT(*) AS total_employees
FROM
    employees
GROUP BY
    department
ORDER BY
    top_performers DESC;

-- Find employees who are close to the next performance level
SELECT
    employee_id,
    name,
    performance_rating,
    salary,
    CASE
        WHEN performance_rating = 'good' AND salary BETWEEN 6000 AND 7000 
            THEN CONCAT('Needs $', 7000 - salary, ' more to reach Top Performer')
        WHEN performance_rating = 'average' AND salary > 4500 
            THEN CONCAT('Needs good rating to reach Good Performer')
        ELSE 'Meeting current level expectations'
    END AS improvement_opportunity
FROM
    employees
WHERE
    (performance_rating = 'good' AND salary BETWEEN 6000 AND 7000)
    OR (performance_rating = 'average' AND salary > 4500)
ORDER BY
    salary DESC;
```
