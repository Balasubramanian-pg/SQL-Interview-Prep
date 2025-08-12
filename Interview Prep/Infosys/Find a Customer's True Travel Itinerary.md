# Find a Customer's True Travel Itinerary

## 1. Overview
This document explains how to solve a complex SQL interview question that involves piecing together a customer's multi-leg flight journey to determine their initial origin and final destination. This problem tests your ability to think logically about data relationships and use self-joins to find records that do not have a corresponding link in the same table.

## 2. Problem Statement

### 2.1. The Scenario
You are given a `flights` table that contains individual flight legs for different customers. A customer's full journey may consist of multiple connected flights.

**flights:**
| customer_id | flight_id | origin    | destination |
|-------------|-----------|-----------|-------------|
| 1           | FL101     | Delhi     | Hyderabad   |
| 1           | FL102     | Hyderabad | Kochi       |
| 1           | FL103     | Kochi     | Mangalore   |
| 2           | FL201     | Mumbai    | Varanasi    |
| 2           | FL202     | Varanasi  | Delhi       |

### 2.2. The Requirement
Write a SQL query to find the initial starting point (true origin) and the final destination for each customer's complete trip.

-   **Customer 1's journey**: Delhi -> Hyderabad -> Kochi -> Mangalore. The true origin is **Delhi** and the final destination is **Mangalore**.
-   **Customer 2's journey**: Mumbai -> Varanasi -> Delhi. The true origin is **Mumbai** and the final destination is **Delhi**.

**Expected Output:**
| customer_id | origin | final_destination |
|-------------|--------|-------------------|
| 1           | Delhi  | Mangalore         |
| 2           | Mumbai | Delhi             |

## 3. Setup Script
You can use the following T-SQL script to create the table and populate it with sample data.

```sql
-- Create the table
CREATE TABLE flights (
    customer_id INT,
    flight_id VARCHAR(10),
    origin VARCHAR(50),
    destination VARCHAR(50)
);
GO

-- Insert sample data
INSERT INTO flights (customer_id, flight_id, origin, destination) VALUES
(1, 'FL101', 'Delhi', 'Hyderabad'),
(1, 'FL102', 'Hyderabad', 'Kochi'),
(1, 'FL103', 'Kochi', 'Mangalore'),
(2, 'FL201', 'Mumbai', 'Varanasi'),
(2, 'FL202', 'Varanasi', 'Delhi');
GO

-- Verify the initial data
SELECT * FROM flights;
```

## 4. Solution and Explanation
The solution involves a three-step process: first, find the true starting points; second, find the final endpoints; and third, join these results together. This is achieved using self-joins.

### 4.1. Step 1: Finding the True Origin
A true origin is a city that appears in a customer's `origin` column but **never** in their `destination` column. This indicates it is a starting point and not an intermediate layover.

-   **Logic**: We perform a `LEFT JOIN` on the table with itself (a self-join), connecting `F1.origin` to `F2.destination` for the same customer. If a match is not found (`F2.customer_id IS NULL`), it means the origin in `F1` was never a destination, making it the true starting point.

### 4.2. Step 2: Finding the Final Destination
Symmetrically, a final destination is a city that appears in a customer's `destination` column but **never** in their `origin` column for a subsequent flight.

-   **Logic**: We perform another `LEFT JOIN` self-join, this time connecting `F1.destination` to `F2.origin` for the same customer. If a match is not found (`F2.customer_id IS NULL`), the destination in `F1` is the final stop.

### 4.3. Step 3: Combining the Results
We use Common Table Expressions (CTEs) to store the results from the first two steps. Then, we join these CTEs on `customer_id` to consolidate the true origin and final destination into a single row for each customer.

## 5. The Complete Solution (Using CTEs)
This query implements the three-step logic in a clean, readable format using CTEs.

```sql
-- CTE to find the true origins
WITH Origins AS (
    SELECT
        F1.customer_id,
        F1.origin
    FROM
        flights AS F1
    LEFT JOIN
        flights AS F2 ON F1.customer_id = F2.customer_id AND F1.origin = F2.destination
    WHERE
        F2.customer_id IS NULL
),
-- CTE to find the final destinations
Destinations AS (
    SELECT
        F1.customer_id,
        F1.destination
    FROM
        flights AS F1
    LEFT JOIN
        flights AS F2 ON F1.customer_id = F2.customer_id AND F1.destination = F2.origin
    WHERE
        F2.customer_id IS NULL
)
-- Final SELECT to join the origins and destinations
SELECT
    o.customer_id,
    o.origin,
    d.destination AS final_destination
FROM
    Origins AS o
JOIN
    Destinations AS d ON o.customer_id = d.customer_id;
```
