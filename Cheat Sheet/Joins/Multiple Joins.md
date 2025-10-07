## Multiple Join Conditions Cheat Sheet Table

| Problem Type | What They're Testing | Solution Pattern | Key Technique |
| :--- | :--- | :--- | :--- |
| **Compound Key Match** | Joining tables that use a combination of two or more columns to define a unique relationship. | Use the `AND` operator to link multiple columns in the `ON` clause. | `ON T1.ID = T2.ID AND T1.Category = T2.Category` |
| **Range/Time Overlap** | Linking a row in one table to a row in a second table only if a date/time falls within a specific range. | Use `>=` and `<` (or `<=`) operators to define the range boundaries. | `ON T1.Date >= T2.StartDate AND T1.Date <= T2.EndDate` |
| **Conditional Status Match** | Joining based on a primary key *and* ensuring a specific status or flag is met simultaneously. | Use `AND` to combine the primary key match with a boolean or value check. | `ON T1.UserID = T2.UserID AND T2.IsActive = TRUE` |
| **Excluding Self-Joins** | Combining a self-join with an exclusion condition (e.g., finding pairs but not the row itself). | Use the `AND` operator with the `<` or `!=` operator on the primary key. | `ON T1.CommonCol = T2.CommonCol AND T1.ID != T2.ID` |

-----

## Multiple Join Conditions Mini Playbook (Realistic Queries)

These snippets illustrate the common use cases for joins with multiple conditions.

### 1\. Compound Key Match (Order Item Linking)

**Use Case:** Link sales orders to specific line items. The combination of `order_id` and `product_sku` uniquely identifies the relationship.

```sql
SELECT
    o.order_date,
    i.quantity,
    i.item_price
FROM
    Orders AS o
INNER JOIN
    Order_Items AS i
    -- Compound Key: Matching both the Order ID and the Product SKU for precision
    ON o.order_id = i.order_id
    AND o.product_sku = i.product_sku;
```

-----

### 2\. Range/Time Overlap (Price Effective Date)

**Use Case:** Retrieve the correct price for a product based on its `transaction_date`. The price must be effective on that specific date, falling between the price's start and end dates.

```sql
SELECT
    t.transaction_id,
    p.price
FROM
    Transactions AS t
INNER JOIN
    Price_List AS p
    -- Match the product ID
    ON t.product_id = p.product_id
    -- Match the transaction date to the price's effective date range
    AND t.transaction_date >= p.effective_start_date
    AND t.transaction_date < p.effective_end_date;
```

-----

### 3\. Conditional Status Match (Active User Tasks)

**Use Case:** List all tasks assigned to users, but **only** for users who are currently active.

```sql
SELECT
    u.user_name,
    t.task_description
FROM
    Users AS u
LEFT JOIN
    Tasks AS t
    -- Match the primary key (user_id)
    ON u.user_id = t.user_id
    -- AND filter to include only active users
    AND u.is_active = TRUE;
    -- Note: This is an important pattern. Putting the condition in the JOIN (not WHERE)
    -- ensures that inactive users are still returned (as NULLs) if using a LEFT JOIN,
    -- allowing for comparison/filtering based on the left table's full data set later.
```

-----

### 4\. Excluding Self-Joins (Finding Non-Identical Duplicates)

**Use Case:** Find records that share the same `email_address` but have different `user_id`s, indicating potential duplicate accounts.

```sql
SELECT
    a.user_id AS user_id_a,
    b.user_id AS user_id_b,
    a.email_address
FROM
    Users AS a
INNER JOIN
    Users AS b
    -- Match the shared attribute (email)
    ON a.email_address = b.email_address
    -- AND exclude the case where a row is joined to itself
    AND a.user_id < b.user_id; -- Using '<' ensures each duplicate pair is listed only once
```
