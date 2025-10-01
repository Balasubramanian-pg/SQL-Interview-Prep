Handling **NULL values in a `CASE` expression** is a critical aspect of SQL logic, as `NULL` comparisons behave uniquely. Since `NULL` is an "unknown value," standard equality checks involving `NULL` always evaluate to `UNKNOWN`, which is treated as **FALSE** by the `CASE` expression.

Here's a breakdown of how `NULL` is handled in the two forms of `CASE` and the correct way to check for it.

-----

## 1\. NULL in Searched CASE (The Preferred Method)

The **Searched `CASE`** expression is the most robust way to handle `NULL` because it allows you to use the **`IS NULL`** operator, which correctly identifies the `NULL` state.

### Syntax and Example

In a Searched `CASE`, you treat the `IS NULL` check as any other conditional branch:

```sql
SELECT
    product_id,
    inventory_level,
    CASE
        WHEN inventory_level IS NULL THEN 'Unknown Stock'  -- Checks for NULL explicitly
        WHEN inventory_level = 0 THEN 'Out of Stock'     -- Checks for zero
        WHEN inventory_level < 10 THEN 'Low Stock'
        ELSE 'In Stock'
    END AS stock_status
FROM
    Products;
```

### Key Behavior

  * The condition `inventory_level IS NULL` evaluates to **TRUE** when the column is `NULL`.
  * The `CASE` expression proceeds sequentially, executing the code for the **first TRUE** condition. Placing `IS NULL` first ensures it's checked before any value comparisons.
  * If `NULL` is not checked explicitly, it falls to the **`ELSE`** clause (if one exists), or results in `NULL` for the expression itself.

-----

## 2\. NULL in Simple CASE (The Trap)

The **Simple `CASE`** expression is designed for equality checks, which makes it ineffective for directly checking for `NULL`.

### Syntax and Example

In a Simple `CASE`, you provide an input expression and then check equality against that input:

```sql
SELECT
    user_id,
    last_login_date,
    CASE last_login_date
        WHEN NULL THEN 'Never Logged In'  -- THIS WILL ALWAYS FAIL!
        ELSE 'Has Logged In'
    END AS login_status_fail
FROM
    Users;
```

### Key Behavior (The Trap)

  * When SQL evaluates `CASE last_login_date WHEN NULL`, it translates to the comparison `last_login_date = NULL`.
  * This comparison always evaluates to **UNKNOWN**, which is treated as **FALSE** by the `CASE` expression.
  * The `WHEN NULL` branch is **never reached**. Rows with a `NULL` `last_login_date` will fall through to the `ELSE` clause.

### Simple CASE Workaround

The only way to handle `NULL` with a Simple `CASE` is to use a function like **`COALESCE`** or **`ISNULL`** on the input expression to replace `NULL` with a unique sentinel value:

```sql
SELECT
    user_id,
    last_login_date,
    CASE COALESCE(CAST(last_login_date AS VARCHAR), 'NO_DATE')
        WHEN 'NO_DATE' THEN 'Never Logged In' -- Now checking equality with the sentinel
        ELSE 'Has Logged In'
    END AS login_status_workaround
FROM
    Users;
```

-----

## Summary: Always Use IS NULL

For clarity, robustness, and performance, the **Searched `CASE` (using `WHEN column IS NULL`) is the standard and correct way** to handle missing values within conditional logic.

| Scenario | Recommended Solution |
| :--- | :--- |
| **Check for NULL** | `WHEN my_column IS NULL THEN 'Result'` |
| **Check for NOT NULL** | `WHEN my_column IS NOT NULL THEN 'Result'` |
| **Check equality with NULL** | (Should be avoided. Use the Searched CASE or `COALESCE`.) |
