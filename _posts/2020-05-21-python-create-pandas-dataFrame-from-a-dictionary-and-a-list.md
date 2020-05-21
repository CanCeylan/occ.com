---
layout: post
title:  "Create Pandas DataFrame from a Dictionary and a List"
date:   2020-05-21 12:44:36 +0100
categories: python pandas
---

```python
import pandas as pd
```

# Create a Pandas DataFrame from Lists

## Row Oriented List


```python
row_list = [
    ('Marissa', 21, 18,16),
    ('Osman', 3, 2, 4),
    ('Bill', 17, 14, 15),    
]
column_names = ['user_name','jan','feb','mar']
row_list
```


    [('Marissa', 21, 18, 16), ('Osman', 3, 2, 4), ('Bill', 17, 14, 15)]


```python
pd.DataFrame(row_list, columns=column_names) #pd.DataFrame.from_records(row_list, columns=column_names)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_name</th>
      <th>jan</th>
      <th>feb</th>
      <th>mar</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marissa</td>
      <td>21</td>
      <td>18</td>
      <td>16</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Osman</td>
      <td>3</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bill</td>
      <td>17</td>
      <td>14</td>
      <td>15</td>
    </tr>
  </tbody>
</table>


## Column Oriented List


```python
column_list = [
    ('user_name', ['Marissa','Osman','Bill']),
    ('jan', [21,3,17]),
    ('feb', [18,2,14]),
    ('mar', [16,4,15])
]
column_list
```


    [('user_name', ['Marissa', 'Osman', 'Bill']),
     ('jan', [21, 3, 17]),
     ('feb', [18, 2, 14]),
     ('mar', [16, 4, 15])]


```python
dict(column_list)
```


    {'user_name': ['Marissa', 'Osman', 'Bill'],
     'jan': [21, 3, 17],
     'feb': [18, 2, 14],
     'mar': [16, 4, 15]}


```python
pd.DataFrame(dict(column_list))
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_name</th>
      <th>jan</th>
      <th>feb</th>
      <th>mar</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marissa</td>
      <td>21</td>
      <td>18</td>
      <td>16</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Osman</td>
      <td>3</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bill</td>
      <td>17</td>
      <td>14</td>
      <td>15</td>
    </tr>
  </tbody>
</table>


# Create a Pandas DataFrame from Dictionaries

## Column Oriented Dictionary


```python
column_dict = {
    'user_name':['Marissa','Osman','Bill'],
    'jan': [21,3,17],
    'feb': [18,2,14],
    'mar': [16,4,15]
}
column_dict
```


    {'user_name': ['Marissa', 'Osman', 'Bill'],
     'jan': [21, 3, 17],
     'feb': [18, 2, 14],
     'mar': [16, 4, 15]}


```python
pd.DataFrame(column_dict)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_name</th>
      <th>jan</th>
      <th>feb</th>
      <th>mar</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marissa</td>
      <td>21</td>
      <td>18</td>
      <td>16</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Osman</td>
      <td>3</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bill</td>
      <td>17</td>
      <td>14</td>
      <td>15</td>
    </tr>
  </tbody>
</table>


## Row Oriented Dictionary


```python
row_dict = [
    {'user_name': 'Marissa', 'jan': 21, 'feb':18, 'mar':16},
    {'user_name': 'Osman', 'jan': 3, 'feb':2, 'mar':4},
    {'user_name': 'Bill', 'jan': 16, 'feb':14, 'mar':15}
]
row_dict
```


    [{'user_name': 'Marissa', 'jan': 21, 'feb': 18, 'mar': 16},
     {'user_name': 'Osman', 'jan': 3, 'feb': 2, 'mar': 4},
     {'user_name': 'Bill', 'jan': 16, 'feb': 14, 'mar': 15}]


```python
pd.DataFrame(row_dict)
```

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_name</th>
      <th>jan</th>
      <th>feb</th>
      <th>mar</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Marissa</td>
      <td>21</td>
      <td>18</td>
      <td>16</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Osman</td>
      <td>3</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bill</td>
      <td>16</td>
      <td>14</td>
      <td>15</td>
    </tr>
  </tbody>
</table>



