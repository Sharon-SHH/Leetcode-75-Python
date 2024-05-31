[584. Find Customer Referee](https://leetcode.com/problems/find-customer-referee/) (E)

Table: `Customer`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| referee_id  | int     |
+-------------+---------+
In SQL, id is the primary key column for this table.
Each row of this table indicates the id of a customer, their name, and the id of the customer who referred them.
```

 

Find the names of the customer that are **not referred by** the customer with `id = 2`.

Return the result table in **any order**.

The result format is in the following example.

**Algorithm**

MySQL uses three-valued logic -- TRUE, FALSE and UNKNOWN. Anything compared to NULL evaluates to the third value: UNKNOWN. That “anything” includes NULL itself! That’s why MySQL provides the `IS NULL` and `IS NOT NULL` operators to specifically check for NULL.

```sql
SELECT name FROM customer WHERE referee_id <> 2 OR referee_id IS NULL;
SELECT name FROM customer WHERE referee_id != 2 OR referee_id IS NULL;
```

1683. [Invalid Tweets](https://leetcode.com/problems/invalid-tweets/) (E) 

Table: `Tweets`

```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| tweet_id       | int     |
| content        | varchar |
+----------------+---------+
tweet_id is the primary key (column with unique values) for this table.
This table contains all the tweets in a social media app.
```

Write a solution to find the IDs of the invalid tweets. The tweet is invalid if the number of characters used in the content of the tweet is **strictly greater** than `15`.

Return the result table in **any order**.

The result format is in the following example.

```sql
select tweet_id from Tweets
where CHAR_LENGTH(content) > 15
-- Length in bytes
-- SELECT LENGTH('Hello, world!')
```

[1378. Replace Employee ID With The Unique Identifier](https://leetcode.com/problems/replace-employee-id-with-the-unique-identifier/)

Table: `Employees`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
+---------------+---------+
id is the primary key (column with unique values) for this table.
Each row of this table contains the id and the name of an employee in a company. 
```

Table: `EmployeeUNI`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| unique_id     | int     |
+---------------+---------+
(id, unique_id) is the primary key (combination of columns with unique values) for this table.
Each row of this table contains the id and the corresponding unique id of an employee in the company.
```

Write a solution to show the **unique ID** of each user, If a user does not have a unique ID replace just show `null`.

Return the result table in **any** order.

The result format is in the following example.

#### Algorithm

We first perform a LEFT JOIN operation to combine data from both tables based on the `id` column. Similarly, we use LEFT JOIN to ensure that all the rows from the `Employees` table are included in the result, even if there are no matching rows in the `EmployeeUNI` table.

```sql
select U.unique_id, E.name from Employees as E
Left Join EmployeeUNI as U
on U.id = E.id
```



[1581. Customer Who Visited but Did Not Make Any Transactions](https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions/) (E)

Table: `Visits`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| visit_id    | int     |
| customer_id | int     |
+-------------+---------+
visit_id is the column with unique values for this table.
This table contains information about the customers who visited the mall. 
```

Table: `Transactions`

```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| transaction_id | int     |
| visit_id       | int     |
| amount         | int     |
+----------------+---------+
transaction_id is column with unique values for this table.
This table contains information about the transactions made during the visit_id.
```

Write a solution to find the IDs of the users who visited without making any transactions and the number of times they made these types of visits.

Return the result table sorted in **any order**.

The result format is in the following example.

```sql
select customer_id, count(visit_id) as count_no_trans from Visits
where visit_id not in (select visit_id from Transactions)
group by customer_id
```

[1661. Average Time of Process per Machine](https://leetcode.com/problems/average-time-of-process-per-machine/) (E)

Table: `Activity`

```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| machine_id     | int     |
| process_id     | int     |
| activity_type  | enum    |
| timestamp      | float   |
+----------------+---------+
The table shows the user activities for a factory website.
(machine_id, process_id, activity_type) is the primary key (combination of columns with unique values) of this table.
machine_id is the ID of a machine.
process_id is the ID of a process running on the machine with ID machine_id.
activity_type is an ENUM (category) of type ('start', 'end').
timestamp is a float representing the current time in seconds.
'start' means the machine starts the process at the given timestamp and 'end' means the machine ends the process at the given timestamp.
The 'start' timestamp will always be before the 'end' timestamp for every (machine_id, process_id) pair.
```

 

There is a factory website that has several machines each running the **same number of processes**. Write a solution to find the **average time** each machine takes to complete a process.

The time to complete a process is the `'end' timestamp` minus the `'start' timestamp`. The average time is calculated by the total time to complete every process on the machine divided by the number of processes that were run.

The resulting table should have the `machine_id` along with the **average time** as `processing_time`, which should be **rounded to 3 decimal places**.

Return the result table in **any order**.

The result format is in the following example.

 

**Example 1:**

```
Input: 
Activity table:
+------------+------------+---------------+-----------+
| machine_id | process_id | activity_type | timestamp |
+------------+------------+---------------+-----------+
| 0          | 0          | start         | 0.712     |
| 0          | 0          | end           | 1.520     |
| 0          | 1          | start         | 3.140     |
| 0          | 1          | end           | 4.120     |
| 1          | 0          | start         | 0.550     |
| 1          | 0          | end           | 1.550     |
| 1          | 1          | start         | 0.430     |
| 1          | 1          | end           | 1.420     |
| 2          | 0          | start         | 4.100     |
| 2          | 0          | end           | 4.512     |
| 2          | 1          | start         | 2.500     |
| 2          | 1          | end           | 5.000     |
+------------+------------+---------------+-----------+
Output: 
+------------+-----------------+
| machine_id | processing_time |
+------------+-----------------+
| 0          | 0.894           |
| 1          | 0.995           |
| 2          | 1.456           |
+------------+-----------------+
```

### Solution

```sql
select a.machine_id, Round(avg(b.timestamp - a.timestamp), 3) as processing_time 
from Activity a, Activity b
where a.machine_id = b.machine_id and
    a.process_id = b.process_id and 
    a.activity_type = "start" and 
    b.activity_type = "end"
group by machine_id

-- Solution 2
SELECT 
    machine_id,
    ROUND(SUM(CASE WHEN activity_type='start' THEN timestamp*-1 ELSE timestamp END)*1.0
    / (SELECT COUNT(DISTINCT process_id)),3) AS processing_time
FROM 
    Activity
GROUP BY machine_id
```

- `CASE WHEN activity_type='start' THEN timestamp*-1 ELSE timestamp END`:
  - **Purpose:** This conditional logic is applied to each row to determine how to treat the `timestamp` value.
  - Explanation:
    - If `activity_type` is `'start'`, the `timestamp` value is multiplied by `-1`. This effectively inverts the timestamp value for start activities.
    - If `activity_type` is not `'start'`, the `timestamp` value is used as-is.

**`\*1.0 / (SELECT COUNT(DISTINCT process_id))`:**

- **Purpose:** This part computes the average processing time by dividing the total time by the count of distinct `process_id` values.
- Explanation:
  - Multiplying by `1.0` ensures the result is a floating-point number, which is necessary for accurate division.

[577. Employee Bonus](https://leetcode.com/problems/employee-bonus/) (E)

Table: `Employee`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| empId       | int     |
| name        | varchar |
| supervisor  | int     |
| salary      | int     |
+-------------+---------+
empId is the column with unique values for this table.
Each row of this table indicates the name and the ID of an employee in addition to their salary and the id of their manager.
```

 

Table: `Bonus`

```
+-------------+------+
| Column Name | Type |
+-------------+------+
| empId       | int  |
| bonus       | int  |
+-------------+------+
empId is the column of unique values for this table.
empId is a foreign key (reference column) to empId from the Employee table.
Each row of this table contains the id of an employee and their respective bonus.
```

 

Write a solution to report the name and bonus amount of each employee with a bonus **less than** `1000`.

Return the result table in **any order**.

The result format is in the following example.

 

**Example 1:**

```
Input: 
Employee table:
+-------+--------+------------+--------+
| empId | name   | supervisor | salary |
+-------+--------+------------+--------+
| 3     | Brad   | null       | 4000   |
| 1     | John   | 3          | 1000   |
| 2     | Dan    | 3          | 2000   |
| 4     | Thomas | 3          | 4000   |
+-------+--------+------------+--------+
Bonus table:
+-------+-------+
| empId | bonus |
+-------+-------+
| 2     | 500   |
| 4     | 2000  |
+-------+-------+
Output: 
+------+-------+
| name | bonus |
+------+-------+
| Brad | null  |
| John | null  |
| Dan  | 500   |
+------+-------+
```

1. Since foreign key **Bonus.empId** refers to **Employee.empId** and some employees do not have bonus records, we can use `OUTER JOIN` to link these two tables as the first step.

   ```sql
   SELECT
       Employee.name, Bonus.bonus
   FROM
       Employee
           LEFT JOIN
       Bonus ON Employee.empid = Bonus.empid
   | name   | bonus |
   | ------ | ----- |
   | Brad   | null  |
   | John   | null  |
   | Dan    | 500   |
   | Thomas | 2000  |
   ```

   

2. bonus is null or bonus < 1000

```sql
select Employee.name, Bonus.bonus from Employee 
left join Bonus
on Employee.empId = Bonus.empId 
where bonus < 1000 or bonus is null
```

[1211. Queries Quality and Percentage](https://leetcode.com/problems/queries-quality-and-percentage/)

Table: `Queries`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| query_name  | varchar |
| result      | varchar |
| position    | int     |
| rating      | int     |
+-------------+---------+
This table may have duplicate rows.
This table contains information collected from some queries on a database.
The position column has a value from 1 to 500.
The rating column has a value from 1 to 5. Query with rating less than 3 is a poor query.
```

We define query `quality` as:

> The average of the ratio between query rating and its position.

We also define `poor query percentage` as:

> The percentage of all queries with rating less than 3.

Write a solution to find each `query_name`, the `quality` and `poor_query_percentage`.

Both `quality` and `poor_query_percentage` should be **rounded to 2 decimal places**.

Return the result table in **any order**.

The result format is in the following example.

 

**Example 1:**

```
Input: 
Queries table:
+------------+-------------------+----------+--------+
| query_name | result            | position | rating |
+------------+-------------------+----------+--------+
| Dog        | Golden Retriever  | 1        | 5      |
| Dog        | German Shepherd   | 2        | 5      |
| Dog        | Mule              | 200      | 1      |
| Cat        | Shirazi           | 5        | 2      |
| Cat        | Siamese           | 3        | 3      |
| Cat        | Sphynx            | 7        | 4      |
+------------+-------------------+----------+--------+
Output: 
+------------+---------+-----------------------+
| query_name | quality | poor_query_percentage |
+------------+---------+-----------------------+
| Dog        | 2.50    | 33.33                 |
| Cat        | 0.66    | 33.33                 |
+------------+---------+-----------------------+
Explanation: 
Dog queries quality is ((5 / 1) + (5 / 2) + (1 / 200)) / 3 = 2.50
Dog queries poor_ query_percentage is (1 / 3) * 100 = 33.33

Cat queries quality equals ((2 / 5) + (3 / 3) + (4 / 7)) / 3 = 0.66
Cat queries poor_ query_percentage is (1 / 3) * 100 = 33.33
```

```sql
select query_name, Round(avg(rating/position), 2) as quality, 
Round(sum(if(rating<3, 1, 0)) * 100/count(query_name-*), 2) as poor_query_percentage
from Queries
where query_name is not null
group by query_name
```



   
