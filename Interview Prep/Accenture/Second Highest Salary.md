**6. Second-highest salary without `LIMIT` or `TOP`**
Assume `Employees(emp_id, emp_name, salary)`.

```sql
SELECT MAX(salary) AS second_highest_salary
FROM Employees
WHERE salary < (SELECT MAX(salary) FROM Employees);
```

The subquery finds the highest salary, then we take the max of all salaries lower than that.
