Tesla is investigating bottlenecks in their production, and they need your help to extract the relevant data. Write a SQL query that determines which parts have begun the assembly process but are not yet finished.

Assumption

Table parts_assembly contains all parts in production.
parts_assembly Table:
Column Name	Type
part	string
finish_date	datetime
assembly_step	integer
parts_assembly Example Input:
part	finish_date	assembly_step
battery	01/22/2022 00:00:00	1
battery	02/22/2022 00:00:00	2
battery	03/22/2022 00:00:00	3
bumper	01/22/2022 00:00:00	1
bumper	02/22/2022 00:00:00	2
bumper		3
bumper		4
Example Output:
part
bumper
Explanation: The only item in the output is "bumper" because step 3 didn't have a finish date.

Hint #1

Do you know how can we filter for rows that have no data present in the finish_date column?

Tip: use a WHERE clause. It might look something like this: WHERE column IS NULL

### solution
The parts table already contains all of the parts that are currently in production, meaning that we do not have to do any additional filtering for the parts that are not in production.

All we need to do is extract the parts that are not yet finished. We can accomplish this by filtering for rows with no data present in the finish_date column. We call these missing values NULL.

Some parts might be represented multiple times in the query data because they have several assembly steps that are not yet complete. To solve this, we can group part to obtain the unique parts.

```sql
SELECT part
FROM parts_assembly
WHERE finish_date IS NULL
GROUP BY part;
```

```sql
SELECTa DISTINCT part FROM parts_assembly WHERE finish_date is NULL;
```
