Assume you are given the tables below about Facebook pages and page likes. Write a query to return the page IDs of all the Facebook pages that don't have any likes. The output should be in ascending order.

pages Table:
Column Name	    Type
page_id	        integer
name	        varchar

pages Example Input:

page_id	name
20001	SQL Solutions
20045	Brain Exercises
20701	Tips for Data Analysts

page_likes Table:
Column_Name	Type
user_id	    integer
page_id	    integer
liked_date	datetime

page_likes Example Input:  

user_id	    page_id	    liked_date  
111	        20001	    04/08/2022 00:00:00  
121	        20045	    03/12/2022 00:00:00  
156	        20001	    07/25/2022 00:00:00  

Example Output:
page_id
20701

Explanation: The page with ID 20701 has no likes.

Hint #1

First, we will have to find a relationship between the two tables pages and page_likes. Which common field can help us achieve that?

That's right! page_id is the correct answer.

And, how can we combine these two tables based on the page_id values?

If you said, JOIN you are right on the money.

Let's read more about the [JOIN](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-joins/) clause.

Which JOIN do you think will be a good choice to solve this problem?

Hint #2

Did you find out which JOIN will be suitable?

No worries if you didn't. We are here to help you.

We definitely need an OUTER JOIN to get the necessary results. We can use either a LEFT JOIN or a RIGHT JOIN. For now, let's choose the LEFT JOIN.

Our query should look similar to this:

SELECT pg.page_id
FROM pages pg
LEFT OUTER JOIN page_likes pl
ON  pg.page_id = pl.page_id
<missing_clause>
Notice that the page_id column from the pages table is on the left side of the join which is critical!

Now, there is one remaining clause missing which can help get the page ids we need.

Any ideas on what missing?

Hint #3

You might already be aware of this.

As mentioned before, since we're using the LEFT JOIN clause, there may be not a match of the page_id column value in the page_likes table. In that case, the query will return a NULL value in all of the columns returned from that table.

Therefore, to know which page_id values in the pages table are not in the page_likes table, we'll need to add this WHERE clause seen in the query below.

Here is your reward for hanging in there. A complete query.

Go ahead, try this query in the editor to see if you get the expected results now!

```sql
SELECT pg.page_id
FROM pages pg
LEFT OUTER JOIN page_likes pl 
ON pg.page_id = pl.page_id
WHERE pl.page_id IS NULL;
```

alternative way..  
```sql
SELECT page_id 
FROM pages 
WHERE page_id NOT IN (
SELECT page_id FROM page_likes 
);
```

```sql
SELECT page_id
FROM pages
WHERE NOT EXISTS (
  SELECT 1
  FROM page_likes AS likes
  WHERE likes.page_id = pages.page_id);
```

```sql
SELECT page_id FROM pages
EXCEPT
SELECT page_id FROM page_likes
```

```sql
SELECT p.page_id 
  FROM pages p
 WHERE NOT EXISTS
       (
       SELECT 1 FROM page_likes l
        WHERE l.page_id = p.page_id
        )
ORDER BY p.page_id;
```

If you run this query, you'll be able to see all possible results of left join 
SELECT a.*, b.*
FROM pages a
LEFT JOIN page_likes b 
ON a.page_id = b.page_id
ORDER BY a.page_id;


Notice that there's no data from table b(page_likes) corresponding to some page_id from table a(pages).
Which is where we apply the is null filter(user_id/ page_id/ liked_date)
```sql
SELECT a.*, b.*
FROM pages a
LEFT JOIN page_likes b 
ON a.page_id = b.page_id
WHERE b.user_id IS NULL
ORDER BY a.page_id;
```

Not the best explaination but I hope this helps.