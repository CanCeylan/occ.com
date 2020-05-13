---
layout: post
title:  "Concatenate two lists element by element"
date:   2020-05-13 21:44:36 +0100
categories: python pandas pivot
---

I encountered this issue when I tried to clean/fix the columns of my pandas DataFrame after I pivoted with more than one values.

Let me quickly demonstrate what I mean by that.

Let's create a pandas DataFrame, where each user might have multiple rows depending on the devices they use. And we have two metrics, `time_spent` and `revenue`

```python
df = pd.DataFrame({'first_name': ['Alice', 'Alice', 'Bill', 'Bill', 'Steve',
                           'Steve'],
                   'device': ['ios', 'web', 'ios', 'web', 'ios', 'web'],
                   'time_spent': [90,20,65,20,80,30],
                   'revenue': [220,55,160,70,120,80]})
```

|      | first_name | device | time_spent | revenue |
| ---: | ---------: | -----: | ---------: | ------: |
|    0 |      Alice |    ios |         90 |     220 |
|    1 |      Alice |    web |         20 |      55 |
|    2 |       Bill |    ios |         65 |     160 |
|    3 |       Bill |    web |         20 |      70 |
|    4 |      Steve |    ios |         80 |     120 |
|    5 |      Steve |    web |         30 |      80 |

Here's how I summarise this table and get a row per each user with [Pandas' pivot function](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.pivot.html) 

```python
pivot_df = df.pivot(index='first_name', columns='device', values=['time_spent','revenue'])
pivot_df
```

|                | time_spent |         | revenue |         |
| -------------: | ---------: | ------: | ------: | ------: |
|     **device** |    **ios** | **web** | **ios** | **web** |
| **first_name** |            |         |         |         |
|          Alice |         90 |      20 |     220 |      55 |
|           Bill |         65 |      20 |     160 |      70 |
|          Steve |         80 |      30 |     120 |      80 |

So we get a DataFrame with MultiIndex. And it's quite challenging to deal with especially if you are going to merge this table with some other tables.

```
MultiIndex([('time_spent', 'ios'),
            ('time_spent', 'web'),
            (   'revenue', 'ios'),
            (   'revenue', 'web')],
           names=[None, 'device'])
```

So to handle this issue, you need to remove the MultiIndex in your pandas DataFrame. 

The easiest way to do is just overwriting the columns by passing a new list of strings with 

```python
df.columns = ['t_ios','t_web','r_ios','r_web']
```

But obviously it's a super manual way and it's not scalable and it's error prone (*if you happen to change the metric names or devices etc.*)

But we can achieve this by using `zip` and `droplevel`  functions in python!

Let's see how does `droplevel` function work in our case:

```python
pivot_df.columns.droplevel()
>> Index(['ios', 'web', 'ios', 'web'], dtype='object', name='device')

pivot_df.columns.droplevel(1)
>> Index(['time_spent', 'time_spent', 'revenue', 'revenue'], dtype='object')
```

So if we merge these two lists element wise and build a string array, we can easily assign it to our DataFrame's columns.

```python
# initializing lists   
metrics = pivot_df.columns.droplevel(1)
devices = pivot_df.columns.droplevel(0)
  
# using list comprehension + zip() 
# interlist element concatenation 
new_columns = [metric + '_' + device for metric, device in zip(metrics, devices)]
new_columns
>> ['time_spent_ios', 'time_spent_web', 'revenue_ios', 'revenue_web']
```

We then assign our new variable to the DataFrame's columns `pivot_df.columns = new_columns` and here's what we get 👌

|                | time_spent_ios | time_spent_web | revenue_ios | revenue_web |
| -------------: | -------------: | -------------: | ----------: | ----------: |
| **first_name** |                |                |             |             |
|          Alice |             90 |             20 |         220 |          55 |
|           Bill |             65 |             20 |         160 |          70 |
|          Steve |             80 |             30 |         120 |          80 |





