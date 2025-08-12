# SQL Interview Question: Find Department-Wise Second Highest Salary

## 1. Overview
This document explains how to solve a common "top-N-per-group" SQL interview question: finding the second highest salary within each department. This problem tests your ability to join tables, group data, and use ranking functions or advanced subqueries. We will cover two popular solutions: one using the modern `DENSE_RANK()` window function and another using a more traditional correlated subquery.

## 2. Problem Statement

### 2.1. The Scenario
You are given two tables: `employee` and `department`.

**employee:**
| employee_id | salary | department_id |
|-------------|--------|---------------|
| 1           | 70000  | 101           |
| 2           | 50000  | 101           |
| 3           | 60000  | 101           |
| 4           | 65000  | 102           |
| 5           | 65000  | 102           |
| 6           | 55000  | 102           |
| 7           | 80000  | 103           |
| 8           | 70000  | 103           |

**department:**
| department_id | department_name |
|---------------|-----------------|
| 101           | HR              |
| 102           | Finance         |
| 103           | Marketing       |

### 2.2. The Requirement
Write a SQL query to find the second highest salary for each department, along with the details of the employee(s) who earn it.

**Expected Output:**
| department_name | employee_id | salary |
|-----------------|-------------|--------|
| HR              | 3           | 60000  |
| Finance         | 6           | 55000  |
| Marketing       | 8           | 70000  |

## 3. Setup Script
You can use the following T-SQL script to create the tables and populate them with sample data.

```sql
-- Create Department Table
CREATE TABLE department (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(50)
);
GO

-- Create Employee Table
CREATE TABLE employee (
    employee_id INT PRIMARY KEY,
    salary INT,
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES department(department_id)
);
GO

-- Insert Data
INSERT INTO department (department_id, department_name) VALUES
(101, 'HR'), (102, 'Finance'), (103, 'Marketing');
GO

INSERT INTO employee (employee_id, salary, department_id) VALUES
(1, 70000, 101),
(2, 50000, 101),
(3, 60000, 101),
(4, 65000, 102),
(5, 65000, 102),
(6, 55000, 102),
(7, 80000, 103),
(8, 70000, 103);
GO
```

## 4. Solutions and Explanations

### 4.1. Solution 1: Using the DENSE_RANK() Window Function
This is the most modern, readable, and scalable approach for solving Nth-highest problems.

-   **Explanation:**
    1.  **Join Tables**: We first join `employee` and `department` tables on `department_id`.
    2.  **Generate Ranks**: We use the `DENSE_RANK()` window function to assign a rank to each employee's salary within their department.
        -   `PARTITION BY d.department_name`: This restarts the ranking for each new department.
        -   `ORDER BY e.salary DESC`: This assigns rank `1` to the highest salary, `2` to the second highest, and so on.
    3.  **Filter for Rank 2**: We wrap the query in a Common Table Expression (CTE) and then select only the rows where the calculated rank is `2`.

> [!IMPORTANT]
> `DENSE_RANK()` is used instead of `RANK()` because `DENSE_RANK` does not skip ranks after a tie. For the 'Finance' department, two employees have the highest salary (rank 1). `DENSE_RANK` correctly assigns the next salary the rank of `2`, whereas `RANK` would have assigned it a rank of `3`.

-   **SQL Query:**
    ```sql
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
        salary_rank = 2;
    ```

### 4.2. Solution 2: Using a Correlated Subquery
This is a more traditional method that does not rely on window functions. It is more complex and generally less performant on large datasets.

-   **Explanation:**
    The logic is to find the maximum salary within each department that is strictly less than the absolute maximum salary of that same department.
    1.  **Outer Query**: Joins the two tables.
    2.  **Correlated Subquery**: The `WHERE` clause checks if an employee's salary is equal to the result of a subquery.
    3.  **The Subquery Logic**:
        -   It finds the `MAX(salary)` for a given department.
        -   Crucially, it has its own `WHERE` clause that excludes the absolute top salary for that department, effectively making it find the *second* highest salary.
        -   The query is "correlated" because the inner subqueries reference the department from the outer query.

-   **SQL Query:**
    ```sql
    SELECT
        d.department_name,
        e1.employee_id,
        e1.salary
    FROM
        employee AS e1
    JOIN
        department AS d ON e1.department_id = d.department_id
    WHERE
        e1.salary = (
            -- Find the highest salary...
            SELECT MAX(e2.salary)
            FROM employee AS e2
            WHERE e2.department_id = e1.department_id
            -- ...that is less than the absolute highest salary in that department
            AND e2.salary < (
                SELECT MAX(e3.salary)
                FROM employee AS e3
                WHERE e3.department_id = e1.department_id
            )
        );
    ```

## 5. Summary of Approaches

| Method                | Pros                                                         | Cons                                                              |
|-----------------------|--------------------------------------------------------------|-------------------------------------------------------------------|
| **`DENSE_RANK()`**    | Modern, highly readable, and easily scalable (e.g., for 3rd, 4th, Nth highest). Generally better performance. | Requires understanding of window functions.                      |
| **Correlated Subquery** | Works on older database systems that may not support window functions. | Complex, difficult to read and maintain, and can be slow on large datasets due to repeated subquery execution. |
