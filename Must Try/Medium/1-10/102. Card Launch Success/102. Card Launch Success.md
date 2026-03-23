# Credit Card Launch Month Performance

## 1. Problem Statement

### 1.1. Objective
Your team at JPMorgan Chase is soon launching a new credit card. You are asked to estimate how many cards you'll issue in the first month. Before you can answer this, you want to get some perspective on how well new credit card launches typically do in their first month.

Write a query that outputs the name of the credit card and how many cards were issued in its launch month. The launch month is defined as the earliest record in the `monthly_cards_issued` table for a given card. Order the results starting from the biggest issued amount.

> [!IMPORTANT]
> The core of the problem is to identify the "first" or "launch" record for each distinct credit card. "First" is defined chronologically, first by year and then by month.

### 1.2. Input Table: `monthly_cards_issued`

|Column Name|Type|
|---|---|
|issue_month|integer|
|issue_year|integer|
|card_name|string|
|issued_amount|integer|

### 1.3. Example

#### 1.3.1. `monthly_cards_issued` Example Input:

|issue_month|issue_year|card_name|issued_amount|
|---|---|---|---|
|1|2021|Chase Sapphire Reserve|170000|
|2|2021|Chase Sapphire Reserve|175000|
|3|2021|Chase Sapphire Reserve|180000|
|3|2021|Chase Freedom Flex|65000|
|4|2021|Chase Freedom Flex|70000|

#### 1.3.2. Example Output:

|card_name|issued_amount|
|---|---|
|Chase Sapphire Reserve|170000|
|Chase Freedom Flex|65000|

#### 1.3.3. Explanation
The launch month for 'Chase Sapphire Reserve' is its earliest entry, which is 1/2021, with an issued amount of 170,000. The launch month for 'Chase Freedom Flex' is its earliest entry, 3/2021, with an issued amount of 65,000.

## 2. Solution Approaches
This is a classic "greatest-n-per-group" problem, where we want to find the first record for each card. This can be solved efficiently using window functions or with a more traditional `GROUP BY` and `JOIN`.

### 2.1. Method 1: Using the `ROW_NUMBER()` Window Function (Recommended)
This is the most common and often most efficient and readable way to solve this type of problem.

> [!NOTE]
> Window functions are ideal for ranking problems. They allow you to assign a rank to each row within a specific group (or "partition") based on a defined order, all in a single pass over the data.

```sql
WITH ranked_issues AS (
  SELECT
    card_name,
    issued_amount,
    ROW_NUMBER() OVER (
      PARTITION BY card_name
      ORDER BY issue_year ASC, issue_month ASC
    ) AS row_num
  FROM monthly_cards_issued
)
SELECT
  card_name,
  issued_amount
FROM ranked_issues
WHERE row_num = 1
ORDER BY issued_amount DESC;
```

#### 2.1.1. Explanation

1.  **CTE: `ranked_issues`**:
    -   This Common Table Expression processes the entire `monthly_cards_issued` table and assigns a rank to each row.
    -   `ROW_NUMBER() OVER (...)`: This is the window function that generates the rank.
    -   `PARTITION BY card_name`: This is a crucial instruction. It tells the function to treat each `card_name` as a separate group and to **restart the ranking at 1** for each new card.
    -   `ORDER BY issue_year ASC, issue_month ASC`: This defines the sorting order *within* each partition. By ordering by year then month, we ensure that the chronologically earliest record for each card gets assigned `row_num = 1`.

2.  **Final `SELECT` Statement**:
    -   `WHERE row_num = 1`: This simple filter is all we need to select the launch month record for every card. It effectively pulls the top-ranked (i.e., earliest) row from each partition.
    -   `ORDER BY issued_amount DESC`: This sorts the final results to meet the output requirement of showing the cards with the highest launch month issuance first.

> [!TIP]
> Other ranking functions like `RANK()` or `DENSE_RANK()` could also be used here. However, `ROW_NUMBER()` is the most straightforward choice since we are guaranteed to have a unique launch month for each card, so ties are not a concern.

### 2.2. Method 2: Using `GROUP BY` and `JOIN`
This is a more traditional approach that works in older versions of SQL that may not support window functions. It involves first finding the launch date and then joining that information back to the original table.

> [!WARNING]
> While this method is functionally correct, it often requires multiple reads of the table (one for the aggregation and one for the join), which can be less performant than the single-pass window function approach.

```sql
WITH card_launch_dates AS (
  SELECT
    card_name,
    MIN(issue_year) AS first_year
  FROM monthly_cards_issued
  GROUP BY card_name
),
card_launch_months AS (
  SELECT
    m.card_name,
    cld.first_year,
    MIN(m.issue_month) AS first_month
  FROM monthly_cards_issued AS m
  JOIN card_launch_dates AS cld
    ON m.card_name = cld.card_name
    AND m.issue_year = cld.first_year
  GROUP BY m.card_name, cld.first_year
)
SELECT
  mci.card_name,
  mci.issued_amount
FROM monthly_cards_issued AS mci
JOIN card_launch_months AS clm
  ON mci.card_name = clm.card_name
  AND mci.issue_year = clm.first_year
  AND mci.issue_month = clm.first_month
ORDER BY mci.issued_amount DESC;
```

#### 2.2.1. Explanation

> [!CAUTION]
> This multi-step process is more verbose and logically complex. A common mistake is to only find the `MIN(issue_year)` and `MIN(issue_month)` in a single step, which is incorrect. For example, a card launched in 2/2022 would incorrectly be identified as 1/2022 if a record for 1/2023 exists, because `MIN(issue_month)` would be 1.

1.  **CTE 1 (`card_launch_dates`)**: This first finds the earliest *year* for each card.
2.  **CTE 2 (`card_launch_months`)**: This CTE joins back to the original table, filtering for records only in that `first_year`, and then finds the earliest *month* within that year. This correctly identifies the launch month and year for each card.
3.  **Final `SELECT`**: This performs a final join between the original table and the CTE containing the exact launch dates to retrieve the `issued_amount` for that specific month and year.

> [!IMPORTANT]
> The `ROW_NUMBER()` approach (Method 1) is demonstrably simpler, more readable, and generally more performant for this kind of problem. It should be your default choice in any modern SQL environment.
