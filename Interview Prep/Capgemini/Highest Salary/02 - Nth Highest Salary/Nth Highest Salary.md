# SQL Interview Questions: Finding Nth Highest Salary  

## 1. **Problem Overview**  
These are common SQL interview questions about finding salary rankings:  
- Find the highest salary.  
- Find the second highest salary.  
- Find the nth highest salary using subqueries.  
- Find the nth highest salary using Common Table Expressions (CTEs).  
- Find any ranked salary (2nd, 3rd, 15th, etc.).  

> [!NOTE]  
> These solutions are applicable to employee salary data and can be adapted to other ranking scenarios.  

---

## 2. **Solutions**  

### 2.1 **Finding the Highest Salary**  
```sql
SELECT MAX(salary) FROM employees;
```  

> [!TIP]  
> This is the simplest and most efficient way to find the highest salary.  

---

### 2.2 **Finding the Second Highest Salary**  
```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```  

> [!WARNING]  
> This method fails if there are duplicate highest salaries. Use with caution or add `DISTINCT` if needed.  


### 2.3 **Finding Nth Highest Salary with Subquery (PostgreSQL)**  
```sql
SELECT salary
FROM (
    SELECT DISTINCT salary
    FROM employees
    ORDER BY salary DESC
    LIMIT 1 OFFSET n-1
) AS subquery;
```  

> [!IMPORTANT]  
> Replace `n` with the desired rank (e.g., `OFFSET 1` for 2nd highest).  
> `DISTINCT` ensures duplicate salaries are handled correctly.  


### 2.4 **Finding Nth Highest Salary with CTE and DENSE_RANK()**  
```sql
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

> [!TIP]  
> Replace `n` with the desired rank.  
> `DENSE_RANK()` is preferred over `RANK()` as it doesn't skip numbers for ties.  


### 2.5 **Finding Any Ranked Salary**  
```sql
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

> [!NOTE]  
> This approach is flexible and can be used to find any ranked salary by changing the value in `WHERE salary_rank = X`.  


## 3. **Key Differences Between Ranking Functions**  

### 3.1 **ROW_NUMBER()**  
- Assigns unique sequential numbers (1, 2, 3...).  
- Duplicate values get different row numbers.  

### 3.2 **RANK()**  
- Assigns the same rank to ties, then skips the next numbers.  
- Example: 1, 2, 2, 4 (skips 3).  

### 3.3 **DENSE_RANK()**  
- Assigns the same rank to ties, but doesn't skip numbers.  
- Example: 1, 2, 2, 3 (no skip).  

> [!IMPORTANT]  
> Use `DENSE_RANK()` when you want consecutive ranking without gaps, even for ties.  


## 4. **Performance Considerations**  

1. **CTE with DENSE_RANK()**:  
   For large tables, this method is generally more efficient than multiple subqueries.  

2. **Indexing**:  
   Adding appropriate indexes on the `salary` column can significantly improve performance.  

3. **Early Filtering**:  
   Consider filtering the dataset early if you only need rankings for a subset of employees.  

> [!WARNING]  
> Avoid using `ORDER BY` in subqueries for large datasets, as it can degrade performance.  


These solutions cover the most common approaches to salary ranking questions in SQL interviews across different database systems.  

> [!TIP]  
> Practice these methods with varying datasets to understand their behavior and performance implications.
