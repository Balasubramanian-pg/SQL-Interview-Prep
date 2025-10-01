This cheat sheet covers the **`INTERSECT`** operator in SQL. It's used to combine the result sets of two or more `SELECT` statements, returning **only the rows that are present and identical in all result sets**.

## INTERSECT Cheat Sheet Table

| Feature | Description | How It Works | Key Use Case |
| :--- | :--- | :--- | :--- |
| **Primary Function** | Returns the **common rows** (the mathematical intersection) between two result sets. | Performs an implicit comparison on all columns; only rows that are **exactly identical** in both queries are included. | Finding entities common to two groups (e.g., customers who bought Product A **and** Product B). |
| **Duplicates** | **Removes** duplicate rows from the final result set (similar to `UNION`). | The intersection process inherently requires a de-duplication step to find unique common rows. | The result set always contains distinct rows. |
| **Performance** | Generally **slower** than `UNION ALL` but often faster than using complex `INNER JOIN` or `EXISTS` logic to achieve the same result on multiple columns. | Requires hashing and sorting both result sets for comparison. | Use for complex, multi-column matching where you need distinct results. |
| **Requirement** | Both queries **must** have the same number of columns, and the corresponding columns **must** have compatible data types. | Required to ensure a valid comparison can be made between the two result sets. | Enforce consistency in your `SELECT` lists. |

-----

## INTERSECT Mini Playbook (Realistic Queries)

These snippets illustrate the primary use cases for the `INTERSECT` operator.

### 1\. Simple Intersection (Common Customers)

**Use Case:** Find the **distinct customer IDs** who placed an order in **both** 2024 and 2025.

```sql
SELECT
    customer_id
FROM
    Sales_2024

INTERSECT

SELECT
    customer_id
FROM
    Sales_2025;
    
-- RESULT: A list of customer_ids that appear in the results of both SELECT statements.
```

-----

### 2\. Multi-Column Intersection (Common Products and Prices)

**Use Case:** Identify the exact **Product ID and Price combination** that exists in both the `Active_Inventory` table and the `Promotional_List` table.

```sql
SELECT
    product_id,
    price
FROM
    Active_Inventory

INTERSECT

SELECT
    product_id,
    price
FROM
    Promotional_List;
    
-- RESULT: Returns rows where both product_id AND price match exactly in both tables.
```

-----

### 3\. Alternative to INNER JOIN (When De-duplication is Needed)

While an `INNER JOIN` is often used to find common records, `INTERSECT` is useful when the goal is a **de-duplicated result set** based on matching *all* selected columns.

**Goal:** Find distinct user IDs that appear in both `Users_List_A` and `Users_List_B`.

```sql
-- INTERSECT Approach (Implicitly returns distinct IDs)
SELECT user_id FROM Users_List_A
INTERSECT
SELECT user_id FROM Users_List_B;
```

```sql
-- INNER JOIN Equivalent (Requires DISTINCT keyword to match INTERSECT's behavior)
SELECT DISTINCT
    A.user_id
FROM
    Users_List_A AS A
INNER JOIN
    Users_List_B AS B ON A.user_id = B.user_id;
```

-----

### 4\. Using GROUPING() with INTERSECT

The `INTERSECT` operator can only be used between two complete `SELECT` statements. You cannot use aggregation or `ORDER BY` within the individual query blocks, but you can use `WHERE` and `GROUP BY`.

**Use Case:** Find departments that had sales exceeding $10,000 in **both** Q1 and Q2.

```sql
-- Departments with sales > 10000 in Q1
SELECT
    department_id
FROM
    Sales
WHERE
    sale_quarter = 1
GROUP BY
    department_id
HAVING
    SUM(amount) > 10000

INTERSECT

-- Departments with sales > 10000 in Q2
SELECT
    department_id
FROM
    Sales
WHERE
    sale_quarter = 2
GROUP BY
    department_id
HAVING
    SUM(amount) > 10000;
```
