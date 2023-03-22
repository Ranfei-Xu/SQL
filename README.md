# SQL

Purpose: Refresh my SQL skill using Leetcode

## Summary.md
### CASE STUDY 
  retention rate, new users daily count, repurchase rate: find the top3, Youtube: playbackRate, Youtube: count followers
### WINDOW FUNCTION
- Order: top xx, median
  - `row_number()` 1234; `rank()` 1224; `dense_rank()` 1223;
- Aggregation
  - `count, avg, sum, max, min, first, last`
  - moving average
  
    `UNBOUNDED PRECEDING`;
    `EXPRESSION PRECEDING` -- only allwed in ROWS mode;
    `CURRENT ROW;
    `EXPRESSION FOLLOWING` -- only allwed in ROWS mode;
    `UNBOUNDED FOLLOWING`
- Distribution: top 10%
  - `percent_rank(), cume_dist()`
- Lead and Lag
  - growth rate, continous/consecutive login
### CASE WHEN
- case when + aggregation
- case when (case when)
- case when (subquery)
### JOIN
- set and different join
  - whether table1 the same as table b
  - where is the difference: A and B exclude the common:  select out student who has exact same skill 
- join multiple tables
- avoid cartesian product
### DATE AND TIME
### OTHER TIPS
- order of execution
- Abnormal Value
  - duplicate
  - when dominator = 0
  - Input NULL value
- `ROLLUP()`:GROUP BY c1,c2,c3 WITH ROLLUP 
- `GROUP_CONCAT`
- data type: CHARACTER STRING, NUMBER, CONVERT and odd/even
- Adjust structure/find schema
  - add row number
  - pivot
  - check schema, col
## Study-plan.md
Study Plan in leetcode including 100 questions.

## Materials:
- MySQL Tutorial for Beginners -- Mosh
- NOTES from TEXTBOOK (DATABASE-SUFE)
- NOTES from HACKATHON (MA656-BU)
- Hands-on SQL (https://space.bilibili.com/1902255723/channel/seriesdetail?sid=413669)
- Other platform for beginners

  • [Codecademy](https://www.codecademy.com/) 
  ([Learn SQL](https://www.codecademy.com/learn/learn-sql) covers the basic statements such as SELECT and JOIN; [SQL: Table Transformation](https://www.codecademy.com/learn/sql-table-transformation) covers subqueries, set operations, conditional aggregates, and special data types.); 

  • [SQL Course](http://www.sqlcourse.com/)

  • [W3schools’ SQL Tutorial](https://www.w3schools.com/sql/)

   • [Khan Academy’s Intro to SQL](https://www.khanacademy.org/computing/computer-programming/sql)


