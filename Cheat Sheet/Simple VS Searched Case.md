This cheat sheet compares the two primary forms of the SQL **`CASE`** expression: **Simple CASE** and **Searched CASE**. Both are used for implementing conditional logic in SQL queries, but they are suited for different types of comparisons.

## Simple CASE vs. Searched CASE Cheat Sheet Table

| Feature | Simple CASE | Searched CASE |
| :--- | :--- | :--- |
| **Primary Use** | Checking for **equality** against a single input column or expression. | Checking for **multiple, complex conditions** (ranges, inequalities, custom logic). |
| **Syntax Structure** | Starts with the input expression, followed by `WHEN` value comparisons. | Starts immediately with `WHEN` condition comparisons. |
| **Flexibility** | **Limited.** Can only check if the input equals a specific value. | **High.** Can check `<, >, LIKE, IS NULL`, or any boolean expression. |
| **Readability** | Generally **better** when checking a single column against many possible values. | Generally **better** for complex, multi-field, or range-based logic. |
| **Final Result** | Both forms produce a single output value based on the first condition that evaluates to TRUE. | Both forms produce a single output value based on the first condition that evaluates to TRUE. |

## Simple CASE vs. Searched CASE Mini Playbook (Realistic Queries)

These snippets illustrate the distinct syntax and appropriate use case for each form of the `CASE` expression.

### 1\. Simple CASE (Equality Check)

**Use Case:** Assign a full description based on a single-column status code.

```sql
SELECT
    order_id,
    status_code,
    CASE status_code -- The input expression is placed here
        WHEN 1 THEN 'Order Placed'
        WHEN 2 THEN 'Processing Payment'
        WHEN 3 THEN 'Awaiting Shipment'
        WHEN 4 THEN 'Shipped'
        ELSE 'Unknown Status' -- The fallback value
    END AS status_description
FROM
    Orders;
```

### 2\. Searched CASE (Complex/Range Check)

**Use Case:** Assign a user tier based on a `lifetime_spend` range. This requires inequality comparisons, which Simple CASE cannot handle.

```sql
SELECT
    customer_id,
    lifetime_spend,
    CASE -- The expression starts immediately with WHEN
        WHEN lifetime_spend > 5000 THEN 'Platinum'
        WHEN lifetime_spend >= 1000 THEN 'Gold' -- Checks for range >= 1000 AND <= 5000
        WHEN lifetime_spend >= 100 THEN 'Silver'
        ELSE 'Bronze' -- The fallback value
    END AS customer_tier
FROM
    Customers;
```

### 3\. Combining Logic (Searched CASE is Superior)

**Use Case:** Determine an employee's bonus multiplier based on their title and their performance score. Simple CASE cannot handle the combined logic required here.

```sql
SELECT
    employee_id,
    title,
    performance_score,
    CASE
        -- Condition 1: High score for Executive
        WHEN title = 'Executive' AND performance_score >= 0.95 THEN 2.0
        
        -- Condition 2: High score for Manager
        WHEN title = 'Manager' AND performance_score >= 0.90 THEN 1.5
        
        -- Default for all others
        ELSE 1.0
    END AS bonus_multiplier
FROM
    Employees;
```
