---
layout: post
title:  "Student Attendance | Facebook Data Science Interview SQL Question"
date:   2020-05-15 20:44:36 +0100
categories: facebook data science interview
---

## Student Attendance

**`attendance_events` table**

| column     | type    |
| ---------- | ------- |
| date       | date    |
| student_id | integer |
| attendance | bool    |

**`all_students` table**

| column        | type    |
| ------------- | ------- |
| student_id    | integer |
| school_id     | integer |
| grade_level   | integer |
| date_of_birth | date    |
| hometown      | string  |

**Question 1: What percent of students attend school on their birthday?**

* If there is a row for every day for every student with attendance data, we can join on `attendance_event` table since we can cover all the days *(See solution 1.1)*. But if, in `attendance_events` table, there are only rows when a student attend to a school, we should only join on `student_id` and do the aggregation on `SELECT` statement *(See solution 1.2)*.

**Solution 1.1**

```sql
SELECT 
	COUNT(DISTINCT l1.student_id) AS all_students
	, COUNT(DISTINCT CASE WHEN l2.attendance THEN l1.student_ID END) AS bday_students
	, bday_students/all_students AS perc_bday_attendance
FROM attendance_events l1
LEFT JOIN all_students l2 ON l2.student_id = l1.student_id
														AND DAY(l1.date_of_birth) = DAY(l2.date)
														AND MONTH(l1.date_of_birth) = MONTH(l2.date)
```

**Solution 1.2**

```sql
SELECT 
	COUNT(DISTINCT l1.student_id) AS cnt_std
	, COUNT(DISTINCT CASE WHEN DAY(l2.date_of_birth) = DAY(l1.date)
         										AND MONTH(l2.date_of_birth) = MONTH(l1.date)
         										AND l1.attendance THEN l1.student_id END) AS cnt_std_bday
	, cnt_std_bday/cnt_std AS perc_std_attendance
FROM attendance_events l1
LEFT JOIN all_students l2 ON l2.student_id = l1.student_id
```



**Question 2: Which grade level had the largest drop in attendance between yesterday and today?**

* If we are asked to do without using window function. We can define the base table and join with itself to get yesterday's data. (See solution 2.1)

**Solution 2.1**

```sql
WITH base AS (
  SELECT 
    l1.date
    , l2.grade_level
    , COUNT(DISTINCT CASE WHEN l1.attendance THEN l2.student_id) AS attendend_students
  FROM attendance_events l1
  INNER JOIN all_Students l2 ON l2.student_id = l1.student_id
  WHERE l1.date >= DATEADD('d', -1, CURRENTDATE)
  GROUP BY 1,2
)
SELECT 
	l1.date AS today
	, l1.grade_level
	, l1.attendend_students AS todays_attendance
	, l2.attendend_students AS yesterdays_attendance
	, yesterdays_attendance-todays_attendance AS attendance_drop
FROM base l1
INNER JOIN l2 ON l2.grade_level = l1.grade_level
								AND DATEADD('d',1, l2.date) = l1.date
ORDER BY attendance_drop DESC								
```

* If we can use window functions, then we can easily use `LAG` function within the `base` table we defined above so we don't need to use Common Table Expressions (CTEs) in our solution. (See solution 2.2)

**Solution 2.2**

```sql
SELECT 
  l1.date
  , l2.grade_level
  , COUNT(DISTINCT CASE WHEN l1.attendance THEN l2.student_id) AS attendend_students
  , LAG(attendend_students) OVER(PARTITION BY l2.grade_level ORDER BY l1.date) AS yesterdays_attended_students
  , yesterdays_attended_students - attendend_students AS attendance_drop
FROM attendance_events l1
INNER JOIN all_Students l2 ON l2.student_id = l1.student_id
WHERE l1.date >= DATEADD('d', -1, CURRENTDATE)
GROUP BY 1,2
ORDER BY attendance_drop DESC 
```

