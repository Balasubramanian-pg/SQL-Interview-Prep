---
Created: 2025-06-28T18:34
Area:
  - MECE
Key Functions:
  - Distinct
  - Subquery
Company:
  -
Difficulty:
Status:
Category:
Sub category:
Question Link:
---
Assume you're given tables with information about TikTok user sign-ups and confirmations through email and text. New users on TikTok sign up using their email addresses, and upon sign-up, each user  
receives a text message confirmation to activate their account.  

Write a query to display the user IDs of those who did not confirm their sign-up on the first day, but confirmed on the second day.

Definition:

- `action_date` refers to the date when users activated their accounts and confirmed their sign-up through text messages.

### `emails` Table:

|   |   |
|---|---|
|**Column Name**|**Type**|
|email_id|integer|
|user_id|integer|
|signup_date|datetime|

### `emails` Example Input:

|   |   |   |
|---|---|---|
|**email_id**|**user_id**|**signup_date**|
|125|7771|06/14/2022 00:00:00|
|433|1052|07/09/2022 00:00:00|

### `texts` Table:

|   |   |
|---|---|
|**Column Name**|**Type**|
|text_id|integer|
|email_id|integer|
|signup_action|string ('Confirmed', 'Not confirmed')|
|action_date|datetime|

### `texts` Example Input:

|   |   |   |   |
|---|---|---|---|
|**text_id**|**email_id**|**signup_action**|**action_date**|
|6878|125|Confirmed|06/14/2022 00:00:00|
|6997|433|Not Confirmed|07/09/2022 00:00:00|
|7000|433|Confirmed|07/10/2022 00:00:00|

### Example Output:

**user_id**

---

1052

---

### Explanation:

Only User 1052 confirmed their sign-up on the second day.

  

## Solution

I apologize for the confusion earlier. Let me explain my approach step by step.

### Understanding the Problem

We have two tables, `emails` and `texts`, which contain information about TikTok user sign-ups and confirmations. The goal is to find users who did not confirm their sign-up on the first day but did confirm on the second day.

### Tables and Columns

1. **emails Table**:
    - `email_id`: Unique identifier for the email.
    - `user_id`: Unique identifier for the user.
    - `signup_date`: Date and time when the user signed up.
2. **texts Table**:
    - `text_id`: Unique identifier for the text.
    - `email_id`: Foreign key referencing the emails table.
    - `signup_action`: Status of the confirmation ('Confirmed' or 'Not Confirmed').
    - `action_date`: Date and time when the confirmation action occurred.

### Objective

Find users who:

1. Did not confirm their sign-up on the first day (i.e., no 'Confirmed' action on the same day as `signup_date`).
2. Did confirm their sign-up on the second day (i.e., a 'Confirmed' action on the day after `signup_date`).

### Approach

1. **Join Tables**: We need to join the `emails` and `texts` tables on `email_id` to associate signups with their confirmation actions.
2. **Filter Conditions**:
    - The confirmation action must be 'Confirmed'.
    - The confirmation date must be exactly one day after the signup date.
    - There should be no 'Confirmed' action on the same day as the signup date.
3. **Select User IDs**: Select the distinct `user_id` of those who meet the above conditions.

### Step-by-Step Explanation

1. **Join the Tables**:
    - We join the `emails` table with the `texts` table using the `email_id` column. This allows us to associate each user's signup date with their confirmation actions.
2. **Filter for Confirmations on the Second Day**:
    - We filter for rows where `signup_action` is 'Confirmed'.
    - We then ensure that the confirmation date (`action_date`) is exactly one day after the signup date (`signup_date`).
3. **Exclude Users Who Confirmed on the First Day**:
    - We use a `NOT EXISTS` subquery to exclude users who have any 'Confirmed' action on the same day as their signup date.
    - The subquery checks for any 'Confirmed' action in the `texts` table where the `action_date` is the same as the `signup_date`.
4. **Select Distinct User IDs**:
    - We select the distinct `user_id` from the filtered and joined tables to get the list of users who meet our criteria.

### SQL Query

Here is the SQL query that implements the above approach:

```SQL
 Copy
SELECT DISTINCT e.user_id
FROM emails e
JOIN texts t ON e.email_id = t.email_id
WHERE t.signup_action = 'Confirmed'
AND (DATE(t.action_date) - DATE(e.signup_date)) = 1
AND NOT EXISTS (
    SELECT 1
    FROM texts t2
    WHERE t2.email_id = e.email_id
    AND t2.signup_action = 'Confirmed'
    AND (DATE(t2.action_date) - DATE(e.signup_date)) = 0
)
```

### Explanation of the Query

1. **JOIN Clause**:
    - We join the `emails` table with the `texts` table on `email_id` to link each user's signup date with their confirmation actions.
2. **WHERE Clause**:
    - `t.signup_action = 'Confirmed'`: Filters for confirmation actions that are 'Confirmed'.
    - `(DATE(t.action_date) - DATE(e.signup_date)) = 1`: Ensures that the confirmation action occurred exactly one day after the signup date.
    - `NOT EXISTS (...)`: Ensures that there is no 'Confirmed' action on the same day as the signup date.
3. **Subquery**:
    - The subquery checks if there is any 'Confirmed' action on the same day as the signup date. If such a record exists, the `NOT EXISTS` condition will exclude the user from the result set.
4. **SELECT DISTINCT**:
    - We select distinct `user_id` to ensure that each user is listed only once, even if they have multiple confirmations on the second day.

### Example Walkthrough

Let's walk through the example provided:

- **emails Table**:
    - User 25777106 signed up on 06/14/2022.
    - User 1052 signed up on 07/09/2022.
- **texts Table**:
    - User 25777106 confirmed on 06/14/2022 (same day as signup, so excluded).
    - User 1052 had a 'Not Confirmed' action on 07/09/2022 (first day) and a 'Confirmed' action on 07/10/2022 (second day).

The query would include user 1052 because they did not confirm on the first day but did confirm on the second day. User 25777106 is excluded because they confirmed on the first day.

### Final Answer

Here is the final SQL query to find users who did not confirm their sign-up on the first day but confirmed on the second day:

```SQL
 Copy
SELECT DISTINCT e.user_id
FROM emails e
JOIN texts t ON e.email_id = t.email_id
WHERE t.signup_action = 'Confirmed'
AND (DATE(t.action_date) - DATE(e.signup_date)) = 1
AND NOT EXISTS (
    SELECT 1
    FROM texts t2
    WHERE t2.email_id = e.email_id
    AND t2.signup_action = 'Confirmed'
    AND (DATE(t2.action_date) - DATE(e.signup_date)) = 0
)
```

This query will return the user IDs of those who confirmed their sign-up on the second day but not on the first day.

```SQL
SELECT DISTINCT e.user_id
FROM emails e
JOIN texts t ON e.email_id = t.email_id
WHERE t.signup_action = 'Confirmed'
AND (DATE(t.action_date) - DATE(e.signup_date)) = 1
AND NOT EXISTS (
    SELECT 1
    FROM texts t2
    WHERE t2.email_id = e.email_id
    AND t2.signup_action = 'Confirmed'
    AND (DATE(t2.action_date) - DATE(e.signup_date)) = 0
)
```