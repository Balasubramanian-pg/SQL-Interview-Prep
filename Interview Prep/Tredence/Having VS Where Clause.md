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




Alright, here’s a sneaky **10-question mini quiz** on `WHERE` vs `HAVING`. I’ll just give the queries and you can guess which one works.

---

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

---

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

---

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

---

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

---

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

---

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

---

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

---

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

---

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

---

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

---

* `AVG(salary)` is unknown to `WHERE`; only `HAVING` can see it
