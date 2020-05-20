---
layout: post
title:  "Create Dummies from Array of Objects in Pandas"
date:   2020-05-20 15:44:36 +0100
categories: python pandas
---

## Create Dummies from Array of Objects in Pandas

While I was navigating interesting datasets in Kaggle, I've stambled upon the [movies dataset](https://www.kaggle.com/rounakbanik/the-movies-dataset?select=movies_metadata.csv). 

And the main dataset(`movies_metadata.csv`) contains metadata for all 45,000 movies listed in the Full MovieLens Dataset. 

`genres` column specifically reminded me that a challenging problem Data Scientists usually run into. Dealing with unstructured data. 

Here's an example value from `genres` column.

```[{'id': 16, 'name': 'Animation'}, {'id': 35, 'name': 'Comedy'}, {'id': 10751, 'name': 'Family'}]```

This value suggests that, this particular movie represents 3 different genres. But since the data is not structured, it is for instance really challenging to look at questions like
* How many different genres are there?
* What proportion of movies have Comedy genres in their list?
* Do popularity/revenue/votes change depending on genres?
and many more.

**In this post, I will show the steps to turn this array of objects to dummy columns for each genre.**

**Steps**

1. Convert `string` object to `list` object in Pandas DataFrame
1. Only select necessary info within the `list` object and create a clean column
1. Convert newly created column into dummies
1. Merge it with the original DataFrame



## 1. Convert `string` object to `list` object in Pandas DataFrame

Let's read the data and look at the `genres` field on the first row:


```python
import pandas as pd

df = pd.read_csv('./data/movies_metadata.csv')
df.iloc[0]['genres']
```


    "[{'id': 16, 'name': 'Animation'}, {'id': 35, 'name': 'Comedy'}, {'id': 10751, 'name': 'Family'}]"

Let's try to access the first element of the first `genres` value. _(It should be ```{'id': 16, 'name': 'Animation'}```)_


```python
df.iloc[0]['genres'][0]
```


    '['

As you see, we got `[`, because that column is interpreted as `string` object.


```python
df[['genres']].dtypes
```


    genres    object
    dtype: object

## 2. Only select necessary info within the list object and create a clean column
_How to convert string object into a list in python?_

Thankfully, there is a class called `ast` (Abstract Syntax Trees) in python which helps Python applications to process trees of the Python abstract syntax grammar. 

In there, we are specifically interested in `ast.literal_eval` function. [Here](https://docs.python.org/3/library/ast.html#ast.literal_eval) in the documentation it says
> Safely evaluate an expression node or a string containing a Python literal or container display. The string or node provided may only consist of the following Python literal structures: strings, bytes, numbers, tuples, lists, dicts, sets, booleans, and None.
This can be used for safely evaluating strings containing Python values from untrusted sources without the need to parse the values oneself. It is not capable of evaluating arbitrarily complex expressions, for example involving operators or indexing.

So let's apply `ast.literal_eval` function to our Pandas DataFrame to see how does it change our `string` object.


```python
from ast import literal_eval
```


```python
df['new_genres'] = df['genres'].apply(literal_eval)
df.head().iloc[0]['new_genres']
```


    [{'id': 16, 'name': 'Animation'},
     {'id': 35, 'name': 'Comedy'},
     {'id': 10751, 'name': 'Family'}]

If you refer back to the first cell we run (`df.iloc[0]['genres']`) you see that the content is the same, but the output looks very different in terms of structure, suggesting `list` object rather than `string` object.

Let's verify that;


```python
print(f"Type of the genres column:\t{type(df.iloc[0]['genres'])}")
print(f"Type of the new_genres column:\t{type(df.iloc[0]['new_genres'])}")
```

    Type of the genres column:	<class 'str'>
    Type of the new_genres column:	<class 'list'>


Looks great! 

Let's think about how do we want to use the information within this column...

For this particular exercise, we are more interested in creating dummy columns with the genre names. So ideally we want something like the following:

`[Animation, Comedy, Family]`

We can use python's inline list iterator to generate that object for the first row:


```python
[element['name'] for element in df.iloc[0]['new_genres']]
```


    ['Animation', 'Comedy', 'Family']

That's exactly what we want in Step 2, let's apply this to all the rows in the DataFrame. _We should also check the edge case here and if the `new_genres` column is not a list for any reason we should return an empty list_


```python
df['genre_names'] = df['new_genres']\
                        .apply(lambda x: [e['name'] for e in x] if isinstance(x,list) else [])
df.head()[['genres','genre_names']]
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>genres</th>
      <th>genre_names</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>[{'id': 16, 'name': 'Animation'}, {'id': 35, '...</td>
      <td>[Animation, Comedy, Family]</td>
    </tr>
    <tr>
      <th>1</th>
      <td>[{'id': 12, 'name': 'Adventure'}, {'id': 14, '...</td>
      <td>[Adventure, Fantasy, Family]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[{'id': 10749, 'name': 'Romance'}, {'id': 35, ...</td>
      <td>[Romance, Comedy]</td>
    </tr>
    <tr>
      <th>3</th>
      <td>[{'id': 35, 'name': 'Comedy'}, {'id': 18, 'nam...</td>
      <td>[Comedy, Drama, Romance]</td>
    </tr>
    <tr>
      <th>4</th>
      <td>[{'id': 35, 'name': 'Comedy'}]</td>
      <td>[Comedy]</td>
    </tr>
  </tbody>
</table>


## 3. Convert newly created column into dummies

Now we have much clean genre data, but they are still not easily accessible for further data exploration.

Remember, our goal is to create dummy columns for the genres and assign them to the corresponding movies.

There are 2 different way to do this, the first one runs much faster. _(92ms vs 12.7s)_


```python
genres_df = pd.DataFrame(df['genre_names'].tolist())
genres_df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Animation</td>
      <td>Comedy</td>
      <td>Family</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Adventure</td>
      <td>Fantasy</td>
      <td>Family</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Romance</td>
      <td>Comedy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Comedy</td>
      <td>Drama</td>
      <td>Romance</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Comedy</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>

```python
genres_df = df['genre_names'].apply(pd.Series)
genres_df.head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Animation</td>
      <td>Comedy</td>
      <td>Family</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Adventure</td>
      <td>Fantasy</td>
      <td>Family</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Romance</td>
      <td>Comedy</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Comedy</td>
      <td>Drama</td>
      <td>Romance</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Comedy</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
So now, we have `N` number of columns (number of distinct genres) for each movie, and non null columns represent each of the movies' genres. 

For instance our first movie is the following:


```python
genres_df.head(1)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
      <th>5</th>
      <th>6</th>
      <th>7</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Animation</td>
      <td>Comedy</td>
      <td>Family</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
In order to create dummies for each of these movies, we need to [stack](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.stack.html) to reshape our DataFrame.


```python
stacked_genres = genres_df.stack()
stacked_genres
```


    0      0    Animation
           1       Comedy
           2       Family
    1      0    Adventure
           1      Fantasy
                  ...    
    45461  1       Family
    45462  0        Drama
    45463  0       Action
           1        Drama
           2     Thriller
    Length: 91106, dtype: object

So, for each of the genres for each movie we now have a row. Here are the rows corresponding our first movie:


```python
stacked_genres[0]
```


    0    Animation
    1       Comedy
    2       Family
    dtype: object

So we converted our cleaned list `[Animation, Comedy, Family]` to 3 rows, each represents only 1 particular genre. We are now ready to convert this to dummies!


```python
raw_dummies = pd.get_dummies(stacked_genres)
raw_dummies.head()[['Animation','Comedy','Family','Crime','Documentary']]
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>Animation</th>
      <th>Comedy</th>
      <th>Family</th>
      <th>Crime</th>
      <th>Documentary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="3" valign="top">0</th>
      <th>0</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th rowspan="2" valign="top">1</th>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
Here's how the new DataFrame looks like, for simplicity I only selected a few columns. But in actual DataFrame, we have _all_ the genres. 

And if you focus on the first 3 rows, you see that `Animation`,`Comedy` and `Family` columns are 1, and the rest are 0.

Again, remember our goal, we need these dummy columns but we only need *one* row per movie. So how do we do that?

We just aggregate the newly created DataFrame by summing on index level.


```python
genre_dummies = raw_dummies.sum(level=0)
genre_dummies[['Animation','Comedy','Family','Crime','Documentary']].head()
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Animation</th>
      <th>Comedy</th>
      <th>Family</th>
      <th>Crime</th>
      <th>Documentary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>


## 4. Merge it with the original DataFrame

That's the simplest step after using `ast.literal_eval`, `stack`, and `get_dummies` functions.


```python
ndf = pd.concat([df, genre_dummies], axis=1)
ndf.head()[['original_title','genres','Animation','Comedy','Family','Crime','Documentary']]
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>original_title</th>
      <th>genres</th>
      <th>Animation</th>
      <th>Comedy</th>
      <th>Family</th>
      <th>Crime</th>
      <th>Documentary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Toy Story</td>
      <td>[{'id': 16, 'name': 'Animation'}, {'id': 35, '...</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Jumanji</td>
      <td>[{'id': 12, 'name': 'Adventure'}, {'id': 14, '...</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Grumpier Old Men</td>
      <td>[{'id': 10749, 'name': 'Romance'}, {'id': 35, ...</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Waiting to Exhale</td>
      <td>[{'id': 35, 'name': 'Comedy'}, {'id': 18, 'nam...</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Father of the Bride Part II</td>
      <td>[{'id': 35, 'name': 'Comedy'}]</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>

