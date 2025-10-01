This cheat sheet covers **Conditional Aggregation** using the **`SUM`** function combined with the **`CASE`** statement. This technique is extremely powerful, as it allows you to calculate multiple, distinct aggregates (like sums or counts) based on different criteria within a single `GROUP BY` operation. This is often used for creating summary reports where categories become columns.

## SUM with CASE (Conditional Aggregation) Cheat Sheet

| Problem Type | Goal | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Pivot/Cross-Tab** | Display different categorical totals (e.g., status, gender, year) as **separate columns**. | Use `SUM()` around a `CASE` statement that returns the value only if the condition is met. | `$SUM(CASE WHEN condition THEN value ELSE 0 END)$` |
| **Conditional Counting** | Count the number of rows that meet a specific criteria within a group. | Use `SUM()` around a `CASE` statement that returns `1` if the condition is met. | `$SUM(CASE WHEN condition THEN 1 ELSE 0 END)$` |
| **Categorical Filtering** | Calculate an aggregate for one subset of data *relative* to another subset in the same pass. | Define multiple `CASE` statements based on distinct categories. | `SUM(CASE WHEN Type='A' THEN Amount END)` vs. `SUM(CASE WHEN Type='B' THEN Amount END)` |

-----

## SUM with CASE Mini Playbook (Realistic Queries)

Assume a table `Orders` with columns: `user_id`, `amount`, and `status` ('SHIPPED', 'PENDING', 'CANCELLED').

### 1\. Pivoting Statuses (Amount as Columns)

**Use Case:** For each user, display the total amount spent on orders that are 'SHIPPED', 'PENDING', and 'CANCELLED', with each status as a separate column.

```sql
SELECT
    user_id,
    -- Column 1: Total amount of SHIPPED orders
    SUM(
        CASE WHEN status = 'SHIPPED' THEN amount ELSE 0 END
    ) AS total_shipped_amount,
    
    -- Column 2: Total amount of PENDING orders
    SUM(
        CASE WHEN status = 'PENDING' THEN amount ELSE 0 END
    ) AS total_pending_amount,
    
    -- Column 3: Total amount of CANCELLED orders
    SUM(
        CASE WHEN status = 'CANCELLED' THEN amount ELSE 0 END
    ) AS total_cancelled_amount
    
FROM
    Orders
GROUP BY
    user_id;
```

-----

### 2\. Conditional Counting (COUNT Replacement)

**Use Case:** For each user, count the number of high-value orders (amount \> 100) and the number of low-value orders (amount $\le$ 100).

*When counting, you return $\mathbf{1}$ for a match and $\mathbf{0}$ for a non-match. The sum of the $1$s is the count.*

```sql
SELECT
    user_id,
    -- Count of high-value orders
    SUM(
        CASE WHEN amount > 100 THEN 1 ELSE 0 END
    ) AS count_high_value,
    
    -- Count of low-value orders
    SUM(
        CASE WHEN amount <= 100 THEN 1 ELSE 0 END
    ) AS count_low_value
    
FROM
    Orders
GROUP BY
    user_id;
```

-----

### 3\. Conditional Aggregation with NULL (The Shorthand)

**Use Case:** Calculate the total amount for orders placed in Q1 versus Q2.

*You can omit the `ELSE 0` clause in the `CASE` statement. If the condition is false, `CASE` returns `NULL` by default, and **`SUM` ignores `NULL`s**, achieving the same result as `ELSE 0`.*

```sql
SELECT
    product_id,
    -- Sums only Q1 orders (NULLs are ignored)
    SUM(
        CASE WHEN order_quarter = 1 THEN amount END
    ) AS q1_sales,
    
    -- Sums only Q2 orders (NULLs are ignored)
    SUM(
        CASE WHEN order_quarter = 2 THEN amount END
    ) AS q2_sales
    
FROM
    Sales
GROUP BY
    product_id;
```

-----

### Summary of Conditional Counting

Using `SUM(CASE WHEN condition THEN 1 ELSE 0 END)` is the most common and robust way to count conditionally. While `COUNT(CASE WHEN condition THEN column ELSE NULL END)` also works (since `COUNT` ignores `NULL`s), the `SUM(CASE...1...0)` pattern is often more explicit and universally understood.
