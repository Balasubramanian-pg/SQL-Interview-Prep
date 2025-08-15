---
Company:
  - Amazon
Difficulty: Intermediate
Created:
Status:
Category:
Sub category:
Question Link:
---
Of course! This is an excellent, practical query for tracking performance over time. It demonstrates grouping by multiple columns and using a date function within the grouping.

Here is the complete breakdown.

***

### 1. The Question

The business question being asked by this SQL query is:

**"What is the average star rating for each product on a monthly basis? Show the results ordered by month and then by product ID."**

This query is perfect for generating a report to see if a product's ratings are trending up or down over time.

---

### 2. Table Schema

The query references a single table, `reviews`. A plausible schema for this table would be:

```sql
-- Table: reviews
-- Stores a record for every product review submitted by a user.
CREATE TABLE reviews (
    review_id    INT PRIMARY KEY,       -- Unique identifier for the review
    user_id      INT NOT NULL,          -- The ID of the user who wrote the review
    submit_date  TIMESTAMP NOT NULL,    -- The date and time the review was submitted
    product_id   INT NOT NULL,          -- The ID of the product being reviewed
    stars        INT NOT NULL           -- The star rating given (e.g., 1 to 5)
);

-- An index on the grouping columns can improve performance.
CREATE INDEX idx_reviews_product_date ON reviews (product_id, submit_date);
```

---

### 3. Structured SQL Query (Method 1: Using `DATE_PART`)

This is your original query, formatted for clarity. This syntax is specific to databases like PostgreSQL and Redshift.

```sql
SELECT
    EXTRACT(MONTH FROM submit_date) AS mth, -- Using EXTRACT is standard SQL
    product_id AS product,
    ROUND(AVG(stars), 2) AS avg_stars
FROM
    reviews
GROUP BY
    mth,
    product
ORDER BY
    mth ASC,
    product ASC;
```
*Note: I've replaced `DATE_PART('month', submit_date)` with the standard SQL equivalent `EXTRACT(MONTH FROM submit_date)` and adjusted the `GROUP BY` and `ORDER BY` clauses to use the column aliases, which most modern databases support. The logic is identical.*

---

### 4. Explanation of the Query

This query aggregates review data to produce a monthly summary for each product.

1.  **`FROM reviews`**
    *   The query targets the `reviews` table as its data source.

2.  **`GROUP BY EXTRACT(MONTH FROM submit_date), product_id`** (or `GROUP BY mth, product`)
    *   This is the core of the aggregation logic. It creates groups, or "buckets," for each unique combination of month and product ID.
    *   For example, all reviews for `product_id = 101` submitted in January (month 1) go into one bucket. All reviews for `product_id = 101` submitted in February (month 2) go into another. All reviews for `product_id = 202` in January go into a third bucket, and so on.

3.  **`SELECT ...`**
    *   The `SELECT` clause is then executed on each of these buckets.
    *   `EXTRACT(MONTH FROM submit_date) AS mth`: It extracts the month number (1-12) from the `submit_date` and assigns it to the `mth` column.
    *   `product_id AS product`: It selects the `product_id` for that bucket.
    *   `ROUND(AVG(stars), 2) AS avg_stars`: It calculates the average of the `stars` column for all reviews *within that specific bucket*, and then rounds the result to two decimal places.

4.  **`ORDER BY mth ASC, product ASC`**
    *   Finally, the resulting summary rows are sorted. The `ORDER BY` clause first sorts all rows by the month number (`mth`) in ascending order. For rows that have the same month, it then sorts them by the product ID (`product`). This creates a clean, chronological, and well-organized report.

---

### 5. Another SQL Method (Method 2: Using `DATE_TRUNC`)

A slightly different but very powerful way to group by time periods is with the `DATE_TRUNC` function (available in PostgreSQL, BigQuery, etc.). This method retains the full date context (including the year), which is often more useful for real-world analysis.

```sql
SELECT
    DATE_TRUNC('month', submit_date)::DATE AS review_month,
    product_id AS product,
    ROUND(AVG(stars), 2) AS avg_stars
FROM
    reviews
GROUP BY
    review_month,
    product
ORDER BY
    review_month ASC,
    product ASC;
```

---

### 6. Explanation of the Alternative Query

This method groups data by the first day of the month, which is a more robust way to handle time-series data that spans multiple years.

1.  **`DATE_TRUNC('month', submit_date)::DATE AS review_month`**
    *   This is the key change. The `DATE_TRUNC` function "truncates" a date or timestamp to a specified precision.
    *   `DATE_TRUNC('month', submit_date)` takes any date like `2023-01-15 10:30:00` and truncates it down to the first moment of that month, resulting in `2023-01-01 00:00:00`.
    *   The `::DATE` cast then converts this timestamp to just the date: `2023-01-01`.
    *   This provides a single, consistent value for every review submitted in a given month and year.

2.  **`GROUP BY review_month, product`**
    *   The query now groups by this truncated date (`review_month`) and the `product_id`. This means all reviews for product 101 from January 2023 will be in one group, while reviews from January 2024 will be in a *different* group.

**Advantages of this method:**
*   **Handles Multiple Years:** It naturally separates data from different years (e.g., Jan 2022 vs. Jan 2023), which the original `DATE_PART('month', ...)` method does not. The original method would combine all January data, regardless of the year.
*   **Clarity:** The output `review_month` column (`2023-01-01`) is more informative than just a month number (`1`), as it retains the year context.
*   **Performance:** `DATE_TRUNC` can sometimes be optimized better by the database than `DATE_PART` or `EXTRACT` in a `GROUP BY` clause.

The rest of the query (`AVG`, `ROUND`, `ORDER BY`) functions identically but operates on these more specific, year-aware groups.