---
layout: post
title:  "Post Events | Facebook Data Science Interview SQL Question"
date:   2020-05-12 20:44:36 +0100
categories: jekyll update
---
## Post Success

In the table below (`post_events` table), column *`event_name`* represents either ('enter', 'post', 'cancel') for when a user starts a post (enter), ends up canceling it (cancel), or ends up posting it (post).

| column_name | type     |
| ----------- | -------- |
| user_id     | int      |
| created_at  | datetime |
| event_name  | varchar  |

**Write a query to get the post success rate for each day over the past week.**

Sample data:

| user_id | created_at | event_name |
| ------- | ---------- | ---------- |
| 123     | 2019-01-01 | enter      |
| 123     | 2019-01-01 | post       |
| 456     | 2019-01-02 | enter      |
| 456     | 2019-01-02 | cancel     |

## Solution

Let's first define the post success rate on a day (`SR_daily`).

``` SR_daily = # users posting a post / # users starting a post``` *(per day)*

~~~sql
SELECT 
	created_at::date day
	, COUNT(DISTINCT CASE WHEN event_name = 'enter' THEN user_id END) post_start
	, COUNT(DISTINCT CASE WHEN event_name = 'post' THEN user_id END) post_complete
FROM post_events
WHERE created_at > DATEADD('d', -7, current_date)
~~~


