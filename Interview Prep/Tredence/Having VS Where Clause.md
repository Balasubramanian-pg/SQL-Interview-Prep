Here’s a tight mini cheat sheet of tricky `WHERE` vs `HAVING` SQL scenarios:

---

**1. Aggregates in WHERE**

```sql
SELECT dept, COUNT(*) 
FROM employees 
WHERE COUNT(*) > 5 
GROUP BY dept;  -- ❌ fails
```

* Aggregate functions **cannot** be used in `WHERE`. Use `HAVING`.

---

**2. Correct use of HAVING**

```sql
SELECT dept, COUNT(*) 
FROM employees 
GROUP BY dept 
HAVING COUNT(*) > 5;  -- ✅ works
```

* Filters **groups** after aggregation.

---

**3. WHERE before aggregation**

```sql
SELECT dept, AVG(salary) 
FROM employees 
WHERE salary > 30000 
GROUP BY dept;  -- ✅ works
```

* Filters **rows** first; aggregation sees only filtered rows.

---

**4. Combining WHERE and HAVING**

```sql
SELECT dept, SUM(salary) 
FROM employees 
WHERE hire_date > '2020-01-01' 
GROUP BY dept 
HAVING SUM(salary) > 50000;  -- ✅ works
```

* `WHERE` filters rows, `HAVING` filters groups.

---

**5. Tricky misassumption**

```sql
SELECT dept, AVG(salary) 
FROM employees 
WHERE AVG(salary) > 40000 
GROUP BY dept;  -- ❌ fails
```

* `AVG(salary)` is unknown to `WHERE`; only `HAVING` can see it.

---
