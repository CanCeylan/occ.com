---
layout: post
title:  "Search Ratings | Facebook Data Science Interview SQL Question"
date:   2020-05-15 20:44:36 +0100
categories: facebook data science interview

---

## Search Ratings

**`search_results` table**

| column    | type    |
| --------- | ------- |
| query     | varchar |
| result_id | integer |
| position  | integer |
| rating    | integer |

You're given a table that represents search results from searches on Facebook. The *query* column is the search term, *position* column represents each position the search result came in, and the *rating* column represents the human rating of the search result from 1 to 5 where 5 is high relevance and 1 is low relevance.

1. **Write a query to compute a metric to measure the quality of the search results for each query.** 

2. **You want to be able to compute a metric that measures the precision of the ranking system based on position. For example, if the results for dog and cat are....**

| query | result_id | position | rating | notes             |
| ----- | --------- | -------- | ------ | ----------------- |
| dog   | 1000      | 1        | 2      | picture of hotdog |
| dog   | 998       | 2        | 4      | dog walking       |
| dog   | 342       | 3        | 1      | zebra             |
| cat   | 123       | 1        | 4      | picture of cat    |
| cat   | 435       | 2        | 2      | cat memes         |
| cat   | 545       | 3        | 1      | pizza shops       |

...we would rank 'cat' as having a better search result ranking precision than 'dog' based on the correct sorting by rating.

**Write a query to create a metric that can validate and rank the queries by their search result precision.**



## Solution

Search results will be the *best* when `rating`*(DESC)* and `position`*(ASC)* fields have the correct orders. As in, in the example above, when `position` is `1` if the corresponding rating was the _highest_ then it would be a good search result.

So I would create a new column to indicate the descending order in `rating` column first, for each query.

```sql
SELECT 
	*
	, ROW_NUMBER() OVER(PARTITION BY query ORDER BY rating DESC) desc_rating
FROM search_results
```

That would give me the following table;

| query | result_id | position | rating | desc_rating |
| ----- | --------- | -------- | ------ | ----------- |
| dog   | 1000      | 1        | 2      | 2           |
| dog   | 998       | 2        | 4      | 1           |
| dog   | 342       | 3        | 1      | 3           |
| cat   | 123       | 1        | 4      | 1           |
| cat   | 435       | 2        | 2      | 2           |
| cat   | 545       | 3        | 1      | 3           |

Then I would define the _precision_ metric either *Mean Absolute Error* (MAE) or *Root Mean Squared Error* (RMSE). It's always useful to explain the differences between MAE and RMSE and use the appropriate one depending on your scenario.

#### Similarities of Mean Absolute Error and Root Mean Squared Error

* Indifferent from direction of errors
* Expresses average model prediction error in units of the variable of interest
* Can range from 0 to ∞
* Negatively orianted scores. *(Lower is better)*

#### Differences of Mean Absolute Error and Root Mean Squared Error

* RMSE gives relatively high weight to large errors. *(Since the errors are squared before they are averaged)*



In our case, if we assume that the number of search results are not too many, and the ratings are from 1-5, it would probably not make any marginal difference from MAE to RMSE, so I will go with MAE.

```sql
WITH base AS (
  SELECT 
    *
    , ROW_NUMBER() OVER(PARTITION BY query ORDER BY rating DESC) desc_rating
    , position - desc_rating residual
    , COUNT(DISTINCT result_id) n
  FROM search_results
  GROUP BY 1,2,3,4
)
SELECT 
	*
  , SUM(residual) OVER(PARTITION BY query) sum_residual
  , sum_residual/n mae
FROM base
```

