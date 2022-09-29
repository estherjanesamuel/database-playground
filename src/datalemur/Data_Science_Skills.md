# Data Science Skills

Given a table of candidates and their skills, you're tasked with finding the candidates best suited for an open Data Science job. You want to find candidates who are proficient in Python, Tableau, and PostgreSQL.

Write a SQL query to list the candidates who possess all of the required skills for the job. Sort the the output by candidate ID in ascending order.

Assumption:

There are no duplicates in the candidates table.

candidates Table:
Column Name	    Type
candidate_id	integer
skill	        varchar

candidates Example Input:
candidate_id	skill
123	Python
123	Tableau
123	PostgreSQL
234	R
234	PowerBI
234	SQL Server
345	Python
345	Tableau

Example Output:
candidate_id
123

Explanation
Candidate 123 is displayed because they have Python, Tableau, and PostgreSQL skills. 345 isn't included in the output because they're missing one of the required skills: PostgreSQL.

Hint #1

First, we'll filter out any rows that don't fit within the necessary skillset. All we care about is Python, PostgreSQL, and Tableau.

We can use the IN clause to accomplish this step:

SELECT
  candidate_id
FROM candidates
WHERE 
skill IN ('Python', 'Tableau', 'PostgreSQL')
The query above will show all candidates having at least one of the required skills. But, this is not what we want. Do we?

Next, we need to determine how many of these skills each candidate have. You might want to look into GROUP BY, HAVING and COUNT functions to get a head start.

Hint #2

Did you put everything together? Let's find out!

We already eliminated the rows with unnecessary skills. Moreover, there are no duplicate records in the table. Therefore, a candidate is qualified for the job if they have all the required skills; in other words, if their skill-count is 3.

We'll separate the entries in the candidates table into groups using the GROUP BY clause. Then, we'll count the number of skills for each group using the COUNT function and filter the records using a HAVING clause.

```sql
SELECT
  candidate_id
FROM candidates
WHERE 
skill IN ('Python', 'Tableau', 'PostgreSQL')
GROUP BY fill_in_column
HAVING COUNT(skill) = 3
```

Can you guess what fill in column might be? Don't forget to sort the output as per problem description :)


```sql
SELECT candidate_id
FROM candidates
WHERE skill IN ('Python' , 'Tableau' , 'PostgreSQL')
GROUP BY candidate_id
HAVING COUNT(skill) = 3
ORDER BY candidate_id;

```

