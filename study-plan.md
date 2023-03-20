SQL Study Plan in leetcodeRanfei-Xu

Link: https://leetcode.com/study-plan/sql/?


# Window Function 
## Lead and Lag
1709. Biggest Window Between Visits
- Assume today's date is '2021-1-1'.
- Write an SQL query that will, for each user_id, find out the largest window of days between each visit and the one right after it (or today if you are considering the last visit).
- Return the result table ordered by user_id.
```sql
SELECT user_id
, MAX(wds) AS biggest_window
FROM(
  SELECT user_id, visit_date
  , IFNULL(DATEDIFF(lag1,visit_date),DATEDIFF('2021-1-1',visit_date)) AS wds
  FROM(
    SELECT user_id, visit_date
    , lag(visit_date,1) OVER(PARTITION BY user_id ORDER BY visit_date DESC) AS lag1
SELECT user_id, MAX(diff) AS biggest_window
FROM
(
SELECT user_id,
DATEDIFF(LEAD(visit_date, 1, '2021-01-01') OVER (PARTITION BY user_id ORDER BY visit_date), visit_date) AS diff
	FROM userVisits
) a
GROUP BY user_id
ORDER BY user_id
    FROM uservisits
    ORDER BY user_id, visit_date DESC
  )t
) t2
GROUP BY user_id
ORDER BY user_id
# simpler

SELECT user_id, MAX(diff) AS biggest_window
FROM
(
SELECT user_id,
DATEDIFF(LEAD(visit_date, 1, '2021-01-01') OVER (PARTITION BY user_id ORDER BY visit_date), visit_date) AS diff
	FROM userVisits
) a
GROUP BY user_id
ORDER BY user_id
```
1747. Leetflex Banned Accounts
Write an SQL query to find the account_id of the accounts that should be banned from Leetflex. An account should be banned if it was logged in at some moment from two different IP addresses.
```sql
SELECT
    DISTINCT
    account_id
FROM
    (SELECT
        account_id,
        ip_address,
        LEAD(ip_address,1) OVER (PARTITION BY account_id ORDER BY login ASC) AS next_ip,
        logout,
        LEAD(login,1) OVER (PARTITION BY account_id ORDER BY login ASC) AS next_in
    FROM LogInfo) A
WHERE logout>= next_in AND ip_address!=next_ip
```

603. Consecutive Available Seats
```sql
SELECT
    seat_id
FROM
    (SELECT
        seat_id,
        free,
        LAG(free,1) OVER (ORDER BY seat_id) as free_lag,
        LEAD(free,1) OVER (ORDER BY seat_id) as free_lead
    FROM cinema ) as t
WHERE (free=1 AND free_lag=1)
OR (free=1 AND free_lead=1)
```

180. Consecutive Numbers
```sql
SELECT DISTINCT num AS ConsecutiveNums
FROM(
SELECT id, num
, LEAD(num,1) OVER() AS lead1
, LAG(num,1) OVER() AS lag1
FROM logs) t
WHERE num = lead1 AND num = lag1
```
1285. Find the Start and End Number of Continuous Ranges
- Write an SQL query to find the start and end number of continuous ranges in the table Logs.
```sql
# special groupby
SELECT 
	min(log_id) AS start_id,
	max(log_id) AS end_id
FROM(
    SELECT
		row_number() over (ORDER BY log_id ASC) AS rn, 
		log_id
    FROM Logs
    ) t
GROUP BY log_id-rn # groupby the difference
```

## aggregation
```sql

```
```sql

```
```sql

```
```sql

```
```sql

```
```sql

```
1308. Running Total for Different Genders
```sql
SELECT gender
, `day`
, SUM(score_points) OVER(PARTITION BY gender ORDER BY `day` ROWS unbounded preceding) AS total
FROM scores
# GROUP BY 1,2 # why it is necessary to use partitionby instead of groupby here
ORDER BY 1,2
```
1321. Restaurant Growth
- Write an SQL query to compute the moving average of how much the customer paid in a seven days window (i.e., current day + 6 days before). average_amount should be rounded to two decimal places.
Return result table ordered by visited_on in ascending order.
```sql
SELECT * FROM
(SELECT visited_on
, SUM(amount) OVER(ROWS 6 PRECEDING) AS amount
, ROUND(AVG(amount) OVER(ROWS 6 PRECEDING),2) AS average_amount
# also can use'ROWS BETWEEN 6 PRECEDING AND CURRENT ROW'
FROM
(SELECT visited_on
, SUM(amount) AS amount
FROM customer
GROUP BY visited_on)t)s
WHERE timestampdiff(DAY,(select min(visited_on) from customer),visited_on)>= 6
# why I can't use datediff
# order: where before group by, so need to buil a tmp table before uisng where to limit time zone
```
Different between groupby and partition by
```sql
# groupby
SELECT product, SUM(sales_amount)
FROM sales_data
GROUP BY product;
# partition by
SELECT product, region, SUM(sales_amount) OVER (PARTITION BY product)
FROM sales_data;
```
1303. Find the Team Size
- Write an SQL query to find the team size of each of the employees.
```sql
# window function
SELECT employee_id, COUNT(*) OVER(PARTITION BY team_id) AS team_size
FROM employee
# join
SELECT employee_id, team_size FROM employee e
JOIN
(SELECT team_id, COUNT(*) AS team_size FROM employee GROUP BY team_id) t 
ON e.team_id=t.team_id
```
1393. Capital Gain/Loss
- Write an SQL query to report the Capital gain/loss for each stock.
The Capital gain/loss of a stock is the total gain or loss after buying and selling the stock one or many times.

```sql
# using Aggretation:
SELECT stock_name, 
	   SUM(CASE WHEN operation = 'Sell' THEN price
		   ELSE -price END) AS capital_gain_loss
FROM Stocks
GROUP BY stock_name
# Window Function (SUM OVER):
SELECT DISTINCT stock_name, 
       SUM(CASE WHEN operation = 'Sell' THEN price ELSE -price END)
       OVER(PARTITION BY stock_name) AS capital_gain_loss
FROM Stocks
```
1112. Highest Grade For Each Student
-
```sql
# window function
SELECT t.student_id, t.course_id, t.grade
FROM 
	(SELECT student_id, course_id, grade, 
	row_number() OVER (PARTITION BY student_id ORDER BY grade DESC, course_id asc) AS r
	FROM Enrollments) t
WHERE t.r=1
ORDER BY t.student_id ASC
# use max(grade), sub-query
SELECT student_id, min(course_id) AS course_id, grade
FROM enrollments
WHERE (student_id, grade) IN 
(SELECT student_id,
MAX(grade) as grade
FROM enrollments
GROUP BY student_id)
GROUP BY student_id # necessary
ORDER BY student_id
```

## order
```sql

```
```sql

```
```sql

```
1972. First and Last Call On the Same Day
```sql
WITH tmp AS (
  SELECT caller_id AS uid1, call_time, recipient_id AS uid2 FROM calls 
  UNION 
  SELECT recipient_id AS uid1, call_time, caller_id AS uid2 FROM calls
)
, tmp1 AS(
  SELECT uid1,uid2,date(call_time) AS date
 , dense_rank() over(PARTITION BY uid1, date(call_time) ORDER BY call_time) AS drnk_asc
 , dense_rank() over(PARTITION BY uid1, date(call_time) ORDER BY call_time DESC) AS drnk_desc
  FROM tmp
)

SELECT DISTINCT uid1 AS user_id 
FROM tmp1
WHERE drnk_asc = 1 OR drnk_desc = 1
GROUP BY uid1, `date`
HAVING COUNT(DISTINCT uid2) = 1 # improtant!
```

1412. Find the Quiet Students in All Exams
- A quiet student is the one who took at least one exam and did not score the high or the low score.
- Write an SQL query to report the students (student_id, student_name) being quiet in all exams. Do not return the student who has never taken any exam.
- Return the result table ordered by student_id.
```sql
SELECT student_id, student_name
FROM student
WHERE student_id NOT IN (
  SELECT DISTINCT student_id FROM(
    SELECT exam_id, student_id
    , rank()over(PARTITION BY exam_id ORDER BY score) AS rnk_asc
    , rank()over(PARTITION BY exam_id ORDER BY score DESC) AS rnk_desc
    FROM exam
    )t
  WHERE rnk_asc = 1 OR rnk_desc = 1
) 
AND student_id IN (SELECT DISTINCT student_id FROM Exam)
# SIMPLER
WITH cte AS(
    SELECT exam_id, exam.student_id, student_name, score, RANK() OVER(PARTITION BY exam_id ORDER BY score) rk1, RANK() OVER(PARTITION BY exam_id ORDER BY score DESC) rk2 
    FROM exam LEFT JOIN student
    ON exam.student_id = student.student_id
)

SELECT DISTINCT student_id, student_name
FROM cte
WHERE student_id NOT IN (SELECT student_id FROM cte WHERE rk1 = 1 or rk2 = 1)
ORDER BY student_id
```
1369. Get the Second Most Recent Activity
- Write an SQL query to show the second most recent activity of each user.
If the user only has one activity, return that one. A user cannot perform more than one activity at the same time.
```sql
SELECT username, activity, startDate, endDate
FROM(
    SELECT a1.username,activity,startDate,endDate,totalactivity 
    , rank()over(PARTITION BY a1.username ORDER BY a1.startdate DESC) AS rnk
    FROM useractivity a1
    LEFT JOIN 
    (SELECT username, count(*) AS totalactivity FROM useractivity GROUP BY username) a2 ON a1.username=a2.username
) t
WHERE rnk = 2 OR totalactivity=1
```
1831. Maximum Transaction Each Day
- Write an SQL query to report the IDs of the transactions with the maximum amount on their respective day. If in one day there are multiple such transactions, return all of them.
- Return the result table ordered by transaction_id in ascending order.
```sql
SELECT transaction_id FROM
(SELECT transaction_id
, date(`day`) AS date
, RANK()OVER(PARTITION BY day(`day`) ORDER BY amount DESC) AS rnk
FROM transactions) t
WHERE rnk=1
ORDER BY transaction_id
```
1532. The Most Recent Three Orders
- Write an SQL query to find the most recent three orders of each user. If a user ordered less than three orders, return all of their orders.
Return the result table ordered by customer_name in ascending order and in case of a tie by the customer_id in ascending order. If there is still a tie, order them by order_date in descending order.
```sql
SELECT name AS customer_name
, cid AS customer_id
, order_id
, order_date
FROM customers c 
JOIN (
    SELECT customer_id AS cid, order_id, order_date
    , RANK()OVER(PARTITION BY customer_id ORDER BY order_date DESC) AS rnk
    FROM orders
)t ON c.customer_id = t.cid
WHERE rnk <=3
ORDER BY customer_name, customer_id, order_date DESC
```

1951. All the Pairs With the Maximum Number of Common Followers
- Write an SQL query to find all the pairs of users with the maximum number of common followers. In other words, if the maximum number of common followers between any two users is maxCommon, then you have to return all pairs of users that have maxCommon common followers. The result table should contain the pairs user1_id and user2_id where user1_id < user2_id.
```sql
SELECT user1_id,user2_id FROM(
  SELECT a1.user_id AS user1_id, a2.user_id AS user2_id
  , count(1) AS com_followers
  , rank() over(ORDER BY count(1) DESC) as rnk
  FROM relations a1, relations a2
  WHERE a1.user_id<a2.user_id AND a1.follower_id=a2.follower_id
  GROUP BY 1,2
  )t
WHERE rnk=1
```

178. Rank Scores
- The scores should be ranked from the highest to the lowest.
- If there is a tie between two scores, both should have the same ranking.
- After a tie, the next ranking number should be the next consecutive integer value. In other words, there should be no holes between ranks.
```sql
SELECT score
, dense_rank() over(ORDER BY score DESC) AS `rank`
FROM scores
ORDER BY `rank`
```
1596. The Most Frequently Ordered Products for Each Customer
- Write an SQL query to find the most frequently ordered product(s) for each customer. The result table should have the product_id and product_name for each customer_id who ordered at least one order.
```sql
SELECT customer_id,product_id, product_name
FROM(
  SELECT customer_id,product_id, product_name
  , rank()OVER(PARTITION BY customer_id ORDER BY cnt DESC) AS rnk
  FROM(
    SELECT o.customer_id, o.product_id, p.product_name
    , count(o.order_id) AS cnt
    FROM orders o
    LEFT JOIN products p ON o.product_id=p.product_id
    GROUP BY customer_id,product_id
    )t
  )t2
WHERE rnk = 1
ORDER BY 1,2
```
1077. Project Employees III
- Write an SQL query that reports the most experienced employees in each project. In case of a tie, report all employees with the maximum number of experience years.
```sql
SELECT project_id, employee_id
FROM(
  SELECT p.project_id, p.employee_id
  , e.name, e.experience_years
  , RANK() OVER(PARTITION BY p.project_id ORDER BY experience_years DESC) as rnk
  FROM project p
  LEFT JOIN employee e ON p.employee_id=e.employee_id
)t
WHERE rnk = 1
```
1549. The Most Recent Orders for Each Product
```sql
SELECT product_name, t.product_id, order_id, order_date
FROM
(SELECT o.product_id AS product_id
, order_id
, order_date 
, RANK() OVER(PARTITION BY product_id ORDER BY order_date DESC) AS rnk
FROM orders o) t
LEFT JOIN Products p ON p.product_id=t.product_id
WHERE rnk = 1
ORDER BY product_name, product_id, order_id
```
184. Department Highest Salary
- Write an SQL query to find employees who have the highest salary in each of the departments.
```sql
SELECT Department, Employee, Salary
FROM
(SELECT e.departmentId 
, d.name AS Department
, e.name AS Employee 
, salary AS Salary
, RANK()OVER(PARTITION BY departmentId ORDER BY salary DESC) AS rnk
FROM employee e 
JOIN department d ON d.id=e.departmentId) t
WHERE rnk=1
```


586. Customer Placing the Largest Number of Orders
- Write an SQL query to find the customer_number for the customer who has placed the largest number of orders.
- The test cases are generated so that exactly one customer will have placed more orders than any other customer.
```sql
# using window function (SLOWER)
SELECT customer_number FROM 
(
SELECT customer_number, RANK() over (ORDER BY COUNT(order_number) DESC) AS rnk
FROM orders
GROUP BY customer_number
) a
where a.rnk = 1

# using Aggregation
select customer_number from orders
group by customer_number
order by count(*) desc limit 1;
```
619. Biggest Single Number
- A single number is a number that appeared only once in the MyNumbers table. Write an SQL query to report the largest single number. If there is no single number, report null.
```sql
# using if, limit
select if(count(*) =1, num, null) as num from number 
group by num order by count(*), num desc limit 1
# using sub-query, max
SELECT MAX(group1.num) AS num 
FROM (
    SELECT num, COUNT(num) AS num_count
    FROM my_numbers
    GROUP BY num
) group1
WHERE num_count = 1
```
176. Second Highest Salary
- Write an SQL query to report the second highest salary from the Employee table. If there is no second highest salary, the query should report null.
```sql
# using (LIMIT, OFFSET)
SELECT(SELECT DISTINCT
    Salary 
FROM
    Employee
ORDER BY Salary DESC
LIMIT 1 OFFSET 1) AS SecondHighestSalary;
```
177. Nth Highest Salary
- Write an SQL query to report the nth highest salary from the Employee table. If there is no nth highest salary, the query should report null.
```sql
# using (LIMIT, OFFSET)
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
DECLARE M INT;
SET M = N-1;
  RETURN (
      SELECT DISTINCT salary FROM employee
      ORDER BY salary DESC
      LIMIT 1 OFFSET M      
  );
END
```

1407. Top Travellers
- Return the result table ordered by travelled_distance in descending order, if two or more users traveled the same distance, order them by their name in ascending order.
```sql
SELECT u.name, IFNULL(SUM(r.distance),0)  AS travelled_distance
FROM users u
LEFT JOIN rides r on u.id = r.user_id
GROUP BY u.id # donot use name, since id is primary key
ORDER BY travelled_distance DESC, u.name ASC
# ATTENTION TO THE NULL
```
1341. Movie Rating
- Find the name of the user who has rated the greatest number of movies. In case of a tie, return the lexicographically smaller user name. Find the movie name with the highest average rating in February 2020. In case of a tie, return the lexicographically smaller movie name.
```sql
#using limit
SELECT name AS results FROM(
  SELECT name
  , COUNT(movie_id) AS num_movie
  FROM movierating m
  LEFT JOIN users ON m.user_id=users.user_id
  GROUP BY m.user_id
  ORDER BY num_movie DESC, name ASC LIMIT 1)  t1
UNION
SELECT movie_name AS results FROM(
  SELECT movies.title AS movie_name
  , AVG(rating) AS avg_rating
  FROM movierating m
  LEFT JOIN movies ON m.movie_id=movies.movie_id
  WHERE month(created_at) = "2"
  GROUP BY movie_name
  ORDER BY avg_rating DESC, movie_name ASC LIMIT 1) t2
```
# RECURSIVE
-
```sql

```
1270. All People Report to the Given Manager
- Write an SQL query to find employee_id of all employees that directly or indirectly report their work to the head of the company.
The indirect relation between managers will not exceed three managers as the company is small.
```sql
WITH RECURSIVE cte AS(
	SELECT employee_id FROM employees WHERE manager_id=1 AND employee_id!=1
	UNION ALL
	SELECT a.employee_id FROM employees a 
	JOIN cte b ON (b.employee_id=a.manager_id) 
)
SELECT employee_id FROM cte
```
1613. Find the Missing IDs
- Write an SQL query to find the missing customer IDs. The missing IDs are ones that are not in the Customers table but are in the range between 1 and the maximum customer_id present in the table.
Notice that the maximum customer_id will not exceed 100.
Return the result table ordered by ids in ascending order.
```sql
WITH RECURSIVE id_seq AS (
    SELECT 1 as continued_id
    UNION 
    SELECT continued_id + 1
    FROM id_seq
    WHERE continued_id < (SELECT MAX(customer_id) FROM Customers) 
)
SELECT continued_id AS ids
FROM id_seq
WHERE continued_id NOT IN (SELECT customer_id FROM Customers)  
```
# CASE

-
```sql

```

-
```sql

```

-
```sql

```

-
```sql

```

-
```sql

```
262. Trips and Users
-
```sql
# not work very well on the day only have cancell trip
- The cancellation rate is computed by dividing the number of canceled (by client or driver) requests with unbanned users by the total number of requests with unbanned users on that day.
Write a SQL query to find the cancellation rate of requests with unbanned users (both client and driver must not be banned) each day between "2013-10-01" and "2013-10-03". Round Cancellation Rate to two decimal points.
Return the result table in any order.
```sql
with tmp AS(
    SELECT * FROM trips
    WHERE client_id NOT IN (select users_id FROM users WHERE banned = 'Yes')
    AND driver_id NOT IN (select users_id FROM users WHERE banned = 'Yes'))

SELECT request_at AS `Day`
, IF(ROUND(incomplete/total,2)=1,0,ROUND(incomplete/total,2)) AS `Cancellation Rate`
FROM(    
    SELECT request_at
    , count(CASE WHEN status <>'completed' THEN id END) AS incomplete
    , count(id) AS total
    FROM tmp
    GROUP BY request_at
    )t
# shorten but still not very well on the day only have cancell trip
 SELECT Request_at AS Day, ROUND(SUM(IF(Status = 'completed', 0, 1))/COUNT(Status),2) as 'Cancellation Rate' 
FROM Trips 
WHERE client_id NOT IN (SELECT Users_Id FROM Users WHERE Banned = 'Yes') 
    AND client_id NOT IN (SELECT Users_Id FROM Users WHERE Banned = 'Yes')
    AND Request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY Trips.Request_at

```


1783. Grand Slam Titles
- Write an SQL query to report the number of grand slam tournaments won by each player. Do not include the players who did not win any tournament.
```sql
SELECT * 
FROM (
  SELECT 
   player_id,
   player_name,
   SUM( CASE WHEN player_id = Wimbledon THEN 1 ELSE 0 END +
        CASE WHEN player_id = Fr_open THEN 1 ELSE 0 END +
        CASE WHEN player_id = US_open THEN 1 ELSE 0 END +
        CASE WHEN player_id = Au_open THEN 1 ELSE 0 END ) AS grand_slams_count
  FROM Players CROSS JOIN Championships 
  GROUP BY player_id, player_name ) t
WHERE grand_slams_count > 0
```
626. Exchange Seats
- Write an SQL query to swap the seat id of every two consecutive students. If the number of students is odd, the id of the last student is not swapped.
Return the result table ordered by id in ascending order.
```sql
SELECT  
      CASE 
        WHEN id % 2 = 0 THEN id - 1
        WHEN id % 2 = 1 AND id < (select count(*) from seat) THEN id + 1
	# why I can't use max(id) with groupby id
        ELSE id
      END AS id, 
student FROM seat
ORDER BY id;
```

1294. Weather Type in Each Country
- Write an SQL query to find the type of weather in each country for November 2019.
The type of weather is:
Cold if the average weather_state is less than or equal 15,
Hot if the average weather_state is greater than or equal to 25, and
Warm otherwise. 
```
SELECT country_name
, CASE WHEN AVG(weather_state) <=15 THEN 'Cold'
    WHEN AVG(weather_state)>=25 THEN 'Hot'
    ELSE 'Warm' END AS weather_type 
FROM weather w 
LEFT JOIN countries c ON w.country_id = c.country_id 
WHERE month(day) = '11'
GROUP BY country_name 
```

580. Count Student Number in Departments
- Write an SQL query to report the respective department name and number of students majoring in each department for all departments in the Department table (even ones with no current students). Return the result table ordered by student_number in descending order. In case of a tie, order them by dept_name alphabetically.
```
# aggregation
SELECT dept_name
, IFNULL(COUNT(student_id ),0) AS student_number
# OR: , SUM(CASE WHEN s.dept_id THEN 1 ELSE 0 END) AS student_number
FROM student s
RIGHT JOIN department d ON d.dept_id=s.dept_id
GROUP BY s.dept_id
ORDER BY student_number DESC, dept_name
```
1264. Page Recommendations
- Write an SQL query to recommend pages to the user with user_id = 1 using the pages that your friends liked. It should not recommend pages you already liked. Return result table in any order without duplicates.
```sql
# case
SELECT DISTINCT(page_id) AS recommended_page 
FROM likes
WHERE user_id IN
 (SELECT CASE 
    WHEN user1_id = 1 THEN user2_id # NO COMMA
    WHEN user2_id = 1 THEN user1_id
    END AS friends
FROM Friendship)
AND page_id NOT IN (SELECT page_id FROM likes WHERE user_id = 1)

# subquery
SELECT DISTINCT(page_id) AS recommended_page FROM likes
WHERE user_id IN (SELECT user2_id  FROM Friendship WHERE user1_id = 1)
OR user_id IN (SELECT user1_id FROM Friendship WHERE user2_id = 1)
AND page_id NOT IN (SELECT page_id FROM likes WHERE user_id = 1)
```

**1440. Evaluate Boolean Expression
- Write an SQL query to evaluate the boolean expressions in Expressions table.
```
SELECT e.left_operand, e.operator, e.right_operand,
    (
        CASE
            WHEN e.operator = '<' AND v1.value < v2.value THEN 'true'
            WHEN e.operator = '=' AND v1.value = v2.value THEN 'true'
            WHEN e.operator = '>' AND v1.value > v2.value THEN 'true'
            ELSE 'false'
        END
    ) AS value
FROM Expressions e
JOIN Variables v1
ON e.left_operand = v1.name
JOIN Variables v2
ON e.right_operand = v2.name
```
1193. Monthly Transactions
- Write an SQL query to find for each month and country, the number of transactions and their total amount, the number of approved transactions and their total amount.
```sql
SELECT 
LEFT(trans_date, 7) AS month, country, 
COUNT(id) AS trans_count, 
SUM(state = 'approved') AS approved_count, 
SUM(amount) AS trans_total_amount, 
SUM(CASE 
    WHEN state = 'approved' THEN amount 
    ELSE 0 
    END) AS approved_total_amount
FROM Transactions
GROUP BY month, country

# or: approved_count,DATE_FORMAT(trans_date,"%Y-%m") as month
# or: sum(case when state = 'approved' then 1 else 0 end) 
```
- 
1445. Apples & Oranges
- Write an SQL query to report the difference between the number of apples and oranges sold each day. Return the result table ordered by sale_date.
```sql
SELECT sale_date, 
   SUM(CASE WHEN fruit = 'apples' THEN sold_num
		   ELSE - sold_num END) AS diff
FROM sales
GROUP BY sale_date
ORDER BY sale_date
```
1393. Capital Gain/Loss
- Write an SQL query to report the Capital gain/loss for each stock.
The Capital gain/loss of a stock is the total gain or loss after buying and selling the stock one or many times.

```sql
# using Aggretation:
SELECT stock_name, 
	   SUM(CASE WHEN operation = 'Sell' THEN price
		   ELSE -price END) AS capital_gain_loss
FROM Stocks
GROUP BY stock_name
# Window Function (SUM OVER):
SELECT DISTINCT stock_name, 
       SUM(CASE WHEN operation = 'Sell' THEN price ELSE -price END)
       OVER(PARTITION BY stock_name) AS capital_gain_loss
FROM Stocks
```

1699. Number of Calls Between Two Persons
- Write an SQL query to report the number of calls and the total call duration between each pair of distinct persons (person1, person2) where person1 < person2.
Return the result table in any order.
```sql
# using FUNCTION LEAST/GREATEST
SELECT LEAST(from_id,to_id) AS person1,
GREATEST(from_id,to_id) as person2,
COUNT(*) AS call_count,
SUM(duration) AS total_duration
FROM calls
GROUP BY person1,person2;

# using CASE
SELECT 
    CASE
        WHEN from_id > to_id THEN to_id
        ELSE from_id
    END AS person1,
    CASE
        WHEN from_id > to_id THEN from_id
        ELSE to_id
    END AS person2,
    COUNT(duration) AS call_count,
    SUM(duration) AS total_duration       
FROM Calls
GROUP BY person2,person1
```

627. Swap Salary
- Write an SQL query to swap all 'f' and 'm' values (i.e., change all 'f' values to 'm' and vice versa) with a single update statement and no intermediate temporary tables. Note that you must write a single update statement, do not write any select statement for this problem.
```sql
UPDATE Salary
    SET sex  = (CASE WHEN sex = 'm' 
        THEN  'f' 
        ELSE 'm' 
        END)
```

- extension: exchange column a and b when a>b
```sql
UPDATE table_name
SET
    A = CASE
        WHEN A > B THEN B
        ELSE A
    END,
    B = CASE
        WHEN A > B THEN A
        ELSE B
    END;
  ```


# DATE AND TIME
- 

```

```

550. Game Play Analysis IV
- first day retention rate
```
WITH first_log AS(
    SELECT player_id
    , MIN(event_date) AS first_dt
    FROM activity
    GROUP BY player_id
)

SELECT 
ROUND(sum(IF(DATEDIFF(activity.event_date,first_dt)=1,1,0))
        /count(distinct activity.player_id)
        ,2) AS fraction
FROM activity
LEFT JOIN first_log ON activity.player_id = first_log.player_id
```
1741. Find Total Time Spent by Each Employee
- Write an SQL query to calculate the total time in minutes spent by each employee on each day at the office. Note that within one day, an employee can enter and leave more than once. The time spent in the office for a single entry is out_time - in_time.
```sql
SELECT
    event_day AS day, 
    emp_id,
    SUM(out_time-in_time) AS total_time
FROM Employees
GROUP BY event_day, emp_id
```
1890. The Latest Login in 2020
- Write an SQL query to report the latest login for all users in the year 2020. Do not include the users who did not login in 2020.
```sql
# why not accepted?
SELECT  user_id, MAX(time_stamp) AS last_stamp
FROM logins
WHERE time_stamp BETWEEN '2020-01-01' AND '2020-12-31'
GROUP BY user_id;
# way2:
select user_id, max(time_stamp) 'last_stamp'
from logins
where time_stamp like '2020%'
group by user_id ;
```
511. Game Play Analysis
- Write an SQL query to report the first login date for each player.Return the result table in any order.
```sql
SELECT player_id, MIN(event_date) AS first_login
FROM Activity 
GROUP BY player_id
ORDER BY player_id;
```

1141. User Activity for the Past 30 Days
- Write an SQL query to find the daily active user count for a period of 30 days ending 2019-07-27 inclusively. A user was active on someday if they made at least one activity on that day.
```sql
SELECT activity_date AS day, COUNT(DISTINCT user_id) AS active_users
FROM Activity
WHERE DATEDIFF('2019-07-27', activity_date) BETWEEN 0 AND 29
GROUP BY activity_date;
```
197. Rising Temperature
- Write an SQL query to find all dates' Id with higher temperatures compared to its previous dates (yesterday).
```sql
SELECT wt1.id 
FROM Weather wt1, Weather wt2
WHERE wt1.Temperature > wt2.Temperature AND 
      TO_DAYS(wt1.recordDate )-TO_DAYS(wt2.recordDate )=1;
# way2:
SELECT w2.id 
FROM Weather w1,Weather w2
WHERE datediff(w2.recordDate, w1.recordDate) = 1 
AND w2.temperature > w1.temperature;
```

# Concat

1045. Customers Who Bought All Products
- Write an SQL query to report the customer ids from the Customer table that bought all the products in the Product table.
```sql
WITH c as (SELECT customer_id, 
GROUP_CONCAT(DISTINCT product_key 
                     ORDER BY product_key ASC
                     separator ',') AS total_product_key
FROM customer
GROUP BY customer_id),
p as (
SELECT 
GROUP_CONCAT(DISTINCT product_key 
				ORDER BY product_key ASC
                 separator ',') AS uid_product_key
FROM product)

SELECT customer_id 
FROM c, p
WHERE c.total_product_key = p.uid_product_key
```
1484. Group Sold Products By The Date
- Write an SQL query to find for each date the number of different products sold and their names.
The sold products names for each date should be sorted lexicographically.
Return the result table ordered by sell_date.
```sql
SELECT sell_date,
    COUNT(DISTINCT(product)) AS num_sold,
    GROUP_CONCAT(DISTINCT(product) ORDER BY product ASC SEPARATOR ",") AS products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date ASC
```
1667. Fix Names in a Table
- Write an SQL query to fix the names so that only the first character is uppercase and the rest are lowercase.
Return the result table ordered by user_id.
```sql
SELECT user_id,CONCAT(UPPER(SUBSTR(name,1,1)),LOWER(SUBSTR(name,2,length(name)))) AS name
FROM Users ORDER BY user_id;
# SUBSTR(string_name , start_index ,end_index)
```

# IF
- 
```sql

```

- 
```sql

```
- 
```sql

```
- 
```sql

```
608. Tree Node
- Each node in the tree can be one of three types:
"Leaf": if the node is a leaf node.
"Root": if the node is the root of the tree.
"Inner": If the node is neither a leaf node nor a root node.
Write an SQL query to report the type of each node in the tree.
Return the result table in any order.
```sql
SELECT id, 
IF(ISNULL(p_id), 'Root', IF(id IN (SELECT p_id FROM tree), 'Inner', 'Leaf')) AS type 
FROM tree
# attention to shoose which direction as the 'TRUE'
```
1873. Calculate Special Bonus
- Write an SQL query to calculate the bonus of each employee. The bonus of an employee is 100% of their salary if the ID of the employee is an odd number and the employee name does not start with the character 'M'. The bonus of an employee is 0 otherwise.
Return the result table ordered by employee_id.
```sql
SELECT employee_id,
IF (employee_id%2=1 AND name NOT LIKE 'M%', salary, 0) AS bonus
FROM Employees 
ORDER BY employee_id;
```

# Join

-
```sql

```

-
```sql
SELECT business_id FROM(
    SELECT business_id
    , SUM(IF(occurences-avg>0,1,0)) AS activity
    FROM(
        SELECT events.* , avg
        FROM events
        LEFT JOIN (
            SELECT event_type, AVG(occurences) AS avg
            FROM events
            GROUP BY event_type
        )t ON events.event_type=t.event_type
    )t2
    GROUP BY business_id)t3
WHERE activity>1
# SIMPLER: use where after join before groupby
select 
business_id

from events as a
left join
    (
    select event_type, avg(occurences) as av
    from events
    group by event_type
    ) as b
on a.event_type = b.event_type
where a.occurences > b.av
group by business_id
having count(*)>1;
```
1949. Strong Friendship
- A friendship between a pair of friends x and y is strong if x and y have at least three common friends.
Write an SQL query to find all the strong friendships.
Note that the result table should not contain duplicates with user1_id < user2_id.
```sql
with tmp as (
    select user1_id as uid, user2_id as fid
    from Friendship
    UNION ALL
    select user2_id as uid, user1_id as fid
    from Friendship
    )
SELECT a1.uid AS user1_id, a2.uid AS user2_id, count(1) AS common_friend
FROM tmp a1, tmp a2
WHERE a1.uid < a2.uid AND a1.fid = a2.fid 
    # ensure they are friends
    AND  (a1.uid, a2.uid) IN (SELECT uid, fid from tmp)
GROUP BY 1,2
HAVING common_friend >=3
```
**1988. Find Cutoff Score for Each School**
```sql
# my way
with tmp AS(
SELECT score, student_count AS ct1
, LEAD(student_count,1) OVER() AS ct2
FROM (
  SELECT * FROM exam ORDER BY score DESC
)t)

SELECT school_id,
IFNULL(score,-1) AS score
FROM schools
left join tmp
ON capacity >= ct1 AND (capacity <ct2 OR ct2 IS NULL) 
ORDER BY score DESC
# why can't use case when

# easy way
select school_id, ifnull(min(score),-1) as score
from Schools left join Exam
on capacity >= student_count
group by school_id
```
1459. Rectangles Area
- Write an SQL query to report all possible axis-aligned rectangles with a non-zero area that can be formed by any two points from the Points table. Each row in the result should contain three columns (p1, p2, area) where:
```sql
# MINE
SELECT p1,p2
, abs(p1x-p2x)*abs(p1y-p2y) AS area
FROM
(SELECT t1.id AS p1, t2.id AS p2
, t1.x_value AS p1x
, t1.y_value AS p1y
, t2.x_value AS p2x
, t2.y_value AS p2y
FROM points t1
JOIN points t2 ON t1.id < t2.id) t
WHERE abs(p1x-p2x)*abs(p1y-p2y) <> 0
ORDER BY area DESC, p1 ASC, p2 ASC 

# SIMPLER
select * from(
select t1.id as P1, t2.id as P2, abs((t1.x_value - t2.x_value) * (t1.y_value - t2.y_value)) as AREA
from points t1 join points t2 on t1.id < t2.id) t where AREA <> 0
order by AREA desc, P1 asc, P2 asc 
```
181. Employees Earning More Than Their Managers
```sql
SELECT Employee FROM
(SELECT e1.name AS Employee
, e1.salary AS emp_salary
, e2.salary AS mgr_salary
FROM employee e1
LEFT JOIN employee e2 ON e1.managerid=e2.id
) t
WHERE emp_salary > mgr_salary
```
1731. The Number of Employees Which Report to Each Employee
- For this problem, we will consider a manager an employee who has at least 1 other employee reporting to them. Write an SQL query to report the ids and the names of all managers, the number of employees who report directly to them, and the average age of the reports rounded to the nearest integer.
Return the result table ordered by employee_id.
```sql
SELECT e1.reports_to AS employee_id
, e2.name AS name
, count(*) AS reports_count
, ROUND(AVG(e1.age),0) AS average_age
FROM employees e1
LEFT JOIN employees e2 ON e1.reports_to = e2.employee_id
GROUP BY e2.employee_id # can't use name, coz same name mgr
HAVING employee_id IS NOT NULL # don't forget
ORDER BY employee_id
```


1164. Product Price at a Given Date
- Write an SQL query to find the prices of all products on 2019-08-16. Assume the price of all products before any change is 10.
```sql
# using where () in (table)
SELECT product_id, 10 AS price
FROM products
GROUP BY product_id
HAVING MIN(change_date) > '2019-08-16'
UNION
SELECT product_id, new_price as price
FROM products
WHERE (product_id,change_date) IN 
(SELECT product_id,MAX(change_date) 
FROM products 
WHERE change_date <= '2019-08-16'
GROUP BY product_id)
```


1280. Students and Examinations
- Write an SQL query to find the number of times each student attended each exam. Return the result table ordered by student_id and subject_name.
```sql
SELECT a.student_id, a.student_name, b.subject_name AS subject_name
, COUNT(a.student_id=c.student_id AND b.subject_name=c.subject_name) AS attended_exams
FROM students a
JOIN subjects b
LEFT JOIN examinations c ON a.student_id=c.student_id AND b.subject_name=c.subject_name
GROUP BY 1,3
ORDER BY 1,3
```

1158. Market Analysis
- Write an SQL query to find for each user, the join date and the number of orders they made as a buyer in 2019.
Return the result table in any order.
```sql
SELECT u.user_id AS buyer_id, u.join_date,
    IFNULL(COUNT(order_id),0) AS orders_in_2019 FROM users u
LEFT JOIN orders o
ON u.user_id = o.buyer_id AND YEAR(order_date) = '2019'
GROUP BY user_id;
# can't use WHERE CLAUSE because we need return null for people who don't order

```
607. Sales Person
- Write an SQL query to report the names of all the salespersons who did not have any orders related to the company with the name "RED".
```sql
# not work
SELECT s.name FROM SalesPerson s
RIGHT JOIN Orders o ON o.sales_id = s.sales_id
Left JOIN Company c ON c.com_id = o.com_id
WHERE c.name NOT IN ('RED');
# fix the problem: not have any orders relate to 'red'
SELECT s.name
FROM salesperson s
WHERE sales_id NOT IN (
    SELECT sales_id FROM orders o 
    LEFT JOIN company c ON o.com_id = c.com_id
    WHERE c.name = 'RED');
```
1581. Customer Who Visited but Did Not Make Any Transactions
- Write a SQL query to find the IDs of the users who visited without making any transactions and the number of times they made these types of visits.
Return the result table sorted in any order.
#### join with null
```sql
SELECT customer_id, COUNT(v.visit_id) as count_no_trans 
FROM Visits v
LEFT JOIN Transactions t ON v.visit_id = t.visit_id
WHERE transaction_id IS NULL
GROUP BY customer_id
```
1795. Rearrange Products Table
- Write an SQL query to rearrange the Products table so that each row has (product_id, store, price). If a product is not available in a store, do not include a row with that product_id and store combination in the result table.
Return the result table in any order.
```sql
SELECT product_id, 'store1' AS store, store1 AS price FROM Products WHERE store1 IS NOT NULL
UNION
SELECT product_id, 'store2' AS store, store2 AS price FROM Products WHERE store2 IS NOT NULL
UNION
SELECT product_id, 'store3' AS store, store3 AS price FROM Products WHERE store3 IS NOT NULL 
```
1965. Employees With Missing Information
- Write an SQL query to report the IDs of all the employees with missing information. The information of an employee is missing if:
The employee's name is missing, or
The employee's salary is missing.
Return the result table ordered by employee_id in ascending order.
```sql
SELECT employee_id FROM Employees WHERE employee_id not in (Select employee_id from Salaries)
UNION 
SELECT employee_id FROM Salaries WHERE employee_id not in (Select employee_id from Employees)
ORDER BY employee_id;
```
# Dataframe

- 
```sql

```

- 
```sql

```

- 
```sql

```

-
```sql

```


196. Delete Duplicate Emails
- Write an SQL query to swap all 'f' and 'm' values (i.e., change all 'f' values to 'm' and vice versa) with a single update statement and no intermediate temporary tables.
```sql
DELETE P1
FROM Person p1, Person p2
WHERE p1.email = p2.email AND p1.id > p2.id 
# attention, don't use <>
```



# Basic
1867. Orders With Maximum Quantity Above Average
- You are running an e-commerce site that is looking for imbalanced orders. An imbalanced order is one whose maximum quantity is strictly greater than - the average quantity of every order (including itself).
- The average quantity of an order is calculated as (total quantity of all products in the order) / (number of different products in the order). The maximum quantity of an order is the highest quantity of any single product in the order.
- Write an SQL query to find the order_id of all imbalanced orders.
```sql
# subquery
WITH tmp AS (
    SELECT order_id,
    AVG(quantity) as avg_qty,
    MAX(quantity) as max_qty
    FROM ordersdetails
    GROUP BY order_id
)
SELECT order_id FROM tmp WHERE max_qty>(SELECT MAX(avg_qty) FROM tmp);
# window function
with cte as (
    select order_id,
    max(avg(quantity)) over() as max_avg_qty,
    max(quantity) as max_qty
    from ordersdetails
    group by order_id
)
select order_id from cte where max_qty>max_avg_qty
```

1398. Customers Who Bought Products A and B but Not C
- Write an SQL query to report the customer_id and customer_name of customers who bought products "A", "B" but did not buy the product "C" since we want to recommend them to purchase this product.
```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE customer_id IN (SELECT customer_id FROM orders WHERE product_name = 'A')
AND customer_id IN (SELECT customer_id FROM orders WHERE product_name = 'B')
AND customer_id NOT IN (SELECT customer_id FROM orders WHERE product_name = 'C')
```
1211. Queries Quality and Percentage
- Write an SQL query to find each query_name, the quality and poor_query_percentage.Both quality and poor_query_percentage should be rounded to 2 decimal places.
```sql
# caculation
SELECT query_name,
ROUND(SUM(rating/position) / COUNT(DISTINCT result),2) AS quality,
ROUND(SUM(rating < 3)*100 / COUNT(DISTINCT result),2) AS poor_query_percentage
FROM queries
GROUP BY query_name
ORDER BY query_name DESC
```
1173. Immediate Food Delivery I
- If the customer's preferred delivery date is the same as the order date, then the order is called immediate; otherwise, it is called scheduled.
Write an SQL query to find the percentage of immediate orders in the table, rounded to 2 decimal places.
```sql
SELECT 
ROUND(
    (SELECT COUNT(*) FROM delivery WHERE order_date = customer_pref_delivery_date) 
    / 
    (SELECT COUNT(*) FROM delivery)
    *100,2) 
    AS immediate_percentage
# DON'T ADD 'FROM delivery' AT END
# SIMPLER WAY
select round(100 * sum(order_date = customer_pref_delivery_date) / count(*), 2) as immediate_percentage from Delivery;
```
1633. Percentage of Users Attended a Contest
- Write an SQL query to find the percentage of the users registered in each contest rounded to two decimals. Return the result table ordered by percentage in descending order. In case of a tie, order it by contest_id in ascending order.
```sql
# calculation
SELECT contest_id,
ROUND(COUNT(user_id)/(SELECT COUNT(*) FROM Users)*100,2) AS percentage 
FROM register
GROUP BY contest_id
ORDER BY percentage DESC, contest_id;
```
1571. Warehouse Manager
- Write an SQL query to report the number of cubic feet of volume the inventory occupies in each warehouse.
```sql
# calculation
SELECT w.name AS warehouse_name,
SUM(units*width*length*height) AS volume
FROM warehouse w
LEFT JOIN products p ON w.product_id = p.product_id
GROUP BY w.name
```
1251. Average Selling Price
- Write an SQL query to find the average selling price for each product. average_price should be rounded to 2 decimal places.
```sql
# calculation
SELECT p.product_id,
ROUND(SUM(price*units)/SUM(units),2) AS average_price 
FROM prices p
RIGHT JOIN unitssold u ON p.product_id = u.product_id
WHERE purchase_date BETWEEN start_date AND end_date
GROUP BY product_id
```

182. Duplicate Emails
- Write an SQL query to report all the duplicate emails. Note that it's guaranteed that the email field is not NULL. Return the result table in any order.
```sql
SELECT email AS Email FROM Person
GROUP BY email
HAVING count(email) > 1;
# using having count() after groupby = where
```
1050. Actors and Directors Who Cooperated At Least Three Times
- Write a SQL query for a report that provides the pairs (actor_id, director_id) where the actor has cooperated with the director at least three times.
Return the result table in any order.
```sql
SELECT actor_id, director_id 
FROM actordirector
GROUP BY actor_id, director_id 
HAVING COUNT(timestamp)>2;
```
570. Managers with at Least 5 Direct Reports
- Write an SQL query to report the managers with at least five direct reports.
```sql
SELECT name FROM employee
WHERE id IN 
(SELECT managerid FROM employee 
GROUP BY managerId
HAVING COUNT(*) >= 5)
```

1527. Patients With a Condition
- Write an SQL query to report the patient_id, patient_name and conditions of the patients who have Type I Diabetes. Type I Diabetes always starts with DIAB1 prefix. Return the result table in any order.
```sql
SELECT patient_id, patient_name, conditions
FROM Patients
WHERE conditions LIKE '% DIAB1%' OR conditions LIKE 'DIAB1%'; 
# ATTENTION TO SPACE
```


1607. Sellers With No Sales
- Write an SQL query to report the names of all sellers who did not make any sales in 2020.
```sql
SELECT seller_name 
FROM seller 
WHERE seller_id NOT IN (
    SELECT seller_id FROM orders 
    WHERE year(sale_date) = 2020
)
ORDER BY seller_name ASC
```
183. Customers Who Never Order
```sql
SELECT name AS Customers FROM Customers
WHERE id NOT IN (SELECT customerId FROM Orders)
```
584. Find Customer Referee
- Write an SQL query to report the names of the customer that are not referred by the customer with id = 2.
```sql
SELECT name FROM Customer
WHERE referee_id  <> 2 OR referee_id IS NULL;
# another way:
SELECT name FROM Customer
WHERE id NOT IN (SELECT id FROM Customer WHERE referee_id=2);
```
0. Logical Operation
```sql
# 1757. Recyclable and Low Fat Products
SELECT product_id FROM Products
WHERE low_fats = 'Y' AND recyclable = 'Y'; 
# 595. Big Countries
SELECT name,population,area FROM World
WHERE area >= 3000000 OR population >= 25000000;
```
