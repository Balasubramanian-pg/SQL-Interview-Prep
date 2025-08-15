---
Created:
Company:
  -
Difficulty:
Status:
Category:
Sub category:
Question Link:
---
Of course! This is a fantastic query with a complex `HAVING` clause, which is perfect for a detailed breakdown. It showcases how to define and identify entities based on multiple, combined criteria.

Here is the complete analysis.

***

### 1. The Question

The business question being asked by this SQL query is:

**"Which employees qualify as 'Subject Matter Experts' (SMEs) based on our company's specific criteria? An SME is defined as an employee who meets either of the following conditions:**
*   **They have expertise in exactly one domain with at least 8 years of experience in it.**
*   **They have expertise in exactly two domains with a combined total of at least 12 years of experience across them."**

---

### 2. Table Schema

The query references a single table, `employee_expertise`. This table is structured to hold multiple skill/domain entries for each employee. A plausible schema would be:

```sql
-- Table: employee_expertise
-- Stores a record for each domain of expertise an employee has.
CREATE TABLE employee_expertise (
    employee_id         INT NOT NULL,           -- The ID of the employee
    domain              VARCHAR(100) NOT NULL,  -- The name of the expertise domain (e.g., 'Cloud Computing')
    years_of_experience INT NOT NULL            -- The years of experience in that specific domain
);

-- A composite primary key is appropriate to ensure an employee doesn't have duplicate entries for the same domain.
ALTER TABLE employee_expertise ADD PRIMARY KEY (employee_id, domain);

-- An index on employee_id is crucial for the GROUP BY performance.
CREATE INDEX idx_employee_expertise_id ON employee_expertise (employee_id);
```

---

### 3. Structured SQL Query (Method 1: Complex `HAVING` Clause)

This is your original query, which is a very direct and efficient way to solve this problem.

```sql
SELECT
    employee_id
FROM
    employee_expertise
GROUP BY
    employee_id
HAVING
    (COUNT(DISTINCT domain) = 2 AND SUM(years_of_experience) >= 12)
    OR (COUNT(DISTINCT domain) = 1 AND SUM(years_of_experience) >= 8);
```

---

### 4. Explanation of the Query

This query identifies SMEs by aggregating each employee's expertise and then applying a compound filter in the `HAVING` clause.

1.  **`FROM employee_expertise`**: The query starts by reading from the table containing all expertise records.
2.  **`GROUP BY employee_id`**: This is the most important step. It groups all rows that belong to the same `employee_id` together. This creates a "summary bucket" for each employee, containing all their listed domains and associated years of experience.
3.  **`HAVING ...`**: The `HAVING` clause is used to filter these *groups* (the employees) based on aggregate calculations. It will only keep the groups that satisfy the complex condition.
    *   **The `OR` Operator**: An employee's group is kept if it meets *either* the condition on the left `OR` the condition on the right.
    *   **Condition 1: `(COUNT(DISTINCT domain) = 2 AND SUM(years_of_experience) >= 12)`**
        *   `COUNT(DISTINCT domain) = 2`: Checks if the employee has exactly two unique domains listed.
        *   `SUM(years_of_experience) >= 12`: Checks if the total years of experience across those domains is 12 or more.
        *   Both must be true for this condition to pass.
    *   **Condition 2: `(COUNT(DISTINCT domain) = 1 AND SUM(years_of_experience) >= 8)`**
        *   `COUNT(DISTINCT domain) = 1`: Checks if the employee has exactly one domain listed.
        *   `SUM(years_of_experience) >= 8`: Checks if their experience in that one domain is 8 years or more.
4.  **`SELECT employee_id`**: Finally, it selects the `employee_id` for each of the groups that passed the `HAVING` clause filter.

---

### 5. Another SQL Method (Method 2: Using a Common Table Expression - CTE)

An alternative approach, which is often more readable for complex filtering, is to first calculate the aggregate metrics for all employees in a CTE and then filter the results in a simple `WHERE` clause.

```sql
WITH employee_summary AS (
    -- Step 1: Calculate the total domains and experience for every employee.
    SELECT
        employee_id,
        COUNT(DISTINCT domain) AS domain_count,
        SUM(years_of_experience) AS total_experience
    FROM
        employee_expertise
    GROUP BY
        employee_id
)
-- Step 2: Filter the summary results based on the SME criteria.
SELECT
    employee_id
FROM
    employee_summary
WHERE
    (domain_count = 2 AND total_experience >= 12)
    OR (domain_count = 1 AND total_experience >= 8);
```

---

### 6. Explanation of the Alternative Query

This method breaks the problem into two distinct, easier-to-understand steps.

**Step 1: The `employee_summary` CTE**
*   This first part of the query creates a temporary result set named `employee_summary`.
*   It calculates two key metrics for **every single employee**:
    1.  `COUNT(DISTINCT domain) AS domain_count`: The number of unique domains for each employee.
    2.  `SUM(years_of_experience) AS total_experience`: The total combined years of experience for each employee.
*   The result is a simple summary table, with one row per employee, containing their ID, domain count, and total experience.

**Step 2: The Final `SELECT` Statement**
*   This query now operates on the clean, pre-aggregated data in `employee_summary`.
*   **`WHERE ...`**: Instead of a complex `HAVING` clause, it uses a standard `WHERE` clause to filter the summary rows. The logic in the `WHERE` clause is identical to the original `HAVING` clause, but it's applied to the simple, aggregated columns (`domain_count`, `total_experience`), which can make the query easier to read and maintain.

This CTE approach is often preferred in complex scenarios because it improves readability by separating the aggregation logic from the filtering logic. Performance is typically very similar to the first method in modern databases.