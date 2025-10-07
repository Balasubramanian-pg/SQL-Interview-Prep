## FIRST\_VALUE/LAST\_VALUE Cheat Sheet Table

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **First Value in Group** | Finding a key metric (e.g., starting balance, first sale) for a partition. | Use **`FIRST_VALUE`** over a partitioned and ordered set. | `$FIRST\_VALUE(column) OVER (PARTITION BY group\_col ORDER BY sort\_col)$` |
| **Last Value in Group** | Finding a final metric (e.g., ending balance, most recent status) for a partition. | Use **`LAST_VALUE`** with an **unbounded following** frame to ensure the final row is captured. | `$LAST\_VALUE(column) OVER (PARTITION BY group\_col ORDER BY sort\_col ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)$` |
| **Comparing to Start** | Measuring all subsequent values against the partition's starting value. | Use **`FIRST_VALUE`** to establish a baseline for comparison/calculation. | `$column - FIRST\_VALUE(column) OVER (PARTITION BY group\_col ORDER BY sort\_col)$` |
| **Tracking High/Low** | Finding the initial or final record based on a specific ordering criteria. | Use `FIRST_VALUE` or `LAST_VALUE` after ordering by the target column (e.g., highest price). | `$FIRST\_VALUE(return\_col) OVER (ORDER BY target\_col DESC)$` |

## FIRST\_VALUE/LAST\_VALUE Mini Playbook (SQL Snippets)

These snippets illustrate the common use cases for `FIRST_VALUE` and `LAST_VALUE` functions.

### 1\. First Value in Group (Baseline)

Find the **starting salary** for every employee group (e.g., department).

```sql
SELECT
    employee_id,
    department,
    hire_date,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
    ) AS initial_department_salary
FROM
    employee_history;
```

### 2\. Last Value in Group (Final Status)

Determine the **final project status** for each project based on the most recent update date.

*Note: The **unbounded following** frame is crucial for `LAST_VALUE` to correctly identify the final row in the partition.*

```sql
SELECT
    project_id,
    update_date,
    status_name,
    LAST_VALUE(status_name) OVER (
        PARTITION BY project_id
        ORDER BY update_date
        -- Frame ensures we look at the last row of the partition
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS final_project_status
FROM
    project_updates;
```

### 3\. Comparing to Start (Deviation)

Calculate the **cumulative deviation** of a product's price from its **initial launch price**.

```sql
SELECT
    product_sku,
    effective_date,
    current_price,
    current_price - FIRST_VALUE(current_price) OVER (
        PARTITION BY product_sku
        ORDER BY effective_date
    ) AS price_deviation_from_start
FROM
    price_changes;
```

### 4\. Tracking High/Low

Identify the **user who holds the current highest score** in a game across all attempts.

*Note: Here, the `PARTITION BY` is omitted to treat the entire dataset as one window.*

```sql
SELECT
    user_id,
    score,
    score_date,
    FIRST_VALUE(user_id) OVER (
        -- Orders by score descending to put the highest score first
        ORDER BY score DESC, score_date DESC
    ) AS current_high_score_user
FROM
    game_scores;
```
