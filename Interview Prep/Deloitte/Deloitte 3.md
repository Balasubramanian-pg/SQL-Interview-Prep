---
Area:
  - Employee Salary
Difficulty: Easy
Main Function:
  - Window
---
### Database Schema

Before we dive into the problems, let's establish the database structure we will be working with.

**Tables:**

- `Employees` (employee_id, name, salary, department_id, date_hired)
- `Department` (department_id, department_name)

**Relationships:**

- `Employees.department_id` is a foreign key that references `Department.department_id`.

---

### Problem 1: Find the top 3 highest-paid employees in each department.

This is a classic "Top N per Group" problem. The key is to rank employees _within_ their department based on salary and then pick the top 3 from each group.

### Explanation

To find who gets paid the most in each department, you first need to match employees to their respective departments. Then, for each department, you assign a rank to every employee based on their salary—the highest salary gets rank 1, the second-highest gets rank 2, and so on. Finally, you select only those employees with a rank of 1, 2, or 3.

### SQL Solution

```SQL
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
    department_name
FROM
    RankedSalaries
WHERE
    rank_num <= 3;
```

### How it Works:

1. **Join Tables**: We start by joining `Employees` and `Department` on `department_id` to get the department name for each employee.
2. **Rank with a Window Function**: We use the `ROW_NUMBER()` window function.
    - `PARTITION BY d.department_id`: This is the crucial part. It divides the data into groups (partitions) based on the department. The ranking will restart from 1 for each new department.
    - `ORDER BY e.salary DESC`: Within each department partition, it orders employees by salary from highest to lowest.
3. **Use a CTE**: The entire ranking query is wrapped in a Common Table Expression (CTE) called `RankedSalaries`. This makes the main query cleaner and easier to read.
4. **Filter by Rank**: Finally, we select from our `RankedSalaries` CTE and use a simple `WHERE` clause to keep only the rows where the rank is 3 or less.

---

### Problem 2: Find the average salary of employees hired in the last 5 years.

This problem tests your ability to work with dates and aggregate functions.

### Explanation

To find out how much new people make on average, you first need to identify everyone who joined the company within the last 5 years. Once you have this group of recent hires, you simply calculate the average of all their salaries.

### SQL Solution

```SQL
SELECT
    AVG(salary) AS average_salary_recent_hires
FROM
    Employees
WHERE
    date_hired >= CURRENT_DATE - INTERVAL '5 years';
```

_Note: The exact syntax for date functions may vary slightly between SQL dialects (e.g., `GETDATE()` in SQL Server, `DATE('now', '-5 years')` in SQLite)._

### How it Works:

1. **Filter by Date**: The `WHERE` clause filters the `Employees` table to include only those records where the `date_hired` is within the last five years. We calculate the cutoff date by subtracting a 5-year interval from the `CURRENT_DATE`.
2. **Calculate Average**: The `AVG(salary)` aggregate function then computes the average salary for this filtered subset of employees.

---

### Problem 3: Find employees whose salary is below the average salary of recent hires.

This problem combines the logic from the previous problem with a subquery to compare values.

### Explanation

First, you need the number calculated in the previous step: the average salary of recent hires. Then, you go through the entire list of employees and compare each person's salary to that average. If an employee's salary is less than that specific average, they should be included in the final list.

### SQL Solution

```SQL
SELECT
    e.name,
    e.salary,
    d.department_name
FROM
    Employees e
JOIN
    Department d ON e.department_id = d.department_id
WHERE
    e.salary < (
        SELECT AVG(salary)
        FROM Employees
        WHERE date_hired >= CURRENT_DATE - INTERVAL '5 years'
    );
```

### How it Works:

1. **Subquery**: The query inside the parentheses is a **subquery**. It is executed first and returns a single value: the average salary of all employees hired in the last 5 years (the exact query from Problem 2).
2. **Outer Query**: The main (or outer) query selects employee details and joins with the `Department` table to get the department name.
3. **Comparison**: The `WHERE` clause of the outer query compares each employee's salary (`e.salary`) to the single value returned by the subquery. Only employees whose salary is strictly less than this average are included in the final result set.