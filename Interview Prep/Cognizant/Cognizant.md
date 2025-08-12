---
Difficulty: Easy
---
# SQL Joins Interview Question Summary

## Question

Given two tables:

- Table1: ID column with values (1, 1, 1, 2, 2, 3, 4, null, null) - 9 rows
- Table2: ID column with values (1, 1, 1, 2, 2, 3, null) - 7 rows

Determine how many records each type of join will produce when joining on the ID column:

1. INNER JOIN
2. LEFT JOIN
3. RIGHT JOIN
4. FULL JOIN
5. CROSS JOIN

## Easiest Solution

### INNER JOIN (14 records)

```SQL
SELECT COUNT(*)
FROM Table1
INNER JOIN Table2 ON Table1.ID = Table2.ID;
```

### LEFT JOIN (17 records)

```SQL
SELECT COUNT(*)
FROM Table1
LEFT JOIN Table2 ON Table1.ID = Table2.ID;
```

### RIGHT JOIN (15 records)

```SQL
SELECT COUNT(*)
FROM Table1
RIGHT JOIN Table2 ON Table1.ID = Table2.ID;
```

### FULL JOIN (18 records)

```SQL
SELECT COUNT(*)
FROM Table1
FULL JOIN Table2 ON Table1.ID = Table2.ID;
```

### CROSS JOIN (63 records)

```SQL
SELECT COUNT(*)
FROM Table1
CROSS JOIN Table2;
```

## Code Explanation

1. **INNER JOIN**: Returns only matching non-null records from both tables.
    - Counts all pairs where [Table1.ID](http://table1.id/) = [Table2.ID](http://table2.id/) (and neither is null)
    - Result: 14 records
2. **LEFT JOIN**: Returns all records from Table1 plus matching records from Table2.
    - Includes all 14 matches from INNER JOIN plus 3 unmatched records from Table1 (4 and two nulls)
    - Result: 17 records (14 + 3)
3. **RIGHT JOIN**: Returns all records from Table2 plus matching records from Table1.
    - Includes all 14 matches from INNER JOIN plus 1 unmatched record from Table2 (null)
    - Result: 15 records (14 + 1)
4. **FULL JOIN**: Returns all records when there's a match in either table.
    - Includes all 14 matches from INNER JOIN + 3 unmatched from LEFT + 1 unmatched from RIGHT
    - Result: 18 records (14 + 3 + 1)
5. **CROSS JOIN**: Returns Cartesian product (every row from Table1 paired with every row from Table2).
    - Simple multiplication: 9 rows × 7 rows = 63 records

Note: NULL values don't match with each other in joins (NULL ≠ NULL in SQL).