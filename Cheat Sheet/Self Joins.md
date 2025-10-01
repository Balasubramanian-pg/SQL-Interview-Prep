This cheat sheet focuses on **Self-Joins**, which is a technique used to combine a table with itself. This is essential for comparing rows within the same dataset, such as finding hierarchical relationships or sequences.

## Self-Joins (Comparing Rows Within Same Table) Cheat Sheet Table

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Hierarchical Data** | Finding a parent/child relationship (e.g., employee and manager). | Join the table to itself using two different **aliases** based on an ID $\implies$ Parent ID link. | `SELECT T1.ParentCol, T2.ChildCol FROM Table T1 JOIN Table T2 ON T1.ID = T2.ParentID` |
| **Sequential Events/Comparisons** | Comparing adjacent records (e.g., finding customers who placed two orders in a row). | Join the table to itself on a $\pm 1$ offset relationship in a sequence or time column. | `ON T1.SequenceCol = T2.SequenceCol - 1` |
| **Finding Duplicates/Related** | Identifying pairs of rows that share a common characteristic (e.g., same city, different name). | Join the table to itself on a shared attribute, ensuring the IDs are not equal. | `ON T1.SharedCol = T2.SharedCol AND T1.ID < T2.ID` |
| **Range Overlap** | Identifying records whose time ranges overlap with other records in the same table. | Join on the condition that one range starts before the other ends, and vice versa. | `ON T1.Start < T2.End AND T2.Start < T1.End` |

-----

## Self-Joins Mini Playbook (Realistic Queries)

These snippets illustrate the common use cases for the self-join technique.

### 1\. Hierarchical Data (Employee $\implies$ Manager)

**Use Case:** List all employees alongside the name of their direct manager.

```sql
SELECT
    e.employee_name,
    m.employee_name AS manager_name
FROM
    Employees AS e         -- Alias 'e' for the employee (child)
LEFT JOIN
    Employees AS m         -- Alias 'm' for the manager (parent)
    ON e.manager_id = m.employee_id;
    -- Use a LEFT JOIN to include employees who don't have a manager (e.g., CEO)
```

-----

### 2\. Sequential Events/Comparisons (Consecutive Orders)

**Use Case:** Find all order pairs placed by the same user where the second order (`o2`) was placed on the day immediately following the first order (`o1`).

```sql
SELECT
    o1.user_id,
    o1.order_date AS first_order_date,
    o2.order_date AS second_order_date
FROM
    Orders AS o1
INNER JOIN
    Orders AS o2
    ON o1.user_id = o2.user_id        -- Must be the same user
    AND o2.order_date = o1.order_date + INTERVAL '1 day'; -- PostgreSQL date arithmetic example (syntax varies by DB)
```

-----

### 3\. Finding Duplicates/Related (Same City)

**Use Case:** Find pairs of distinct customers who live in the same city. The condition `c1.customer_id < c2.customer_id` is used to avoid duplicate pairs (A, B) and (B, A) and comparing a row to itself.

```sql
SELECT
    c1.customer_name AS customer_1,
    c2.customer_name AS customer_2,
    c1.city
FROM
    Customers AS c1
INNER JOIN
    Customers AS c2
    ON c1.city = c2.city
    AND c1.customer_id < c2.customer_id; -- Ensures distinct pairs are returned once
```

-----

### 4\. Range Overlap (Scheduling Conflict)

**Use Case:** Identify all meeting pairs that have a time conflict.

```sql
SELECT
    m1.meeting_title AS meeting_a,
    m2.meeting_title AS meeting_b
FROM
    Meetings AS m1
INNER JOIN
    Meetings AS m2
    ON m1.meeting_id < m2.meeting_id  -- Ensures distinct pairs
    AND m1.start_time < m2.end_time  -- A starts before B ends
    AND m2.start_time < m1.end_time;  -- B starts before A ends (the overlap condition)
```
