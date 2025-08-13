They usually try to test your assumptions about syntax, behavior, and edge cases. Common tricks around “which query works” include:

1. **`JOIN` vs `WHERE` on NULLs**

```sql
SELECT * FROM A INNER JOIN B ON A.id = B.id;
SELECT * FROM A, B WHERE A.id = B.id;
```

* Looks similar, but if `id` has NULLs, behavior differs with `JOIN` vs comma join.

2. **`GROUP BY` with non-aggregated columns**

```sql
SELECT id, name FROM employees GROUP BY id;
SELECT id, MAX(name) FROM employees GROUP BY id;
```

* First query fails in strict SQL (e.g., MySQL with ONLY\_FULL\_GROUP\_BY), second works.

3. **`DELETE` with `JOIN` vs subquery**

```sql
DELETE FROM A USING A JOIN B ON A.id = B.id WHERE B.flag = 1;
DELETE FROM A WHERE id IN (SELECT id FROM B WHERE flag = 1);
```

* Both can delete duplicates, but some engines don’t support `USING JOIN`.

4. **`UNION` vs `UNION ALL`**

```sql
SELECT col FROM t1 UNION SELECT col FROM t2;
SELECT col FROM t1 UNION ALL SELECT col FROM t2;
```

* First removes duplicates, second doesn’t; people often assume both work the same.

5. **`NULL` comparisons**

```sql
SELECT * FROM t WHERE col = NULL;
SELECT * FROM t WHERE col IS NULL;
```

* First fails (returns nothing), second works. A classic gotcha.

Basically, they like to trick you with **syntax validity, NULLs, duplicates, aggregation rules, and engine-specific quirks**.
