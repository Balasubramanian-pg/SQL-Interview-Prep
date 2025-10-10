The **`CASE`** expression is SQL's powerful conditional logic tool, allowing you to execute `IF/THEN/ELSE` logic. Its placement within a query dictates its purpose: transforming data in the output, filtering rows, or controlling the final sort order.

## CASE in SELECT (Data Transformation)

When used in the `SELECT` clause, `CASE` acts as a **data transformer**, creating new columns or modifying the values of existing columns based on defined conditions.

### Use Case

Categorizing or re-labeling data based on complex business logic.

### Example

Assigning an order to a specific processing lane based on its total amount.

```sql
SELECT
    order_id,
    order_amount,
    CASE
        WHEN order_amount >= 500 THEN 'Priority Lane' -- High value
        WHEN order_amount >= 100 THEN 'Standard Lane'
        ELSE 'Economy Lane'
    END AS processing_lane -- New calculated column
FROM
    Orders;
```

## CASE in WHERE (Conditional Filtering)

When used in the `WHERE` clause, `CASE` acts as a **conditional filter**, allowing the filtering logic itself to change based on the data.

### Use Case

Applying different filter criteria based on a specific input or condition. This is often more complex than standard filtering and can be achieved with `AND`/`OR`, but `CASE` offers structure.

### Example

Only include tasks that are past due **if** the task's priority is 'High'. Otherwise, include tasks that are just 'Active'.

```sql
SELECT
    task_id,
    due_date,
    priority
FROM
    Tasks
WHERE
    CASE
        -- If priority is HIGH, check if the task is past due
        WHEN priority = 'High' THEN due_date < CURRENT_DATE
        -- For all other priorities, check if the task is simply active (not closed)
        ELSE status = 'Active'
    END;
```

## CASE in ORDER BY (Custom Sorting)

When used in the `ORDER BY` clause, `CASE` defines a **custom sort order**, allowing you to override the default alphabetical or numerical sorting.

### Use Case

Sorting categorical data in a specific business-mandated order (e.g., Status: PENDING, PROCESSING, SHIPPED, CANCELLED).

### Example

Sorting orders so that 'High Priority' items appear first, followed by 'Medium', and then 'Low', regardless of their alphabetical order.

```sql
SELECT
    order_id,
    priority
FROM
    Orders
ORDER BY
    CASE priority
        WHEN 'High' THEN 1      -- Sorts High first
        WHEN 'Medium' THEN 2    -- Sorts Medium second
        WHEN 'Low' THEN 3       -- Sorts Low third
        ELSE 4                  -- Puts all others last
    END,
    order_id; -- Secondary sort by ID
```
