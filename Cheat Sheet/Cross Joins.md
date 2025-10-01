This cheat sheet covers the **`CROSS JOIN`** operation in SQL, which produces the Cartesian product of two tables. This means every row from the first table is combined with every row from the second table.

## CROSS JOIN Use Cases Cheat Sheet Table

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Generating Combinations** | Creating all possible pairings between elements of two sets. | Use `CROSS JOIN` to multiply the row counts. | `SELECT T1.col, T2.col FROM Table1 T1 CROSS JOIN Table2 T2` |
| **Time Series/Date Padding** | Creating a continuous time series for every group, ensuring no date is missing for analysis. | Cross join a set of unique groups (e.g., products) with a calendar table. | `SELECT g.group, d.date FROM Groups g CROSS JOIN Dates d` |
| **Scenario Testing/Permutations** | Combining all values of several parameters to simulate different possibilities. | Cross join parameter tables to generate a large test set. | `SELECT p1.val, p2.val FROM Params1 p1 CROSS JOIN Params2 p2` |
| **Pivot Table Data Structure** | Creating a fixed set of columns (e.g., all months) to ensure they appear in the result, even if data is missing for some. | Cross join the data with a table containing all desired column headers. | `FROM Data d CROSS JOIN ColumnHeaders h` |

-----

## CROSS JOIN Mini Playbook (Realistic Queries)

These snippets illustrate the common and advanced uses for the `CROSS JOIN` function.

### 1\. Generating Combinations (Menu Design)

**Use Case:** Create a complete list of all possible meal combinations by pairing every appetizer with every main course.

```sql
SELECT
    a.appetizer_name,
    m.main_course_name,
    a.price + m.price AS combo_price
FROM
    appetizers AS a
CROSS JOIN
    main_courses AS m;
```

-----

### 2\. Time Series/Date Padding

**Use Case:** Ensure you have a record for **every product on every day** within a date range, even if a product had no sales on a specific day. This is vital for time-series analysis.

```sql
-- Assume 'Product_IDs' table has unique product identifiers
-- Assume 'Calendar' table has one row for every day in the range
SELECT
    p.product_id,
    c.sale_date,
    COALESCE(s.daily_sales, 0) AS daily_sales -- Use COALESCE to fill in 0 for missing dates
FROM
    Product_IDs AS p
CROSS JOIN
    Calendar AS c
LEFT JOIN
    Daily_Sales AS s
    ON p.product_id = s.product_id AND c.sale_date = s.sale_date;
```

-----

### 3\. Scenario Testing (Permutations)

**Use Case:** Generate all possible test cases by combining different browser types and operating systems.

```sql
SELECT
    b.browser_name,
    o.os_name
FROM
    Browsers AS b
CROSS JOIN
    Operating_Systems AS o
WHERE
    b.supported = TRUE AND o.is_active = TRUE; -- Can still apply filters
```

-----

### 4\. Implicit CROSS JOIN Syntax (Old Style)

Note that using multiple tables in the `FROM` clause without specifying a `JOIN` condition also results in a `CROSS JOIN` (Cartesian product). This style is generally discouraged in favor of the explicit `CROSS JOIN` keyword for clarity.

```sql
SELECT
    c.color,
    s.size
FROM
    Colors AS c, Sizes AS s; -- Implicit CROSS JOIN
```
