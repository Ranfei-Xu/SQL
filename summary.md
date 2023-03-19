# CASE STUDY

- retention rate
    
    ```sql
    # 3 step: find the first day, find the t+n day, calculate the rate
    # activity[player_id,device_id,event_date,games_played]
    with install_date as(
    SELECT player_id, min(event_date) AS install_dt
    FROM activity
    GROUP BY player_id)
    
    SELECT a.player_id
    , a.install_dt
    , count(1) as total_cnt
    , count(case when event_date is not null then event_date) AS retention_cnt
    FROM install_date a
    LEFT JOIN activity b ON a.player_id=b.player_id
    AND install_dt+interval'1' day = event_date
    
    SELECT install_dt
    ,sum(total_cnt) AS installs
    ,round(sum(retention_cnt)/sum(total_cnt),2) AS day1_retention
    FROM install_date
    GROUP BY 1 # don't forget to write groupby after uisng aggretation function
    ```
    
- new users daily count
    
    ```sql
    # new users daily count (recent 90 days)
    # [user_id, activity, activity_date]
    # step: find the first_date, activitity_date = first_date, calculate
    with min_log as(
    SELECT user_id
    , min(activity_date) AS first_date
    FROM traffic
    WHERE activity = 'login'
    GROUP BY 1)
    
    SELECT activity_date = login_date
    , count(DISTINCT case when activity_date =  first_date then a.user_id END) AS user_count
    FROM traffic a
    JOIN min_log b ON a.user_id=b.user_id
    WHERE activity_date BETWEEN date_sub(date('2019-06-30',interval 90 day) AND date('2019-06-30')
    GROUP BY 1
    HAVING (count(DISTINCT case when activity_date =  first_date then a.user_id END) )>0
    ```
    
- repurchase rate: find the top3
    
    ```sql
    # step:calculate the times of multiple buying, calculate the ratio
    # tb_product_info[id, product_id,shop_id, tag, int,quantity,release_time]
    # tb_order_overall[id,order_id,uid,event_time, total_amount, total_cnt,status]
    # tb_order_detail[id, order_id, product_id, price, cn]
    with tmp as(
    SELECT b.product_id
    , a.uid
    , count(distinct a.order_id) AS cnt
    FROM tb_order_overall a 
    JOIN tb_order_detail b ON a.order_id=b.order_id
    JOIN tb_product_info c ON b.product_id= c.product_id
    WHERE tag='snack'
    AND DATEDIFF((select max(event_time) from tb_order_overall), date(event_time))<90
    GROUP BY product_id, uid
    )
    SELECT product_id
    , round(count(case when cnt>1 then uid end)/count(uid),3) AS repurchase_rate
    FROM tmp
    GROUP BY 1
    ORDER BY 2 DESC,1
    LIMIT 3
    ```
    
- Youtube: **playbackRate**
    
    ```sql
    timestampdiff(interval,datetime1,datetime2)
    # interval could be second,minute,hour,day,week,month,quarter,year
    # user_video_log[id, uid, video_id, start_time, end_time, if_follow, if_like, if_retweet,comment_id]
    # video_info[id, video_id, author, tag, duration, release_time]
    
    SELECT a.video_id
    , round(count(case when timestampdiff(second,start_time,end_time) >= duration then a.id end)/count(a.id),3) as avg_comp_play_rate
    FROM user_video_log a 
    JOIN video_info b ON a.video_id=b.video_id
    WHERE year(start_time) = 2021
    GROUP BY 1
    ORDER BY avg_comp_play_rate DESC
    ```
    
- Youtube: count followers
    
    # followers by month: newfollow- unfollowed — func(args)over()
    
    effective of video = (newfollow- unfollowed)/# palyback — count case when
    
    ```sql
    # user_video_log[id, uid, video_id, start_time, end_time, if_follow(2:UNFOLLOW), if_like, if_retweet,comment_id]
    # video_info[id, video_id, author, tag, duration, release_time]
    with fans as(
    SELECT author
    , `month`
    , sum(new_fans-loss_fans) over(partition by author order by `month`) AS total_fans 
    # we don't change the structure of table -- no need groupby
    FROM
    (SELECT author
    , substrt(start_time,1,7) as `month`
    , count(case when if_follow = 1 then uid end) as new_fans
    , count(case when if_follow = 2 then uid end) as loss_fans
    FROM user_video_log a 
    	JOIN video_info b ON a.video_id=b.video_id
    	AND year(start_time) = 2021
    GROUP BY 1,2) t)
    
    , growth as(
    SELECT author
    , substrt(start_time,1,7) as `month`
    , (count(case when if_follow = 1 then uid end)-count(case when if_follow = 2 then uid end))/count(uid) as fans_growth_rate
    FROM user_video_log a 
    	JOIN video_info b ON a.video_id=b.video_id
    	AND year(start_time) = 2021
    GROUP BY 1,2)
    
    SELECT a.author,a.`month`,fans_growth_rate,total_fans
    FROM growth a
    	JOIN fans b ON a.author=b.author AND a.`month`=b.`month`
    
    ```
    

# Window Function

```sql
function(args) OVER (
[PARTITION BY expression]
[ORDER BY expression DESC|ASC] 
[frame] # AS 'rank' (necessary using ``, since rank is a function in sql)
)
```

- order: top xx
    - 3 types
        
        `row_number()` 1234; `rank()` 1224; `dense_rank()` 1223;
        
    - examples
        
        ```sql
        # median
        SELECT id,company,salary
        FROM
        (SELECT id,company,salary,
        	count(1) OVER(PARTITION BY company) AS cnt
        , ROW_NUMBER() OVER ((PARTITION BY company ORDER BY salary, id) AS rank1
        , ROW_NUMBER() OVER ((PARTITION BY company ORDER BY salary DESC, id DESC) AS rank2
        FROM employee) t
        WHERE rank1=round(cnt/2,0) 
        OR rank2=round(cnt/2,0) 
        # when we have even employees, we need select out two rows
        # when odd employess, rank1=rnak2=rount(cnt/2,0)
        
        # rank the score
        SELECT score
        , DENSE_RANK() OVER(ORDER BY score DESC) AS 'rank'
        FROM scores
        
        # rank the salary in different department
        SELECT department, employy, salary FROM 
        (SELECT
        b.name AS department
        , a.name AS employee
        , salary
        , RANK()OVER(PARTITION BY deparmentid ORDER BY salary DESC) AS rank_num
        FROM employee a 
        JOIN department b ON a.departmentid = b.id) t
        # necessary to add another name for df generated by subquery
        WHERE rank_num = 1
        ```
        
- aggregation
    - `count, avg, sum, max, min, first, last`
        
        ```sql
        # accumulated monthly sales for each sub_cate
        SELECT main_cate, sub_cate, year_month, sales
        , SUM(sales) OVER(PARTITION BY sub_cate ORDER BY year_month) AS sum_by_sub_cate
        FROM sales
        ```
        
    - SILEDING WINDOW: moving average
        
        ```sql
        # frame
        UNBOUNDED PRECEDING
        EXPRESSION PRECEDING -- only allwed in ROWS mode
        CURRENT ROW
        EXPRESSION FOLLOWING -- only allwed in ROWS mode
        UNBOUNDED FOLLOWING
        
        {RANGE|ROW} frame_start
        {RANGE|ROW} BETWEEN frame_start AND frame_end
        ```
        
        ```sql
        # rows: 物理行
        SELECT main_cate
        , sub_cate
        , year_month
        , sales
        , SUM(sales) OVER (
        	ORDER BY year_month
        	ROWS unbounded preceding) AS sum_by_sub_cate
        FROM tmp
        # everytime: add from first row until current row
        
        # range: 数值逻辑
        SELECT main_cate
        , sub_cate
        , year_month
        , sales
        , SUM(sales) OVER (
        	ORDER BY year_month
        	RANGE unbounded preceding) AS sum_by_sub_cate
        FROM tmp
        # everytime: add from the group that first row involved to the group current invovled
        
        # moving avg of 3 month gmv
        SELECT product, year_month, gmv
        , AVG(gmv) OVER(PARTITION BY department, product ORDER BY year_month
        	ROWS 2 PRECEDING) AS avg_gmv
        FROM product
        
        # from start to current [defalt]
        SELECT product, year_month, gmv
        , AVG(gmv) OVER(PARTITION BY department, product ORDER BY year_month
        	) AS avg_gmv
        FROM product
        
        # 7 days growth rate
        SELECT * FROM
        (SELECT visited_on,
        	SUM(amount) OVER(ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS amount,
          ROUND(AVG(amount) OVER(ORDER BY visited_on ROWS BETWEEN 6 PRECEDING AND CURRENT ROW),2) AS average_amount
        FROM (
        SELECT visited_on, SUM(amount) AS amount
        FROM customers
        GROUP BY visited_on) s)T
        WHERE timestampdiff(day,(select min(visited_on) from customer), visited_on)>=6
        # start from the 7th day of the month
        
        ```
        
- distribution: top 10%
    
    `percent_rank()` = (分组内当前行的Rank-1)/(分组内总行-1)
    
    `cume_dist()` = 小于等于当前值的行数/分组内总行数
    
    ```sql
    # skusales[product(#10), sales]
    # top 10% sales
    SELECT product
    , percent_rank() OVER(ORDER BY sales DESC) AS 'percent_rank' # 0,0.125,0.25,...,0.875,1
    , cume_dist() OVER(ORDER BY sales DESC) AS 'cume_dist' # 0.1111,0.222,...,0.888,1
    FROM skusales
    WHERE percent_rank < 0.1
    
    # way2
    WITH tmp AS(
    SELECT product, row_number() over (order by sales DESC) as 'rank' FROM skusales)
    
    SELECT * FROM tmp a 
    	LEFT JOIN (
    		SELECT CEILING(COUNT(1)*0.1) AS rank_bar
    		FROM skusales) b ON 1=1
    WHERE 'rank' <= rank_bar
    
    # way3: ntile
    NTILE(N) OVER (
    	[PARTITION BY expression]
    	[ORDER BY expression[ASC|DESC]]
    	[frame]
    )
    
    SELECT product,
    NTILE(10) OVER (ORDER BY sales DESC) AS ntile_rank # divide in 10 groups
    FROM skusales
    # if indivisible, the first group will have one more element
    
    ```
    
- lead and lag
    
    `lag(expression.n)` 返回当前行的前n行expression的值； `lead(expression.n)` 返回当前行的后n行expression的值
    
    ```sql
    [year_month,sales]
    # month-on-month
    SELECT DATE(year_month) as `year_month`
    , sales
    , LAG(sales,1) OVER(ORDER BY `year_month`) AS lag_1_sales
    , sales/LAG(sales,1) OVER(ORDER BY `year_month`) - 1 AS mom_growth
    FROM table1
    # year-on-year
    SELECT DATE(year_month) as `year_month`
    , sales
    , LAG(sales,12) OVER(ORDER BY `year_month`) AS lag_1_sales
    , sales/LAG(sales,12) OVER(ORDER BY `year_month`) - 1 AS mom_growth
    FROM table1
    
    [product,year_month,department,gmv]
    # month-on-month
    SELECT *
    , LAG(gmv,1) OVER (PARTITION BY department, product ORDER BY year_month) AS lag_1_gmv
    , CAST(gmv AS DOUBLE)/lag(gmv,1) OVER (PARTITION BY department, product ORDER BY year_month) - 1 AS growth_rate
    # use double change int into double
    FROM table1
    # if the year_month not continues, start by right joining a calendar
    
    # find the number continuously appear at least 3 times
    SELECT DISTINCT num AS consecutiveNums
    (SELECT id, num
    , LAG(num,1) OVER (ORDER BY id) AS lag_1
    , LEAD(num,1) OVER(ORDER BY id) AS lead_1
    FROM table1) t
    WHERE num = lag_1 AND num = lead_1
    
    # continously login in 4 days
    # logins[id,login_date]
    WITH tmp AS(
    SELECT id, login_date
    , LEAD(login_date,4) OVER(PARTITION BY id ORDER BY login_date) AS lead_4_date
    FROM
    (SELECT DISTINCT * FROM logins) t # remove duplicate)
    
    SELECT DISTINCT a.id, name
    FROM tmp a
    JOIN account b ON a.id=b.id
    WHERE DATEDIFF(lead_4_date,login_date)=4
    ORDER BY a.id
    
    ```
    

# CASE WHEN

```sql
CASE WHEN condition THEN result # the type of result must keep consistant: number or string
[WHEN...THEN..]
[ELSE result]
END
```

- case when + aggregation
    
    ```sql
    CASE WHEN aggregation_function = 1 THEN aggregation_function
    ELSE aggregation_function
    END
    ...
    GROUP BY XXX
    ```
    
- case when (case when)
    
    ```sql
    CASE WHEN XX THEN XX
    ELSE (CASE WHEN XX THEN XX ELSE XX END)
    END
    
    # select out single club and main club
    SELECT student_id,
    CASE 
    	WHEN COUNT(*) = 1 THEN MAX(club_id)
    	ELSE MAX(CASE 
    		WHEN main_club_flag = 'Y' THEN club_id
    	END)
    END club_id
    FROM student_club_info
    GROUP BY student_id; # using groupby, so MAX() is necessary
    
    ```
    
- case when (subquery)
    
    ```sql
    CASE WHEN XX THEN XX
    WHEN XX=(SELECT...) THEN XX
    END
    
    # select out the club of selected_student
    SELECT student_id,
    CASE 
    	WHEN 
    		sdudent_id IN (SELECT student_id FROM selected_student)
    	THEN club_id
    	# better to have ELSE 0 to avoid null
    END AS club_id
    FROM student_club_info;
    ```
    

# JOIN

- set and different join
    
    ```sql
    # common 
    SELECT xx FROM A LEFT JOIN B ON A.key=B.key # all A
    SELECT xx FROM A RIGHT JOIN B ON A.key=B.key # all B
    SELECT xx FROM A INNER JOIN B ON A.key=B.key # A,B common
    # attention
    SELECT xx FROM A LEFT JOIN B ON A.key=B.key WHERE B.key IS NULL # A excluding B
    SELECT xx FROM A RIGHT JOIN B ON A.key=B.key WHERE A.key IS NULL # B excluding A
    SELECT xx FROM A FULL OUT JOIN B ON A.key=B.key # all A and B 
    # A and B exclude the common
    SELECT xx FROM A FULL OUT JOIN B ON A.key=B.key WHERE A.key IS NULL OR B.key IS NULL
    # mysql not support full out join, using left+right+union
    ```
    
    - e.g.  whether table1 the same as table b
    
    ```sql
    studentskill[student_name, skill],studentskill2[student_name, skill]
    SELECT COUNT(1) # if # rows the same, show #, or NULL
    FROM 
    	(SELECT * FROM studentskill
    	 UNION
    	 SELECT * FROM studentskill2) a
    HAVING COUNT(1) = (SELECT COUNT(1) FROM studentskill)
    	AND COUNT(1) = (SELECT COUNT(1) FROM studentskill2)
    ```
    
    - e.g. where is the difference: A and B exclude the common
    
    ```sql
    SELECT *
    FROM
    	(SELECT a.student_name AS old_student_name,
    					a.skill AS old_skill,
    					b.student_name AS new_student_name,
    					b.skill AS new_skill,
    	 FROM studentskill a 
    	 LEFT JOIN studentskill2 b ON a.student_name=b.student_name AND a.skill=b.skill
    	 UNION 
    	 SELECT a.student_name AS old_student_name,
    					a.skill AS old_skill,
    					b.student_name AS new_student_name,
    					b.skill AS new_skill,
    	 FROM studentskill a  
    	 RIGHT JOIN studentskill2 b ON a.student_name=b.student_name AND a.skill=b.skill
    	) a
    WHERE old_student_name IS NULL OR new_student_name IS NULL
    ```
    
    - e.g. select out student who has exact same skill
    
    ```sql
    # studentskill[student_name, skill]
    SELECT a1.student_name, a2.student_name, count(1) # the # common skill
    FROM studentskill a1, studentskill a2
    WHERE a1.student_name < a2.student_name AND a1.skill = a2.skill
    GROUP BY 1,2 #delete duplicate
    HAVING COUNT(1) = (SELECT COUNT(1) FROM studentskill a3 WHERE a3.student_name=a1.student_name)
    	AND COUNT(1) = (SELECT COUNT(1) FROM studentskill a4 WHERE a4.student_name=a2.student_name)
    ```
    
- join multiple tables
    - a LEFT JOIN b ON a.id=b.id LEFT JOIN c ON **a.id**=c.id v.s. a LEFT JOIN b ON a.id=b.id LEFT JOIN c ON **b.id**=c.id
    - ON VS. WHERE
- avoid cartesian product
    
    ```sql
    # discount_info[productid:A,B,discount], product_info[productid,modelid.price)
    # (A productid link with 4 modelid with different price)
    
    # wrong answer
    SELECT a.*, b.price
    FROM discount_info a
    	LEFT JOIN product_info b ON a.productid=b.productid
    # it will show 4A+1B
    
    # adjust:把颗粒度较细的表向上聚合
    SELECT a.*, b.price
    FROM discount_info a
    	LEFT JOIN
    	(SELECT productid, CAST(AVG(price) AS DECIMAL(6,2) AS price
    		FROM product_info
    		GROUP BY productid) b ON a.productid=b.productid
    ```
    

# DATE, TIME

- `date_format(date,format)`: change format
- CURRENT TIME `NOW()` `CURRENT_DATE()`
    
    now including the specific time
    
    DATE( NOW()) = CURRENT_DATE()
    
- RETREIVE DATE `year(),month(),week(),day()`
    
    week: xx/52
    
    day: xx/30,31,28
    
- moving average
    
    `datediff(date1,date2)` (date2 before date1)
    
    `date_add(current_date,interval-2day)` before_yesterday
    
    `date_add(current_date,interval+2day)`after_tomorrow
    
    `date_add(current_date,interval-1month)` last_month
    
    `date_format(current_date,'%Y-%M-01'` first day
    
    `date_add(current_date, interval - day(current_date)+1day)`  first day
    
    `last_day(current_date)`  last day 
    
- exchange type
    
    `str_to_date(string,format_mask)`
    
    `from_unixtime(), unix_timestamp(current_date)`
    
- the boundary
    
    before 2019-11-09: (Y) ts<’2019-11-09’; (N) ts ≤’2019-11-08’
    

```sql
# str_to_date(string,format_mask)
SELECT str_to_date('09/01/2009','%m/%d/%y') AS str_to_date_result; -> 2009-09-01
# date_format
SELECT date_format('2013-03-09','%Y %m %d') AS new_date; -> 2013 03 09
```

# Other Tips

- order of execution:
    
    from, join, on, where, group by, aggregation(avg,sum,..), having, select, **distinct, order by, limit (effective area of alias**)
    
- **Abnormal Value**
    - **duplicate**
        
        ```sql
        # IDENTIFY AND DELETE
        # detect
        GROUP BY X1
        HAVING COUNT(X2)>1
        # delete way1
        SELECT DISTINCT * FROM users
        # delete way2
        SELECT `name`, MAX(`id`)
        FROM users
        GROUP BY `name`
        ```
        
    - **when dominator = 0**
        
        ```sql
        # way1: case when
        # way2: nullif
        # NULLIF(expression1,expression2): if ex1=ex2, result in ‘null’, otherwise ex1
        
        SELECT price,sku,
        price/NULLIF(sku,0) AS test1, #include null
        COALESCE(price/NULLIF(sku,0),0) AS test2,# null->0
        CASE
        	WHEN sku=0 THEN NULL
        	ELSE price/sku
        END AS test3 # null
        CASE
        	WHEN sku=0 THEN 'inf'
        	ELSE price/sku
        END AS test4 # null -> inf
        FROM table0
        
        ```
        
    - **Input NULL value**
        
        `IFNULL`: IFNULL(sales,0)
        
        `COALESCE`: COALESCE(sales, fillsalesway1, 0); using fillsalesway1 to input the null in sales, if still null, using 0
        
        `NULLIF(expression1,expression2)`: if ex1=ex2, result in ‘null’, otherwise ex1
        
- Aggregation`count, avg, sum, max, min, first, last`
    
    don’t forget adding groupby at end (whether effect structure of table)
    
    `count(*)` vs. `count(col_name)`: `count(*)` including null
    
- `ROLLUP():GROUP BY c1,c2,c3 WITH ROLLUP` (c1>c2>c3)
    
    ```sql
    # sales in each product, each year
    # plus sales in each year
    # plus sales in total
    SELECT `year`, product, SUM(sales) AS totalsale
    FROM sales
    GROUP BY `year`, product WITH ROLLUP
    # impute the null 
    SELECT 
    	COALESCE(`year`,TTL) AS `year`,
    	COALESCE(product,TTL) AS product,
    	SUM(sales) AS totalsale
    FROM sales
    GROUP BY `year`, product WITH ROLLUP
    ```
    
- retrieve data (column, row, order)
    - filter row
        - single condition: =,>,<,; between…and…;IN, NOT IN; LIKE “%”,”_”
            
            ```sql
            # <>, =, null
            SELECT * FROM sales WHERE product <> 'A';
            SELECT * FROM sales WHERE product <> 'A' OR product IS NULL;
            ```
            
        - multiple condition: NOT>AND>OR: a AND b OR c = (a AND b) OR c
        - base on situation: CASE WHEN
        - ORDER: select the second one
            
            LIMIT, WINDOW FUNCTION, SELF JOIN
            
            ```sql
            # LIMIT
            LIMIT[offset],rows # the offset of first row is 0
            e.g. LIMIT 5,10 -> row 6-15
            LIMIT 1,1, -> row 2
            # WINDOW FUNCTION
            # SELF JOIN
            ```
            
- `GROUP_CONCAT`: one to many query
    
    ```sql
    SELECT gender, GROUP_CONCAT(student_id) AS name_array
    FROM studentinfo
    GROUP BY gender
    
    SELECT sell_date,
        COUNT(DISTINCT(product)) AS num_sold,
        GROUP_CONCAT(DISTINCT(product) ORDER BY product ASC SEPARATOR ",") AS products
    FROM Activities
    GROUP BY sell_date
    ORDER BY sell_date ASC
    ```
    
- data type: CHARACTER STRING, NUMBER, CONVERT and odd/even
    
    ```sql
    # CHARACTER STRING
    # left(), right(), substring()
    LEFT('apple',2) -> 'ap'
    RIGHT('apple',2) -> 'le'
    SUBSTRING('apple',2,3) -> 'ppl' # startposition,length
    SUBSTRING('apple',1) -> 'apple'
    
    # deal with blank: rtrim(), ltrim(),trim()
    RTRIM('  apple  ') -> '  apple'
    LTRIM('  apple  ') -> 'apple  '
    TRIM('  apple  ') -> 'apple'
    
    # REPLACE
    REPLACE(’apple’,’a’,’o’) -> ‘opple’
    
    # CONCAT
    CONCAT('abc','-','def') -> 'abc-def'
    CONCAT_WS(',','abc','def','ghi') -> 'abc,def,ghi'
    
    # NUMBER
    # TRUNCATE(X,D)
    SELECT TRUNCATE(12.4567,1) -> 12.4
    SELECT TRUNCATE(12.4567,0) -> 12
    SELECT TRUNCATE(12.4567,-1) -> 10 # when D<0, input 0 at that position
    SELECT TRUNCATE(12.4567,-1) -> 0
    # ROUND(X,Y)
    SELECT ROUND(12.4567,1) ->12.5
    # CEIL(X), CEILING(X)
    SELECT CEILING(12.4567) ->13
    # FLOOR(X)
    SELECT FLOOR(12.4567) ->12
    
    # CONVERT
    CAST(value as type), CONVERT(value,type) # type: DECIMAL(P,D), SIGNED(int), UNSIGNED
    SELECT CAST('1.3434' AS signed) ->1
    SELECT CAST('1.3434' AS DECIMAL(2,1)) -> 1.3 #D<=P, P:significant digit
    
    # odd or even: MOD(X,Y)
    SELECT MOD(3,2) -> 1
    # OR
    SELECT 3%2 -> 1
    ```
    
- adjust structure/find schema
    - add row number
        
        ```sql
        # add rowNumber for each student
        SELECT a.*
        , (@ROW:;=@ROW+1) AS rowNumber # second step: traversing the data
        FROM student_info a, (SELECT @ROW:=0) b  # first step: setup 0; @: self-defined var.
        
        # add rowNumber only for male
        SELECT a.*
        , (@ROW:= CASE WHEN gender='male' THEN @ROW+1 ELSE 0 END) AS rowNumber
        FROM student_info a, (SELECT @ROW:=0) b 
        ```
        
    - pivot
        
        ```sql
        # row: better storage:[student_id,subject,score]
        # col: better read:[studen_id,subject1,subject2]
        
        # row to col
        # way1: case when
        SELECT student_id
        , SUM(CASE WHEN subject='math' THEN score END) AS'math' # SUM,MAX,MIN,AVG the same, necessary to use or lots of null
        , SUM(CASE WHEN subject='english' THEN score END) AS'english'
        FROM student_score
        
        # way2: if()
        SELECT student_id
        , SUM(IF(subject='math',score,0)) AS 'math'
        , SUM(IF(subject='english',score,0 AS'english'
        FROM student_score
        GROUP BY student_id
        
        # way3: with rollup
        SELECT IFNULL(student_id,'summary') AS student_id
        , SUM(IF(subject='math',score,0)) AS 'math'
        , SUM(IF(subject='english',score,0 AS'english'
        , SUM(IF(subject='total',score,0 AS'total'
        FROM (
        	SELECT student_id
        	, IFNULL(subject,'total') AS subject
        	, SUM(score) AS score
        	FROM student_score
        	GROUP BY student_id, subject WITH ROLLUP # for each student
        	HAVING sTudent_id IS NOT NULL
        	) a
        GROUP BY student_id with ROLLUP  # for each subject
        
        # col to row
        # UNION
        SELECT student_id,'math' AS subject, math AS score FROM sudent_score_2
        UNION ALL # won't delete duplicate, faster than UNION
        SELECT student_id,'english' AS subject, english AS score FROM sudent_score_2
        ```
        
    - check schema, col
        
        ```sql
        # schema: fuzzy search
        SHOW TABLES LIKE "%product%";
        
        # col: given colname, search out schema
        # there is an default table: information_schema
        SELECT information_schema.columns
        WHERE column_name like 'price';
        # common use: table_schema, table_name, column_name
        ```
        
- Some Concepts
    
    group by will destroy the structure of table
    
    put null at last or remove: order by (-age) DESC; or order by ISNULL(age)
    
    - `like` and `regexp`
        
        ```sql
        st1: "garage"
        LIKE "ga" # null
        REGEXP "ga" # garage
        
        # zifu set
        with tmp as (select 'lassssas' as str1)
        # match 'la' or 'sa'
        SELECT * FROM tmp
        WHERE REGEXP_LIKE(str1,'[ls]a') = TRUE # lassssas
        # match 'xa', x represent any instead of 'l'
        SELECT * FROM tmp
        WHERE REGEXP_LIKE(str1,'[^l]a') # sa
        # start with a 
        '^a'
        # end with a
        'a$'
        
        # any number
        [0123456789] 
        # or 
        [0-9]
        # any alpha
        [a-zA-Z]
        # any alpha and num
        [a-zA-Z0-9]
        # how to match string start with num
        '^[0-9]'
        
        with tmp as (select 'lassssas' as str1)
        SELECT REGEXP_EXTRACT(str1,'as?') AS result1 # before?, 's' only shows up 0 or 1 time
        , REGEXP_EXTRACT(str1,'as*') AS result2 # before*, 's' shows up more than 0 time (included)
        , REGEXP_EXTRACT(str1,'as+') AS result3 # before*, 's' shows up more and 1 time (included)
        FROM tmp
        > result1: 'as'
        > result2: 'asssss'
        > result3: 'asssss'
        
        ```
        
    - upgrade the code
        
        multiple joins: using `where` first, then `join`
        
        avoid `count(distinct col)`
        
        prefer `union all` rather than `or`
        
        prefer `exists` rather than `in`
        
- SHORTCUT
