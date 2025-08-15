# Calculating Average Deal Size (Part 2)

## 1. Overview
> [!NOTE]
> This document analyzes a SQL query designed to answer the business question: **"What is the average deal size across all of our contracts, rounded to two decimal places?"** A "deal size" is defined as the `yearly_seat_cost` multiplied by the `num_seats`.

## 2. Table Schema
The query references a single table, `contracts`. A plausible schema for this table is as follows:

```sql
-- Table: contracts
-- Stores information about each sales contract with a customer.
CREATE TABLE contracts (
    contract_id       INT PRIMARY KEY,        -- Unique identifier for the contract
    customer_id       INT NOT NULL,           -- ID of the customer for this contract
    num_seats         INT NOT NULL,           -- The number of user seats purchased
    yearly_seat_cost  DECIMAL(10, 2) NOT NULL,-- The cost per seat per year
    start_date        DATE,                   -- The start date of the contract
    status            VARCHAR(20)             -- e.g., 'Active', 'Expired', 'Cancelled'
);
```
> [!TIP]
> Using the `DECIMAL(10, 2)` data type is a best practice for storing currency values. It prevents the floating-point precision errors that can occur with types like `FLOAT` or `REAL`.

## 3. Method 1: Using `AVG()`
This method is the most direct and readable way to calculate an average.

### 3.1. SQL Query
```sql
SELECT
    ROUND(AVG(yearly_seat_cost * num_seats), 2) AS average_deal_size
FROM
    contracts;
```
> [!NOTE]
> Formatting SQL queries with indentation and newlines, as shown above, significantly improves readability and maintainability, especially for more complex queries.

### 3.2. Explanation
The query calculates a single aggregate value for the entire `contracts` table. The calculation happens from the inside out:
*   **`FROM contracts`**: The query targets the `contracts` table as its data source.
*   **`yearly_seat_cost * num_seats`**: For each row, the database performs this multiplication to compute the total value ("deal size") for that individual contract.
*   **`AVG(...)`**: The `AVG()` aggregate function takes all individual deal size values and calculates their statistical average.
> [!TIP]
> SQL has a logical order of operations. The calculation within the `AVG()` function (`yearly_seat_cost * num_seats`) is performed for each row *before* the results are passed to the aggregation function.
*   **`ROUND(..., 2)`**: The result of `AVG()` is passed to the `ROUND()` function, which rounds the number to two decimal places.
*   **`AS average_deal_size`**: The final value is placed into a column named `average_deal_size`.
> [!IMPORTANT]
> Because there is no `GROUP BY` clause, the `AVG()` aggregate function collapses the entire result set into a single row, providing one summary value for the whole table.

## 4. Method 2: Using `SUM()` and `COUNT()`
An alternative way to calculate an average is to manually perform the underlying math: `Average = Total Sum / Total Count`.

### 4.1. SQL Query
```sql
SELECT
    ROUND(
        SUM(yearly_seat_cost * num_seats) / COUNT(*),
        2
    ) AS average_deal_size
FROM
    contracts;
```

### 4.2. Explanation
This query arrives at the exact same result by breaking the average calculation into its fundamental parts:
*   **`SUM(yearly_seat_cost * num_seats)`**: This calculates the deal size for each row and then adds them all together, providing the total value of all contracts combined.
*   **`COUNT(*)`**: This counts the total number of rows (i.e., total number of contracts) in the table.
> [!WARNING]
> If the `contracts` table is empty, `COUNT(*)` will return `0`. Attempting to divide by zero will cause an error in most SQL database systems. The `AVG()` function handles this more gracefully by returning `NULL`.
*   **`SUM(...) / COUNT(*)`**: The total value of all contracts is divided by the total number of contracts. This is the mathematical definition of an average.
> [!CAUTION]
> Be mindful of integer division. In some SQL dialects (like older versions of SQL Server or PostgreSQL), if both the numerator and denominator were integers, the result would be an integer (e.g., `7 / 2 = 3`). Since our calculation involves a `DECIMAL` type, this is not an issue here, but it's a common pitfall to watch for.
*   **`ROUND(..., 2) AS average_deal_size`**: As in the first method, the result is rounded and aliased.

## 5. Method Comparison
> [!IMPORTANT]
> Both methods are functionally identical and will likely have the same performance in most database systems, as the query optimizer often translates `AVG(expression)` into `SUM(expression) / COUNT(expression)` internally.

> [!TIP]
> For calculating a simple average, `AVG()` is almost always the preferred method. It is more concise, more readable, and clearly states the intent of the query.

> [!NOTE]
> While `AVG()` is simpler, the `SUM()`/`COUNT()` method can be useful if you need to display the total sum and the total count as separate columns in the same query, in addition to the calculated average.
