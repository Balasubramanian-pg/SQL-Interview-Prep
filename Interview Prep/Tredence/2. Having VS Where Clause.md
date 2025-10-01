Here’s a tight mini cheat sheet of tricky `WHERE` vs `HAVING` SQL scenarios:

**1. Aggregates in WHERE**

```sql
SELECT dept, COUNT(*) 
FROM employees 
WHERE COUNT(*) > 5 
GROUP BY dept;  -- ❌ fails
```

* Aggregate functions **cannot** be used in `WHERE`. Use `HAVING`.

**2. Correct use of HAVING**

```sql
SELECT dept, COUNT(*) 
FROM employees 
GROUP BY dept 
HAVING COUNT(*) > 5;  -- ✅ works
```

* Filters **groups** after aggregation.

**3. WHERE before aggregation**

```sql
SELECT dept, AVG(salary) 
FROM employees 
WHERE salary > 30000 
GROUP BY dept;  -- ✅ works
```

* Filters **rows** first; aggregation sees only filtered rows.

**4. Combining WHERE and HAVING**

```sql
SELECT dept, SUM(salary) 
FROM employees 
WHERE hire_date > '2020-01-01' 
GROUP BY dept 
HAVING SUM(salary) > 50000;  -- ✅ works
```

* `WHERE` filters rows, `HAVING` filters groups.

**5. Tricky misassumption**

```sql
SELECT dept, AVG(salary) 
FROM employees 
WHERE AVG(salary) > 40000 
GROUP BY dept;  -- ❌ fails
```




Alright, here’s a sneaky **10-question mini quiz** 
**Q1**

```sql
SELECT dept, COUNT(*) 
FROM employees 
WHERE COUNT(*) > 5 
GROUP BY dept;
```

```sql
SELECT dept, COUNT(*) 
FROM employees 
GROUP BY dept 
HAVING COUNT(*) > 5;
```
**Q2**

```sql
SELECT dept, AVG(salary) 
FROM employees 
WHERE salary > 30000 
GROUP BY dept;
```

```sql
SELECT dept, AVG(salary) 
FROM employees 
HAVING salary > 30000 
GROUP BY dept;
```
**Q3**

```sql
SELECT dept, SUM(salary) 
FROM employees 
WHERE hire_date > '2020-01-01' 
GROUP BY dept 
HAVING SUM(salary) > 50000;
```

```sql
SELECT dept, SUM(salary) 
FROM employees 
HAVING SUM(salary) > 50000 
WHERE hire_date > '2020-01-01';
```
**Q4**

```sql
SELECT dept, COUNT(*) 
FROM employees 
WHERE dept IS NOT NULL 
GROUP BY dept 
HAVING COUNT(*) > 10;
```

```sql
SELECT dept, COUNT(*) 
FROM employees 
WHERE dept IS NOT NULL 
HAVING COUNT(*) > 10;
```
**Q5**

```sql
SELECT dept, AVG(salary) 
FROM employees 
WHERE AVG(salary) > 40000 
GROUP BY dept;
```

```sql
SELECT dept, AVG(salary) 
FROM employees 
GROUP BY dept 
HAVING AVG(salary) > 40000;
```
**Q6**

```sql
SELECT dept, SUM(salary) 
FROM employees 
GROUP BY dept 
HAVING SUM(salary) < 100000;
```

```sql
SELECT dept, SUM(salary) 
FROM employees 
WHERE SUM(salary) < 100000 
GROUP BY dept;
```

**Q7**

```sql
SELECT dept, MAX(salary) 
FROM employees 
WHERE salary = MAX(salary) 
GROUP BY dept;
```

```sql
SELECT dept, MAX(salary) 
FROM employees 
GROUP BY dept 
HAVING salary = MAX(salary);
```
**Q8**

```sql
SELECT dept, COUNT(*) 
FROM employees 
WHERE dept LIKE 'S%' 
GROUP BY dept 
HAVING COUNT(*) > 5;
```

```sql
SELECT dept, COUNT(*) 
FROM employees 
WHERE dept LIKE 'S%' 
HAVING COUNT(*) > 5;
```
**Q9**

```sql
SELECT dept, SUM(salary) 
FROM employees 
WHERE salary > 20000 
GROUP BY dept;
```

```sql
SELECT dept, SUM(salary) 
FROM employees 
GROUP BY dept 
HAVING salary > 20000;
```
**Q10**

```sql
SELECT dept, AVG(salary) 
FROM employees 
WHERE hire_date > '2021-01-01' 
GROUP BY dept 
HAVING AVG(salary) > 50000;
```

```sql
SELECT dept, AVG(salary) 
FROM employees 
WHERE hire_date > '2021-01-01' 
HAVING AVG(salary) > 50000;
```

* `AVG(salary)` is unknown to `WHERE`; only `HAVING` can see it


Here’s a clean table with the **answers and explanations** for the 10-question `WHERE` vs `HAVING` quiz:

| Q# | Correct Query                                                   | Explanation                                                                               |
| -- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| 1  | `HAVING COUNT(*) > 5`                                           | Aggregates cannot be used in `WHERE`; `HAVING` filters groups.                            |
| 2  | `WHERE salary > 30000`                                          | Row-level filter before aggregation; `HAVING salary > 30000` is invalid.                  |
| 3  | `WHERE ... HAVING SUM(salary) > 50000`                          | `WHERE` filters rows first; `HAVING` filters aggregated groups; reversing order fails.    |
| 4  | `WHERE dept IS NOT NULL ... HAVING COUNT(*) > 10`               | Proper combination: `WHERE` filters rows, `HAVING` filters groups.                        |
| 5  | `HAVING AVG(salary) > 40000`                                    | `AVG(salary)` is aggregate; cannot be in `WHERE`.                                         |
| 6  | `HAVING SUM(salary) < 100000`                                   | Aggregates must be filtered using `HAVING`.                                               |
| 7  | Neither                                                         | `salary = MAX(salary)` cannot be used directly; would need a subquery or window function. |
| 8  | `WHERE dept LIKE 'S%' ... HAVING COUNT(*) > 5`                  | Correct use: `WHERE` filters rows, `HAVING` filters groups.                               |
| 9  | `WHERE salary > 20000 ... GROUP BY dept`                        | Filtering rows before aggregation; cannot put non-aggregated column in `HAVING`.          |
| 10 | `WHERE hire_date > '2021-01-01' ... HAVING AVG(salary) > 50000` | `WHERE` filters rows; `HAVING` filters groups based on aggregate.                         |
