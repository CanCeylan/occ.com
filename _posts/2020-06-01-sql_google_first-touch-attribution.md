---
layout: post
title:  "First Touch Attribution | Google Data Science Interview SQL Question"
date:   2020-06-01 11:24:36 +0100
categories: google data science interview
---

***`attribution`*** table

| column     | type     |
| ---------- | -------- |
| id         | int      |
| created_at | datetime |
| session_id | int      |
| channel    | varchar  |
| conversion | boolean  |

***`user_sessions`*** table

| column     | user_id |
| ---------- | ------- |
| session_id | int     |
| user_id    | int     |

The schema above is for a retail online shopping company consisting of two tables, *attribution* and *user_sessions.* 

- The attribution table logs a session visit for each row.
- If *conversion* is *true*, then the user converted to buying on that session.
- The *channel* column represents which advertising platform the user was attributed to for that specific session.
- Lastly the **`user_sessions`** table maps many to one session visits back to one user.

First touch attribution is defined as the channel to which the converted user was associated with when they first discovered the website.

Calculate the first touch attribution for each *user_id* that converted.

**Example output:**

| user_id | channel  |
| ------- | -------- |
| 123     | facebook |
| 145     | google   |
| 153     | facebook |
| 172     | organic  |
| 173     | email    |

## Solution 1

Let's find the converted sessions by user first:

```sql
SELECT 
	l2.user_id
	, MIN(l1.created_at) first_converted_session_ts
FROM attribution l1
INNER JOIN user_sessions l2 ON l2.user_id = l1.user_id
WHERE l1.conversion IS TRUE
GROUP BY 1
```

This query gives us the timestamp of a firstly converted session by user. Let's join that back to `attribution` table to get the `channel` information.

```sql
WITH cte_first_converted_session AS (
  SELECT 
    l2.user_id
    , MIN(l1.created_at) first_converted_session_ts
  FROM attribution l1
  INNER JOIN user_sessions l2 ON l2.user_id = l1.user_id
  WHERE l1.conversion IS TRUE
  GROUP BY 1
)
SELECT
	l1.user_id
	, l2.channel
FROM cte_first_converted_session l1
INNER JOIN attribution l2 ON l2.user_id = l1.user_id
													AND l2.created_at = l1.first_converted_session_ts
```

## Solution 2

We can also do this by using window functions by calculating the row number of each converted session by user and then simply selecting the first row (ordered by session creation date.)

```sql
WITH cte_first_converted_session AS (
  SELECT 
    l1.user_id
    , l2.channel
    , ROW_NUMBER() OVER(PARTITION BY l1.user_id ORDER BY l1.created_at) rn
  FROM attribution l1
  INNER JOIN user_sessions l2 ON l2.user_id = l1.user_id
  WHERE l1.conversion IS TRUE
  GROUP BY 1
)
SELECT 
	user_id
	, channel
FROM cte_first_converted_session
WHERE rn = 1
```

