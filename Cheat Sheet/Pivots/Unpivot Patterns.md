Unpivot patterns, also known as **reverse pivoting** or **row-to-column transformation**, convert data from wide format (where categories are columns) into tall format (where categories are rows). This is essential for reporting, data warehousing, and preparing data for systems that require normalized formats.

There are three primary methods for unpivoting data in SQL, ranging from portable to vendor-specific.

## 1\. UNION ALL (Portable and Explicit) 📝

The `UNION ALL` method is the most **portable** solution, working across virtually all SQL databases (SQL Server, PostgreSQL, MySQL, Oracle, etc.). It is highly explicit but can become tedious with many columns.

### Technique

Select each column you want to unpivot, casting its value as the new "Value" column and providing a literal string for the new "Category" column. Combine all these selects using `UNION ALL`.

### Example

Unpivoting a `Sales` table with columns: `Region`, `Q1_Sales`, `Q2_Sales`, and `Q3_Sales`.

```sql
SELECT
    Region,
    'Q1_Sales' AS Quarter, -- New category column
    Q1_Sales AS Amount     -- New value column
FROM
    Sales

UNION ALL

SELECT
    Region,
    'Q2_Sales' AS Quarter,
    Q2_Sales AS Amount
FROM
    Sales

UNION ALL

SELECT
    Region,
    'Q3_Sales' AS Quarter,
    Q3_Sales AS Amount
FROM
    Sales
ORDER BY
    Region, Quarter;
```

### Result (Conceptual)

| Region | Quarter | Amount |
| :--- | :--- | :--- |
| East | Q1\_Sales | 1500 |
| East | Q2\_Sales | 1800 |
| East | Q3\_Sales | 1900 |
| West | Q1\_Sales | 2200 |
| ... | ... | ... |

## 2\. CROSS APPLY/LATERAL JOIN (SQL Server/PostgreSQL) 🔀

This method is generally cleaner than `UNION ALL` because it avoids repeating the main query multiple times. It's often used when the `UNPIVOT` operator (see below) is not flexible enough.

  * **SQL Server:** Uses `CROSS APPLY`.
  * **PostgreSQL:** Uses `LATERAL JOIN` or the `UNNEST` function.

### Example (SQL Server CROSS APPLY)

Using the same `Sales` table with `Q1_Sales`, `Q2_Sales`, and `Q3_Sales`.

```sql
SELECT
    s.Region,
    q.Quarter, -- New category column
    q.Amount   -- New value column
FROM
    Sales AS s
CROSS APPLY (
    -- Define the columns to be unpivoted as a virtual table
    VALUES
        ('Q1_Sales', s.Q1_Sales),
        ('Q2_Sales', s.Q2_Sales),
        ('Q3_Sales', s.Q3_Sales)
) AS q(Quarter, Amount) -- Assign aliases for the new columns
ORDER BY
    s.Region;
```

## 3\. UNPIVOT Operator (SQL Server/Oracle) ⚙️

The `UNPIVOT` operator is a proprietary, dedicated function in SQL Server and Oracle that provides a concise, single-statement solution.

### Technique

The `UNPIVOT` clause takes the names of the new value and category columns, followed by the explicit list of source columns to unpivot.

### General Syntax (SQL Server)

```sql
SELECT
    [Grouping_Columns],
    [New_Category_Column],
    [New_Value_Column]
FROM
    <Source_Table>
UNPIVOT
(
    [New_Value_Column] FOR [New_Category_Column] IN ([Col_A], [Col_B], [Col_C], ...)
) AS UnpivotAlias;
```

### Example (SQL Server UNPIVOT)

Using the same `Sales` table.

```sql
SELECT
    Region,
    Quarter,    -- New category column
    SalesAmount -- New value column
FROM
    Sales
UNPIVOT
(
    -- Store the values from the source columns here
    SalesAmount FOR Quarter 
    
    -- These are the source columns being unpivoted
    IN (Q1_Sales, Q2_Sales, Q3_Sales) 
) AS SalesUnpivoted
ORDER BY
    Region;
```

### Limitation of UNPIVOT

The primary limitation is that all columns in the `IN` list **must be of the same data type**. If you have mixed types (e.g., `Q1_Sales` is `DECIMAL` and `Q1_Count` is `INT`), you must cast them to a common type (like `DECIMAL` or `VARCHAR`) in a subquery or CTE *before* applying the `UNPIVOT` operator.
