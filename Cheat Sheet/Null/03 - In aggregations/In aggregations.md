This cheat sheet focuses on the specific behavior of **`NULL` values within SQL aggregate functions** (`SUM`, `AVG`, `COUNT`, `MIN`, `MAX`). Understanding this behavior is crucial for accurate calculations, as `NULL` is **ignored** by most aggregate functions, which can lead to unexpected results, particularly with `COUNT` and `AVG`.

## NULL in Aggregations Cheat Sheet Table

| Aggregate Function | Treatment of NULLs | Effect on Calculation | Key Pitfall/Note |
| :--- | :--- | :--- | :--- |
| **`SUM(column)`** | **Ignores** all `NULL` values. | Only sums non-NULL numeric values. | If **all** values in the group are `NULL`, the result is typically **`NULL`** (or 0 in some contexts/dialects). |
| **`COUNT(column)`** | **Ignores** all `NULL` values. | Counts only the rows where the specified `column` is **not `NULL`**. | Use `COUNT(*)` or `COUNT(1)` to count all rows (including those with `NULL`s in specific columns). |
| **`AVG(column)`** | **Ignores** all `NULL` values. | Calculates the sum of non-NULLs, divided by the **count of non-NULLs**. | Can skew results if you intend to include `NULL` values as zero in the average calculation. |
| **`MIN(column)`** | **Ignores** all `NULL` values. | Finds the smallest non-NULL value. | If all values are `NULL`, the result is `NULL`. |
| **`MAX(column)`** | **Ignores** all `NULL` values. | Finds the largest non-NULL value. | If all values are `NULL`, the result is `NULL`. |

-----

## NULL in Aggregations Mini Playbook (Realistic Queries)

These snippets illustrate the critical differences in how aggregate functions handle `NULL`s and provide solutions for accurate reporting. Assume a table `Performance` with `user_id`, `score`, and `bonus_paid`.

### Setup: Illustrative Data Behavior

Assume a group of 5 rows where 2 rows have a score of 10, and 3 rows have a score of `NULL`.

| Function | Result | Reasoning |
| :--- | :--- | :--- |
| `COUNT(*)` | 5 | Counts all rows. |
| `COUNT(score)` | 2 | Counts only the 2 non-NULL scores. |
| `SUM(score)` | 20 | Sums only the 2 scores (10 + 10). |
| `AVG(score)` | 10 | Calculates 20 / 2 (Sum / Count of non-NULLs). |

-----

### 1\. Counting All Rows vs. Counting Non-NULLs

**Use Case:** Compare the total number of records (`COUNT(*)`) against the number of records with an actual performance score (`COUNT(score)`).

```sql
SELECT
    COUNT(*) AS total_records,
    COUNT(score) AS scored_records,
    COUNT(*) - COUNT(score) AS missing_scores
FROM
    Performance;
```

-----

### 2\. The AVG Pitfall (NULLs Ignored)

**Use Case:** Calculate the average score. If you want `NULL` scores to be treated as 0 for averaging (which may be a business requirement), you **must** handle the `NULL`s explicitly.

```sql
SELECT
    -- AVG treats NULLs as non-existent (20 / 2 = 10)
    AVG(score) AS avg_score_ignoring_nulls,
    
    -- Solution: Treat NULL scores as 0 (20 / 5 = 4)
    AVG(COALESCE(score, 0)) AS avg_score_treating_nulls_as_zero
FROM
    Performance;
```

-----

### 3\. SUM and the `NULL` Result

**Use Case:** Calculate the total bonus paid. If a whole group has no bonuses recorded (all `NULL`), `SUM` returns `NULL`. Use `COALESCE` to display 0 instead for cleaner reporting.

```sql
SELECT
    user_id,
    SUM(bonus_paid) AS total_bonus_paid,
    -- Provides a cleaner 0 for users with no bonus records
    COALESCE(SUM(bonus_paid), 0) AS total_bonus_paid_clean
FROM
    Performance
GROUP BY
    user_id;
```

-----

### 4\. MIN/MAX Ignoring NULLs

**Use Case:** Find the lowest and highest reported scores. `NULL`s do not affect the result unless *all* scores are `NULL`.

```sql
SELECT
    MIN(score) AS lowest_score,
    MAX(score) AS highest_score
FROM
    Performance;
```
