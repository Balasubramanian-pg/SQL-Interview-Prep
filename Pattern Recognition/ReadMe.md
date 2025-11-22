# The Discipline of SQL Interview Preparation

The study of SQL reveals a deeper encounter with order itself. Every query becomes a small act of structuring chaos, and every pattern reflects a principle that governs the movement from raw complexity to usable clarity. This roadmap outlines thirty distinct problem types that appear in interviews not by accident but because they expose a candidate’s ability to impose structure, recognize patterns, and act responsibly within a relational system.

## **I. Core Pattern Recognition (1–10)**

These patterns test whether an individual can identify regularity within variation. Each represents a fundamental mode of analytical reasoning.

1. **Finding Top N Records per Group**
   The task examines one’s capacity to distinguish hierarchy within categories, imposing ranking where none is explicitly provided.

2. **Calculating Running Totals**
   Running totals reflect accumulation through time, demonstrating how present states emerge from prior actions.

3. **Identifying Gaps in Sequences**
   Gaps reveal discontinuity. They are signs of missing information and demand recognition before responsible decision making.

4. **Finding Islands of Consecutive Values**
   Islands show coherence within apparent randomness. This pattern reveals the hidden boundaries between order and fragmentation.

5. **Calculating Moving Averages**
   Moving averages smooth volatility, allowing trends to become intelligible without distortion from noise.

6. **Pivoting Data**
   Pivoting demonstrates the ability to reorient perspective, transforming detail-oriented structure into comparative form.

7. **Unpivoting Data**
   Unpivoting restores underlying simplicity by returning overly complex structures to a normalized foundation.

8. **Finding Duplicate Records**
   Duplicates indicate disorder in the system. Identifying them is a step toward restoring accuracy and integrity.

9. **Navigating Hierarchical Data**
   Hierarchies represent nested order. Understanding them means understanding how systems embed meaning at multiple levels.

10. **Identifying First and Last Events per Group**
    These boundaries provide orientation. They define beginnings and conclusions within a dataset’s internal narrative.

## **II. Relational Logic and Joins (11–16)**

Joins examine whether one understands interconnectedness. Data rarely stands alone. Meaning emerges in relation.

11. **Inner vs Outer Join Structures**
    Each join type reflects a philosophical stance on what counts as relevant information.

12. **Anti-Joins and Semi-Joins**
    These patterns identify absence, a necessary complement to presence in any coherent system.

13. **Unmatched Records**
    Unmatched data signals broken relationships, demanding careful investigation.

14. **Multi-table Joins with Constraints**
    Such queries test whether complexity can be navigated without losing orientation.

15. **Conditional Joins**
    Conditional logic reflects nuanced associations beyond simple equivalence.

16. **Self Joins for Pattern Detection**
    A self-join forces reflection: a dataset must understand itself to reveal its deeper structure.

## **III. Window Function Mastery (17–22)**

Window functions provide a framework for examining an element in context, not isolation.

17. **Ranking Patterns**
    Rankings impose order where competition or priority must be discerned.

18. **Lag and Lead**
    These functions allow comparison across time, revealing causality and progression.

19. **Sessionization**
    Sessions mark boundaries of activity, distinguishing one coherent period from another.

20. **Percentiles and Distribution Analysis**
    Distributions reveal natural variation. Percentiles impose structure upon that variation.

21. **Frame Clauses and Moving Windows**
    Frames define scope, limiting attention to what is immediately relevant.

22. **Cumulative Distinct Counts**
    Distinct accumulation highlights uniqueness within repetition, a subtle but essential observation.

## **IV. Data Integrity and Cleaning Patterns (23–27)**

Disorder in data reflects disorder in systems. Cleaning restores reliability.

23. **Imputing Missing Values**
    Missing data must be treated with caution, neither ignored nor fabricated irresponsibly.

24. **Standardizing Categories**
    Standardization transforms multiplicity into unity, enabling consistent interpretation.

25. **Outlier Detection**
    Outliers mark deviation. Understanding them prevents misdirection.

26. **Deduplication with Window Functions**
    Deduplication eliminates redundancy and improves the coherence of downstream analysis.

27. **Referential Integrity Validation**
    A system is only as strong as the relationships that bind it. These must be maintained.

## **V. Real-world Analytical Patterns (28–30)**

These topics reflect scenarios where technical clarity meets organizational responsibility.

28. **Revenue Recognition Patterns**
    Revenue must be understood through time, obligation, and fulfillment. The pattern is ethical as much as technical.

29. **Churn and Retention Calculations**
    These analyses reveal the stability of a system’s relationships and the cost of neglect.

30. **Cohort Analysis**
    Cohorts expose developmental trajectories. They show how different groups evolve within the same environment.

## **Closing Reflection**

This roadmap outlines not only technical exercises but invitations to think more clearly. SQL becomes a discipline when approached with seriousness, responsibility, and an interest in uncovering the underlying order within complexity. Mastery emerges when one recognizes that every query is a small argument for structure in a world that naturally tends toward disorder.

What to expect for **SQL queries in data analysis, user queries, testing, and low-medium complexity ETL logic**, tailored to your background (B.Com + MBA in Business Analytics) and job search context (3.5 years of experience, Power BI/Tally/Excel, C-suite reporting).

### **1. Data Analysis Queries**
**Purpose:** Extract insights, validate hypotheses, or answer ad-hoc business questions.
**Expectations:**
- **Aggregations & Grouping** (e.g., sales trends, customer segmentation).
- **Joins** (merging tables like `transactions` + `customer_demographics`).
- **Window Functions** (ranking, running totals, YoY growth).
- **CTEs/Subqueries** (breaking complex logic into readable steps).

#### **Example Queries**
```sql
-- 1. Monthly Revenue Growth (YoY)
WITH MonthlyRevenue AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        SUM(revenue) AS revenue,
        LAG(SUM(revenue), 12) OVER (ORDER BY DATE_TRUNC('month', order_date)) AS prev_year_revenue
    FROM orders
    GROUP BY 1
)
SELECT
    month,
    revenue,
    prev_year_revenue,
    (revenue - prev_year_revenue) / prev_year_revenue * 100 AS yoy_growth_pct
FROM MonthlyRevenue
ORDER BY month;

-- 2. Customer Lifetime Value (CLV) by Cohort
SELECT
    customer_cohort,
    AVG(total_spend) AS avg_clv,
    COUNT(DISTINCT customer_id) AS customers
FROM (
    SELECT
        customer_id,
        DATE_TRUNC('year', MIN(order_date)) AS customer_cohort,
        SUM(amount) AS total_spend
    FROM orders
    GROUP BY 1, 2
)
GROUP BY 1
ORDER BY 1;
```

> **🔹 Why It Matters:**
> - Tests your ability to **translate business questions into SQL** (e.g., "Why did revenue drop in Q3?").
> - Expect **follow-ups** like:
>   - *"How would you handle missing data in the `order_date` column?"*
>   - *"Can you modify this to exclude refunded orders?"*

### **2. User Queries (Ad-Hoc Requests)**
**Purpose:** Quick, accurate responses to stakeholder asks (e.g., "Show me top 10 products by margin in Karnataka").
**Expectations:**
- **Filtering** (`WHERE`, `HAVING`).
- **Sorting/Limiting** (`ORDER BY`, `LIMIT`).
- **Conditional Logic** (`CASE WHEN`).
- **Performance Awareness** (avoiding `SELECT *` on large tables).

#### **Example Queries**
```sql
-- 1. Top 10 High-Margin Products in Karnataka
SELECT
    product_id,
    product_name,
    SUM(revenue - cost) AS gross_margin
FROM sales
WHERE state = 'Karnataka'
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 10;

-- 2. Customer Churn Risk (Inactive for 90+ days)
SELECT
    customer_id,
    MAX(order_date) AS last_order_date,
    CASE
        WHEN DATEDIFF(day, MAX(order_date), CURRENT_DATE) > 90 THEN 'High Risk'
        ELSE 'Active'
    END AS churn_status
FROM orders
GROUP BY 1;
```

> **🔹 Pro Tip:**
> - **Always clarify requirements** before writing:
>   - *"Should ‘margin’ be absolute or percentage?"*
>   - *"Do we exclude cancelled orders?"*
> - Use **temp tables** for intermediate steps if the query is complex.

### **3. Testing Queries**
**Purpose:** Validate data quality, ETL outputs, or edge cases.
**Expectations:**
- **Data Validation** (e.g., check for duplicates, nulls, referential integrity).
- **Edge Cases** (e.g., "What if a customer has no orders?").
- **Assertions** (e.g., "Ensure all `order_ids` in `payments` exist in `orders`").

#### **Example Queries**
```sql
-- 1. Find Orphaned Records (Payments without Orders)
SELECT p.payment_id
FROM payments p
LEFT JOIN orders o ON p.order_id = o.order_id
WHERE o.order_id IS NULL;

-- 2. Test ETL: Verify Row Counts Post-Load
SELECT
    (SELECT COUNT(*) FROM source_table) AS source_count,
    (SELECT COUNT(*) FROM target_table) AS target_count,
    CASE
        WHEN (SELECT COUNT(*) FROM source_table) = (SELECT COUNT(*) FROM target_table)
        THEN '✅ Match'
        ELSE '❌ Mismatch'
    END AS status;

-- 3. Check for Negative Revenue (Data Anomaly)
SELECT order_id, revenue
FROM orders
WHERE revenue < 0;
```

> **🔹 Red Flags in Interviews:**
> - Not testing for **nulls** or **divide-by-zero** errors.
> - Assuming all joins are `INNER` (forgetting `LEFT/RIGHT` for completeness).

### **4. Low-Medium Complexity ETL Logic**
**Purpose:** Transform raw data into analysis-ready formats.
**Expectations:**
- **Cleaning** (handling nulls, standardizing formats).
- **Transformations** (pivoting, unpivoting, derived columns).
- **Incremental Loads** (updating only new/changed data).
- **Basic Scheduling** (e.g., "Run this daily at 2 AM").

#### **Example Queries**
```sql
-- 1. Standardize Customer Names (ETL Cleaning)
UPDATE customers
SET customer_name = TRIM(UPPER(customer_name))
WHERE customer_name LIKE '%  %' OR customer_name LIKE '%a%';  -- Fix whitespace/casing

-- 2. Pivot Sales by Product Category (For Power BI)
SELECT
    region,
    SUM(CASE WHEN category = 'Electronics' THEN revenue ELSE 0 END) AS electronics_revenue,
    SUM(CASE WHEN category = 'Clothing' THEN revenue ELSE 0 END) AS clothing_revenue
FROM sales
GROUP BY 1;

-- 3. Incremental Load (Only New Orders)
INSERT INTO target_orders
SELECT * FROM source_orders
WHERE order_date > (SELECT MAX(order_date) FROM target_orders);
```

> **🔹 Common Pitfalls:**
> - **Performance:** Avoid `UPDATE` on large tables without indexes.
> - **Idempotency:** Ensure rerunning the ETL doesn’t duplicate data.
> - **Logging:** Always log errors (e.g., failed rows) for debugging.

### **5. Interview-Specific Expectations**
| **Topic**               | **What They Test**                          | **Example Question**                          |
|-------------------------|--------------------------------------------|-----------------------------------------------|
| **SQL Fundamentals**    | Syntax, joins, aggregations                | *"Write a query to find the 2nd highest salary."* |
| **Business Acumen**     | Translating metrics to SQL                 | *"How would you calculate monthly active users (MAU)?"* |
| **Problem-Solving**     | Debugging, optimizing slow queries         | *"This query takes 10 minutes. How would you fix it?"* |
| **ETL Design**          | Handling data pipelines                    | *"How would you load daily sales data from CSV to a database?"* |
| **Edge Cases**          | Robustness (e.g., nulls, duplicates)       | *"What if a customer has two ‘first orders’ on the same day?"* |

### **6. Tools & Extensions You Might Encounter**
| **Tool**               | **Relevance**                              | **Example**                                  |
|------------------------|--------------------------------------------|----------------------------------------------|
| **CTEs**               | Improve readability                        | `WITH cte_name AS (SELECT ...)`              |
| **Window Functions**   | Rank, running totals, moving averages      | `ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date)` |
| **JSON/XML Handling**  | Semi-structured data (e.g., APIs)          | `SELECT json_value(column, '$.price') FROM table` |
| **Stored Procedures**  | Reusable logic (common in ETL)             | `CREATE PROCEDURE update_customer_status() BEGIN ... END;` |
| **Indexing**           | Performance tuning                         | `CREATE INDEX idx_customer_id ON orders(customer_id);` |

### **7. How to Prepare**
1. **Practice Platforms:**
   - [LeetCode (SQL Problems)](https://leetcode.com/problemset/database/)
   - [StrataScratch](https://www.stratascratch.com/) (Real-world scenarios)
   - [HackerRank (SQL)](https://www.hackerrank.com/domains/sql)

2. **Focus Areas for You:**
   - **Financial Data:** YoY growth, cohort analysis (aligns with your B.Com/MBA).
   - **ETL:** Mimic your Tally/Excel workflows in SQL (e.g., reconciling ledgers).
   - **Power BI Integration:** Write SQL for direct queries (e.g., parameterized reports).

3. **Behavioral + Technical:**
   - *"Tell me about a time you found a data error with SQL."*
   - *"How would you explain a complex query to a non-technical stakeholder?"*

### **8. Sample Interview Workflow**
1. **Clarify Requirements:**
   - *"What’s the expected output format?"*
   - *"Are there any known data quality issues?"*

2. **Write the Query:**
   - Start with a **CTE** for readability.
   - Use **comments** to explain steps.

3. **Test & Optimize:**
   - *"How would you test this query?"* → **"I’d check row counts, sample outputs, and edge cases."**
   - *"Can this be optimized?"* → **"Adding an index on `order_date` would speed up the GROUP BY."**

4. **Explain Your Thought Process:**
   - *"I used a window function here because we need to compare each row to its previous value."*

### **9. Your Advantage**
- **Domain Knowledge:** Your finance/taxation background is a **plus** for queries involving:
  - **Reconciliations** (e.g., matching invoices to payments).
  - **Time-Based Aggregations** (fiscal years in Tally → `DATE_TRUNC` in SQL).
  - **Executive Reporting** (e.g., "Show me top 5 high-value customers with declining spend").

> **Tailor Examples to Your Experience:**
> - *"In my current role, I used SQL to automate a monthly P&L report previously done in Excel, reducing errors by 30%."*

### **Next Steps**
1. **Mock Scenario:** Try this problem:
   > *"Write a query to identify customers whose average order value (AOV) dropped by >20% compared to their first 3 months."*
2. **Share Your Solution:** I can review it and suggest optimizations.
3. **Deep Dive:** Want to explore ETL tools like **dbt** or **Airflow**? Let me know!
