# Average Review Ratings
Given the reviews table, write a query to get the average stars for each product every month.

The output should include the month in numerical value, product id, and average star rating rounded to two decimal places. Sort the output based on month followed by the product id.

P.S. If you've read the Ace the Data Science Interview, and liked it, consider writing us a review?

reviews Table:
Column Name	Type
review_id	integer
user_id	integer
submit_date	datetime
product_id	integer
stars	integer (1-5)
reviews Example Input:
review_id	user_id	submit_date	product_id	stars
6171	123	06/08/2022 00:00:00	50001	4
7802	265	06/10/2022 00:00:00	69852	4
5293	362	06/18/2022 00:00:00	50001	3
6352	192	07/26/2022 00:00:00	69852	3
4517	981	07/05/2022 00:00:00	69852	2
Example Output:
mth	product	avg_stars
6	50001	3.50
6	69852	4.00
7	69852	2.50
Explanation
In June (month #6), product 50001 had two ratings - 4 and 3, resulting in an average star rating of 3.5.


Hint #1

Let's take the first step. The question is asking for the month in the output, but we do not see any month column in the reviews table. So, how do we get this column?

If you're thinking of extracting it from the submit_date column, then you're on the right path! The EXTRACT function will help us out here.

Your code snippet should look similar to this:

SELECT EXTRACT(MONTH FROM submit_date) AS mth
FROM reviews;
Try it out in the editor!

Hint #2

O-kay! Next, let's introduce you to one of our good friends, the AVG function which is very useful and frequently asked in technical interviews.

Do you which column we should use to find the average of star ratings?

SELECT 
  EXTRACT(MONTH FROM submit_date) AS mth,
  product_id,
  AVG(_____) AS avg_stars
FROM reviews
GROUP BY EXTRACT(MONTH FROM submit_date), _____;
Want to understand more about the AVG function? Head over here for some reading.

Hint #3

Now that you know how the AVG function works and what the output looks like, we'll introduce AVG's other friend, the ROUND function which you can read about it here. They usually come in pairs.

Here's how you can apply the ROUND function with the AVG function.

SELECT 
  EXTRACT(MONTH FROM submit_date) AS mth,
  product_id,
  ROUND(AVG(stars), 2) AS avg_stars
FROM reviews
GROUP BY EXTRACT(MONTH FROM submit_date), product_id
ORDER BY _____, _____;
Can you complete the blanks in the ORDER BY clause based on "Sort the output based on month followed by the product id"?

## Solution
As we can see, there is no month column in the reviews table. First, we have to extract the month from the submit_date column.

There is a simple function to extract month from a date. Here's the syntax: EXTRACT(MONTH from column_name)

You can look at this page for more explanation on the EXTRACT function.

After extracting the month in numerical values, get the average of the star ratings and round them to two decimal places. It can be achieved using the functions AVG() and ROUND(). Please refer [1] & [2] for some reading on the functions.

Solution:
```sql
SELECT 
  EXTRACT(MONTH FROM submit_date) AS mth,
  product_id,
  ROUND(AVG(stars), 2) AS avg_stars
FROM reviews
GROUP BY EXTRACT(MONTH FROM submit_date), product_id
ORDER BY mth, product_id;
```
Why can't we write mth in the GROUP BY clause, but we can do so in the ORDER BY clause?

This is the order of sequence of how SQL executes the solution's query from top to bottom:

1 - FROM reviews
2 - GROUP BY EXTRACT(MONTH FROM submit_date) ...
3 - SELECT EXTRACT(MONTH FROM submit_date) AS mth ...
4 - ORDER BY mth ...
SQL runs the GROUP BY clause BEFORE the SELECT statement. Hence, when SQL executes the grouping, it cannot say GROUP BY mth because the mth column only exists AFTER the SELECT statement is executed.

The reason why we can execute mth column in the ORDER BY clause is because it's run after the SELECT statement and mth column has been created.

This is a very popular question in technical interviews so make sure that you learn the order of SQL execution well!
#### Answer
```sql
SELECT 
 EXTRACT(MONTH FROM submit_date) AS mth,
 product_id as product,
 ROUND(AVG(stars),2)
FROM reviews
GROUP BY 
  EXTRACT(MONTH FROM submit_date),
 product_id
ORDER BY  mth ,product 
;

```

### Discusion
```sql
SELECT DISTINCT extract(Month from submit_date) as mth,
        product_id as product,
        round(avg(stars) over(partition by extract(Month from submit_date), product_id), 2) avg_stars
FROM reviews
order by 1, 2;

SELECT 
date_part('month',submit_date) as mth,
product_id AS product,
round(avg(stars),2) as avg_stars
FROM reviews
GROUP BY mth,product
ORDER BY mth,product_id;

with t1 as
(
select review_id, user_id, date_part('month', submit_date) as mth, product_id, stars
from reviews
)

select DISTINCT mth, product_id, 
ROUND(AVG(stars) OVER(PARTITION BY mth, product_id),2) as avg_stars
from t1
ORDER BY mth, product_id

SELECT date_part('month', submit_date ) as mth, product_id as product, ROUND(AVG(stars),2) as avg_stars FROM reviews GROUP BY product_id, date_part('month', submit_date ) ORDER BY mth, product

WITH avg_reviews AS 
(SELECT 
EXTRACT(MONTH FROM submit_date) as mth, 
product_id as product, 
AVG(stars) as avg_stars 
FROM reviews 
GROUP BY product_id, mth) 
SELECT mth, product, ROUND(avg_stars,2) as avg_stars
FROM avg_reviews
ORDER BY mth, product;

SELECT
    MONTH, 
    PRODUCT_ID AS PRODUCT,
    ROUND(AVG(STARS), 2) AS AVG_STARS
FROM
    (
    SELECT
        PRODUCT_ID,
        EXTRACT(MONTH FROM SUBMIT_DATE) AS MONTH,
        STARS 
    FROM
        REVIEWS
    ) AS INNER_TABLE
GROUP BY 
    MONTH, PRODUCT
ORDER BY
    MONTH, PRODUCT
    

    with cte as (SELECT EXTRACT(month from submit_date) as month ,product_id,stars,
Round(avg(stars) over(partition by EXTRACT(month from submit_date),product_id),2)
as avg_stars,row_number() over(partition by extract(month from submit_date),product_id order by stars) as rn 
FROM reviews)

select month,product_id,avg_stars from cte 
where rn = 1


with temp_b as(select distinct  mh,count(stars) over(partition by product_id,mh) as total_no_rev,product_id from(SELECT EXTRACT(Month from submit_date) as mh,product_id,stars FROM reviews
order by submit_date)a
order by mh),
temp_a as(
select distinct mh,Sum(stars) over(partition by product_id,mh) as sum_reviews,product_id from( SELECT EXTRACT(Month from submit_date) as mh,product_id,stars FROM reviews
order by submit_date)a
order by a.mh
)
select tb.mh as month,ta.product_id as product,round(((sum_reviews*1.0)/(total_no_rev)),2) as avg_stars from
temp_a ta,temp_b tb
where 
ta.mh=tb.mh
and ta.product_id=tb.product_id
order by tb.mh,ta.product_id;

WITH CTE (product, rating, bymonth) AS 
(SELECT 
product_id, 
stars, 
EXTRACT(MONTH FROM submit_date) as monthnum 
FROM reviews),
CTE2 (bymonth2, product2, avgStars2) AS (SELECT 
bymonth,
product,
AVG(rating) OVER (Partition BY product ORDER BY bymonth asc) as avgStars FROM CTE)
SELECT bymonth2, product2, CAST(avgStars2 as DECIMAL(5,2)) as finalAVG
FROM CTE2
GROUP BY product2, bymonth2, finalAVG
ORDER BY bymonth2, product2
```


