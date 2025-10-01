Finding records in one table that **do not exist** in another table is a very common SQL problem known as the **Anti-Join** pattern. This task identifies "missing," "orphan," or "unmatched" records based on a key column.

There are three primary, high-performance methods for achieving this: **`LEFT JOIN` with `WHERE IS NULL`**, **`NOT EXISTS`**, and the **`EXCEPT`** (or `MINUS`) set operator.

-----

## 1\. LEFT JOIN with WHERE IS NULL (The Most Common) 🎣

This is the most traditional, widely supported, and often highly performant method.

### Technique

1.  Perform a **`LEFT JOIN`** from the primary table (A) to the secondary table (B) on the matching key.
2.  In the `WHERE` clause, check if the key column (or any non-nullable column) from the secondary table (B) is **`NULL`**. This condition is only true for rows in A that had **no match** in B.

### Example

Find all **Customers** who have **not placed an Order**.

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM
    Customers AS c -- Table A: The source of the desired records
LEFT JOIN
    Orders AS o ON c.customer_id = o.customer_id -- Join to the target table
WHERE
    o.order_id IS NULL; -- Only keep rows where the match failed (right side is NULL)
```

-----

## 2\. NOT EXISTS (Robust and Efficient) 🚫

The `NOT EXISTS` operator uses a correlated subquery to check for the absence of a matching row.

### Technique

The subquery runs once for every row in the outer query (Table A). If the subquery finds *any* matching row in Table B, `EXISTS` returns true, but the surrounding `NOT` operator makes the row fail the filter. If the subquery finds *no* matching row, `NOT EXISTS` allows the row to pass.

### Example

Find all **Products** that have **not been sold**.

```sql
SELECT
    p.product_id,
    p.product_name
FROM
    Products AS p -- Outer Query (The desired records)
WHERE
    NOT EXISTS (
        SELECT 1 -- Any value works; only existence is checked
        FROM Sales AS s -- Subquery (The records to exclude)
        WHERE s.product_id = p.product_id -- The correlation link
    );
```

  * **Performance Note:** For large tables, `NOT EXISTS` is often the most efficient method because the subquery can stop searching immediately upon finding the first match.

-----

## 3\. EXCEPT / MINUS (Set Operator) ➖

The `EXCEPT` (or `MINUS` in Oracle) operator performs a mathematical set difference, returning all rows from the first `SELECT` statement that are *not* found in the second.

### Technique

1.  Select the key column (and any other columns you need) from the primary table (A).
2.  Use `EXCEPT` (or `MINUS`).
3.  Select the matching key column from the secondary table (B).

### Example

Find all **Employee IDs** that are in the `AllEmployees` list but **not** in the `ActivePayroll` list.

```sql
SELECT
    employee_id
FROM
    AllEmployees -- Set A: The superset

EXCEPT -- MINUS in Oracle

SELECT
    employee_id
FROM
    ActivePayroll; -- Set B: The subset to be removed
```

  * **Limitation:** The `SELECT` lists in both halves of the `EXCEPT` statement **must be identical** in column count, data types, and order.
  * **Result:** Returns only the distinct `employee_id` values that are in `AllEmployees` but not in `ActivePayroll`.
