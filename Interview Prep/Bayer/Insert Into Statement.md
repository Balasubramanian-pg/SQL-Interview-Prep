Here's the `INSERT INTO` query to populate the tables for the "Department-Wise Second Highest Salary" problem:

```sql
-- Create Department Table
CREATE TABLE department (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50)
);

-- Create Employee Table
CREATE TABLE employee (
    employee_id INT PRIMARY KEY,
    salary INT,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES department(department_id)
);

-- Insert Department Data
INSERT INTO department (department_id, department_name) VALUES
(101, 'HR'),
(102, 'Finance'),
(103, 'Marketing'),
(104, 'Engineering'),
(105, 'Operations');

-- Insert Employee Data with various scenarios:
-- 1. Normal case with clear 2nd highest
-- 2. Department with salary ties
-- 3. Department with only 1 employee (should return nothing)
-- 4. Department with all employees having same salary
INSERT INTO employee (employee_id, salary, department_id) VALUES
-- HR Department (101)
(1, 70000, 101),  -- Highest
(2, 50000, 101),  -- Lowest
(3, 60000, 101),  -- 2nd highest
(4, 60000, 101),  -- Tie for 2nd highest

-- Finance Department (102)
(5, 65000, 102),  -- Highest (tie)
(6, 65000, 102),  -- Highest (tie)
(7, 55000, 102),  -- 2nd highest
(8, 50000, 102),  -- 3rd highest

-- Marketing Department (103)
(9, 80000, 103),  -- Highest
(10, 70000, 103), -- 2nd highest
(11, 70000, 103), -- 2nd highest (tie)
(12, 60000, 103), -- 3rd highest

-- Engineering Department (104) - only 1 employee
(13, 90000, 104), -- Only employee

-- Operations Department (105) - all same salary
(14, 50000, 105),
(15, 50000, 105),
(16, 50000, 105);
```

This test data includes:
1. A normal department (HR) with clear salary differences
2. A department (Finance) with ties for highest salary
3. A department (Marketing) with ties for second highest salary
4. A department (Engineering) with only one employee (shouldn't appear in results)
5. A department (Operations) where all employees have the same salary (shouldn't appear in results)

You can verify the solution works with this query:

```sql
-- Using DENSE_RANK() solution
WITH RankedSalaries AS (
    SELECT
        d.department_name,
        e.employee_id,
        e.salary,
        DENSE_RANK() OVER (PARTITION BY d.department_name ORDER BY e.salary DESC) AS salary_rank
    FROM
        employee AS e
    JOIN
        department AS d ON e.department_id = d.department_id
)
SELECT
    department_name,
    employee_id,
    salary
FROM
    RankedSalaries
WHERE
    salary_rank = 2
ORDER BY
    department_name;
```

Expected results should show:
- HR: employee(s) with 60000 salary
- Finance: employee with 55000 salary
- Marketing: employee(s) with 70000 salary
(Engineering and Operations shouldn't appear as they don't have a true second highest salary)
