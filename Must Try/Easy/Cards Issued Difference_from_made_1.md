---
Created: 2025-06-28T18:34
Area:
  - Statistical Aggregation
Key Functions:
  - MAX
Company:
  -
Difficulty:
Status:
Category:
Sub category:
Question Link:
---
Your team at JPMorgan Chase is preparing to launch a new credit card,  
and to gain some insights, you're analyzing how many credit cards were  
issued each month.  

Write a query that outputs the name of each credit card and the  
difference in the number of issued cards between the month with the  
highest issuance cards and the lowest issuance. Arrange the results  
based on the largest disparity.  

### `monthly_cards_issued` Table:

|   |   |
|---|---|
|Column Name|Type|
|card_name|string|
|issued_amount|integer|
|issue_month|integer|
|issue_year|integer|

### `monthly_cards_issued` Example Input:

|   |   |   |   |
|---|---|---|---|
|card_name|issued_amount|issue_month|issue_year|
|Chase Freedom Flex|55000|1|2021|
|Chase Freedom Flex|60000|2|2021|
|Chase Freedom Flex|65000|3|2021|
|Chase Freedom Flex|70000|4|2021|
|Chase Sapphire Reserve|170000|1|2021|
|Chase Sapphire Reserve|175000|2|2021|
|Chase Sapphire Reserve|180000|3|2021|

### Example Output:

|   |   |
|---|---|
|card_name|difference|
|Chase Freedom Flex|15000|
|Chase Sapphire Reserve|10000|

Chase Freedom Flex's best month was 70k cards issued and the worst month was 55k cards, so the difference is 15k cards.

Chase Sapphire Reserve’s best month was 180k cards issued and the worst month was 170k cards, so the difference is 10k cards.

  

  

### SQL Query

```SQL
SELECT
  card_name,
  MAX(issued_amount) - MIN(issued_amount) AS difference
FROM
  monthly_cards_issued
GROUP BY
  card_name
ORDER BY
  difference DESC;
```

### Explanation

1. `**SELECT card_name, MAX(issued_amount) - MIN(issued_amount) AS difference**`
    - We select the `card_name` to identify which card we are analyzing.
    - The core of the logic is `MAX(issued_amount) - MIN(issued_amount)`. We use the `MAX()` and `MIN()` aggregate functions to find the highest and lowest values in the `issued_amount` column for each card. Subtracting the minimum from the maximum gives us the required difference.
    - `AS difference` assigns a clear and readable alias to our calculated column.
2. `**FROM monthly_cards_issued**`
    - This specifies that we are retrieving data from the `monthly_cards_issued` table.
3. `**GROUP BY card_name**`
    - This is a crucial step. It groups all the rows with the same `card_name` together. This ensures that the `MAX()` and `MIN()` functions are applied independently to each credit card's set of monthly data, rather than across the entire table.
4. `**ORDER BY difference DESC**`
    - Finally, this clause sorts the final result set. We order by our calculated `difference` column in descending (`DESC`) order, so the card with the largest disparity between its best and worst month appears first, as requested.