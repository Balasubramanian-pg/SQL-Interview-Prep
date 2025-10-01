This cheat sheet covers **Recursive Common Table Expressions (CTEs)**, a powerful SQL feature used to perform iterative, self-referencing queries. Recursive CTEs are the standard solution for traversing hierarchical or graph-like data structures of unknown depth, such as organizational charts, bill of materials, or thread replies.

## Recursive CTEs Cheat Sheet Table

| Component | Purpose | Key Requirement | Example (Conceptual) |
| :--- | :--- | :--- | :--- |
| **`WITH RECURSIVE`** | Initiates the recursive process. | Must be declared immediately before the CTE definition. | `WITH RECURSIVE Hierarchy AS (...)` |
| **Anchor Member** | The **non-recursive** part. | Must be the first `SELECT` statement. Defines the starting point (e.g., the root/top-level node). | `SELECT id, parent_id, 1 AS level FROM Table WHERE parent_id IS NULL` |
| **`UNION ALL`** | Connects the anchor and recursive parts. | **Must** be used to combine the anchor and recursive results. | `... UNION ALL ...` |
| **Recursive Member** | The **recursive** part. | Must reference the CTE itself (`Hierarchy`). Defines the step-by-step logic to find the *next* level of the hierarchy. | `SELECT t.id, t.parent_id, h.level + 1 FROM Table t JOIN Hierarchy h ON t.parent_id = h.id` |
| **Termination** | Prevents infinite loops. | Achieved implicitly by the `JOIN` condition when no more matching rows are found. | The process stops when the `JOIN` of the recursive member returns zero rows. |

-----

## Recursive CTEs Mini Playbook (Realistic Query)

This example demonstrates traversing an **Organizational Hierarchy** where employees are linked to their managers.

### Scenario: Finding All Reports Below a Manager

Assume an `Employees` table with columns: `employee_id`, `manager_id`, and `employee_name`. We want to list a manager (ID 10) and everyone reporting, directly or indirectly, to them.

```sql
WITH RECURSIVE OrgHierarchy AS (
    -- 1. ANCHOR MEMBER: Defines the starting point (the top-level manager)
    SELECT
        employee_id,
        manager_id,
        employee_name,
        1 AS level  -- Start level at 1
    FROM
        Employees
    WHERE
        employee_id = 10 -- Start with the specific manager's ID

    UNION ALL

    -- 2. RECURSIVE MEMBER: Defines the step to find the next level of reports
    SELECT
        e.employee_id,
        e.manager_id,
        e.employee_name,
        oh.level + 1 AS level -- Increment the level
    FROM
        Employees AS e
    INNER JOIN
        OrgHierarchy AS oh -- Recursively join back to the CTE result set
        ON e.manager_id = oh.employee_id -- Link the employee's manager_id to the previous level's employee_id
)

-- 3. FINAL SELECT: Retrieve the full result set
SELECT
    employee_name,
    level,
    manager_id
FROM
    OrgHierarchy
ORDER BY
    level, employee_name;
```

### Key Rules of Recursive CTEs

1.  **`UNION ALL` Only:** You must use `UNION ALL` to combine the anchor and recursive members; `UNION` (which removes duplicates) is not allowed.
2.  **No Aggregation in Recursion:** Aggregate functions (`SUM`, `AVG`, `COUNT`, `GROUP BY`) cannot be used in the recursive member's `SELECT` list.
3.  **No External References (in Recursive Part):** The recursive member can only join to the base tables and the CTE itself (the `OrgHierarchy` alias). It cannot directly reference the base tables unless they are part of the join from the CTE.
4.  **Base Case:** The anchor member must define a *finite* start, and the recursive member's `JOIN` must eventually return zero rows to terminate the loop.
