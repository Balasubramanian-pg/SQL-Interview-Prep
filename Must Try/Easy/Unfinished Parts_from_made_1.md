---
Company:
  - Tesla
Difficulty: Intermediate
Created:
Status:
Category:
Sub category:
Question Link:
---
### 1. The Question

The business or operational question being asked by this SQL query is:

**"Which types of parts are currently in the assembly process and have not yet been finished? Provide a unique list of these part names."**

---

### 2. Table Schema

The query references a single table, `parts_assembly`. A plausible schema for this table, which tracks the status of individual parts, would be:

```sql
-- Table: parts_assembly
-- Tracks the assembly status of individual car parts.
CREATE TABLE parts_assembly (
    assembly_id   INT PRIMARY KEY,        -- A unique ID for each individual part being assembled
    part          VARCHAR(255) NOT NULL,  -- The name of the part, e.g., 'steering wheel', 'battery pack'
    assembly_line VARCHAR(50),            -- The assembly line where the part is being built
    start_date    DATE NOT NULL,          -- When the assembly for this specific part began
    finish_date   DATE NULL               -- When assembly was completed. NULL if it is still in progress.
);
```

---

### 3. Structured SQL Query (Method 1: Using `SELECT DISTINCT`)

This is your original query, which is the most direct and declarative way to ask this question.

```sql
SELECT DISTINCT
    part
FROM
    parts_assembly
WHERE
    finish_date IS NULL;
```

---

### 4. Explanation of the Query

This query efficiently finds all unique part types that are currently unfinished. The database executes this in a couple of logical steps:

1.  **`FROM parts_assembly`**
    *   The query starts by specifying that it will read data from the `parts_assembly` table.

2.  **`WHERE finish_date IS NULL`**
    *   This is the filtering step. It scans the table and keeps only the rows where the `finish_date` column has a `NULL` value. In the context of this table, a `NULL` `finish_date` signifies that the part's assembly has started but has not yet been completed.

3.  **`SELECT DISTINCT part`**
    *   This part operates on the filtered result set (only the unfinished parts).
    *   `SELECT part`: It selects the `part` column from these rows.
    *   `DISTINCT`: This is a crucial keyword. Without it, if there were 500 steering wheels currently in assembly, you would get "steering wheel" listed 500 times. `DISTINCT` removes all duplicate values from the final result set, ensuring that each part name appears only once.

The final output is a clean, unique list of the names of all parts that are still being assembled.

---

### 5. Another SQL Method (Method 2: Using `GROUP BY`)

An alternative way to get a unique list of values is to use the `GROUP BY` clause. This method achieves the exact same result.

```sql
SELECT
    part
FROM
    parts_assembly
WHERE
    finish_date IS NULL
GROUP BY
    part;
```

---

### 6. Explanation of the Alternative Query

This method uses grouping to implicitly create a unique list of parts.

1.  **`FROM parts_assembly WHERE finish_date IS NULL`**
    *   This part is identical to the first method. The query first filters the table to get a list of all individual parts that are unfinished.

2.  **`GROUP BY part`**
    *   This is the key difference. The `GROUP BY` clause takes the filtered rows and groups them into "buckets" based on the `part` name. All rows for "steering wheel" go into one bucket, all rows for "battery pack" go into another, and so on. By definition, this creates one group for each unique part name.

3.  **`SELECT part`**
    *   The `SELECT` clause then simply retrieves the `part` name from each of these unique groups. The result is the same unique list of part names as the `SELECT DISTINCT` method.

**Comparison:**
*   For the simple task of getting a unique list of values from a column, `SELECT DISTINCT` is often considered more readable and directly states the intent.
*   `GROUP BY` is a more powerful tool that is essential when you need to perform aggregate calculations (like `COUNT(*)`, `SUM()`, `AVG()`) on each group.
*   In most modern database systems, the query optimizer is smart enough to treat `SELECT DISTINCT column` and `SELECT column ... GROUP BY column` identically, so there is typically no performance difference between them for this specific task.