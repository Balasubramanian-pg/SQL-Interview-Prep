# Predicting Join Record Counts

## 1. Overview
This document explains a common SQL interview question that tests your understanding of how different `JOIN` types work, especially with duplicate values and `NULL`s. The goal is to predict the number of records returned by `INNER`, `LEFT`, `RIGHT`, `FULL OUTER`, and `CROSS` joins for a given pair of tables.

> [!IMPORTANT]
> A key concept tested here is that in standard SQL, `NULL` is not equal to `NULL`. Therefore, a join condition like `ON T1.ID = T2.ID` will not match rows where the ID is `NULL` in both tables.

## 2. Problem Statement

### 2.1. The Scenario
You are given two tables, `Table1` and `Table2`, with the following data in their `ID` columns.

**Table1:**
| ID |
|----|
| 1  |
| 1  |
| 2  |
| NULL |
| NULL |

**Table2:**
| ID |
|----|
| 1  |
| 3  |
| NULL |

### 2.2. The Question
For the tables above, how many records will be returned for each of the following join operations when joining `ON Table1.ID = Table2.ID`?
-   INNER JOIN
-   LEFT JOIN
-   RIGHT JOIN
-   FULL OUTER JOIN
-   CROSS JOIN

## 3. Setup Script
You can use the following T-SQL script to create the tables and populate them with the sample data to run the queries yourself.

```sql
-- Create the tables
CREATE TABLE Table1 (ID INT);
CREATE TABLE Table2 (ID INT);
GO

-- Insert data into Table1
INSERT INTO Table1 (ID) VALUES
(1),
(1),
(2),
(NULL),
(NULL);
GO

-- Insert data into Table2
INSERT INTO Table2 (ID) VALUES
(1),
(3),
(NULL);
GO

-- Verify the data
SELECT * FROM Table1;
SELECT * FROM Table2;
```

## 4. Solutions and Explanations

### 4.1. INNER JOIN
-   **Explanation**: An `INNER JOIN` returns only the rows where the join condition is met. It finds the intersection of the two tables based on the `ID` column.
-   **Logic**:
    - The first `1` in `Table1` matches the `1` in `Table2`. (1 record)
    - The second `1` in `Table1` also matches the `1` in `Table2`. (1 record)
    - The `2` in `Table1` has no match in `Table2`.
    - The `NULL` values in `Table1` do not match the `NULL` in `Table2` because `NULL` is not equal to `NULL`.
-   **Result**: **2 Records**
-   **SQL Query**:
    ```sql
    SELECT T1.ID, T2.ID
    FROM Table1 T1
    INNER JOIN Table2 T2 ON T1.ID = T2.ID;
    ```

### 4.2. LEFT JOIN
-   **Explanation**: A `LEFT JOIN` returns all records from the left table (`Table1`) and the matched records from the right table (`Table2`). If there is no match, the columns from the right table will be `NULL`.
-   **Logic**: It starts with the result of the `INNER JOIN` and adds the non-matching records from the left table.
    - Matching records from `INNER JOIN`: 2 records.
    - Non-matching records from `Table1`: `2`, `NULL`, `NULL` (3 records).
-   **Result**: 2 (matching) + 3 (non-matching left) = **5 Records**
-   **SQL Query**:
    ```sql
    SELECT T1.ID, T2.ID
    FROM Table1 T1
    LEFT JOIN Table2 T2 ON T1.ID = T2.ID;
    ```

### 4.3. RIGHT JOIN
-   **Explanation**: A `RIGHT JOIN` returns all records from the right table (`Table2`) and the matched records from the left table (`Table1`). If there is no match, the columns from the left table will be `NULL`.
-   **Logic**: It starts with the result of the `INNER JOIN` and adds the non-matching records from the right table.
    - Matching records from `INNER JOIN`: 2 records.
    - Non-matching records from `Table2`: `3`, `NULL` (2 records).
-   **Result**: 2 (matching) + 2 (non-matching right) = **4 Records**
-   **SQL Query**:
    ```sql
    SELECT T1.ID, T2.ID
    FROM Table1 T1
    RIGHT JOIN Table2 T2 ON T1.ID = T2.ID;
    ```

### 4.4. FULL OUTER JOIN
-   **Explanation**: A `FULL OUTER JOIN` returns all records when there is a match in either the left or the right table. It effectively combines the results of `LEFT JOIN` and `RIGHT JOIN`.
-   **Logic**: This join includes all matching records, all non-matching records from the left, and all non-matching records from the right.
    - Matching records (`INNER JOIN`): 2 records.
    - Non-matching records from `Table1`: `2`, `NULL`, `NULL` (3 records).
    - Non-matching records from `Table2`: `3`, `NULL` (2 records).
-   **Result**: 2 (matching) + 3 (non-matching left) + 2 (non-matching right) = **7 Records**
-   **SQL Query**:
    ```sql
    SELECT T1.ID, T2.ID
    FROM Table1 T1
    FULL OUTER JOIN Table2 T2 ON T1.ID = T2.ID;
    ```

### 4.5. CROSS JOIN
-   **Explanation**: A `CROSS JOIN` produces a Cartesian product, meaning it returns every row from the left table paired with every row from the right table. It does not use an `ON` clause.
-   **Logic**: The total number of records is the number of rows in `Table1` multiplied by the number of rows in `Table2`.
    - Rows in `Table1`: 5
    - Rows in `Table2`: 3
-   **Result**: 5 * 3 = **15 Records**
-   **SQL Query**:
    ```sql
    SELECT T1.ID, T2.ID
    FROM Table1 T1
    CROSS JOIN Table2 T2;
    ```

## 5. Summary of Results

| Join Type       | Number of Records Returned |
|-----------------|----------------------------|
| INNER JOIN      | 2                          |
| LEFT JOIN       | 5                          |
| RIGHT JOIN      | 4                          |
| FULL OUTER JOIN | 7                          |
| CROSS JOIN      | 15                         |
