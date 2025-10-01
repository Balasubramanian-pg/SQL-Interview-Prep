This cheat sheet covers the specific, and often confusing, behavior of **`NULL` values within SQL `JOIN` conditions**. Understanding how `NULL` is treated in a comparison is essential because it violates standard equality logic.

## NULL in JOINs Cheat Sheet Table

| Scenario | Join Type | Condition | NULL Behavior | Key Result |
| :--- | :--- | :--- | :--- | :--- |
| **INNER/Standard JOIN** | `INNER JOIN` | `T1.col = T2.col` | **`NULL = NULL` is FALSE.** Rows where both columns are `NULL` will **never match** and are **excluded**. | Only rows with non-NULL, matching values are returned. |
| **LEFT JOIN (Right Side)** | `LEFT JOIN` | `T1.col = T2.col` | **Unmatched `NULL`s are preserved.** If a row on the left has no match (including matches that fail due to `NULL` equality), the right side returns **`NULL`**. | All left rows are returned. If the join column on the right is `NULL` but the left column is not, it's considered a non-match. |
| **Matching NULLs (Workaround)** | `INNER/LEFT JOIN` | `T1.col = T2.col OR (T1.col IS NULL AND T2.col IS NULL)` | Explicitly checks for the case where both values are `NULL` using `IS NULL`. | Forces rows where the join columns are both `NULL` to match. |
| **Filtering NULLs** | `LEFT JOIN` | `T2.col IS NULL` (in `WHERE`) | Used after a `LEFT JOIN` to identify rows that had **no match** in the right table. | Finds "orphan" records in the left table (Anti-Join Pattern). |

-----

## NULL in JOINs Mini Playbook (Realistic Queries)

These snippets illustrate how `NULL` values are handled when joining a primary table (`Employees`) to a configuration table (`Configs`) on an optional field, `department_code`.

### Setup: Data with NULLs

Assume the following data:

| Employees.employee\_id | Employees.department\_code | Configs.department\_code | Configs.setting |
| :--- | :--- | :--- | :--- |
| 101 | 'SALES' | 'SALES' | High |
| 102 | NULL | 'HR' | Medium |
| 103 | NULL | NULL | Low |

### 1\. Standard INNER JOIN (Excludes NULLs)

**Use Case:** Find matching configurations. The rows where `department_code` is `NULL` (Employees 102, 103, and Configs row 3) are *not* considered a match.

```sql
SELECT
    e.employee_id,
    c.setting
FROM
    Employees AS e
INNER JOIN
    Configs AS c ON e.department_code = c.department_code;

/*
RESULT:
employee_id | setting
---------------------
101         | High

(Rows for 102 and 103 are excluded because NULL != NULL)
*/
```

-----

### 2\. LEFT JOIN (Preserves Unmatched Left Rows)

**Use Case:** List all employees and their settings, if available. Employees 102 and 103 fail the equality check, resulting in `NULL` on the right side.

```sql
SELECT
    e.employee_id,
    e.department_code,
    c.setting
FROM
    Employees AS e
LEFT JOIN
    Configs AS c ON e.department_code = c.department_code;

/*
RESULT:
employee_id | department_code | setting
---------------------------------------
101         | SALES           | High
102         | NULL            | NULL  <-- Unmatched, right side is NULL
103         | NULL            | NULL  <-- Unmatched, right side is NULL
*/
```

-----

### 3\. Matching NULLs Explicitly (Forcing the Match)

**Use Case:** Join configurations to employees, ensuring that records where **both** `department_code` fields are `NULL` are treated as a match.

```sql
SELECT
    e.employee_id,
    c.setting
FROM
    Employees AS e
INNER JOIN
    Configs AS c ON e.department_code = c.department_code
    OR (e.department_code IS NULL AND c.department_code IS NULL);

/*
RESULT:
employee_id | setting
---------------------
101         | High
103         | Low  <-- Matched because both department_code were IS NULL
*/
```

-----

### 4\. Anti-Join to Find Employees with NULL Code

**Use Case:** Find employees who do *not* have a matching, non-NULL configuration setting.

```sql
SELECT
    e.employee_id,
    e.department_code
FROM
    Employees AS e
LEFT JOIN
    Configs AS c ON e.department_code = c.department_code
WHERE
    c.setting IS NULL;

/*
RESULT:
employee_id | department_code
-----------------------------
102         | NULL
103         | NULL

(If employee 102 had a non-NULL code that wasn't in Configs, they'd also be here)
*/
```
