# How OFFSET and LIMIT fit into the true SQL execution order

OFFSET and LIMIT look simple, but they sit **after ORDER BY**, which means they operate on the *final, sorted result set* just before rows are returned.

To keep this clean and vertical, here is the full order with JOINs, WHERE, GROUP BY, HAVING, SELECT, ORDER BY, and finally OFFSET/LIMIT.

# The Actual Evaluation Order

## With JOINs, WITH/CTEs, aggregates, windows, LIMIT, OFFSET

### 1. FROM

This is where all JOINs are executed.
Inside this step the engine internally performs:

* table selection
* join optimization
* join execution
* subquery materialization
* CTE expansion logic

Every row that will ever be seen by later stages is born here.

### 2. WHERE

Filters rows produced by FROM/JOIN.
Removes rows; cannot use aggregates; cannot use window functions.

### 3. GROUP BY

Forms groups from the surviving WHERE rows.

### 4. HAVING

Filters after aggregation.
This can use SUM, AVG, COUNT, etc.

### 5. SELECT

Computes expressions, window functions, CASE, aliases.
ORDER OF EXECUTION inside SELECT (internal, not visible syntactically):

1. scalar expressions
2. window functions
3. aliases

### 6. ORDER BY

Sorting happens after SELECT, which is why ORDER BY can use aliases and window functions.

This produces a fully sorted final result set.

# 7. OFFSET

Now the engine discards the first N rows of the sorted result.

OFFSET does **not** reorder anything.
OFFSET does **not** filter based on conditions.
OFFSET just skips rows.

Example:
OFFSET 10 means: remove the first 10 rows from the sorted dataset.

# 8. LIMIT / FETCH / TOP

After OFFSET removes the first chunk, LIMIT takes the next chunk.

LIMIT says: return only N rows *after* OFFSET has done its slicing.

So the combination:

```sql
ORDER BY amount DESC
OFFSET 20
LIMIT 10
```

Means:

1. Sort all rows by amount descending
2. Skip the first 20 rows
3. Return rows 21 through 30

# Why OFFSET/LIMIT must be last

Because OFFSET/LIMIT control *presentation*, not *logic*.
They are not filters and cannot be optimized the same way WHERE or HAVING can.

They only make sense **after** you have:

* formed the working dataset
* filtered it
* aggregated it
* computed window functions
* sorted it

Only at the end does the engine decide how many rows to show you.


# Full Final Order (including OFFSET/LIMIT)

Vertical and exact:

1. FROM
2. JOIN
3. WHERE
4. GROUP BY
5. HAVING
6. SELECT
7. WINDOW FUNCTIONS
8. ORDER BY
9. OFFSET
10. LIMIT

This is the real execution path inside the SQL engine.
