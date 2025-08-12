---
Difficulty: Easy
---
# SQL Interview Question: Pivoting Sales Data

## Problem Statement

Given a sales_data table with:

- `month`
- `category` (electronics/clothing)
- `amount`

We need to:

1. Pivot the data to show months as rows
2. Transform categories into columns
3. Display amounts in the corresponding cells

## PostgreSQL Solution (Using CROSSTAB)

```SQL
SELECT * FROM CROSSTAB(
    'SELECT
        month,
        category,
        amount
     FROM sales_data
     ORDER BY 1, 2'
) AS ct (
    month VARCHAR,
    clothing NUMERIC,
    electronics NUMERIC
);
```

## Key Components Explained

1. **CROSSTAB Function**:
    - PostgreSQL's pivot function
    - Requires a query returning exactly 3 columns:
        - Row identifier (month)
        - Category (values to become columns)
        - Value (amounts to display)
2. **Output Definition**:
    - `AS ct` defines the result table alias
    - Column definitions specify data types:
        - `month VARCHAR` (text column)
        - `clothing NUMERIC` (first category)
        - `electronics NUMERIC` (second category)
3. **Ordering**:
    - `ORDER BY 1, 2` ensures proper grouping
    - Sorts by month then category

## Alternative Solutions

### SQL Server (Using PIVOT)

```SQL
SELECT
    month,
    [clothing] AS clothing,
    [electronics] AS electronics
FROM
    (SELECT month, category, amount FROM sales_data) AS SourceTable
PIVOT
(
    SUM(amount)
    FOR category IN ([clothing], [electronics])
) AS PivotTable;
```

### Standard SQL (Using CASE)

```SQL
SELECT
    month,
    SUM(CASE WHEN category = 'clothing' THEN amount END) AS clothing,
    SUM(CASE WHEN category = 'electronics' THEN amount END) AS electronics
FROM
    sales_data
GROUP BY
    month;
```

## Key Concepts

1. **Pivoting Data**:
    - Transforming rows to columns
    - Common requirement for reports and dashboards
2. **Database-Specific Functions**:
    - PostgreSQL: CROSSTAB (requires tablefunc extension)
    - SQL Server: PIVOT
    - Oracle: PIVOT
3. **Performance Considerations**:
    - Pivot operations can be resource-intensive
    - Filter source data first if possible
    - Consider materialized views for frequent pivots

This solution demonstrates how to restructure data for better readability and analysis, a common task in business intelligence and reporting scenarios.