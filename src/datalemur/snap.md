This is the same question as problem #25 in the SQL Chapter of Ace the Data Science Interview!

Assume you are given the tables below containing information on Snapchat users, their ages, and their time spent sending and opening snaps. Write a query to obtain a breakdown of the time spent sending vs. opening snaps (as a percentage of total time spent on these activities) for each of the different age groups.

Output the age bucket and percentage of sending and opening snaps. Round the percentages to 2 decimal places.

Notes:

You should calculate these percentages:
time sending / (time sending + time opening)
time opening / (time sending + time opening)
To avoid integer division in percentages, multiply by 100.0 and not 100.
activities Table:
Column Name	Type
activity_id	integer
user_id	integer
activity_type	string ('send', 'open', 'chat')
time_spent	float
activity_date	datetime
activities Example Input:
activity_id	user_id	activity_type	time_spent	activity_date
7274	123	open	4.50	06/22/2022 12:00:00
2425	123	send	3.50	06/22/2022 12:00:00
1413	456	send	5.67	06/23/2022 12:00:00
1414	789	chat	11.00	06/25/2022 12:00:00
2536	456	open	3.00	06/25/2022 12:00:00
age_breakdown Table:
Column Name	Type
user_id	integer
age_bucket	string ('21-25', '26-30', '31-25')
age_breakdown Example Input:
user_id	age_bucket
123	31-35
456	26-30
789	21-25
Example Output:
age_bucket	send_perc	open_perc
26-30	65.40	34.60
31-35	43.75	56.25
Explanation
For the age bucket 26-30, the time spent sending snaps was 5.67 and opening 3. The percent of time sending snaps was 5.67/(5.67+3)=65.4%, and the percent of time opening snaps was 3/(5.67+3)=34.6%.


Hint #1

We'll need a table containing all the necessary Snapchat information from both tables. Can you start off with joining both of the tables?

SELECT *
FROM activities
JOIN age_breakdown AS age
  ON activities.user_id = age.user_id
Can you filter the joined table for 'send' and 'open' activity types and find the total of the time spent by sessions using the conditional CASE statement?

CASE statement are very useful and often tested in technical interviews so be sure to familiarise yourself with it!

Hint #2

Here's how you should write the CASE statements:
```sql
SELECT 
  age.fill_in_column, 
  SUM(CASE WHEN activities.activity_type = 'send' 
      THEN activities.time_spent ELSE 0 END) AS send_timespent, 
  SUM(
    CASE WHEN activities.fill_in_column_2 = 'open' 
      THEN activities.fill_in_column_3 ELSE 0 END) AS open_timespent, 
  SUM(activities.fill_in_column_3) AS total_timespent 
FROM activities
JOIN age_breakdown AS age 
  ON activities.user_id = age.user_id 
WHERE activities.activity_type IN ('send', 'open') 
GROUP BY age.fill_in_column
```
Let's breakdown the CASE statement into parts:
```sql
SUM( -- 2
  CASE WHEN activities.activity_type = 'send' -- 1
    THEN activities.time_spent ELSE 0 END) AS send_timespent
```

1 - When activity_type is 'send', we take the value of time_spent. If activity_type is anything but 'send', then we take the value of 0.

2 - Then, we sum up the values as the time spent for sending snaps and alias it as send_timespent.

Now, we have to wrap the code above in a CTE and find the percentage of sending and opening the snaps. Do you know how to do it?

Hint #3

Try and run the code below and complete the empty columns:
```sql
WITH snaps_statistics 
AS ( 
  SELECT 
    age.age_bucket, 
    SUM(CASE WHEN activities.activity_type = 'send' 
      THEN activities.time_spent ELSE 0 END) AS send_timespent, 
    SUM(CASE WHEN activities.activity_type = 'open' 
      THEN activities.time_spent ELSE 0 END) AS open_timespent, 
    SUM(activities.time_spent) AS total_timespent 
  FROM activities
  JOIN age_breakdown AS age 
    ON activities.user_id = age.user_id 
  WHERE activities.activity_type IN ('send', 'open') 
  GROUP BY age.age_bucket
) 

SELECT 
  fill_in_column_1, 
  (send_timespent / total_timespent) AS send_perc, 
  (fill_in_column_2 / fill_in_column_3) AS open_perc 
FROM snaps_statistics; 
Missing something? You'll have to round the percentages to 2 decimal places. Use the ROUND function to help you out!
```

# Solution
First, we join the age_bucket field from the age_breakdown table on the corresponding user_id field with the activities table. Next, we filter for 'send' and 'open' activity_type because we have to find the percentage of sending and opening snaps and then group by age_bucket.

Subsequently, we obtain the total time spent on sending and opening snaps using conditional CASE statements for each activity type ('send' and 'open') and also find the total time spent.

```sql
SELECT 
  age.age_bucket, 
  SUM(CASE WHEN activities.activity_type = 'send' 
      THEN activities.time_spent ELSE 0 END) AS send_timespent, 
  SUM(CASE WHEN activities.activity_type = 'open' 
      THEN activities.time_spent ELSE 0 END) AS open_timespent, 
  SUM(activities.time_spent) AS total_timespent 
FROM activities
INNER JOIN age_breakdown AS age 
  ON activities.user_id = age.user_id 
WHERE activities.activity_type IN ('send', 'open') 
GROUP BY age.age_bucket;
```

Here's how the output looks like:

age_bucket	send_timespent	open_timespent	total_timespent
21-25	6.24	5.25	11.49
26-30	13.91	3.00	16.91
31-35	3.50	5.75	9.25
Then, we convert the query into a CTE called snaps_statistics and using the formula below, we calculate the percentages of the time spent sending and opening snaps.

Percentage of time spend on sending snaps = Time spent on sending snaps / Total time spend on sending and opening snaps

```sql
WITH snaps_statistics AS (
-- Insert above query here
)

SELECT 
  age_bucket, 
  ROUND(100.0 * send_timespent / total_timespent, 2) AS send_perc, 
  ROUND(100.0 * open_timespent / total_timespent, 2) AS open_perc 
FROM snaps_statistics;
```

When dividing 2 integers, the division / operator in PostgreSQL only considers the integer part of the results and do not deal with the remainder (ie. .123, .352). Hence, to avoid integer division which happens when we multiply by 100 and to convert the numerical values into decimal values, we multiply the values with 100.0.

Results:

age_bucket	send_perc	open_perc
21-25	54.31	45.69
26-30	82.26	17.74
31-35	37.84	62.16
Solution:

```sql
WITH snaps_statistics AS (
  SELECT 
    age.age_bucket, 
    SUM(CASE WHEN activities.activity_type = 'send' 
      THEN activities.time_spent ELSE 0 END) AS send_timespent, 
    SUM(CASE WHEN activities.activity_type = 'open' 
      THEN activities.time_spent ELSE 0 END) AS open_timespent, 
    SUM(activities.time_spent) AS total_timespent 
  FROM activities
  JOIN age_breakdown AS age 
    ON activities.user_id = age.user_id 
  WHERE activities.activity_type IN ('send', 'open') 
  GROUP BY age.age_bucket) 

SELECT 
  age_bucket, 
  ROUND(100.0 * send_timespent / total_timespent, 2) AS send_perc, 
  ROUND(100.0 * open_timespent / total_timespent, 2) AS open_perc 
FROM snaps_statistics;
```

## Alternative
```sql
WITH cte AS (
  SELECT 
    user_id
    , activity_type
    , SUM(time_spent) AS time_spent
  FROM activities
  WHERE activity_type != 'chat'
  GROUP BY
    user_id
    , activity_type
),

cte2 AS (
  SELECT 
    B.age_bucket
    , activity_type
    , time_spent
    , SUM(time_spent) OVER (PARTITION BY age_bucket) AS total_time_spent
  FROM cte A
  JOIN age_breakdown B
  ON A.user_id = B.user_id
)

SELECT
  age_bucket
  , ROUND(SUM(CASE 
      WHEN activity_type = 'send' THEN time_spent / total_time_spent
      ELSE NULL
      END)*100.0,2) AS send_perc
  , ROUND(SUM(CASE 
      WHEN activity_type = 'open' THEN time_spent / total_time_spent
      ELSE NULL
      END)*100.0,2) AS open_perc
FROM cte2
GROUP BY age_bucket;

with cte as (
select user_id,
       activity_type,
       sum(time_spent) as time_spent from activities
       group by user_id,activity_type)

SELECT d.age_bucket,
       b.send_perc,b.open_perc 
       from(
       select a.user_id,
              round(a.time_spent/(a.time_spent + b.time_spent)* 100.0,2) as send_perc,
              round(b.time_spent/(a.time_spent + b.time_spent) * 100.0,2) as open_perc
              from cte a,cte b
              where a.user_id = b.user_id and a.activity_type= 'send' and b.activity_type='open') b
              inner join age_breakdown d on b.user_id = d.user_id order by d.age_bucket asc;

SELECT ab.age_bucket, 
ROUND(sum(
case when activity_type = 'send' 
     then time_spent else 0 
end
) * 100.0 / SUM(time_spent), 2) as send_perc,
ROUND(sum(
case when activity_type = 'open' 
     then time_spent else 0 
end
) * 100.0 / SUM(time_spent), 2) as open_perc
FROM activities a
join age_breakdown ab
on a.user_id = ab.user_id
where activity_type in ('open', 'send')
group by ab.age_bucket

with CTE_Total_spent_time as (
SELECT
user_id
,sum(CASE WHEN activity_type ='send' then time_spent end) as send_time
,sum(CASE WHEN activity_type ='open' then time_spent end) as open_time
FROM activities
group by user_id
)

select
b.age_bucket as age_bucket
,cast(send_time / (send_time + open_time) * 100.0 as decimal(18,2)) as send_perc
,cast(open_time / (send_time + open_time) * 100.0 as decimal(18,2)) as open_perc
from CTE_Total_spent_time a JOIN age_breakdown b
On a.user_id = b.user_id
order by age_bucket;

WITH cte AS (
        SELECT
        act.activity_type,
        abd.age_bucket,
        SUM(act.time_spent) OVER (PARTITION BY act.activity_type, age_bucket) AS time_send_or_open,
        SUM(act.time_spent) OVER (PARTITION BY age_bucket) AS total_time_spent
        FROM activities act
        LEFT JOIN age_breakdown abd ON act.user_id = abd.user_id
        WHERE act.activity_type IN ('send', 'open'))
        
SELECT
        DISTINCT(cte1.age_bucket),
        ROUND((cte2.time_send_or_open / cte2.total_time_spent) * 100, 2) AS send_perc,
        ROUND((cte3.time_send_or_open / cte3.total_time_spent) * 100, 2) AS open_perc
  FROM cte AS cte1
JOIN cte AS cte2 ON cte1.age_bucket = cte2.age_bucket
JOIN cte AS cte3 ON cte1.age_bucket = cte3.age_bucket
  WHERE cte2.activity_type = 'send' AND cte3.activity_type = 'open'


select 
b.age_bucket,
round( 100.0*(sum(CASE WHEN a.activity_type = 'send' then a.time_spent else 0 end)/sum(a.time_spent)),2) as send,
round( 100.0*(sum(CASE WHEN a.activity_type = 'open' then a.time_spent else 0 end) /sum(a.time_spent)),2) as open

-- sum(a.time_spent) as total
from activities  a left join age_breakdown as b
on a.user_id = b.user_id 
where a.activity_type in ('open' , 'send')
GROUP BY age_bucket

WITH base AS 
(SELECT age_bucket, activity_type, time_spent
FROM activities JOIN age_breakdown USING (user_id)
WHERE activity_type in ('open', 'send')),

base_2 AS 
(SELECT age_bucket, activity_type,
SUM(time_spent) AS total
FROM base
GROUP BY age_bucket, activity_type
),

base_3 AS
(SELECT age_bucket, 
SUM(CASE WHEN activity_type = 'open' THEN total END) as open, 
SUM(CASE WHEN activity_type = 'send' THEN total END) as send
FROM base_2
GROUP BY age_bucket)

SELECT age_bucket, ROUND(send / (send + open) * 100.0, 2) AS send_perc, ROUND(open / (send + open) * 100.0, 2) AS open_perc
FROM base_3;


;with activity_time AS(select age_bucket, activity_type,
      case when activity_type = 'chat' then  SUM(time_spent) else 0 end as chat,
      case when activity_type = 'send' then  SUM(time_spent) else 0 end as send,
      case when activity_type = 'open' then  SUM(time_spent) else 0 end as open
from activities a
inner join age_breakdown ab on ab.user_id = a.user_id
group by age_bucket, activity_type)

SELECT age_bucket, CAST(((sum(send)/(sum(send)+sum(open))) * 100.00) AS decimal(4,2)) send_perc,
        CAST(((sum(open)/(sum(send)+sum(open))) * 100.00) AS decimal(4,2))  open_perc
    FROM activity_time
    GROUP BY age_bucket
    ORDER BY age_bucket


with age_grouping as (SELECT a.user_id,a.activity_type,a.time_spent, ab.age_bucket FROM activities as a
inner join age_breakdown as ab 
on a.user_id=ab.user_id
where a.activity_type!='chat'),
open_time as(
Select age_bucket,SUM(time_spent)as opentime
from age_grouping
where activity_type='open'
group by age_bucket
order by age_bucket),
send_time as(
Select age_bucket,SUM(time_spent)as sendtime
from age_grouping
where activity_type='send'
group by age_bucket
order by age_bucket)
select open_time.age_bucket, 
round(((100.0*send_time.sendtime)/(open_time.opentime+send_time.sendtime)),2) as send_perc,
round(((100.0*open_time.opentime)/(open_time.opentime+send_time.sendtime)),2) as open_perc
from open_time
inner join send_time
on open_time.age_bucket=send_time.age_bucket

select ages,round(100 * sts/(ots + sts),2) as send_perc,
ROUND(100 *ots/(ots + sts),2) as open_perc
from(
SELECT age_bucket as ages,
SUM(case when activity_type = 'open'
then time_spent end) as ots, sum(
case when activity_type = 'send'
then time_spent end) as sts
from activities a
join age_breakdown g
on a.user_id = g.user_id
where activity_type in ('send','open')
GROUP BY age_bucket) as bn
-- ) as bbb
group by ages,ots,sts
ORDER BY ages

with output(activity_type,pct,age_bucket)
as 
(select activity_type,round(sum(s1)/sum(s2) * 100.0,2) as pct,age_bucket from 
(SELECT user_id,activity_type,sum(time_spent) as s1,sum(sum(time_spent)) 
OVER(partition by user_id) as s2
FROM activities
group by user_id,activity_type having activity_type!='chat'
order by user_id) a 
inner join age_breakdown b using (user_id) 
group by age_bucket,activity_type)


select 
age_bucket,
sum(coalesce(CASE when activity_type='send' then pct end,0)) as send_perc,
sum(coalesce(CASE when activity_type='open' then pct end,0)) as open_perc
from output
group by age_bucket
order by age_bucket;

WITH   --send total
SEND as (
select 
age_breakdown.age_bucket, 
activities.activity_type as send_activity_type,
sum(time_spent) as sum_time_spent from activities  
inner join age_breakdown on age_breakdown.user_id = activities.user_id
group by activities.activity_type,age_breakdown.age_bucket
having activity_type = 'send'

),

OPEN as (  --open total
select 
age_breakdown.age_bucket, 
activities.activity_type as open_activity_type,
sum(time_spent) as sum_time_spent from activities  
inner join age_breakdown on age_breakdown.user_id = activities.user_id
group by activities.activity_type,age_breakdown.age_bucket
having activity_type = 'open'


)

select OPEN.age_bucket,    
ROUND(SEND.sum_time_spent / (SEND.sum_time_spent + OPEN.sum_time_spent ) * 100.0, 2) AS send_perc, 
ROUND(OPEN.sum_time_spent / (SEND.sum_time_spent + OPEN.sum_time_spent ) * 100.0, 2 ) AS open_perc 
from SEND
inner join OPEN on OPEN.age_bucket = SEND.age_bucket

WITH cte as (
SELECT user_id ,
SUM (CASE when activity_type = 'send' THEN time_spent end  ) as Tot_spent
from activities
GROUP BY user_id ) ,

abc as (
SELECT user_id ,
SUM (CASE when activity_type = 'open' THEN time_spent end  ) as Tot_spent
from activities
GROUP BY user_id ) ,

mnb as ( 
SELECT user_id ,
SUM(CASE when activity_type = 'open' OR  activity_type = 'send' 
THEN time_spent end  ) as Tot_spent
from activities
GROUP BY user_id)

SELECT age_breakdown.age_bucket,ROUND((cte.Tot_spent/ mnb.Tot_spent)* 100.0 ,2 )as send_perc
,ROUND((abc.Tot_spent/ mnb.Tot_spent)* 100.0,2) as open_perc
FROM mnb JOIN  abc  on abc.user_id = mnb.user_id
 JOIN  cte  on abc.user_id = cte.user_id
 join age_breakdown on age_breakdown.user_id = cte.user_id
ORDER BY age_breakdown.age_bucket

select age_bucket, 
CAST(sum(case when activity_type = 'send' then time_spent else 0 END)/(sum(case when activity_type = 'send' then time_spent else 0 END)+sum(case when activity_type = 'open' then time_spent else 0 end))*100 AS DECIMAL(8,2)) AS "send_perc",
CAST(sum(case when activity_type = 'open' then time_spent else 0 END)/(sum(case when activity_type = 'send' then time_spent else 0 END)+sum(case when activity_type = 'open' then time_spent else 0 end))*100 AS DECIMAL (8,2))  AS "open_perc"
from activities a join age_breakdown b on a.user_id = b.user_id
group by age_bucket
order by age_bucket

SELECT age_breakdown.age_bucket,
      ROUND(100.0 * SUM(CASE WHEN activities.activity_type = 'send' 
      THEN activities.time_spent END)/SUM(activities.time_spent),2) AS send_perc,
      ROUND(100.0 * SUM(CASE WHEN activities.activity_type = 'open'
      THEN activities.time_spent END)/SUM(activities.time_spent), 2) AS open_perc
      FROM activities JOIN age_breakdown 
      ON activities.user_id = age_breakdown.user_id
      AND activities.activity_type != 'chat'
      GROUP BY age_breakdown.age_bucket;

WITH joined AS(
  SELECT a.user_id, a.activity_type, a.time_spent, a.activity_date, b.age_bucket
  FROM activities a JOIN age_breakdown b ON a.user_id = b.user_id
  WHERE a.activity_type = 'open' OR a.activity_type = 'send'
),
sends AS(
  SELECT age_bucket,
         SUM(time_spent) OVER(PARTITION BY age_bucket) AS send_time
  FROM joined
  WHERE activity_type = 'send'
),
opens AS(
  SELECT age_bucket,
         SUM(time_spent) OVER(PARTITION BY age_bucket) AS open_time
  FROM joined
  WHERE activity_type = 'open'
)
SELECT DISTINCT s.age_bucket,
       ROUND(100.0*(send_time)/(send_time + open_time),2) AS send_perc,
       ROUND(100.0*(open_time)/(send_time + open_time),2) AS open_perc
FROM sends s JOIN opens o ON s.age_bucket = o.age_bucket

SELECT
  age_bucket,
  round(((send_time/(send_time + open_time))*100.0),2) as send_perc,
  round(((open_time / (send_time + open_time))*100),2) as open_perc
from
(SELECT
				age_bucket,
				sum(case when activity_type = 'send' then time_spent else 0 end) as send_time,
			sum(case when activity_type = 'open' then time_spent else 0 end) as open_time
			from
				((select
				user_id,
				activity_type,
				time_spent
				FROM activities
				where activity_type <> 'chat') a
				JOIN
				age_breakdown age
				on a.user_id = age.user_id) sub1
				group by age_bucket
) sub2;

SELECT age_bucket,
       ROUND(SUM(time_spent) filter(WHERE activity_type = 'send') / SUM(time_spent) filter(WHERE activity_type = 'open' OR activity_type = 'send') * 100.0, 2) AS send_perc,
       ROUND(SUM(time_spent) filter(WHERE activity_type = 'open') / SUM(time_spent) filter(WHERE activity_type = 'open' OR activity_type = 'send') * 100.0, 2) AS open_perc
FROM activities ac
JOIN age_breakdown a ON ac.user_id = a.user_id
GROUP BY 1

with cte as (SELECT 
ab.age_bucket, sum(Case WHEN a.activity_type='open' then a.time_spent END) as open,
sum(Case WHEN a.activity_type='send' then a.time_spent END) as send
FROM activities a
join age_breakdown ab
on a.user_id=ab.user_id
where activity_type != 'chat'
GROUP BY ab.age_bucket)
select age_bucket,round((send/(open+send)*100.0),2) as pct_send,round((open/(open+send)*100.0),2) as pct_open
from cte;

WITH split_sum as (
SELECT ab.age_bucket, 
       SUM(CASE 
           WHEN a.activity_type in ('send', 'open')
           THEN a.time_spent
           END) AS total_sum,
       SUM(CASE 
           WHEN a.activity_type = 'send'
           THEN a.time_spent
           END) AS send_sum,
       SUM(CASE
           WHEN a.activity_type = 'open'
           THEN a.time_spent
           END) AS open_sum
FROM activities as a 
JOIN age_breakdown as ab 
ON a.user_id = ab.user_id
GROUP BY ab.age_bucket
)
SELECT age_bucket,
       ROUND((100.0*send_sum/total_sum),2) as send_perc,
       ROUND((100.0*open_sum/total_sum),2) as open_perc
FROM split_sum

SELECT s.age_bucket, 
        round(100 * sending / (sending + opening), 2) as send_perc,
        round(100 * opening / (sending + opening), 2) as open_perc
  FROM sends s
  join opens o on s.age_bucket = o.age_bucket;

  WITH cte AS
(
  SELECT a.user_id , a.activity_type , a.time_spent , 
  b.age_bucket
  FROM activities AS a INNER JOIN
  age_breakdown AS b 
  ON a.user_id = b.user_id AND 
  a.activity_type IN ('open' , 'send')
) , 
cte1 AS
(
  SELECT activity_type , age_bucket , total_time_spent ,
  SUM(time_spent) AS activity_type_time_spent
  FROM
  (
  SELECT activity_type , age_bucket , time_spent ,
  SUM(time_spent) OVER(PARTITION BY age_bucket) AS total_time_spent
  FROM cte AS a
  ) AS a
  GROUP BY activity_type , age_bucket , total_time_spent
  ORDER BY age_bucket
)
SELECT age_bucket , 
CAST(MAX(CASE WHEN activity_type = 'open' THEN 
activity_type_time_spent/total_time_spent END)*100.0 AS DECIMAL(10,2))
AS open_perc , 
CAST(MAX(CASE WHEN activity_type = 'send' THEN 
activity_type_time_spent/total_time_spent END)*100.0 AS DECIMAL(10,2))
AS send_perc
FROM cte1
GROUP BY age_bucket
ORDER BY age_bucket;


with cte as (SELECT a.user_id,age_bucket,
       sum(case when activity_type = 'send' then time_spent end) as time_sending,
       sum(case when activity_type = 'open' then time_spent end) as time_opening
FROM activities a 
join age_breakdown ag
on a.user_id  = ag.user_id
group by a.user_id,age_bucket)


select age_bucket,round((time_sending::numeric/(time_sending+time_opening))*100,2) as send_perc,
round((time_opening::numeric/(time_sending+time_opening))*100,2) as open_perc
FROM cte
order by age_bucket 


WITH User_2_posts as(
SELECT 
      user_id,
      COUNT(DISTINCT post_date) as num_of_posts
      FROM
      Posts
WHERE EXTRACT(YEAR FROM post_date ) IN(2021)
      GROUP BY 1
      HAVING COUNT(DISTINCT post_date)>1
),
Last_post as(
SELECT 
      user_id,
      post_id,
      post_date,
    LAG(post_date) over(PARTITION BY user_id ORDER BY post_date) as last_post_date
FROM posts
WHERE user_id in (SELECT user_id FROM User_2_posts)
),
days_calc as(
SELECT
      user_id,
      post_id,
      post_date,
      last_post_date,
      EXTRACT(DAY FROM post_date - last_post_date) AS DateDifference
  FROM
    Last_post)
    SELECT
        user_id,
        ROUND(AVG(DateDifference),1) as days_between
    FROM
    days_calc
    GROUP BY 1;

    

WITH SEN AS (SELECT a.age_bucket, SUM(ac.time_spent) as t_s 
FROM activities as ac
JOIN age_breakdown as a
ON a.user_id = ac.user_id
WHERE ac.activity_type = 'send'
GROUP BY a.age_bucket),

OPE AS (SELECT a.age_bucket, SUM(ac.time_spent) as t_o 
FROM activities as ac
JOIN age_breakdown as a
ON a.user_id = ac.user_id
WHERE ac.activity_type = 'open'
GROUP BY a.age_bucket)

SELECT s.age_bucket, 
ROUND((t_s / (t_o + t_s))*100.0, 2),
ROUND((t_o / (t_o + t_s))*100.0, 2)
FROM SEN as s
JOIN OPE as o
ON o.age_bucket = s.age_bucket

WITH SendCNT (user1, totalsend) AS (
 SELECT user_id, SUM(time_spent) FROM activities
  WHERE activity_type = 'send'
   GROUP BY user_id),
OpenCNT (user2, totalopen) AS (
 SELECT user_id, SUM(time_spent) FROM activities
  WHERE activity_type = 'open'
   GROUP BY user_id)
SELECT age_bucket, 
  ROUND(totalsend * 100.0/ (totalsend + totalopen),2) as send_perc,
  ROUND(totalopen * 100.0 / (totalsend + totalopen),2) as open_perc
FROM sendcnt
  JOIN opencnt ON sendCNT.user1 = opencnt.user2
  LEFT JOIN age_breakdown ab ON opencnt.user2 = ab.user_id
 ORDER BY age_bucket

 SELECT ab.age_bucket,
       ROUND( (SUM ( CASE WHEN a.activity_type = 'send' THEN a.time_spent ELSE 0 END ) /  SUM(time_spent))*100, 2) as send_per,
       ROUND( (SUM ( CASE WHEN a.activity_type = 'open' THEN a.time_spent ELSE 0 END ) /  SUM(time_spent)) *100,2) as open_per
FROM activities as a
LEFT JOIN age_breakdown as ab
ON a.user_id = ab.user_id
WHERE a.activity_type in ('open', 'send')
GROUP BY ab.age_bucket

SELECT a_b.age_bucket,ROUND((SELECT SUM(a.time_spent) FROM activities a
WHERE a.user_id=a_b.user_id AND a.activity_type='send')*100/((SELECT SUM(a.time_spent) FROM activities a
WHERE a.user_id=a_b.user_id AND a.activity_type='open')+(SELECT SUM(a.time_spent) FROM activities a
WHERE a.user_id=a_b.user_id AND a.activity_type='send')),2) AS send_per,ROUND((SELECT SUM(a.time_spent) FROM activities a
WHERE a.user_id=a_b.user_id AND a.activity_type='open')*100/((SELECT SUM(a.time_spent) FROM activities a
WHERE a.user_id=a_b.user_id AND a.activity_type='open')+(SELECT SUM(a.time_spent) FROM activities a
WHERE a.user_id=a_b.user_id AND a.activity_type='send')),2) AS open_per FROM
age_breakdown a_b
ORDER BY a_b.age_bucket

with cte as (
select user_id, sum(case when activity_type='send' then time_spent else 0 end)  as time_spent_send
,sum(case when activity_type='open' then time_spent else 0 end) as time_spent_open,sum(time_spent) as time_spent_total
from activities
group by user_id
)
SELECT age_bucket,(sum(time_spent_send)*100.0)/sum(time_spent_total) as time_spent_send_percent,
(sum(time_spent_open)*100.0)/sum(time_spent_total) as time_spent_open_percent FROM age_breakdown
inner join cte on age_breakdown.user_id = cte.user_id
group by age_bucket;


WITH cte AS
(
SELECT age_bucket, SUM(CASE WHEN activity_type = 'send' THEN time_spent ELSE 0 END) as total_time_sending, 
  SUM(CASE WHEN activity_type = 'open' THEN time_spent ELSE 0 END) as total_time_opening
      
FROM activities A LEFT JOIN age_breakdown ab 
      ON A.user_id = ab.user_id 
      
GROUP BY 1 )

SELECT age_bucket, ROUND(100.0 * total_time_sending/ (total_time_sending+total_time_opening),2) as send_perc, 
 ROUND(100.0 * total_time_opening/ (total_time_sending+total_time_opening),2)  as open_perc 
FROM cte;

select ab.age_bucket,
  round(sum(case when a.activity_type = 'send' then time_spent else 0 end)/ sum(case when a.activity_type in ('send', 'open') then time_spent else 0 end), 2) as send_perc,
  round(sum(case when a.activity_type = 'open' then time_spent else 0 end) / sum(case when a.activity_type in ('send', 'open') then time_spent else 0 end), 2) as open_perc
from activities a
left join age_breakdown ab on a.user_id = ab.user_id
group by 1;

with cte_open as (SELECT age_bucket,sum(time_spent) as open_time
FROM activities a1 join age_breakdown a2 on a1.user_id = a2.user_id
where activity_type = 'open'
group by age_bucket),

cte_send as (SELECT age_bucket,sum(time_spent) as send_time
FROM activities a1 join age_breakdown a2 on a1.user_id = a2.user_id
where activity_type = 'send'
group by age_bucket),

cte_total as (SELECT age_bucket,sum(time_spent) as total_time
FROM activities a1 join age_breakdown a2 on a1.user_id = a2.user_id
where activity_type = 'send' or activity_type = 'open'
group by age_bucket)

select * from
(select s.age_bucket, round(send_time/total_time,2) as send_perc from cte_send s join cte_total t on s.age_bucket = t.age_bucket) as tmp2
join
(select o.age_bucket, round(open_time/total_time,2) as open_perc from cte_open o join cte_total t on o.age_bucket = t.age_bucket) as tmp1
on tmp2.age_bucket = tmp1.age_bucket
```

## mine
```sql

WITH snaps_statistic 
AS(
  SELECT 
  b.age_bucket AS age,
    SUM(CASE WHEN a.activity_type = 'send' 
      THEN a.time_spent ELSE 0 END) AS send_timespent,
    SUM(CASE WHEN a.activity_type = 'open'
      THEN a.time_spent ELSE 0 END) AS open_timespent,
    SUM(a.time_spent) as total_timespent
  FROM activities a
  JOIN age_breakdown b 
  ON a.user_id = b.user_id
  WHERE a.activity_type IN ('send', 'open')
  GROUP BY b.age_bucket
)

SELECT
  age,
  ROUND((send_timespent / total_timespent * 100.0),2)  AS send_perc,
  ROUND((open_timespent / total_timespent * 100.0),2)  AS open_perc
FROM snaps_statistic;

/*
WITH snaps_statistics 
AS ( 
  SELECT 
    age.age_bucket, 
    SUM(CASE WHEN activities.activity_type = 'send' 
      THEN activities.time_spent ELSE 0 END) AS send_timespent, 
    SUM(CASE WHEN activities.activity_type = 'open' 
      THEN activities.time_spent ELSE 0 END) AS open_timespent, 
    SUM(activities.time_spent) AS total_timespent 
  FROM activities
  JOIN age_breakdown AS age 
    ON activities.user_id = age.user_id 
  WHERE activities.activity_type IN ('send', 'open') 
  GROUP BY age.age_bucket
) 

SELECT 
  fill_in_column_1, 
  (send_timespent / total_timespent) AS send_perc, 
  (fill_in_column_2 / fill_in_column_3) AS open_perc 
FROM snaps_statistics; 



SELECT 
  age.fill_in_column, 
  SUM(CASE WHEN activities.activity_type = 'send' 
      THEN activities.time_spent ELSE 0 END) AS send_timespent, 
  SUM(
    CASE WHEN activities.fill_in_column_2 = 'open' 
      THEN activities.fill_in_column_3 ELSE 0 END) AS open_timespent, 
  SUM(activities.fill_in_column_3) AS total_timespent 
FROM activities
JOIN age_breakdown AS age 
  ON activities.user_id = age.user_id 
WHERE activities.activity_type IN ('send', 'open') 
GROUP BY age.fill_in_column


SELECT 
  fill_in_column_1, 
  (send_timespent / total_timespent) AS send_perc, 
  (fill_in_column_2 / fill_in_column_3) AS open_perc 
FROM snaps_statistics; 
*/
```