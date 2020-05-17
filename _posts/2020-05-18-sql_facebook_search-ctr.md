---
layout: post
title:  "Search CTR | Facebook Data Science Interview SQL Question"
date:   2020-05-18 15:44:36 +0100
categories: facebook data science interview

---

## Search CTR

You're given a table that represents search results from searches on Facebook. The `query` column is the search term, `position` column represents each position the search result came in, and the `rating` column represents the human rating from 1 to 5 where 5 is high relevance and 1 is low relevance.

**`search_results` table**

| column    | type    |
| --------- | ------- |
| query     | varchar |
| result_id | integer |
| position  | integer |
| rating    | integer |

Each row in the `search_events` table represents a single search with the `has_clicked` column representing if a user clicked on a result or not. We have a hypothesis that the CTR is dependent on the search result rating.

**`search_events` table**

| column      | type    |
| ----------- | ------- |
| search_id   | integer |
| query       | varchar |
| has_clicked | boolean |

### Question: Write a query to return data to support or disprove this hypothesis.

We can define multiple metrics to understand relationship between search results and CTR.

Here are a few of the metrics I defined:

* **Average rating of the search.** *(It shows the quality of the overall search results. But it doesn't take into account anything on position. For instance, if there are 4 results from the search, and the top result is really relevant to a user with a score of 5, but the rest of the results are 1, we get really low average, but still show the most relevant result on the top which might cause user to click on the result.)*
* **Maximum rating of the search result.** *(As the name suggests, this shows the rating of the "best" search result. And again, this doesn't take into account the position of the search results. We might have the most relevant result in the query, but maybe at the bottom of the result list, so that the user doesn't click on the result)*
* **Has most relevant.** *(It shows whether there is any result with the highest relevance without taking into account the position.)*
* **Is most relevant top**. *(It shows whether there is any result with the highest relevance **and** whether it's at the top.)*

Once we have these metrics, we can then join them to `search_events` table to see the relationship between one of these metrics with CTR. In the example below I choose `avg_rating`. But it's worth exploring the other metrics too before jumpin into any conclusions.

### Solution

```sql
WITH base AS (
  SELECT 
    query
    , ROUND(AVG(rating),0) AS avg_rating
    , MAX(rating) AS max_rating
    , MAX(CASE WHEN rating = 5 THEN 1 ELSE 0 END) AS has_most_relevant
    , MAX(CASE WHEN rating = 5 AND position = 1 THEN 1 ELSE 0 END) AS is_most_relevant_top
  FROM search_results
  GROUP BY 1
)  
SELECT 
	avg_rating
	, COUNT(DISTINCT l1.query) AS searches
	, COUNT(DISTINCT CASE WHEN has_clicked THEN l1.query END) AS clicks
	, clicks/searches AS CTR
FROM base l1
INNER JOIN search_events l2 ON l2.query = l1.query
GROUP BY 1
```

