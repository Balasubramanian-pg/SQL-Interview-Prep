This cheat sheet compares the three main variations of the SQL **`COUNT`** aggregate function, highlighting their fundamental differences in how they handle rows and `NULL` values.

## COUNT(\*) vs COUNT(column) vs COUNT(DISTINCT) Cheat Sheet Table

| Function | Purpose | NULL Handling | Includes Duplicates | Key Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **`COUNT(*)`** | Counts **all rows** returned by the query or within a group. | **Ignores `NULL` handling**—it counts the existence of the row itself. | **Yes** (counts every row). | Getting the total size of a table or the total count within a group (e.g., total transactions). |
| **`COUNT(column)`** | Counts **non-NULL values** in the specified column. | **Ignores** `NULL` values. | **Yes** (counts every non-NULL value, even if identical). | Counting how many records have data (e.g., how many users provided a phone number). |
| **`COUNT(DISTINCT column)`** | Counts the **unique, non-NULL values** in the specified column. | **Ignores** `NULL` values. | **No** (only counts unique values once). | Counting the total number of unique entities (e.g., how many distinct products were sold). |

-----

## COUNT Variations Mini Playbook (Realistic Queries)

Assume a table `Orders` with the following illustrative data:

| order\_id | customer\_id | product\_id | shipped\_date |
| :--- | :--- | :--- | :--- |
| 1 | 101 | A | 2025-10-01 |
| 2 | 101 | A | 2025-10-01 |
| 3 | 102 | B | NULL |
| 4 | 103 | C | 2025-10-02 |
| 5 | 104 | C | 2025-10-03 |
| 6 | 104 | C | NULL |

### 1\. COUNT(\*) (Total Rows)

**Use Case:** Calculate the total number of individual orders recorded.

```sql
SELECT
    COUNT(*) AS total_orders
FROM
    Orders;

-- Result: 6 (Counts every row, including those with NULL values in other columns.)
```

-----

### 2\. COUNT(column) (Total Non-NULL Values)

**Use Case:** Calculate the number of orders that have actually been shipped (i.e., have a non-NULL `shipped_date`).

```sql
SELECT
    COUNT(shipped_date) AS shipped_orders
FROM
    Orders;

-- Result: 4 (Orders 1, 2, 4, 5. Orders 3 and 6 are ignored because shipped_date is NULL.)
```

-----

### 3\. COUNT(DISTINCT column) (Total Unique Values)

**Use Case:** Calculate the total number of unique customers who placed an order.

```sql
SELECT
    COUNT(DISTINCT customer_id) AS unique_customers,
    COUNT(DISTINCT product_id) AS unique_products_ordered
FROM
    Orders;

/*
Result:
unique_customers: 4 (IDs: 101, 102, 103, 104)
unique_products_ordered: 3 (IDs: A, B, C)
*/
```

-----

### 4\. Handling NULL with COUNT(DISTINCT)

If `customer_id` had a `NULL` value, `COUNT(DISTINCT customer_id)` would **ignore** it.

| order\_id | customer\_id |
| :--- | :--- |
| 7 | 105 |
| 8 | NULL |
| 9 | 105 |

```sql
SELECT
    COUNT(DISTINCT customer_id) AS unique_customers_non_null
FROM
    Orders;

-- Result: 1 (Only 105 is counted, NULL is ignored.)
```
