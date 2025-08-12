---
Area:
  - Employee Salary
Difficulty: Easy
Main Function:
  - CTE
---
# SQL Interview Questions: Finding Nth Highest Salary

## Problem Overview

These are common SQL interview questions about finding salary rankings:

1. Find the highest salary
2. Find the second highest salary
3. Find the nth highest salary using subqueries
4. Find the nth highest salary using Common Table Expressions (CTEs)
5. Find any ranked salary (2nd, 3rd, 15th, etc.)

## Solutions

### 1. Finding the Highest Salary

```SQL
SELECT MAX(salary) FROM employees;
```

### 2. Finding the Second Highest Salary

```SQL
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

### 3. Finding Nth Highest Salary with Subquery (PostgreSQL)

```SQL
SELECT salary
FROM (
    SELECT DISTINCT salary
    FROM employees
    ORDER BY salary DESC
    LIMIT 1 OFFSET n-1
) AS subquery;
```

- Replace `n` with desired rank (e.g., OFFSET 1 for 2nd highest)
- `DISTINCT` ensures we handle duplicate salaries correctly

### 4. Finding Nth Highest Salary with CTE and DENSE_RANK()

```SQL
WITH ranked_salaries AS (
    SELECT
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM employees
)
SELECT salary
FROM ranked_salaries
WHERE salary_rank = n;
```

- Replace `n` with desired rank
- `DENSE_RANK()` is preferred over `RANK()` as it doesn't skip numbers for ties

### 5. Finding Any Ranked Salary

```SQL
-- For 5th highest salary:
WITH ranked_salaries AS (
    SELECT
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
    FROM employees
)
SELECT salary
FROM ranked_salaries
WHERE salary_rank = 5;
```

## Key Differences Between Ranking Functions

1. **ROW_NUMBER()**:
    - Assigns unique sequential numbers (1, 2, 3...)
    - Duplicate values get different row numbers
2. **RANK()**:
    - Assigns same rank to ties, then skips next numbers
    - Example: 1, 2, 2, 4 (skips 3)
3. **DENSE_RANK()**:
    - Assigns same rank to ties, but doesn't skip numbers
    - Example: 1, 2, 2, 3 (no skip)

## Performance Considerations

1. For large tables, CTE with DENSE_RANK() is generally more efficient than multiple subqueries
2. Adding appropriate indexes on salary columns can significantly improve performance
3. Consider filtering early if you only need rankings for a subset of employees

These solutions cover the most common approaches to salary ranking questions in SQL interviews across different database systems.