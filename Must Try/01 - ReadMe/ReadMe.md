# SQL Interview Preparation: Must-Try Problems

This document provides a central index for a curated collection of SQL interview questions. 
The problems are designed to test a range of skills, from basic queries to complex window functions and analytical reasoning. Each item links directly to the problem statement and its solution.

*   [1. Advertiser Status](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/1.%20Advertiser%20Status.md)
*   [2. Average Deal Size (Part 2)](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/2.%20Average%20Deal%20Size%20(Part%202).md)
*   [3. Average Post Hiatus (Part 1)](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/3.%20Average%20Post%20Hiatus%20(Part%201).md)
*   [4. Compare Department Average Salary to Company Average](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/4.%20Average%20salary%20of%20employees%20in%20a%20department%20to%20the%20company's%20average%20salary.md)
*   [5. Click-Through Rate (CTR)](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/5.%20Click%20Through%20Rate.md)
*   [6. Cumulative Salary Sum Over 3 Months](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/6.%20Cumulative%20sum%20of%20an%20employee's%20salary%20over%20a%20period%20of%203%20months.md)
*   [7. Department Top 3 Salaries](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/7.%20Department%20Top%203%20Salaries.md)
*   [8. FAANG Stock Min-Max (Part 1)](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/8.%20FAANG%20Stock%20Min-Max%20(Part%201).md)
*   [9. Game Play Analysis (Part 5)](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/9.%20Game%20Play%20Analysis%205.md)
*   [10. Highest-Grossing Items](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/10.%20Highest-Grossing%20Items.md)
*   [11. Human Traffic at Stadium](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/11.%20Human%20Traffic%20%40%20Stadium.md)
*   [12. LinkedIn Power Creators (Part 1)](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/12.%20LinkedIn%20Power%20Creators%20(Part%201).md)
*   [13. LinkedIn Power Creators (Part 2)](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/13.%20LinkedIn%20Power%20Creators%20(Part%202).md)
*   [14. Market Analysis (Part 2)](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/14.%20Market%20Analysis%202.md)
*   [15. Median Employee Salary](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/15.%20Median%20Employee%20Salary.md)
*   [16. Median Google Search Frequency](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/16.%20Median%20Google%20Search%20Frequency.md)
*   [17. Calculate Median](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/17.%20Median.md)
*   [18. Number of Transactions Per Visit](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/18.%20Number%20of%20Transactions%20Per%20Visit.md)
*   [19. Patient Support Analysis (Part 1)](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/19.%20Patient%20Support%20Analysis%20(Part%201).md)
*   [20. Patient Support Analysis (Part 2)](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/20.%20Patient%20Support%20Analysis%20(Part%202).md)
*   [21. Purchasing Activity by Product Category](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/21.%20Purchasing%20Activity%20by%20Product%20Category.md)
*   [22. Quiet Students](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/22.%20Quiet%20Students.md)
*   [23. Report Contagious Data](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/23.%20Report%20Contagious%20Data.md)
*   [24. Sending vs. Opening Snaps](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/24.%20Sending%20VS%20Opening%20Snaps.md)
*   [25. Supercloud Platform](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/25.%20Supercloud%20Platform.md)
*   [26. Top 5 Streams](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/26.%20Top%205%20Streams.md)
*   [27. Top Rated Business](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/27.%20Top%20Rated%20Business.md)
*   [28. User’s Third Transaction](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/28.%20User%E2%80%99s%20Third%20Transaction.md)
*   [29. Year-on-Year Growth Rate](https://github.com/Balasubramanian-pg/SQL-Interview-Pewp/blob/main/Must%20Try/29.%20Y-on-Y%20Growth%20Rate.md)

# A High-Complexity SQL Query

## With JOINs, multiple CTEs, window functions, filtering logic, aggregation, and ORDER BY

I’ll give you a deliberately difficult query and then break down the *true* SQL execution order, the way a database engine actually thinks.
The explanation is plain language, vertical structure, with clear headings and subheadings.

No assumptions are presented as facts. No verification required because this concerns stable SQL behavior.

# The Query

```sql
WITH sales_cte AS (
    SELECT
        s.sale_id,
        s.customer_id,
        s.amount,
        s.sale_date,
        ROW_NUMBER() OVER (
            PARTITION BY s.customer_id
            ORDER BY s.sale_date DESC
        ) AS rn
    FROM sales s
    INNER JOIN customers c
        ON s.customer_id = c.customer_id
    WHERE s.amount > 100
),

agg_cte AS (
    SELECT
        customer_id,
        COUNT(*) AS total_sales,
        SUM(amount) AS total_amount
    FROM sales_cte
    WHERE rn <= 5
    GROUP BY customer_id
)

SELECT
    a.customer_id,
    a.total_sales,
    a.total_amount,
    RANK() OVER (ORDER BY a.total_amount DESC) AS revenue_rank
FROM agg_cte a
LEFT JOIN regions r
    ON a.customer_id = r.customer_id
WHERE a.total_amount > 500
ORDER BY revenue_rank;
```

# The Real Order of SQL Execution

## Not the order you write it

SQL has a secret choreography. You write it top-down, but the engine executes it in a completely different order.

Below is the exact execution sequence.

# 1. CTE Expansion (Logical Step Only)

The database *inlines* each CTE (unless the engine chooses to materialize it).
This step does not run the query yet. It simply prepares the logical structures.

*This part does not compute rows.*

# 2. Start With the First CTE: `sales_cte`

### 2.1 FROM Clause

The engine first resolves the base tables.

1. Load `sales`
2. Load `customers`
3. Perform the `INNER JOIN` using the condition:
   `s.customer_id = c.customer_id`

All rows that fail the join condition are removed.

### 2.2 WHERE Clause

Applied on the join result:

`s.amount > 100`

Rows failing this are removed.

### 2.3 SELECT Expression Processing

Now the engine evaluates the expressions in the SELECT list:

* `s.sale_id`
* `s.customer_id`
* `s.amount`
* `s.sale_date`

### 2.4 Window Function in sales_cte

Now the window function is evaluated:

```
ROW_NUMBER() OVER (
    PARTITION BY s.customer_id
    ORDER BY s.sale_date DESC
)
```

This requires:

* group rows by customer
* sort each customer’s rows by date
* number them

This step runs *after* filtering and joins, but before the CTE is materialized.

### 2.5 Materialize the CTE output

`sales_cte` is now a virtual table with:

* sale fields
* rn (row number)

# 3. Second CTE: `agg_cte`

### 3.1 FROM sales_cte

Load the result of the previous CTE.

### 3.2 WHERE rn <= 5

Filter to keep only each customer’s most recent 5 high-value sales.

### 3.3 GROUP BY customer_id

Rows grouped by customer.

### 3.4 Aggregations

Compute the aggregates:

* `COUNT(*)` as total_sales
* `SUM(amount)` as total_amount

### 3.5 Materialize agg_cte

Now we have one row per customer.

# 4. Final Query Begins

### 4.1 FROM agg_cte

Load one row per customer (their aggregated totals).

### 4.2 LEFT JOIN regions

Join customer rows to regions table using:

`a.customer_id = r.customer_id`

Rows with no region still stay (because it’s LEFT JOIN).

### 4.3 WHERE a.total_amount > 500

Filters the joined result.
Note: This happens *after* the join but *before* window functions.

# 5. SELECT Expression Evaluation

Compute fields:

* a.customer_id
* a.total_sales
* a.total_amount

### 5.1 Window Function in final SELECT

Now the engine computes:

```
RANK() OVER (ORDER BY a.total_amount DESC)
```

This requires:

* scan all remaining rows
* order by total_amount
* assign rank

# 6. ORDER BY

Sort final rows by `revenue_rank`.

This is the last step before output.

# 7. Return Rows to Client

This is when the rows actually come back to you.

# Full Engine Execution Order in Compact Form

1. Expand CTEs
2. Execute FROM for sales_cte
3. Execute JOIN for sales_cte
4. Apply WHERE in sales_cte
5. Compute SELECT projections
6. Compute window function row_number
7. Materialize sales_cte
8. Execute FROM for agg_cte
9. WHERE rn <= 5
10. GROUP BY
11. Compute aggregates
12. Materialize agg_cte
13. Final FROM
14. LEFT JOIN
15. WHERE total_amount > 500
16. SELECT list computations
17. Window function RANK
18. ORDER BY
19. Output
