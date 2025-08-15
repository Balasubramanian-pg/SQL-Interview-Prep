# Department High Earners

## 1. Problem Statement

### 1.1. Objective
As part of an ongoing analysis of salary distribution within the company, your manager has requested a report identifying high earners in each department. You're tasked with identifying these high earners across all departments.

### 1.2. Definition
-   A 'high earner' within a department is defined as an employee with a salary ranking among the **top three distinct salaries** within that department.

> [!IMPORTANT]
> The query must correctly handle duplicate salaries. If multiple employees share one of the top three salary amounts, they should all be included in the results.

### 1.3. Sorting Requirement
-   In case of duplicates, the final results should be sorted by department name (ascending), then by salary (descending), and finally by the employee's name (alphabetically).

## 2. Input Schemas

> [!TIP]
> The two tables are linked by the `department_id` column. An `INNER JOIN` will be used to combine them to get the department names for each employee.

### 2.1. `employee` Schema:

|column_name|type|description|
|---|---|---|
|employee_id|integer|The unique ID of the employee.|
|name|string|The name of the employee.|
|salary|integer|The salary of the employee.|
|department_id|integer|The department ID of the employee.|
|manager_id|integer|The manager ID of the employee.|

### 2.2. `department` Schema:

|column_name|type|description|
|---|---|---|
|department_id|integer|The department ID of the employee.|
|department_name|string|The name of the department.|

## 3. Example

### 3.1. Example Input:
`employee` table:
|employee_id|name|salary|department_id|
|---|---|---|---|
|1|Emma Thompson|3800|1|
|2|Daniel Rodriguez|2230|1|
|3|Olivia Smith|2000|1|
|4|Noah Johnson|6800|2|
|5|Sophia Martinez|1750|1|
|...|...|...|...|
|8|William Davis|6800|2|
|...|...|...|...|
|10|James Anderson|4000|1|

`department` table:
|department_id|department_name|
|---|---|
|1|Data Analytics|
|2|Data Science|
|3|Engineering|

### 3.2. Example Output:

|department_name|name|salary|
|---|---|---|
|Data Analytics|James Anderson|4000|
|Data Analytics|Emma Thompson|3800|
|Data Analytics|Daniel Rodriguez|2230|
|Data Science|Noah Johnson|6800|
|Data Science|William Davis|6800|

### 3.3. Explanation
-   In the **Data Analytics** department, the top three salaries are $4,000 (James), $3,800 (Emma), and $2,230 (Daniel).
-   In the **Data Science** department, the top salary is $6,800. Since two employees, Noah and William, share this salary, they are both included. Noah is listed first due to alphabetical sorting.

## 4. Conceptual Approach
To solve this, we need to rank employees within each department by their salary and then select the top three ranks.

1.  **Join Data**: Combine the `employee` and `department` tables to get employee names, salaries, and their corresponding department names in one place.
2.  **Rank Salaries**: Use a window function to rank employees by salary *within* each department. The ranking must handle ties correctly.
3.  **Filter Top 3**: Select only the employees whose salary rank is less than or equal to 3.
4.  **Sort Results**: Order the final output according to the specified multi-level criteria.

### 4.1. Choosing the Right Ranking Function
The key to this problem is selecting the correct window function to handle potential salary ties.

> [!IMPORTANT]
> `DENSE_RANK()` is the ideal choice for this scenario. It ensures that if multiple employees have the same salary, they receive the same rank, and critically, the next rank number is sequential (no gaps are created).

> [!NOTE]
> Let's compare the ranking functions for a salary list of (100k, 100k, 90k):
> - `ROW_NUMBER()`: Would give ranks 1, 2, 3. Incorrect, as it arbitrarily separates the tied salaries.
> - `RANK()`: Would give ranks 1, 1, 3. Incorrect, as it would skip rank 2, potentially excluding the true third-highest salary.
> - `DENSE_RANK()`: Would give ranks 1, 1, 2. This is correct. It groups the ties and ensures we can reliably select the top N *distinct* salary levels.

## 5. SQL Solution

```sql
WITH ranked_employees AS (
  SELECT
    d.department_name,
    e.name,
    e.salary,
    DENSE_RANK() OVER (
      PARTITION BY e.department_id
      ORDER BY e.salary DESC
    ) AS salary_rank
  FROM employee AS e
  INNER JOIN department AS d
    ON e.department_id = d.department_id
)
SELECT
  department_name,
  name,
  salary
FROM ranked_employees
WHERE salary_rank <= 3
ORDER BY department_name ASC, salary DESC, name ASC;
```

## 6. Code Breakdown

### 6.1. CTE: `ranked_employees`
This Common Table Expression is the core of the solution, preparing the ranked data.
-   **`INNER JOIN`**: It first joins the `employee` and `department` tables.
-   **`DENSE_RANK() OVER (...)`**: This window function assigns a rank to each employee.
    -   `PARTITION BY e.department_id`: This is the most critical clause. It tells the function to perform the ranking independently for each department, resetting the rank counter for each new `department_id`.

> [!CAUTION]
> Without the `PARTITION BY` clause, the query would rank all employees across the entire company, which would not answer the question of finding the top earners *within each department*.

    -   `ORDER BY e.salary DESC`: This sorts employees within each department partition by their salary in descending order, so the highest salary gets rank 1.

### 6.2. Final `SELECT` Statement
This part of the query filters and sorts the ranked data to produce the final report.
-   `FROM ranked_employees`: It queries the temporary result set created by the CTE.
-   `WHERE salary_rank <= 3`: This is the filter that selects only the employees who are in one of the top three distinct salary tiers for their department.

> [!TIP]
> This simple filter is possible because of the careful work done by `DENSE_RANK()` in the previous step.

-   **`ORDER BY department_name ASC, salary DESC, name ASC`**: This multi-level sorting clause is crucial for meeting the output requirements.
    -   It first sorts by `department_name` alphabetically.
    -   Then, within each department, it sorts by `salary` from highest to lowest.
    -   Finally, if two employees in the same department have the same salary, it sorts them by `name` alphabetically.
