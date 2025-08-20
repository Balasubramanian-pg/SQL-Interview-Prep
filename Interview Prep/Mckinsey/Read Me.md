### Category 1: Complex Joins and Logic

**1. The "Gap Analysis" Question:**
*   **Question:** "Given a table of `projects` with `project_id`, `start_date`, and `end_date`, how would you find all continuous ranges of dates that are not covered by any project? Assume no NULLs."
*   **Tests:** Self-joins or window functions (LEAD/LAG), date arithmetic, and complex conditional logic.

**2. The "Non-Equi Self-Join" Question:**
*   **Question:** "You have a table `transactions` with `user_id`, `transaction_id`, and `timestamp`. Find all pairs of transactions for the same user that occurred within 10 minutes of each other."
*   **Tests:** Understanding of self-joins, non-equi joins (on a time interval), and avoiding duplicate pairs (e.g., A-B and B-A).

**3. The "Greatest-N-Per-Group" Classic:**
*   **Question:** "Given an `employees` table with `id`, `name`, `department_id`, and `salary`, write a query to find the employee with the highest salary in each department. If there is a tie, return all employees with the highest salary."
*   **Tests:** Knowledge of window functions (`ROW_NUMBER`, `RANK`, `DENSE_RANK`) vs. a correlated subquery approach and how to handle ties.

**4. The "Three-Way Join with Condition" Question:**
*   **Question:** "We have three tables: `users`, `orders`, and `order_items`. Write a query to find all users who have *never* ordered a specific product (e.g., product_id = 123)."
*   **Tests:** Ability to structure a query using `NOT EXISTS` or a `LEFT JOIN` with a `NULL` check, combining multiple tables correctly.

**5. The "Rolling Average" Question:**
*   **Question:** "Given a `sales` table with `sale_date` and `amount`, calculate the 7-day rolling average of sales for each day."
*   **Tests:** Mastery of window functions with frame clauses (e.g., `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW`), handling dates, and potential NULLs.

---

### Category 2: Advanced Aggregation and Filtering

**6. The "Multiple Aggregates in One Query" Question:**
*   **Question:** "Write a single query to return the total number of orders, the number of shipped orders, and the number of cancelled orders from an `orders` table."
*   **Tests:** Use of conditional aggregation (e.g., `SUM(CASE WHEN status = 'shipped' THEN 1 ELSE 0 END) AS shipped_orders`).

**7. The "Histogram" Question:**
*   **Question:** "Using the `employees` table, create a histogram showing how many employees have a salary in ranges like 0-50000, 50001-75000, 75001-100000, etc."
*   **Tests:** Ability to use `CASE` statements or `WIDTH_BUCKET` (if supported) inside a `GROUP BY` to "bucket" continuous data.

**8. The "Percent of Total" Question:**
*   **Question:** "For the `sales` table, show each product's total sales and its percentage contribution to overall total sales."
*   **Tests:** Understanding of how to combine a regular aggregate with a window function that calculates a sum over the entire result set (e.g., `SUM(amount) / SUM(SUM(amount)) OVER()` ).

---

### Category 3: Window Functions Mastery

**9. The "Compare Row to Previous Row" Question:**
*   **Question:** "In a `temperature_readings` table with `reading_time` and `temperature`, for each reading, show the temperature and the difference from the *previous* reading's temperature."
*   **Tests:** Using `LAG()` to access data from a previous row in an ordered set.

**10. The "Running Total vs. Group Total" Question:**
*   **Question:** "For each user and their order, show the order amount, the running total of their orders (chronologically), and what percentage that specific order is of their lifetime total value."
*   **Tests:** Combining different window function frames: unbounded preceding for the running total and an entire partition for the lifetime total.

---

### Category 4: Handling Hierarchies and Recursion

**11. The "Manager Hierarchy" Question:**
*   **Question:** "An `employees` table has `employee_id`, `name`, and `manager_id`. Write a query to print the full hierarchy for a given employee (e.g., their manager, their manager's manager, etc., all the way to the top)."
*   **Tests:** Knowledge of **Recursive Common Table Expressions (CTEs)**, which is a must-know for hierarchical data.

---

### Category 5: Performance and Deep Understanding

**12. The "Indexing" Question:**
*   **Question:** "A query `SELECT * FROM users WHERE last_name = 'Smith' AND first_name = 'John';` is slow. What index would you create to speed it up, and why?"
*   **Tests:** Understanding of composite indexes and the order of columns (the most selective column, `last_name`, should often come first).

**13. The "NULL Pitfall" Question:**
*   **Question:** "What will this query return: `SELECT * FROM table WHERE column <> 'value';` if some rows have `NULL` in `column`? Why?"
*   **Tests:** Deep understanding of three-valued logic (TRUE, FALSE, UNKNOWN). Comparisons with `NULL` yield `UNKNOWN`, which is not `TRUE`, so those rows are filtered out.

**14. The "EXPLAIN" Question:**
*   **Question:** "You've written a query that's running slowly. What is your step-by-step process to diagnose and fix the performance issue?"
*   **Tests:** Practical knowledge of using `EXPLAIN` (or `EXPLAIN ANALYZE`) to look for full table scans, missing indexes, expensive operations like sorts, or bad join strategies.

---

### Category 6: Set Operations and Unconventional Thinking

**15. The "Find Duplicates" Question:**
*   **Question:** "Find all duplicate rows in a table based on a set of columns (e.g., `email`), and show how many times they appear."
*   **Tests:** Using `GROUP BY` and `HAVING COUNT(*) > 1`. A follow-up might be to delete duplicates, which tests using a CTE or subquery with `ROW_NUMBER()`.

**16. The "Pivot" Question:**
*   **Question:** "Given a `sales` table with `year`, `quarter`, and `revenue`, write a query to pivot the data so that each year is a row and the four quarters are columns (Q1, Q2, Q3, Q4)."
*   **Tests:** Knowledge of the `CASE` statement method for pivoting or the vendor-specific `PIVOT` function (e.g., in SQL Server or Oracle).

**17. The "Anti-Join" Question:**
*   **Question:** "Find all customers in the `customers` table who do *not* have an order in the `orders` table. Write the query using both a `LEFT JOIN` and a `NOT EXISTS` and explain the potential performance difference."
*   **Tests:** Understanding different methods to achieve the same logical result and their implications (often `NOT EXISTS` is more performant for this pattern).

**18. The "Date Dimension" Question:**
*   **Question:** "Generate a series of all dates for the last calendar year (e.g., Jan 1 to Dec 31 of last year) without using a pre-existing date table."
*   **Tests:** Knowledge of recursive CTEs or database-specific functions like `GENERATE_SERIES` (PostgreSQL) to create data on the fly.

**19. The "Correlated Subquery vs. Join" Question:**
*   **Question:** "When would you use a correlated subquery over a JOIN, and what are the performance considerations?"
*   **Tests:** Conceptual understanding. Correlated subqueries can be easier to read for existential checks (`EXISTS`) but can be inefficient if they execute once for every row in the outer query, unlike a set-based join.

**20. The "Self-Referencing Update" Question:**
*   **Question:** "You need to swap the `sex` values in a `users` table ('m' becomes 'f', and 'f' becomes 'm'). Write a single UPDATE statement to do this."
*   **Tests:** Logical thinking and avoiding a intermediate value trap. The solution often involves a `CASE` statement: `UPDATE users SET sex = CASE WHEN sex = 'm' THEN 'f' WHEN sex = 'f' THEN 'm' ELSE sex END;`.

These questions move beyond syntax and into problem-solving, which is what companies truly value in a senior SQL developer or data analyst.
