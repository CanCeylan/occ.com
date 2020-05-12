---
layout: post
title:  "Acceptance Rate | Facebook Data Science Interview SQL Question"
date:   2020-05-12 20:44:36 +0100
categories: facebook data science interview

---

## Acceptance Rate

`friend_requests` table

| column       | type     |
| ------------ | -------- |
| requester_id | integer  |
| requested_id | integer  |
| created_at   | datetime |

`friend_accepts` table

| column       | type     |
| ------------ | -------- |
| acceptor_id  | integer  |
| requester_id | integer  |
| created_at   | datetime |

You're given two tables. *`friend_requests`* holds all the friend requests made and *`friend_accepts`* is all of acceptances.

Write a query to find the overall acceptance rate of friend requests.

**Write a query to find the overall acceptance rate of friend requests.**

## Solution

~~~sql
WITH base AS (
  SELECT
    l1.requester_id 
    , l1.requested_id
    , CASE WHEN l2.requester_id IS NOT NULL THEN 1 ELSE 0 END accepted
  FROM friend_requests l1
  LEFT JOIN friend_accepts l2 ON l2.acceptor_id = l1.requested_id
)
SELECT 
	SUM(CASE WHEN accepted = 1 THEN 1 END) accepts
	, COUNT(*) requests
	, accepts/requests acceptance_rate
FROM base

~~~

