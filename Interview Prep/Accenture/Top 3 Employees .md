**1. Top 3 employees with highest salaries**
We assume table `Employees(emp_id, emp_name, salary)`.

```sql
SELECT emp_id, emp_name, salary
FROM Employees
ORDER BY salary DESC
LIMIT 3;   -- SQL Server would use TOP 3
```

This sorts by salary descending and picks the first 3 rows.
