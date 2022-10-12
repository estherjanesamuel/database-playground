# This is the same question as problem #30 in the SQL Chapter of Ace the Data Science Interview!

Assume you are given the following tables on Walmart transactions and products. Find the top 3 products that are most frequently bought together (purchased in the same transaction).

Output the name of product #1, name of product #2 and number of combinations in descending order.

Assumption:

Each transaction contains the purchase of 2 products at most.
transactions Table:
Column Name	Type
transaction_id	integer
product_id	integer
user_id	integer
transaction_date	datetime
transactions Example Input:
transaction_id	product_id	user_id	transaction_date
231574	111	234	03/01/2022 12:00:00
231574	444	234	03/01/2022 12:00:00
231574	222	234	03/01/2022 12:00:00
137124	111	125	03/05/2022 12:00:00
137124	444	125	03/05/2022 12:00:00
products Table:
Column Name	Type
product_id	integer
product_name	string
products Example Input:
product_id	product_name
111	apple
222	soy milk
333	instant oatmeal
444	banana
555	chia seed
Example Output:
product1	product2	combo_num
banana	apple	2
banana	soy milk	1
soy milk	apple	1
Explanation
Banana and apple are both present in two transactions (231574 and 137124), so they're at the top of the results with a combo_num of 2.


Hint #1

To start off, let's join the transactions and products tables using a related column so that we can combine this information.

Can you tell which is the related column? It's a column that exists in both tables.

SELECT *
FROM transactions
INNER JOIN products
	ON transactions._____ = products._____;
Hint #2

Yep, it's the product_id column!

SELECT 
  transactions.user_id, 
  transactions.product_id, 
  transactions.transaction_id, 
  products.product_name 
FROM transactions
INNER JOIN products 
  ON transactions.product_id = products.product_id;
Now that we have a table containing the product_names for each transaction, we can perform a SELF_JOIN with the above table's output to pull the products that were purchased together in the same transaction.

Here is an example of a code with SELF JOIN.

WITH [name_your_CTE_table] AS (
-- Insert the above query
)

SELECT *
FROM [name_your_CTE_table] AS table_1
INNER JOIN [name_your_CTE_table] AS table_2
	ON table_1._____ = table_2._____
    	AND table_1._____ __ table_2._____;
We want all pairs of products, but we also want to be mindful not to overcount them. For example, we only want to count (apple, instant oatmeal) once and not count (instant oatmeal, apple) as another pair.

How do we ensure that this does not happen? If you don't know, click to get another hint...

Hint #3

We can ensure that overcounting does not occur by including an additional condition as part of the SELF JOIN:

WITH [name_your_CTE_table] AS (
-- Insert the above query
)

SELECT *
FROM [name_your_CTE_table] AS table_1
INNER JOIN [name_your_CTE_table] AS table_2
  ON table_1.transaction_id = table_2.transaction_id
    AND table_1.product_id > table_2.product_id; -- Additional condition
Finally, in the SELECT clause, you should select only product #1, product #2 and count of the product combination for the final output. Remember to sort our output in the requested order and limit it to the top 3 only!

### Solution 
Goal: Top 3 products that are most frequently bought together (purchased in a single transaction).

Let's start small by joining the transactions and products tables together based on the related field product_id.

```sql
SELECT 
  transactions.product_id,
  transactions.transaction_id, 
  products.product_name 
FROM transactions 
INNER JOIN products 
  ON transactions.product_id = products.product_id;
```
Showing the first 5 rows of output:

user_id	product_id	transaction_id	product_name
234	111	231574	apple
234	444	231574	banana
234	222	231574	soy milk
125	444	137124	banana
311	222	256234	soy milk
Using the query above, we create a CTE and perform a SELF JOIN to fetch products that were purchased together by a single user by joining on transaction_id.

Note that we want all pairs of the products, but we do not want to overcount. For example, if user A purchased an apple and instant oatmeal in the same transaction, then we only want to count the (apple, instant oatmeal) transaction once, and not also (instant oatmeal, apple).

To avoid double-counting, we can use a condition within the SELF JOIN that the product_id of A is less than that of product B. Lastly, we use a GROUP BY clause for each pair of products.
```sql
WITH purchase_info 
AS (
-- Insert the above query
) 

SELECT 
  p1.product_name AS product1, 
  p2.product_name AS product2, 
  COUNT(*) AS combo_num 
FROM purchase_info AS p1 
INNER JOIN purchase_info AS p2 
  ON p1.transaction_id = p2.transaction_id 
    AND p1.product_id > p2.product_id 
GROUP BY p1.product_name, p2.product_name 
Output:
```

product1	product2	combo_num
banana	apple	2
banana	soy milk	2
instant oatmeal	soy milk	1
soy milk	apple	1
Did you see how there is no overcounting of the different combinations of the same products?

Lastly, we use a GROUP BY clause for each pair of products and sort the resulting count by descending, taking the top 3 using the LIMIT clause:

Results:

product1	product2	combo_num
banana	apple	2
banana	soy milk	2
instant oatmeal	soy milk	1
Solution:
```sql
WITH purchase_info AS (
  SELECT 
    transactions.product_id,
    transactions.transaction_id, 
    products.product_name 
  FROM transactions 
  INNER JOIN products 
    ON transactions.product_id = products.product_id
) 

SELECT 
  p1.product_name AS product1, 
  p2.product_name AS product2, 
  COUNT(*) AS combo_num 
FROM purchase_info AS p1 
INNER JOIN purchase_info AS p2 
  ON p1.transaction_id = p2.transaction_id 
    AND p1.product_id > p2.product_id 
GROUP BY p1.product_name, p2.product_name 
ORDER BY combo_num DESC 
LIMIT 3;
```

```sql
WITH cte AS (
  SELECT 
  transactions.user_id, 
  transactions.product_id, 
  transactions.transaction_id, 
  products.product_name As product_name
FROM transactions
INNER JOIN products 
  ON transactions.product_id = products.product_id
)
		
SELECT 
  table_1.product_name AS product1,  table_2.product_name AS product2, COUNT(table_1.product_name) AS combo_num
FROM cte AS table_1
INNER JOIN cte AS table_2
  ON table_1.transaction_id = table_2.transaction_id
    AND table_1.product_id > table_2.product_id
GROUP BY table_1.product_name,  table_2.product_name
ORDER BY combo_num DESC
LIMIT 3 
```
### Discussion
```sql
SELECT transaction_id, product_name, t.product_id
FROM transactions t
join
products p
on
t.product_id = p.product_id
),
cte1 as (
select c.transaction_id, c1.transaction_id, c.product_name as product1, 
c1.product_name as product2 
from cte c
join
cte c1
on
c.transaction_id = c1.transaction_id
and
c.product_id > c1.product_id
),
cte2 as (
select product1, product2, count(*) as combo_num,
row_number()over(order by count(*) desc) as rn
from cte1
group by 1,2
)
select product1, product2, combo_num
from cte2 where rn <= 3;

with product AS
(
SELECT t1.product_id, t1.user_id, p.product_name, t1.transaction_id
from transactions t1 join products p
on t1.product_id = p.product_id
)

select p2.product_name as product1, p1.product_name as product2,
count(*) as combo_num
from product p1 join product p2
on p1.transaction_id = p2.transaction_id and p1.product_id < p2.product_id
group by p1.product_name, p2.product_name
order by combo_num desc, product1, product2
limit 3


SELECT
p2.product_name AS product1,
p1.product_name AS product2,
COUNT(*) AS combo_num
FROM
transactions t1 JOIN transactions t2 ON t1.transaction_id = t2.transaction_id AND t1.product_id < t2.product_id
JOIN products p1 ON t1.product_id = p1.product_id
JOIN products p2 ON t2.product_id = p2.product_id
GROUP BY 1,2
ORDER BY 3 DESC
LIMIT 3;


WITH CTE AS (
SELECT * ,
ROW_NUMBER() OVER(PARTITION BY user_id ORDER BY  TR. product_id ) AS RW
FROM transactions AS TR
INNER JOIN products AS PR
ON TR.product_id = PR.product_id)


SELECT C2.product_name AS product1 ,
C1.product_name AS product2 ,
 COUNT( C1.user_id ) AS combo_num
FROM CTE AS C1
INNER JOIN CTE AS C2
ON C1.user_id = C2.user_id AND C2.RW > C1.RW
GROUP BY C1.product_name , C2.product_name
ORDER BY product1  , product2  , combo_num DESC 
LIMIT 3 


WITH product_combos AS (
    SELECT
        t1.product_id AS product1,
        t2.product_id AS product2,
        COUNT(DISTINCT t1.transaction_id) AS combo_num
    FROM transactions t1
        JOIN transactions t2 ON t1.transaction_id = t2.transaction_id
            AND t1.product_id > t2.product_id
    GROUP BY t1.product_id, t2.product_id
)

SELECT
    p1.product_name AS product1,
    p2.product_name AS product2,
    c.combo_num
FROM product_combos c
    JOIN products p1 ON c.product1 = p1.product_id
    JOIN products p2 ON c.product2 = p2.product_id
ORDER BY combo_num DESC--, p1.product_name
LIMIT 3
;


with product_by_transaction as (
select  
  transaction_id, 
  product_id,
  row_number() over (PARTITION BY transaction_id order by product_id desc) as ranking

from transactions

group by 1,2

)
  SELECT 
    p1.product_name as product1
    , p2.product_name as product2
    , count(*) as combo_num
  
  FROM (select * from product_by_transaction where ranking=1) t1
    INNER JOIN transactions t2
      on (t1.transaction_id = t2.transaction_id
          AND t1.product_id != t2.product_id)
          
    INNER JOIN products p1
      on t1.product_id = p1.product_id
    INNER JOIN products p2
      on t2.product_id = p2.product_id
      
  group by 1,2

order by combo_num desc
    ;


    WITH t_1 AS(
SELECT t1.transaction_id,t1.product_id AS p_1,t2.product_id AS p_2 FROM transactions t1,
transactions t2
WHERE t1.transaction_id=t2.transaction_id AND t1.product_id>t2.product_id),

t_2 AS(
SELECT p_1,p_2,COUNT(transaction_id) AS cnt FROM t_1
GROUP BY p_1,p_2),

t_3 AS(
SELECT p_1,p_2,cnt,DENSE_RANK() OVER(ORDER BY cnt DESC)AS rnk
FROM t_2),

t_4 AS(
SELECT p1.product_name AS product1,p2.product_name AS product2,cnt AS combo_num FROM products p1,products p2,t_3
WHERE p_1=p1.product_id AND p_2=p2.product_id)

SELECT * FROM t_4
ORDER BY combo_num DESC,product1,product2
LIMIT 3;

with cte as
(
SELECT 
transaction_id
,product_id,
coalesce (lag(product_id) over(partition by transaction_id order by product_id) 
,first_value(product_id) over(partition by transaction_id order by product_id desc)) as purchased_pair 
FROM transactions t
)

SELECT p1.product_name AS product1
,p.product_name AS product2
,count(distinct transaction_id) as bought_together
FROM cte t JOIN  products p
on p.product_id = t.product_id
JOIN  products p1
on p1.product_id = t.purchased_pair
group by p.product_name,p1.product_name
order by count(distinct transaction_id) desc
limit 3

count(distinct table_1.transaction_id) as combo_num
from 
(SELECT     t.user_id, 
    t.product_id, 
    t.transaction_id, 
    p.product_name
FROM transactions t inner join products p
on t.product_id = p.product_id) table_1 join 
(SELECT     t.user_id, 
    t.product_id, 
    t.transaction_id, 
    p.product_name
FROM transactions t inner join products p
on t.product_id = p.product_id) table_2
ON table_1.transaction_id = table_2.transaction_id 
    AND table_1.product_id > table_2.product_id 
  group by 1,2
limit 3


WITH cte AS
(
SELECT a.product_id AS product_1 ,
b.product_id AS product_2 , a.user_id , a.transaction_id
FROM transactions AS a 
INNER JOIN transactions AS b
ON a.user_id = b.user_id AND a.product_id > b.product_id
) , 
cte1 AS
(
SELECT a.* , 
ROW_NUMBER() OVER (ORDER BY combo_num DESC) AS seq FROM
(
SELECT b.product_name AS product1 ,
c.product_name AS product2, COUNT(1) AS combo_num
FROM
cte AS a 
INNER JOIN
products AS b ON a.product_1 = b.product_id 
INNER JOIN 
products AS c ON a.product_2 = c.product_id
GROUP BY b.product_name , c.product_name
) AS a
) 
SELECT a.product1 , a.product2 , a.combo_num
FROM cte1 a WHERE seq <= 3;


WITH CTE AS (SELECT  
  a.product_id as ProdA,
  p.product_name as NameA,
  b.product_id as ProdB,
  pr.product_name NameB,
  RANK() OVER (PARTITION BY a.product_id, b.product_id ORDER BY a.transaction_id)
 FROM transactions a
  JOIN transactions b ON a.transaction_id = b.transaction_id 
  AND a.product_id < b.product_id
  JOIN products p ON p.product_id = a.product_id
  JOIN products pr ON pr.product_id = b.product_id)
  
SELECT NameB, NameA, MAX(rank) as combo FROM CTE
 GROUP BY NameB, NameA
  ORDER BY MAX(rank) desc, NameB, NameA
```