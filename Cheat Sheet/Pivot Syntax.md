The **`PIVOT`** operator in SQL Server is a dedicated, proprietary extension used to rotate data from a row-based format into a column-based format for summarized reporting. It is a specific alternative to the manual pivot achieved with `SUM` and `CASE` (conditional aggregation).

The `PIVOT` operator simplifies the syntax for aggregating data from a **single categorical column** into multiple summary columns.

-----

## SQL Server PIVOT Syntax Cheat Sheet

The `PIVOT` operation works by defining three key elements:

1.  **Grouping Column(s):** The column(s) that will remain as rows (e.g., `Region`).
2.  **Spreading Column:** The categorical column whose unique values will become the new column headers (e.g., `Status`).
3.  **Aggregating Value:** The column whose values will be summarized (e.g., `SalesAmount`).

### General Syntax

```sql
SELECT
    -- 1. GROUPING COLUMNS: Include the column(s) that were NOT aggregated or pivoted
    <Grouping_Column(s)>,
    
    -- 2. PIVOTED COLUMNS: The actual values from the spreading column that are now headers
    [Value_1], [Value_2], [Value_3], ... 
FROM
    (<SELECT statement that produces the data>) AS SourceQuery
PIVOT
(
    -- 3. AGGREGATING FUNCTION: Define the aggregate function and the value column
    <Aggregate_Function>(<Aggregating_Value>) 
    
    -- 4. SPREADING COLUMN: Define the column containing the values that will become headers
    FOR <Spreading_Column> 
    IN ([Value_1], [Value_2], [Value_3], ...) -- Must explicitly list the values to become columns
) AS PivotTableAlias;
```

-----

## PIVOT Mini Playbook (Realistic Query)

### Scenario: Pivoting Sales by Status

Assume an `Orders` table with columns: `Region`, `OrderStatus` ('Pending', 'Shipped', 'Cancelled'), and `OrderAmount`.

**Goal:** For each **Region**, show the **total `OrderAmount`** split by the three `OrderStatus` values as new columns.

```sql
SELECT
    Region, -- The grouping column
    [Pending],  -- New columns derived from the OrderStatus values
    [Shipped],
    [Cancelled]
FROM
    (
        -- Source Query: Select the minimum required data for the pivot
        SELECT Region, OrderStatus, OrderAmount 
        FROM Orders
    ) AS SourceData
PIVOT
(
    -- 1. Aggregation: SUM the OrderAmount
    SUM(OrderAmount) 
    
    -- 2. Spreading Column: Use OrderStatus values for column headers
    FOR OrderStatus 
    
    -- 3. Pivot List: Explicitly list the values to be used as column names
    IN ([Pending], [Shipped], [Cancelled]) 
) AS SalesPivot
ORDER BY 
    Region;
```

### Result (Conceptual)

| Region | Pending | Shipped | Cancelled |
| :--- | :--- | :--- | :--- |
| East | 500.00 | 8000.00 | 120.00 |
| West | 0.00 | 9500.00 | 0.00 |

-----

## Key PIVOT Limitations

1.  **Static Columns:** The `IN` list **must** be static (known at the time the query is written). You cannot dynamically generate the column names based on the data in the table without using dynamic SQL.
2.  **Single Aggregate:** You can only pivot on a **single aggregate function** (e.g., only `SUM`, not `SUM` and `AVG` simultaneously) unless you first calculate both in the source query.
3.  **SQL Server Only:** This syntax is proprietary and will not work in PostgreSQL, MySQL, or standard ANSI SQL. The manual `SUM(CASE...)` approach is required for portability.
