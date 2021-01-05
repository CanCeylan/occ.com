---
layout: post
title:  "Rolling Bank Transactions | Dropbox Data Science Interview SQL Question"
date:   2020-12-10 18:24:16 +0100
categories: dropbox data science interview
---

******

**`bank_transactions` table**

| column            | type     |
| ----------------- | -------- |
| user_id           | int      |
| created_at        | datetime |
| transaction_value | float    |

We're given a table bank transactions with three columns, `user_id`, `transaction_value` a deposit or withdrawal value determined if the value is positive or negative, and `created_at` time for each transaction.

Write a query to get the total three day rolling average for deposits by day.

**Output:**

| column            | type     |
| ----------------- | -------- |
| rolling_three_day | float    |
| dt                | datetime |



## Solution 1 (without using the window function)

Let's find the total daily transaction values for deposits first

```sql
SELECT
	created_at::date dt
	, SUM(transaction_value) daily_deposits
FROM bank_transactions
WHERE transaction_value > 0
GROUP BY 1
```

Here's an example output of the table

| dt         | daily_deposits |
| ---------- | -------------- |
| 2020-12-17 | 328            |
| 2020-12-16 | 211            |
| 2020-12-15 | 300            |
| 2020-12-14 | 287            |

If we use this table as a base table, and join with itself on `dt` we can get the previous 3 days' values;

```sql
WITH base AS (
  SELECT
    created_at::date dt
    , SUM(transaction_value) daily_deposits
  FROM bank_transactions
  WHERE transaction_value > 0
  GROUP BY 1
)
SELECT 
	*
FROM base l1
LEFT JOIN base l2 ON l2.dt <= l1.dt
					AND l2.dt > DATEADD('d', -3, l1.dt)
WHERE l1.dt = '2020-12-17'		
GROUP BY 1
```

| dt         | daily_deposits | dt         | daily_deposits |
| ---------- | -------------- | ---------- | -------------- |
| 2020-12-17 | 328            | 2020-12-17 | 328            |
| 2020-12-17 | 328            | 2020-12-16 | 211            |
| 2020-12-17 | 328            | 2020-12-15 | 300            |
|            |                |            |                |

Above table shows an example output for date = 2020-12-17. *(See, it doesn't include the value from 2020-12-14)*

```sql
WITH base AS (
  SELECT
    created_at::date dt
    , SUM(transaction_value) daily_deposits
  FROM bank_transactions
  WHERE transaction_value > 0
  GROUP BY 1
)
SELECT 
	l1.dt
	, SUM(l2.daily_deposits)/3 deposits_3d_rolling_avg
FROM base l1
LEFT JOIN base l2 ON l2.dt <= l1.dt
					AND l2.dt > DATEADD('d', -3, l1.dt)
WHERE l1.dt = '2020-12-17'		
GROUP BY 1
```