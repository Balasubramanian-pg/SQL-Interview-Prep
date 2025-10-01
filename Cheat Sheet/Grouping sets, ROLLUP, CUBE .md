## GROUPING SETS, ROLLUP, and CUBE Cheat Sheet Table

| Function | Purpose | How It Works | Key Result |
| :--- | :--- | :--- | :--- |
| **GROUPING SETS** | **Custom Subtotals** | Explicitly defines every combination of grouping columns for which an aggregate is needed. | Generates subtotals for *only* the specific combinations listed (including the grand total). |
| **ROLLUP** | **Hierarchical Subtotals** | Generates subtotals for every possible prefix of the grouping columns, creating a hierarchy. | Generates subtotals for level 1, level 1 + level 2, ..., and the grand total. Order matters\! |
| **CUBE** | **All Combinations** | Generates subtotals for *all possible combinations* of the grouping columns. | Generates a complete cross-tabular report showing subtotals for every dimension and the grand total. |
| **GROUPING()** | **Identifying Subtotals** | A function used to indicate if a column in the result set is part of the grouping (0) or represents a subtotal (1). | Helps distinguish aggregated rows from regular detail rows in the output. |

-----

## GROUPING SETS, ROLLUP, and CUBE Mini Playbook (Realistic Queries)

These snippets illustrate how to generate complex subtotals from a `Sales` table with columns: `year`, `country`, and `product`.

### 1\. GROUPING SETS (Custom Totals)

**Use Case:** Calculate sales aggregated by **(Year, Country)** and separately by **Product**, plus a **Grand Total**, all in one query.

```sql
SELECT
    year,
    country,
    product,
    SUM(sale_amount) AS total_sales
FROM
    Sales
GROUP BY
    GROUPING SETS (
        (year, country), -- Group 1: By year and country
        (product),       -- Group 2: By product only
        ()               -- Group 3: The grand total
    );
```

-----

### 2\. ROLLUP (Hierarchical Totals)

**Use Case:** Calculate sales for a fixed hierarchy: **(Country, Year)**, then subtotal by **Country**, and finally the **Grand Total**.

```sql
SELECT
    country,
    year,
    SUM(sale_amount) AS total_sales,
    -- GROUPING(country) = 1 means it's a subtotal row for country
    GROUPING(country) AS is_country_subtotal
FROM
    Sales
GROUP BY
    ROLLUP (country, year);
    
/*
ROLLUP (A, B) is equivalent to GROUPING SETS ( (A, B), (A), () )
*/
```

-----

### 3\. CUBE (All Combinations)

**Use Case:** Calculate sales for all possible combinations of **(Year, Product)**, including subtotals for Year, Product, and the Grand Total.

```sql
SELECT
    year,
    product,
    SUM(sale_amount) AS total_sales,
    -- Example of using GROUPING() to label the rows
    CASE
        WHEN GROUPING(year) = 1 AND GROUPING(product) = 1 THEN 'Grand Total'
        WHEN GROUPING(year) = 1 THEN 'Product Subtotal'
        WHEN GROUPING(product) = 1 THEN 'Year Subtotal'
        ELSE 'Detail Row'
    END AS total_type
FROM
    Sales
GROUP BY
    CUBE (year, product);
    
/*
CUBE (A, B) is equivalent to GROUPING SETS ( (A, B), (A), (B), () )
*/
```
