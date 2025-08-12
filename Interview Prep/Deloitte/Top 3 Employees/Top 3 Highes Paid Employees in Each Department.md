# Employee Salary Analysis with SQL  

## 1. **Database Schema**  

**Tables:**  
- **`Employees`**:  
  - `employee_id`  
  - `name`  
  - `salary`  
  - `department_id`  
  - `date_hired`  

- **`Department`**:  
  - `department_id`  
  - `department_name`  

**Relationships:**  
- `Employees.department_id` is a foreign key referencing `Department.department_id`.  

> [!NOTE]  
> This schema is used for all subsequent queries.  

---

## 2. **Problem 1: Find the Top 3 Highest-Paid Employees in Each Department**  

### 2.1 **SQL Solution**  
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
    department_name
FROM
    RankedSalaries
WHERE
    rank_num <= 3;
```  

### 2.2 **Explanation**  
1. **Join Tables**: Combines `Employees` and `Department` on `department_id` to include department names.  
2. **Rank with Window Function**: Uses `ROW_NUMBER()` with `PARTITION BY department_id` to rank employees within each department by salary in descending order.  
3. **CTE**: Encapsulates the ranking logic in a Common Table Expression (`RankedSalaries`) for clarity.  
4. **Filter by Rank**: Selects only employees with a rank of 1, 2, or 3.  

> [!TIP]  
> `ROW_NUMBER()` is ideal for this problem because it assigns a unique rank without ties, ensuring exactly 3 employees per department are selected.  

---

## 3. **Problem 2: Find the Average Salary of Employees Hired in the Last 5 Years**  

### 3.1 **SQL Solution**  
```sql
SELECT
    AVG(salary) AS average_salary_recent_hires
FROM
    Employees
WHERE
    date_hired >= CURRENT_DATE - INTERVAL '5 years';
```  

### 3.2 **Explanation**  
1. **Filter by Date**: Uses `WHERE` to include only employees hired in the last 5 years.  
2. **Calculate Average**: Applies `AVG(salary)` to compute the average salary for recent hires.  

> [!NOTE]  
> The `INTERVAL` syntax may vary across SQL dialects (e.g., `DATE_SUB(CURRENT_DATE, INTERVAL 5 YEAR)` in MySQL).  

---

## 4. **Problem 3: Find Employees Whose Salary is Below the Average Salary of Recent Hires**  

### 4.1 **SQL Solution**  
```sql
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

### 4.2 **Explanation**  
1. **Subquery**: Calculates the average salary of employees hired in the last 5 years.  
2. **Outer Query**: Joins `Employees` and `Department` to include department names.  
3. **Comparison**: Filters employees whose salary is less than the average salary from the subquery.  

> [!IMPORTANT]  
> The subquery must return a single value for the comparison to work. Ensure the subquery logic is correct.  

---

## 5. **Key Concepts**  

### 5.1 **Window Functions**  
- Used for ranking, aggregation, and analysis within partitions (e.g., `ROW_NUMBER()`, `PARTITION BY`).  

### 5.2 **CTEs (Common Table Expressions)**  
- Simplify complex queries by breaking them into reusable parts.  

### 5.3 **Date Arithmetic**  
- Essential for time-based filtering (e.g., `CURRENT_DATE - INTERVAL '5 years'`).  

### 5.4 **Subqueries**  
- Used to embed queries within queries for dynamic comparisons.  

> [!WARNING]  
> Subqueries can impact performance if not optimized. Test with large datasets to ensure efficiency.  

---

This document covers common SQL interview questions related to employee salary analysis, leveraging window functions, CTEs, and date arithmetic.  

> [!TIP]  
> Practice these queries with sample data to reinforce your understanding of how partitioning, ranking, and subqueries work together.  
