SQL Study Plan in leetcode

Link: https://leetcode.com/study-plan/sql/?


# Window Function and Aggregation

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

176. Second Highest Salary
- Write an SQL query to report the second highest salary from the Employee table. If there is no second highest salary, the query should report null.
```sql
# using Aggregation (LIMIT, OFFSET)
SELECT(SELECT DISTINCT
    Salary 
FROM
    Employee
ORDER BY Salary DESC
LIMIT 1 OFFSET 1)AS SecondHighestSalary;
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

# Case
- 

```

```
- 

```

```
- 

```

```
- 

```

```
- 

```

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
627. Swap Salary
- Write an SQL query to swap all 'f' and 'm' values (i.e., change all 'f' values to 'm' and vice versa) with a single update statement and no intermediate temporary tables. Note that you must write a single update statement, do not write any select statement for this problem.
```sql
UPDATE Salary
    SET sex  = (CASE WHEN sex = 'm' 
        THEN  'f' 
        ELSE 'm' 
        END)
```


# DATE AND TIME
- 

```

```
- 

```

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
- 

```sql

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
- sql
```

```



```sql

```
- 

```sql

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
```sqlfds

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
- sql
```

```
- sql
```

```
- sql
```

```
- sql
```

```
- sql
```

```

- sql
```

```

- sql
```

```

- sql
```

```


182. Duplicate Emails
- Write an SQL query to report all the duplicate emails. Note that it's guaranteed that the email field is not NULL. Return the result table in any order.
```sql
SELECT email AS Email FROM Person
GROUP BY email
HAVING count(email) > 1;
# using having count() after groupby
# using sum() before groupby
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
1527. Patients With a Condition
- Write an SQL query to report the patient_id, patient_name and conditions of the patients who have Type I Diabetes. Type I Diabetes always starts with DIAB1 prefix. Return the result table in any order.
```sql
SELECT patient_id, patient_name, conditions
FROM Patients
WHERE conditions LIKE '% DIAB1%' OR conditions LIKE 'DIAB1%'; 
# ATTENTION TO SPACE
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
