Employee-Manager hierarchies, often called **organizational charts** or **self-referencing relationships**, are stored in a single database table where a **foreign key column** points back to the **primary key** of the same table. The standard and most powerful way to query these structures in SQL is by using **Recursive Common Table Expressions (CTEs)**.

## The Hierarchy Data Structure 

The structure requires a minimum of two columns to establish the relationship:

| Column | Role | Example Value |
| :--- | :--- | :--- |
| **Primary Key** | Unique identifier for the employee. | `employee_id`: 100 |
| **Foreign Key** | References the $\text{Primary Key}$ of the manager. | `manager_id`: 50 |

| employee\_id | employee\_name | manager\_id |
| :--- | :--- | :--- |
| 10 | CEO Alex | NULL |
| 50 | Manager Bob | 10 |
| 100 | Employee Carol | 50 |
| 101 | Employee David | 50 |

## 1\. Traversing Down the Hierarchy (Finding All Reports)

This is the most common task: starting at a manager's ID, find all employees who report to them, directly or indirectly. This requires a **Recursive CTE**.

### Technique: Recursive CTE

The CTE starts with the known manager (the **Anchor Member**) and recursively joins the table back to the CTE on the condition that the employee's `manager_id` matches the manager's `employee_id` from the previous step.

```sql
WITH RECURSIVE OrgReports AS (
    -- ANCHOR MEMBER: Starts at the top of the desired branch (e.g., Manager Bob, ID 50)
    SELECT
        employee_id,
        manager_id,
        employee_name,
        1 AS level -- Start the level counter at 1
    FROM
        Employees
    WHERE
        employee_id = 50 -- Start with the specific manager's ID (Bob)

    UNION ALL

    -- RECURSIVE MEMBER: Finds the next level of direct reports
    SELECT
        e.employee_id,
        e.manager_id,
        e.employee_name,
        oh.level + 1 AS level -- Increment level
    FROM
        Employees AS e
    INNER JOIN
        OrgReports AS oh -- Join back to the CTE
        ON e.manager_id = oh.employee_id -- Link reports (e.manager_id) to the current level's employee (oh.employee_id)
)
-- Final SELECT: Retrieve the full result set
SELECT
    employee_name,
    level
FROM
    OrgReports;
```

## 2\. Traversing Up the Hierarchy (Finding the Management Chain)

This task involves starting at a specific employee and finding their manager, their manager's manager, and so on, up to the top CEO (the root node).

### Technique: Recursive CTE (Reversed Join)

The logic is similar, but the recursive join condition is **reversed**. The anchor starts at the employee, and the recursive member links the current row's `manager_id` to the previous row's `employee_id`.

```sql
WITH RECURSIVE ManagementChain AS (
    -- ANCHOR MEMBER: Start at the specific employee (e.g., Employee Carol, ID 100)
    SELECT
        employee_id,
        manager_id,
        employee_name
    FROM
        Employees
    WHERE
        employee_id = 100 -- Start with Carol's ID

    UNION ALL

    -- RECURSIVE MEMBER: Finds the next manager up the chain
    SELECT
        m.employee_id,
        m.manager_id,
        m.employee_name
    FROM
        Employees AS m
    INNER JOIN
        ManagementChain AS mc
        ON m.employee_id = mc.manager_id -- Link the manager (m.employee_id) to the current employee's manager_id (mc.manager_id)
)
SELECT
    employee_name
FROM
    ManagementChain;
```

## 3\. Finding Direct Reports Only

To find only the immediate direct reports of a manager, a simple non-recursive `WHERE` clause is sufficient.

```sql
SELECT
    employee_name
FROM
    Employees
WHERE
    manager_id = 50; -- The ID of the manager (Bob)
```
