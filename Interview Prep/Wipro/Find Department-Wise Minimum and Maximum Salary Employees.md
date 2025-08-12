# Find Department-Wise Minimum and Maximum Salary Employees  

## 1. **Overview**  
This document explains how to solve a common SQL interview question that requires identifying employees with the minimum and maximum salaries within each department. This problem tests your understanding of advanced SQL window functions and their application in grouping and ranking data.  

> [!NOTE]  
> This solution is widely applicable in HR analytics, payroll systems, and any scenario requiring departmental salary comparisons.  

## 2. **Problem Statement**  

### 2.1 **The Scenario**  
You are given an `employee` table with employee details, including their department and salary.  

**employee:**  
| employee_name | department_id | salary |  
|---------------|---------------|--------|  
| Shiva         | 1             | 10000  |  
| Prasad        | 1             | 25000  |  
| Anna          | 2             | 10000  |  
| Ravi          | 2             | 40000  |  
| David         | 2             | 20000  |  
| ...           | ...           | ...    |  

### 2.2 **The Requirement**  
Write a SQL query that returns the `department_id`, the name of the employee with the minimum salary, and the name of the employee with the maximum salary for each department.  

**Expected Output:**  
| department_id | min_salary_emp_name | max_salary_emp_name |  
|---------------|---------------------|---------------------|  
| 1             | Shiva               | Prasad              |  
| 2             | Anna                | Ravi                |  

> [!IMPORTANT]  
> The solution must handle ties (e.g., multiple employees with the same minimum or maximum salary) and return only one result per department.  

## 3. **Setup Script**  
You can use the following T-SQL script to create the table and populate it with sample data.  

```sql
-- Create the table
CREATE TABLE employee (
    employee_name VARCHAR(50),
    department_id INT,
    salary INT
);
GO

-- Insert sample data
INSERT INTO employee (employee_name, department_id, salary) VALUES
('Shiva', 1, 10000),
('Prasad', 1, 25000),
('Anna', 2, 10000),
('Ravi', 2, 40000),
('David', 2, 20000);
GO

-- Verify the initial data
SELECT * FROM employee;
```  

## 4. **Solutions and Explanations**  

### 4.1 **Solution 1: Using the `FIRST_VALUE()` Window Function**  
This is a clean and efficient approach that uses the `FIRST_VALUE()` function, which retrieves the first value in an ordered set of data within a partition.  

- **Explanation**:  
  1. **`PARTITION BY department_id`**: This divides the data into groups for each department.  
  2. **`ORDER BY salary ASC`**: When finding the minimum salary employee, we order the salaries in ascending order. `FIRST_VALUE()` then picks the name of the employee with the lowest salary.  
  3. **`ORDER BY salary DESC`**: To find the maximum salary employee, we order in descending order. `FIRST_VALUE()` then picks the name of the employee with the highest salary.  
  4. **`SELECT DISTINCT`**: Since the window functions return the same result for every row within a partition, we use `SELECT DISTINCT` on the `department_id` to show only one row per department.  

> [!TIP]  
> `FIRST_VALUE()` is ideal for this problem as it directly retrieves the first value in the ordered partition.  

- **SQL Query**:  
  ```sql
  SELECT DISTINCT
      department_id,
      FIRST_VALUE(employee_name) OVER (PARTITION BY department_id ORDER BY salary ASC) AS min_salary_emp_name,
      FIRST_VALUE(employee_name) OVER (PARTITION BY department_id ORDER BY salary DESC) AS max_salary_emp_name
  FROM
      employee
  ORDER BY
      department_id;
  ```  

### 4.2 **Solution 2: Using `ROW_NUMBER()`, `CASE`, and `GROUP BY`**  
This approach uses `ROW_NUMBER()` to rank employees by salary within each department and then uses conditional aggregation to select the corresponding names.  

- **Explanation**:  
  1. **Step 1 (CTE): Generate Ranks**: We use `ROW_NUMBER()` to create two ranks: `rn_asc` (ascending salary order) and `rn_desc` (descending salary order). `rn_asc = 1` identifies the minimum salary employee, and `rn_desc = 1` identifies the maximum salary employee.  
  2. **Step 2 (Conditional Aggregation)**: We `GROUP BY department_id`. Within the `GROUP BY` query, we use `CASE` statements to pick the `employee_name` when the respective rank is `1`.  
  3. **`MAX()` or `MIN()` in the `SELECT`**: The `MAX()` aggregate function is used on the `CASE` statement results. Since the `CASE` statement returns `NULL` for all employees except the rank `1` employee, `MAX()` simply picks the non-`NULL` value.  

> [!IMPORTANT]  
> This approach is more flexible and can be adapted to find the Nth highest or lowest values.  

- **SQL Query**:  
  ```sql
  WITH RankedEmployees AS (
      SELECT
          employee_name,
          department_id,
          salary,
          ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary ASC) AS rn_asc,
          ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rn_desc
      FROM
          employee
  )
  SELECT
      department_id,
      MAX(CASE WHEN rn_asc = 1 THEN employee_name END) AS min_salary_emp_name,
      MAX(CASE WHEN rn_desc = 1 THEN employee_name END) AS max_salary_emp_name
  FROM
      RankedEmployees
  GROUP BY
      department_id
  ORDER BY
      department_id;
  ```  

## 5. **Summary**  
Both `FIRST_VALUE()` and `ROW_NUMBER()` are valid and common solutions for this problem. `FIRST_VALUE()` often results in a cleaner, more direct query, while `ROW_NUMBER()` is a powerful technique applicable to many ranking and filtering problems.  

| **Method**                | **Pros**                                                         | **Cons**                                                              |  
|---------------------------|------------------------------------------------------------------|-----------------------------------------------------------------------|  
| **`FIRST_VALUE()`**       | Concise and efficient; expresses intent clearly.                 | Less intuitive than `ROW_NUMBER` if you are unfamiliar with it.       |  
| **`ROW_NUMBER()`**        | Highly flexible and useful for various ranking problems.         | Requires two-step logic (ranking in a CTE, then filtering/aggregating). |  

---

This solution demonstrates advanced SQL techniques, including window functions and conditional aggregation, making it a valuable addition to your SQL toolkit.  

> [!TIP]  
> Practice this pattern with different datasets to reinforce your understanding of departmental salary analysis and ranking techniques.  
