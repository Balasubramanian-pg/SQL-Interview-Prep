# Mask Credit Card Numbers

## 1. Overview
This document explains a common SQL interview question that tests your knowledge of string manipulation functions. The task is to mask a sensitive piece of data, like a credit card number, leaving only the last few digits visible for identification purposes.

## 2. Problem Statement

### 2.1. The Scenario
You are given a `cards` table with a `card_number` column.

**cards:**
| card_number      |
|------------------|
| 1234567890125678 |
| 9876543210986189 |
| 1111222233334444 |

### 2.2. The Requirement
Write a SQL query that returns the card numbers in a masked format. The first 12 digits should be replaced with asterisks (`*`), and only the last 4 digits should be visible.

**Expected Output:**
| masked_card_number |
|--------------------|
| \*\*\*\*\*\*\*\*\*\*\*\*5678   |
| \*\*\*\*\*\*\*\*\*\*\*\*6189   |
| \*\*\*\*\*\*\*\*\*\*\*\*4444   |

## 3. Setup Script
You can use the following T-SQL script to create the table and populate it with sample data.

```sql
-- Create the table
CREATE TABLE cards (
    card_number VARCHAR(16)
);
GO

-- Insert sample data
INSERT INTO cards (card_number) VALUES
('1234567890125678'),
('9876543210986189'),
('1111222233334444');
GO

-- Verify the initial data
SELECT * FROM cards;
```

## 4. Solution and Explanation
The solution involves combining two parts: a string of masking characters and the last four digits of the original card number. This can be achieved using standard string functions.

-   **`REPLICATE('x', 12)`**: The `REPLICATE` function repeats a given string a specified number of times. Here, we repeat the asterisk character `*` twelve times to create the masked portion.
-   **`RIGHT(card_number, 4)`**: The `RIGHT` function extracts a specified number of characters from the right side of a string. Here, it retrieves the last 4 digits of the `card_number`.
-   **`CONCAT()` or `+`**: We use a concatenation function or the `+` operator to join the replicated mask string with the extracted last four digits.

### 4.1. The SQL Query
```sql
SELECT
    CONCAT(REPLICATE('*', 12), RIGHT(card_number, 4)) AS masked_card_number
FROM
    cards;
```
