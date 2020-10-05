
### Move than N in a group
There is a table courses with columns: student and class. Please list out all classes which have more than or equal to 5 students.

```bash
-- For example, the table:
--
-- +---------+------------+
-- | student | class      |
-- +---------+------------+
-- | A       | Math       |
-- | B       | English    |
-- | C       | Math       |
-- | D       | Biology    |
-- | E       | Math       |
-- | F       | Computer   |
-- | G       | Math       |
-- | H       | Math       |
-- | I       | Math       |
-- +---------+------------+
-- Should output:
--
-- +---------+
-- | class   |
-- +---------+
-- | Math    |
-- +---------+
```

#### solution
* `having`

```sql
select class
from courses
group by class
having count(distinct student) >=5
```

### No holes ranking
Write a SQL query to rank scores. If there is a tie between two scores, both should have the same ranking. Note that after a tie, the next ranking number should be the next consecutive integer value. In other words, there should be no "holes" between ranks.

```bash
-- +----+-------+
-- | Id | Score |
-- +----+-------+
-- | 1  | 3.50  |
-- | 2  | 3.65  |
-- | 3  | 4.00  |
-- | 4  | 3.85  |
-- | 5  | 4.00  |
-- | 6  | 3.65  |
-- +----+-------+
-- For example, given the above Scores table, your query should generate the following report (order by highest score):
--
-- +-------+---------+
-- | score | Rank    |
-- +-------+---------+
-- | 4.00  | 1       |
-- | 4.00  | 1       |
-- | 3.85  | 2       |
-- | 3.65  | 3       |
-- | 3.65  | 3       |
-- | 3.50  | 4       |
-- +-------+---------+
```

#### solution
* `dense_rank()`

```sql
select
    score,
    dense_rank() over (order by Score desc) as Rank
from Scores
```

### Second Highest Salary
Write a SQL query to get the second highest salary from the Employee table.

```bash
-- +----+--------+
-- | Id | Salary |
-- +----+--------+
-- | 1  | 100    |
-- | 2  | 200    |
-- | 3  | 300    |
-- +----+--------+
-- For example, given the above Employee table, the query should return 200 as the second highest salary. If there is no second highest salary, then the query should return null.
--
-- +---------------------+
-- | SecondHighestSalary |
-- +---------------------+
-- | 200                 |
-- +---------------------+
--
```

#### Solution
* `offset`
* `dense_rank()`

```sql
-- Solution 1 use max and sub-query
select
    max(salary) as SecondHighestSalary
from employee
where salary <>
(
    select
        max(Salary)
    from Employee
)

-- Solution 2 use order and offset
select
	distinct
    salary as SecondHighestSalary
from employee
order by Salary desc
limit 1 offset 1

-- Solution 3 use window function with dense_rank()
select
    salary as SecondHighestSalary
	from
(
	select
		salary,
		dense_rank() over (order by Salary desc) as rank
	from Employee
)
where rank = 2
```

### Swap column value
Given a table salary, such as the one below, that has m=male and f=female values. Swap all f and m values (i.e., change all f values to m and vice versa) with a single update statement and no intermediate temp table.

Note that you must write a single update statement, DO NOT write any select statement for this problem.
```bash
-- Example:
--
-- | id | name | sex | salary |
-- |----|------|-----|--------|
-- | 1  | A    | m   | 2500   |
-- | 2  | B    | f   | 1500   |
-- | 3  | C    | m   | 5500   |
-- | 4  | D    | f   | 500    |

-- After running your update statement, the above salary table should have the following rows:

-- | id | name | sex | salary |
-- |----|------|-----|--------|
-- | 1  | A    | f   | 2500   |
-- | 2  | B    | m   | 1500   |
-- | 3  | C    | f   | 5500   |
-- | 4  | D    | m   | 500    |
```

#### solution
* `update`, `set`
* `CASE`, `IF`

```sql
-- Solution 1 user case statement
update salary
set sex = case
		when sex = 'm' then 'f'
        else 'm' end

-- Solution 2 user IF
update salary set sex = IF (sex = "m", "f", "m");
```

### Boring movie
X city opened a new cinema, many people would like to go to this cinema. The cinema also gives out a poster indicating the movies’ ratings and descriptions. Please write a SQL query to output movies with an odd numbered ID and a description that is not 'boring'. Order the result by rating.

```bash
-- For example, table cinema:
--
-- +---------+-----------+--------------+-----------+
-- |   id    | movie     |  description |  rating   |
-- +---------+-----------+--------------+-----------+
-- |   1     | War       |   great 3D   |   8.9     |
-- |   2     | Science   |   fiction    |   8.5     |
-- |   3     | irish     |   boring     |   6.2     |
-- |   4     | Ice song  |   Fantacy    |   8.6     |
-- |   5     | House card|   Interesting|   9.1     |
-- +---------+-----------+--------------+-----------+
-- For the example above, the output should be:
-- +---------+-----------+--------------+-----------+
-- |   id    | movie     |  description |  rating   |
-- +---------+-----------+--------------+-----------+
-- |   5     | House card|   Interesting|   9.1     |
-- |   1     | War       |   great 3D   |   8.9     |
-- +---------+-----------+--------------+-----------+
--
```

#### solution
* `mod()`

```sql
select
    *
from cinema
where mod(id, 2) <> 0
and description <> 'boring'
order by rating desc
```

### Delect duplicate values
Write a SQL query to delete all duplicate email entries in a table named Person, keeping only unique emails based on its smallest Id.

```bash
-- +----+------------------+
-- | Id | Email            |
-- +----+------------------+
-- | 1  | john@example.com |
-- | 2  | bob@example.com  |
-- | 3  | john@example.com |
-- +----+------------------+
-- Id is the primary key column for this table.
-- For example, after running your query, the above Person table should have the following rows:
--
-- +----+------------------+
-- | Id | Email            |
-- +----+------------------+
-- | 1  | john@example.com |
-- | 2  | bob@example.com  |
-- +----+------------------+
```

#### Solution
* `DELETE`, `FROM`, `WHERE`

```sql
DELETE p1
FROM Person p1, Person p2
WHERE p1.Email = p2.Email AND p1.Id > p2.Id
```

### N consective number
SQL query to find all numbers that appear at least three times consecutively.

```bash
-- +----+-----+
-- | Id | Num |
-- +----+-----+
-- | 1  |  1  |
-- | 2  |  1  |
-- | 3  |  1  |
-- | 4  |  2  |
-- | 5  |  1  |
-- | 6  |  2  |
-- | 7  |  2  |
-- +----+-----+
-- For example, given the above Logs table, 1 is the only number that appears consecutively for at least three times.
--
-- +-----------------+
-- | ConsecutiveNums |
-- +-----------------+
-- | 1               |
-- +-----------------+
```

#### Solution
* 'lag()' or 'lead()'
* self-join

```sql
-- Solution 1 user window function
select
    distinct
    Num as ConsecutiveNums
from
(
	select
		Id,
		Num,
		lead(Num) over (order by Id) as lead1,
		lead(Num, 2) over (order by Id) as lead2
	from Logs
)
where Num = lead1 and Num = lead2

-- Solution 2. user self join
select
	Distinct
	a.num as ConsecutiveNums
from logs a, logs b, logs c
where a.Id = b.Id + 1 and a.Id = c.ID + 2 and a.Num = b.Num and a.Num = c.Num
```

### Top N salary of department
The Employee table holds all employees. Every employee has an Id, a salary, and there is also a column for the department Id.

```bash
-- +----+-------+--------+--------------+
-- | Id | Name  | Salary | DepartmentId |
-- +----+-------+--------+--------------+
-- | 1  | Joe   | 70000  | 1            |
-- | 2  | Jim   | 90000  | 1            |
-- | 3  | Henry | 80000  | 2            |
-- | 4  | Sam   | 60000  | 2            |
-- | 5  | Max   | 90000  | 1            |
-- +----+-------+--------+--------------+
-- The Department table holds all departments of the company.
--
-- +----+----------+
-- | Id | Name     |
-- +----+----------+
-- | 1  | IT       |
-- | 2  | Sales    |
-- +----+----------+
-- Write a SQL query to find employees who have the highest salary in each of the departments. For the above tables, your SQL query should return the following rows (order of rows does not matter).
--
-- +------------+----------+--------+
-- | Department | Employee | Salary |
-- +------------+----------+--------+
-- | IT         | Max      | 90000  |
-- | IT         | Jim      | 90000  |
-- | Sales      | Henry    | 80000  |
-- +------------+----------+--------+
-- Explanation:
--
-- Max and Jim both have the highest salary in the IT department and Henry has the highest salary in the Sales department.
```
#### Solution
* `dense_rank()`

```sql
-- Use 3 as example

CREATE TEMP TABLE rank_number AS
SELECT 3 AS rank_number;

select
    Department,
    Employee,
    Salary
From
(
select
    d.Name as Department,
    e.Name as Employee,
    e.salary as Salary,
    dense_rank() over (partition by d.Name order by e.salary desc) as salary_rank
from Employee e
join Department d on e.DepartmentId = d.Id
)
where salary_rank <= (select rank_number from rank_number)

-- If the questions as for just the 1 top records of person of each department
-- Top 1
With top1 as
(
  SELECT
    d.name as Department,
    max(e.Salary) as max_Salary
  from Department d
  join Employee e on d.id = e.departmentid
  group by 1
)
SELECT
  distinct
  a.Department as Department,
  a.employee as Employee,
  a.salary as Salary
from
(
  SELECT
    d.name as Department,
    e.Name as employee,
    e.salary as salary
  from Department d
  join Employee e on d.id = e.departmentid
) a
join top1 b on a.salary = b.max_salary and a.Department = b.Department
```

### Trip and Users
The Trips table holds all taxi trips. Each trip has a unique Id, while Client_Id and Driver_Id are both foreign keys to the Users_Id at the Users table. Status is an ENUM type of (‘completed’, ‘cancelled_by_driver’, ‘cancelled_by_client’).

```bash
--+----+-----------+-----------+---------+--------------------+----------+
--| Id | Client_Id | Driver_Id | City_Id |        Status      |Request_at|
--+----+-----------+-----------+---------+--------------------+----------+
--| 1  |     1     |    10     |    1    |     completed      |2013-10-01|
--| 2  |     2     |    11     |    1    | cancelled_by_driver|2013-10-01|
--| 3  |     3     |    12     |    6    |     completed      |2013-10-01|
--| 4  |     4     |    13     |    6    | cancelled_by_client|2013-10-01|
--| 5  |     1     |    10     |    1    |     completed      |2013-10-02|
--| 6  |     2     |    11     |    6    |     completed      |2013-10-02|
--| 7  |     3     |    12     |    6    |     completed      |2013-10-02|
--| 8  |     2     |    12     |    12   |     completed      |2013-10-03|
--| 9  |     3     |    10     |    12   |     completed      |2013-10-03|
--| 10 |     4     |    13     |    12   | cancelled_by_driver|2013-10-03|
--+----+-----------+-----------+---------+--------------------+----------+
--The Users table holds all users. Each user has an unique Users_Id, and Role is an ENUM type of (‘client’, ‘driver’, ‘partner’).
--
--+----------+--------+--------+
--| Users_Id | Banned |  Role  |
--+----------+--------+--------+
--|    1     |   No   | client |
--|    2     |   Yes  | client |
--|    3     |   No   | client |
--|    4     |   No   | client |
--|    10    |   No   | driver |
--|    11    |   No   | driver |
--|    12    |   No   | driver |
--|    13    |   No   | driver |
--+----------+--------+--------+
--Write a SQL query to find the cancellation rate of requests made by unbanned users (both client and driver must be unbanned) between Oct 1, 2013 and Oct 3, 2013. The cancellation rate is computed by dividing the number of canceled (by client or driver) requests made by unbanned users by the total number of requests made by unbanned users.
--
--For the above tables, your SQL query should return the following rows with the cancellation rate being rounded to two decimal places.
--
--+------------+-------------------+
--|     Day    | Cancellation Rate |
--+------------+-------------------+
--| 2013-10-01 |       0.33        |
--| 2013-10-02 |       0.00        |
--| 2013-10-03 |       0.50        |
--+------------+-------------------+
```

#### Solution
* `CTE`
* multiple join

```sql
-- Solution 1 CTE, divide and conque
with banned_trips as
(
    select
        distinct
        t.ID
    from Trips t
    join Users u
    on (t.Client_Id = u.Users_id  or t.Driver_id = u.Users_id)
    where u.Banned = 'Yes'
    and t.Request_at between '2013-10-01' and '2013-10-03'
),

unbanned_trips as
(
    select
        distinct
        t.ID,
        t.Status,
        t.Request_at
    from Trips t
    join Users u
    on (t.Client_Id = u.Users_id  or t.Driver_id = u.Users_id)
    where t.ID not in (select ID from banned_trips)
    and t.Request_at between '2013-10-01' and '2013-10-03'
)

select
    Request_at as Day,
    round(sum(case
            when Status in ('cancelled_by_driver', 'cancelled_by_client') then 1.0
            else 0.0 end)/count(distinct ID), 2) as "Cancellation rate"
from  unbanned_trips
group by Request_at
order by Request_at

-- Solution 2. Multiple join
select
	request_at as Day,
	cast(sum(case
        when Status!='completed' then 1.0
        else 0.0 end)/count(1) as decimal(4,2)) as Cancellation Rate
from Trips a
inner join
(
    select
        users_id
    from users
    where role='client' and banned='No'
) b on a.Client_id=b.users_id
inner join
(
    select
        users_id from users
    where role='driver' and banned='No'
) c on a.Driver_id=c.users_id
where request_at>='2013-10-01' and request_at<='2013-10-03'
group by request_at
```

### Cumulative Salary of employee
The Employee table holds the salary information in a year. Write a SQL to get the cumulative sum of an employee's salary over a period of 3 months but exclude the most recent month.

The result should be displayed by 'Id' ascending, and then by 'Month' descending.

```bash
-- Example Input
--
-- | Id | Month | Salary |
-- |----|-------|--------|
-- | 1  | 1     | 20     |
-- | 2  | 1     | 20     |
-- | 1  | 2     | 30     |
-- | 2  | 2     | 30     |
-- | 3  | 2     | 40     |
-- | 1  | 3     | 40     |
-- | 3  | 3     | 60     |
-- | 1  | 4     | 60     |
-- | 3  | 4     | 70     |
-- Output
--
-- | Id | Month | Salary |
-- |----|-------|--------|
-- | 1  | 3     | 90     |
-- | 1  | 2     | 50     |
-- | 1  | 1     | 20     |
-- | 2  | 1     | 20     |
-- | 3  | 3     | 100    |
-- | 3  | 2     | 40     |

-- Explanation
-- Employee '1' has 3 salary records for the following 3 months except the most recent month '4': salary 40 for month '3', 30 for month '2' and 20 for month '1' So the cumulative sum of salary of this employee over 3 months is 90(40+30+20), 50(30+20) and 20 respectively.
--
-- | Id | Month | Salary |
-- |----|-------|--------|
-- | 1  | 3     | 90     |
-- | 1  | 2     | 50     |
-- | 1  | 1     | 20     |
-- Employee '2' only has one salary record (month '1') except its most recent month '2'.
--
-- | Id | Month | Salary |
-- |----|-------|--------|
-- | 2  | 1     | 20     |
-- Employ '3' has two salary records except its most recent pay month '4': month '3' with 60 and month '2' with 40. So the cumulative salary is as following.
--
-- | Id | Month | Salary |
-- |----|-------|--------|
-- | 3  | 3     | 100    |
-- | 3  | 2     | 40     |
```

#### Solution
* window function
* `ROW BETWEEN ... AND PROCEEDING`

```sql
select
	id,
	month,
	sum(Salary) over (partition by Id order by Month ROWS between -1 and proceeding) as Salary)
from Employee
```
