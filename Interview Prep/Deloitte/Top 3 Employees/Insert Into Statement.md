Here's the complete `INSERT INTO` query to populate the `Department` and `Employees` tables for testing employee salary analysis scenarios:

```sql
-- Create Department table
CREATE TABLE IF NOT EXISTS Department (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50)
);

-- Create Employees table
CREATE TABLE IF NOT EXISTS Employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10,2),
    department_id INT,
    date_hired DATE,
    FOREIGN KEY (department_id) REFERENCES Department(department_id)
);

-- Clear existing data
TRUNCATE TABLE Department;
TRUNCATE TABLE Employees;

-- Insert departments
INSERT INTO Department (department_id, department_name) VALUES
(1, 'Engineering'),
(2, 'Marketing'),
(3, 'Finance'),
(4, 'HR'),
(5, 'Operations');

-- Insert employees with various salaries and hire dates
INSERT INTO Employees (employee_id, name, salary, department_id, date_hired) VALUES
-- Engineering (top salaries)
(101, 'John Smith', 120000.00, 1, '2018-06-15'),
(102, 'Sarah Johnson', 110000.00, 1, '2020-03-22'),
(103, 'Michael Brown', 105000.00, 1, '2021-09-10'),
(104, 'Emily Davis', 95000.00, 1, '2022-01-05'),
(105, 'Robert Wilson', 85000.00, 1, '2023-11-18'),

-- Marketing (mix of salaries)
(201, 'Jennifer Lee', 90000.00, 2, '2019-04-12'),
(202, 'David Miller', 80000.00, 2, '2020-08-30'),
(203, 'Jessica Taylor', 75000.00, 2, '2021-05-15'),
(204, 'Daniel Anderson', 70000.00, 2, '2022-12-01'),
(205, 'Lisa Martinez', 65000.00, 2, '2023-07-20'),

-- Finance (high salaries, recent hires)
(301, 'Kevin Robinson', 130000.00, 3, '2021-02-14'),
(302, 'Amanda White', 125000.00, 3, '2022-03-25'),
(303, 'James Harris', 115000.00, 3, '2023-01-10'),
(304, 'Maria Garcia', 95000.00, 3, '2023-09-05'),
(305, 'Thomas Martin', 85000.00, 3, '2023-12-15'),

-- HR (lower salaries, older hires)
(401, 'Sophia Clark', 70000.00, 4, '2017-11-08'),
(402, 'William Rodriguez', 65000.00, 4, '2018-05-20'),
(403, 'Olivia Martinez', 60000.00, 4, '2019-10-30'),
(404, 'Benjamin Hernandez', 55000.00, 4, '2020-07-12'),
(405, 'Isabella Lopez', 50000.00, 4, '2021-04-18'),

-- Operations (wide salary range)
(501, 'Mason Gonzalez', 80000.00, 5, '2020-09-15'),
(502, 'Charlotte Wilson', 75000.00, 5, '2021-06-22'),
(503, 'Elijah Anderson', 65000.00, 5, '2022-03-10'),
(504, 'Amelia Thomas', 55000.00, 5, '2023-02-28'),
(505, 'Lucas Moore', 45000.00, 5, '2023-10-15');
```

To test the top 3 highest-paid employees in each department:

```sql
WITH RankedSalaries AS (
    SELECT
        e.name AS employee_name,
        e.salary,
        d.department_name,
        ROW_NUMBER() OVER(PARTITION BY d.department_id ORDER BY e.salary DESC) AS rank_num
    FROM
        Employees e
    JOIN
        Department d ON e.department_id = d.department_id
)
SELECT
    employee_name,
    salary,
    department_name,
    rank_num
FROM
    RankedSalaries
WHERE
    rank_num <= 3
ORDER BY
    department_name,
    rank_num;
```

To test the average salary of employees hired in the last 5 years:

```sql
SELECT
    AVG(salary) AS average_salary_recent_hires,
    COUNT(*) AS number_of_recent_hires
FROM
    Employees
WHERE
    date_hired >= CURRENT_DATE - INTERVAL '5 years';
```

To test employees whose salary is below the average salary of recent hires:

```sql
SELECT
    e.name,
    e.salary,
    d.department_name,
    e.date_hired,
    (SELECT ROUND(AVG(salary), 2) FROM Employees WHERE date_hired >= CURRENT_DATE - INTERVAL '5 years') AS avg_recent_salary
FROM
    Employees e
JOIN
    Department d ON e.department_id = d.department_id
WHERE
    e.salary < (
        SELECT AVG(salary)
        FROM Employees
        WHERE date_hired >= CURRENT_DATE - INTERVAL '5 years'
    )
ORDER BY
    e.salary DESC;
```

Key features of this test data:
1. **Realistic salary distributions** across different departments
2. **Varied hire dates** from 2017 to 2023
3. **Clear department structure** for grouping analysis
4. **Enough employees per department** (5 each) to test ranking
5. **Recent hires with higher salaries** in some departments to test the average salary comparison

The solutions should:
- Correctly identify top earners within each department
- Accurately calculate the average salary for recent hires
- Properly compare individual salaries against the benchmark
- Handle date calculations dynamically (will work in future years)

Additional analysis queries you might want to test:

```sql
-- Department-wise salary statistics
SELECT
    d.department_name,
    COUNT(e.employee_id) AS employee_count,
    ROUND(AVG(e.salary), 2) AS avg_salary,
    ROUND(MIN(e.salary), 2) AS min_salary,
    ROUND(MAX(e.salary), 2) AS max_salary,
    ROUND(MAX(e.salary) - MIN(e.salary), 2) AS salary_range
FROM
    Department d
LEFT JOIN
    Employees e ON d.department_id = e.department_id
GROUP BY
    d.department_name
ORDER BY
    avg_salary DESC;

-- Salary growth analysis by hire year
SELECT
    EXTRACT(YEAR FROM date_hired) AS hire_year,
    COUNT(*) AS employees_hired,
    ROUND(AVG(salary), 2) AS avg_salary,
    ROUND(AVG(salary) - LAG(AVG(salary), 1) OVER (ORDER BY EXTRACT(YEAR FROM date_hired)), 2) AS salary_growth
FROM
    Employees
WHERE
    date_hired IS NOT NULL
GROUP BY
    EXTRACT(YEAR FROM date_hired)
ORDER BY
    hire_year;
```
