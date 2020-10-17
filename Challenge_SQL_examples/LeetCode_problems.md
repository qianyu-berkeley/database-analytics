
# Some of the harder LeetCode database problems and solutions


### No holes ranking
Write a SQL query to rank scores. If there is a tie between two scores, both should have the same ranking. Note that after a tie, the next ranking number should be the next consecutive integer value. In other words, there should be no "holes" between ranks.

```
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

```
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
```
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

```
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

### Delete duplicate values
Write a SQL query to delete all duplicate email entries in a table named Person, keeping only unique emails based on its smallest Id.

```
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

```
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
* 'lag' or 'lead'
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

```
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

```
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
-- Solution 1 CTE, divide and conquer
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

```
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
* `ROW BETWEEN n preceding AND current row`

```sql
with no_recent_month as
(
    select
        Id,
        Month,
        Salary
    from
    (
        select
            Id,
            Month,
            Salary,
            row_number() over (partition by Id order by Month desc) as rn
        from Employee
    ) a
    where rn <> 1
)

select
    Id,
    Month,
    sum(Salary) over (partition by Id order by Month rows between 2 preceding and current row) as Salary
from no_recent_month
group by Id, Month
order by Id, Month desc
```

### Calculate Median without using build-in function
The Employee table holds all employees. The employee table has three columns: Employee Id, Company Name, and Salary.

```
+-----+------------+--------+
|Id   | Company    | Salary |
+-----+------------+--------+
|1    | A          | 2341   |
|2    | A          | 341    |
|3    | A          | 15     |
|4    | A          | 15314  |
|5    | A          | 451    |
|6    | A          | 513    |
|7    | B          | 15     |
|8    | B          | 13     |
|9    | B          | 1154   |
|10   | B          | 1345   |
|11   | B          | 1221   |
|12   | B          | 234    |
|13   | C          | 2345   |
|14   | C          | 2645   |
|15   | C          | 2645   |
|16   | C          | 2652   |
|17   | C          | 65     |
+-----+------------+--------+
Write a SQL query to find the median salary of each company. Bonus points if you can solve it without using any built-in SQL functions.

+-----+------------+--------+
|Id   | Company    | Salary |
+-----+------------+--------+
|5    | A          | 451    |
|6    | A          | 513    |
|12   | B          | 234    |
|9    | B          | 1154   |
|14   | C          | 2645   |
+-----+------------+--------+
```

#### Solution
* Window function

```sql
-- solution 1 step by step non-optimized
with counts as
(
   select
       Company,
       count(Salary) as counts
    from Employee
    group by 1
),

median as
(
    select
        Company,
        case when counts%2 = 0 then counts/2 else ceiling(counts/2) end as median_rows
    from counts
    union
    select
        Company,
        case when counts%2 = 0 then counts/2+1 else ceiling(counts/2) end as median_rows
    from counts
),

row_n as
(
    select
        Id,
        Company,
        Salary,
        row_number() over (partition by Company order by Salary) as row_n
    from Employee
)

select
    a.Id,
    a.Company,
    a.Salary
from row_n a
join median b on a.Company = b.Company and a.row_n = b.median_rows

-- solution 2 Optimized version solution 1
With ranks as
(
    Select
        id,
        company,
        salary,
        count(Company) over (partition by company order by salary rows between unbounded preceding and unbounded following) as count_emp,
        row_number() over(partition by Company) as row_rank
    from Employee
)

select
    id,
    company,
    salary
from ranks
where (count_emp%2 = 1 and row_rank = round(count_emp/2,0))
or (count_emp%2 = 0 and row_rank in (count_emp/2, count_emp/2 + 1))

-- solution 3 using the property of Median
select
    Id,
    Company,
    Salary
from
(
    select
        *,
        cast(row_number() over (partition by Company order by Salary asc) as signed) as rn_asc,
        cast( row_number() over (partition by Company order by Salary desc) as signed) as rn_desc
    from Employee
) a
where abs(rn_asc - rn_desc) <= 1
order by Company, Salary

```

### Caculate median base on frequency of numbers

```
The Numbers table keeps the value of number and its frequency.
+----------+-------------+
|  Number  |  Frequency  |
+----------+-------------|
|  0       |  7          |
|  1       |  1          |
|  2       |  3          |
|  3       |  1          |
+----------+-------------+
In this table, the numbers are 0, 0, 0, 0, 0, 0, 0, 1, 2, 2, 2, 3, so the median is (0 + 0) / 2 = 0.

+--------+
| median |
+--------|
| 0.0000 |
+--------+
Write a query to find the median of all numbers and name the result as median.
```

#### Solution

```sql
with cte as
(
   select
       Number,
       Frequency,
       sum(Frequency) over (order by Number rows between unbounded preceding and current row) - frequency + 1 as range_start,
       sum(Frequency) over (order by Number rows between unbounded preceding and current row) as range_end
   from Numbers
),

median_position as
(
    select
        case
            when sum(frequency)%2 = 1 then ceil(sum(frequency)/2)
            else sum(frequency)/2 end as median_location
    from Numbers
    union
    select
        case
            when sum(frequency)%2 = 1 then ceil(sum(frequency)/2)
            else sum(frequency)/2+1 end as median_location
    from Numbers
)

select
    avg(Number) as median
from cte c
join median_position m
where m.median_location between range_start and range_end
```

### Consecutive Row Selection
X city built a new stadium, each day many people visit it and the stats are saved as these columns: id, visit_date, people

Please write a query to display the records which have 3 or more consecutive rows and the amount of people more than 100(inclusive).

```
For example, the table stadium:
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 1    | 2017-01-01 | 10        |
| 2    | 2017-01-02 | 109       |
| 3    | 2017-01-03 | 150       |
| 4    | 2017-01-04 | 99        |
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-08 | 188       |
+------+------------+-----------+
For the sample data above, the output is:

+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-08 | 188       |
+------+------------+-----------+
Note:
Each day only have one row record, and the dates are increasing with id increasing.
```
#### Solution

```sql
-- User (id - row_number) of generate label of consective row label
with cte as
(
    SELECT
        id,
        visit_date,
        people,
        id - ROW_NUMBER() OVER(ORDER BY id) AS rn
    FROM stadium
    WHERE people >= 100
)

select
    id,
    visit_date,
    people
from cte
where rn in
(
    select
        rn
    from cte
    group by rn
    having count(visit_date) >= 3
)
```

### Averge employee salary
Given two tables as below, write a query to display the comparison result (higher/lower/same) of the average salary of employees in a department to the company's average salary.

```
Table: salary
| id | employee_id | amount | pay_date   |
|----|-------------|--------|------------|
| 1  | 1           | 9000   | 2017-03-31 |
| 2  | 2           | 6000   | 2017-03-31 |
| 3  | 3           | 10000  | 2017-03-31 |
| 4  | 1           | 7000   | 2017-02-28 |
| 5  | 2           | 6000   | 2017-02-28 |
| 6  | 3           | 8000   | 2017-02-28 |


The employee_id column refers to the employee_id in the following table employee.


| employee_id | department_id |
|-------------|---------------|
| 1           | 1             |
| 2           | 2             |
| 3           | 2             |


So for the sample data above, the result is:


| pay_month | department_id | comparison  |
|-----------|---------------|-------------|
| 2017-03   | 1             | higher      |
| 2017-03   | 2             | lower       |
| 2017-02   | 1             | same        |
| 2017-02   | 2             | same        |

```

__Explanation__

In March, the company's average salary is (9000+6000+10000)/3 = 8333.33...


The average salary for department '1' is 9000, which is the salary of employee_id '1' since there is only one employee in this department. So the comparison result is 'higher' since 9000 > 8333.33 obviously.


The average salary of department '2' is (6000 + 10000)/2 = 8000, which is the average of employee_id '2' and '3'. So the comparison result is 'lower' since 8000 < 8333.33.


With he same formula for the average salary comparison in February, the result is 'same' since both the department '1' and '2' have the same average salary with the company, which is 7000.

#### Solution
* `CTE`
* `join`
* `case statement`

```sql
-- get company average
with company_avg as
(
        select
            left(pay_date, 7) as pay_month,
            avg(amount) as avg_salary
        from salary
        group by 1
        order by 1 desc
),

-- get department average
department_avg as
(
    select
        department_id,
        left(pay_date, 7) as pay_month,
        avg(amount) as avg_salary
    from
    (
        select
            e.department_id,
            s.pay_date,
            s.amount
        from salary s
        join employee e on s.employee_id = e.employee_id
    ) a
    group by 1, 2
)

-- Join with case statement
select
    da.pay_month,
    da.department_id,
    case when da.avg_salary > ca.avg_salary then 'higher'
         when da.avg_salary < ca.avg_salary then 'lower'
         else 'same' end as comparison
from department_avg da
join company_avg ca on da.pay_month = ca.pay_month
order by 1 desc, 2
```

### Students Report By Geography

A U.S graduate school has students from Asia, Europe and America. The students' location information are stored in table student as below.

```
| name   | continent |
|--------|-----------|
| Jack   | America   |
| Pascal | Europe    |
| Xi     | Asia      |
| Jane   | America   |


Pivot the continent column in this table so that each name is sorted alphabetically and displayed underneath its corresponding continent. The output headers should be America, Asia and Europe respectively. It is guaranteed that the student number from America is no less than either Asia or Europe.


For the sample input, the output is:


| America | Asia | Europe |
|---------|------|--------|
| Jack    | Xi   | Pascal |
| Jane    |      |        |
```

Follow-up: If it is unknown which continent has the most students, can you write a query to generate the student report?

#### Solution
* `case statement`
* Use `row_number()` and `group by`
* Alternative, we can create a table of each column and do outer join using row number

```sql
with cte as
(
    select
        case when continent = 'America' then name else NULL end as America,
        case when continent = 'Europe' then name else NULL end as Europe,
        case when continent = 'Asia' then name else NULL end as Asia,
        row_number() over (partition by continent order by name) as rn
    from student
)

select
    max(America) as America,
    max(Asia) as Asia,
    max(Europe) as Europe
from cte
group by rn
```

### Game play analysis
```
Table: Activity

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+
(player_id, event_date) is the primary key of this table.
This table shows the activity of players of some game.
Each row is a record of a player who logged in and played a number of games (possibly 0) before logging out on some day using some device.


We define the install date of a player to be the first login day of that player.

We also define day 1 retention of some date X to be the number of players whose install date is X and they logged back in on the day right after X, divided by the number of players whose install date is X, rounded to 2 decimal places.

Write an SQL query that reports for each install date, the number of players that installed the game on that day and the day 1 retention.

The query result format is in the following example:

Activity table:
+-----------+-----------+------------+--------------+
| player_id | device_id | event_date | games_played |
+-----------+-----------+------------+--------------+
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-01 | 0            |
| 3         | 4         | 2016-07-03 | 5            |
+-----------+-----------+------------+--------------+

Result table:
+------------+----------+----------------+
| install_dt | installs | Day1_retention |
+------------+----------+----------------+
| 2016-03-01 | 2        | 0.50           |
| 2017-06-25 | 1        | 0.00           |
+------------+----------+----------------+
Player 1 and 3 installed the game on 2016-03-01 but only player 1 logged back in on 2016-03-02 so the day 1 retention of 2016-03-01 is 1 / 2 = 0.50
Player 2 installed the game on 2017-06-25 but didn't log back in on 2017-06-26 so the day 1 retention of 2017-06-25 is 0 / 1 = 0.00
```

### solution
* `CTE`
* Window function
* The device id can be confusing. One can clarify with the interviewer

```
-- Create row number to identify install date (1st date)
-- calculate day gap use lead to find day 1 retention user
with retention as
(
    select
        player_id,
        device_id,
        event_date,
        row_number() over (partition by player_id order by event_date) as rn,
        lead(event_date) over (partition by player_id order by event_date) - event_date as days
    from activity
)

select
    event_date as install_dt,
    count(device_id) as installs,
    round(1.0 * count(case when days = 1 then player_id else NULL end)/count(device_id), 2) as Day1_retention
from retention
where rn = 1
group by 1
```

### Market Analysis

```
Table: Users

+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| user_id        | int     |
| join_date      | date    |
| favorite_brand | varchar |
+----------------+---------+
user_id is the primary key of this table.
This table has the info of the users of an online shopping website where users can sell and buy items.
Table: Orders

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| order_date    | date    |
| item_id       | int     |
| buyer_id      | int     |
| seller_id     | int     |
+---------------+---------+
order_id is the primary key of this table.
item_id is a foreign key to the Items table.
buyer_id and seller_id are foreign keys to the Users table.
Table: Items

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| item_id       | int     |
| item_brand    | varchar |
+---------------+---------+
item_id is the primary key of this table.
```


Write an SQL query to find for each user, whether the brand of the second item (by date) they sold is their favorite brand. If a user sold less than two items, report the answer for that user as no. It is guaranteed that no seller sold more than one item on a day.

The query result format is in the following example:

```
Users table:
+---------+------------+----------------+
| user_id | join_date  | favorite_brand |
+---------+------------+----------------+
| 1       | 2019-01-01 | Lenovo         |
| 2       | 2019-02-09 | Samsung        |
| 3       | 2019-01-19 | LG             |
| 4       | 2019-05-21 | HP             |
+---------+------------+----------------+

Orders table:
+----------+------------+---------+----------+-----------+
| order_id | order_date | item_id | buyer_id | seller_id |
+----------+------------+---------+----------+-----------+
| 1        | 2019-08-01 | 4       | 1        | 2         |
| 2        | 2019-08-02 | 2       | 1        | 3         |
| 3        | 2019-08-03 | 3       | 2        | 3         |
| 4        | 2019-08-04 | 1       | 4        | 2         |
| 5        | 2019-08-04 | 1       | 3        | 4         |
| 6        | 2019-08-05 | 2       | 2        | 4         |
+----------+------------+---------+----------+-----------+

Items table:
+---------+------------+
| item_id | item_brand |
+---------+------------+
| 1       | Samsung    |
| 2       | Lenovo     |
| 3       | LG         |
| 4       | HP         |
+---------+------------+

Result table:
+-----------+--------------------+
| seller_id | 2nd_item_fav_brand |
+-----------+--------------------+
| 1         | no                 |
| 2         | yes                |
| 3         | yes                |
| 4         | no                 |
+-----------+--------------------+

The answer for the user with id 1 is no because they sold nothing.
The answer for the users with id 2 and 3 is yes because the brands of their second sold items are their favorite brands.
The answer for the user with id 4 is no because the brand of their second sold item is not their favorite brand.
```

#### Solutions
```sql
-- Break seller and non-seller, use union for the final table
with seller as
(
    select
        o.seller_id,
        o.item_id,
        i.item_brand,
        u.favorite_brand,
        o.order_date,
        row_number() over (partition by o.seller_id order by o.order_date) as num_sales
    from Orders o
    join Users u on o.seller_id = u.user_id
    join Items i on o.item_id = i.item_id
),

non_seller as
(
    select
        user_id as seller_id,
        'no' as 2nd_item_fav_brand
    from Users
    where user_id not in (select seller_id from Orders)
)

select
    seller_id,
    case when 2nd_item_fav > 0.0 then 'yes' else 'no' end as 2nd_item_fav_brand
from
(
    select
        seller_id,
        sum(case when item_brand = favorite_brand and num_sales = 2 then 1.0 else 0.0 end) as 2nd_item_fav
    from seller
    group by seller_id
) a
union
select * from non_seller
order by seller_id

--Find the second day sale, left join to users table
WITH second_sold AS
(
    SELECT
      s.seller_id,
      s.item_id,
      s.order_date
    FROM
    (
        SELECT s.seller_id,
        s.item_id,
        RANK() OVER(PARTITION BY s.seller_id ORDER BY s.order_date ASC) AS rk,
        s.order_date
        FROM Orders s
    ) s
    WHERE s.rk = 2
)

SELECT
  u.user_id AS seller_id,
  CASE WHEN u.favorite_brand = i.item_brand THEN 'yes' ELSE 'no' END AS 2nd_item_fav_brand
FROM Users u
LEFT JOIN second_sold s
ON u.user_id = s.seller_id
LEFT JOIN Items i
ON s.item_id = i.item_id;

```

### Tournament Winner

```
Table: Players

+-------------+-------+
| Column Name | Type  |
+-------------+-------+
| player_id   | int   |
| group_id    | int   |
+-------------+-------+
player_id is the primary key of this table.
Each row of this table indicates the group of each player.
Table: Matches

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| match_id      | int     |
| first_player  | int     |
| second_player | int     |
| first_score   | int     |
| second_score  | int     |
+---------------+---------+
match_id is the primary key of this table.
Each row is a record of a match, first_player and second_player contain the player_id of each match.
first_score and second_score contain the number of points of the first_player and second_player respectively.
You may assume that, in each match, players belongs to the same group.
```


The winner in each group is the player who scored the maximum total points within the group. In the case of a tie, the lowest player_id wins. Write an SQL query to find the winner in each group.

The query result format is in the following example:

```
Players table:
+-----------+------------+
| player_id | group_id   |
+-----------+------------+
| 15        | 1          |
| 25        | 1          |
| 30        | 1          |
| 45        | 1          |
| 10        | 2          |
| 35        | 2          |
| 50        | 2          |
| 20        | 3          |
| 40        | 3          |
+-----------+------------+

Matches table:
+------------+--------------+---------------+-------------+--------------+
| match_id   | first_player | second_player | first_score | second_score |
+------------+--------------+---------------+-------------+--------------+
| 1          | 15           | 45            | 3           | 0            |
| 2          | 30           | 25            | 1           | 2            |
| 3          | 30           | 15            | 2           | 0            |
| 4          | 40           | 20            | 5           | 2            |
| 5          | 35           | 50            | 1           | 1            |
+------------+--------------+---------------+-------------+--------------+

Result table:
+-----------+------------+
| group_id  | player_id  |
+-----------+------------+
| 1         | 15         |
| 2         | 35         |
| 3         | 40         |
+-----------+------------+
```

#### Solution

```sql
-- Use union to calculate total score for each users, then rank within group
with scores as
(
    select
        player_id,
        sum(score) as score
    from
    (
        select
            first_player as player_id,
            first_score as score
        from Matches
        union all
        select
            second_player as player_id,
            second_score as score
        from Matches
     ) a
    group by 1
)

select
    group_id,
    player_id
from
(
    select
        p.group_id,
        p.player_id,
        s.score,
        rank() over (partition by group_id order by score desc, player_id) as rk    
    from scores s
    join Players p on s.player_id = p.player_id

) a
where rk = 1
```

### Report Contiguous Dates

```
Table: Failed

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| fail_date    | date    |
+--------------+---------+
Primary key for this table is fail_date.
Failed table contains the days of failed tasks.
Table: Succeeded

+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| success_date | date    |
+--------------+---------+
Primary key for this table is success_date.
Succeeded table contains the days of succeeded tasks.
```

A system is running one task every day. Every task is independent of the previous tasks. The tasks can fail or succeed. Write an SQL query to generate a report of period_state for each continuous interval of days in the period from 2019-01-01 to 2019-12-31. period_state is 'failed' if tasks in this interval failed or 'succeeded' if tasks in this interval succeeded. Interval of days are retrieved as start_date and end_date. Order result by start_date.

The query result format is in the following example:

```
Failed table:
+-------------------+
| fail_date         |
+-------------------+
| 2018-12-28        |
| 2018-12-29        |
| 2019-01-04        |
| 2019-01-05        |
+-------------------+

Succeeded table:
+-------------------+
| success_date      |
+-------------------+
| 2018-12-30        |
| 2018-12-31        |
| 2019-01-01        |
| 2019-01-02        |
| 2019-01-03        |
| 2019-01-06        |
+-------------------+


Result table:
+--------------+--------------+--------------+
| period_state | start_date   | end_date     |
+--------------+--------------+--------------+
| succeeded    | 2019-01-01   | 2019-01-03   |
| failed       | 2019-01-04   | 2019-01-05   |
| succeeded    | 2019-01-06   | 2019-01-06   |
+--------------+--------------+--------------+

The report ignored the system state in 2018 as we care about the system in the period 2019-01-01 to 2019-12-31.
From 2019-01-01 to 2019-01-03 all tasks succeeded and the system state was "succeeded".
From 2019-01-04 to 2019-01-05 all tasks failed and system state was "failed".
From 2019-01-06 to 2019-01-06 all tasks succeeded and system state was "succeeded".
```

#### Solutions

```sql
-- Use rolling sum to create period number, aggregate by period number, union
with success_periods as
(
   select
       success_date,
       sum(start_of_period_flag) over (order by success_date rows unbounded preceding) as period_num
   from
   (
       select
           success_date,
           case when day_gap > 1 then 1 else 0 end as start_of_period_flag
       from
       (
           select
               success_date,
               datediff(success_date, lag(success_date, 1) over (order by success_date)) as day_gap
           from succeeded
           where success_date between '2019-01-01' and '2019-12-31'
       ) a
   ) aa
),

fail_periods as
(
   select
       fail_date,
       sum(start_of_period_flag) over (order by fail_date rows unbounded preceding) as period_num
   from
   (
       select
           fail_date,
           case when day_gap > 1 then 1 else 0 end as start_of_period_flag
       from
       (
           select
               fail_date,
               datediff(fail_date, lag(fail_date, 1) over (order by fail_date)) as day_gap
           from failed
           where fail_date between '2019-01-01' and '2019-12-31'
       ) a
   ) aa
)

select
    'succeeded' as period_state,
    min(success_date) as start_date,
    max(success_date) as end_date
from success_periods
group by period_num
union
select
    'failed' as period_state,
    min(fail_date) as start_date,
    max(fail_date) as end_date
from fail_periods
group by period_num
order by start_date

-- Use rom_number() to find group
-- if the date is consective, the date group be the same
-- if the date is jumps, the date group be change

(
  SELECT
    'succeeded' AS period_state,
    MIN(s.success_date) AS start_date,
    MAX(s.success_date) AS end_date
  FROM
  (   
    SELECT
      s.success_date,
      DATEDIFF(s.success_date, '2000-01-01') - ROW_NUMBER() OVER(ORDER BY s.success_date) AS date_groups
    FROM Succeeded s
    WHERE s.success_date BETWEEN '2019-01-01' AND '2019-12-31'
  ) s
  GROUP BY s.date_groups
)
UNION
(
  SELECT
    'failed' AS period_state,
    MIN(f.fail_date) AS start_date,
    MAX(f.fail_date) AS end_date
  FROM
  (   
    SELECT
      f.fail_date,
      DATEDIFF(f.fail_date, '1970-01-01') - ROW_NUMBER() OVER(ORDER BY f.fail_date) AS date_groups
    FROM Failed f
    WHERE f.fail_date BETWEEN '2019-01-01' AND '2019-12-31'
  ) f
  GROUP BY f.date_groups
)
ORDER BY start_date;
```

```
Table: Visits

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| visit_date    | date    |
+---------------+---------+
(user_id, visit_date) is the primary key for this table.
Each row of this table indicates that user_id has visited the bank in visit_date.


Table: Transactions

+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| user_id          | int     |
| transaction_date | date    |
| amount           | int     |
+------------------+---------+
There is no primary key for this table, it may contain duplicates.
Each row of this table indicates that user_id has done a transaction of amount in transaction_date.
It is guaranteed that the user has visited the bank in the transaction_date.(i.e The Visits table contains (user_id, transaction_date) in one row)


A bank wants to draw a chart of the number of transactions bank visitors did in one visit to the bank and the corresponding number of visitors who have done this number of transaction in one visit.
```

Write an SQL query to find how many users visited the bank and didn't do any transactions, how many visited the bank and did one transaction and so on.

The result table will contain two columns:

* transactions_count which is the number of transactions done in one visit.
* visits_count which is the corresponding number of users who did transactions_count in one visit to the bank.
* transactions_count should take all values from 0 to max(transactions_count) done by one or more users.

Order the result table by transactions_count.

The query result format is in the following example:

```
Visits table:
+---------+------------+
| user_id | visit_date |
+---------+------------+
| 1       | 2020-01-01 |
| 2       | 2020-01-02 |
| 12      | 2020-01-01 |
| 19      | 2020-01-03 |
| 1       | 2020-01-02 |
| 2       | 2020-01-03 |
| 1       | 2020-01-04 |
| 7       | 2020-01-11 |
| 9       | 2020-01-25 |
| 8       | 2020-01-28 |
+---------+------------+
Transactions table:
+---------+------------------+--------+
| user_id | transaction_date | amount |
+---------+------------------+--------+
| 1       | 2020-01-02       | 120    |
| 2       | 2020-01-03       | 22     |
| 7       | 2020-01-11       | 232    |
| 1       | 2020-01-04       | 7      |
| 9       | 2020-01-25       | 33     |
| 9       | 2020-01-25       | 66     |
| 8       | 2020-01-28       | 1      |
| 9       | 2020-01-25       | 99     |
+---------+------------------+--------+
Result table:
+--------------------+--------------+
| transactions_count | visits_count |
+--------------------+--------------+
| 0                  | 4            |
| 1                  | 5            |
| 2                  | 0            |
| 3                  | 1            |
+--------------------+--------------+
* For transactions_count = 0, The visits (1, "2020-01-01"), (2, "2020-01-02"), (12, "2020-01-01") and (19, "2020-01-03") did no transactions so visits_count = 4.
* For transactions_count = 1, The visits (2, "2020-01-03"), (7, "2020-01-11"), (8, "2020-01-28"), (1, "2020-01-02") and (1, "2020-01-04") did one transaction so visits_count = 5.
* For transactions_count = 2, No customers visited the bank and did two transactions so visits_count = 0.
* For transactions_count = 3, The visit (9, "2020-01-25") did three transactions so visits_count = 1.
* For transactions_count >= 4, No customers visited the bank and did more than three transactions so we will stop at transactions_count = 3
```

#### Solution
The tricky part of the problem is to generate all the transaction_count

```sql
with cte as
(
    select
        v.user_id,
        v.visit_date,
        count(t.transaction_date) as transactions_count
    from Visits v
    left join Transactions t on v.user_id = t.user_id and v.visit_date = t.transaction_date
    group by 1, 2
),

transaction_num as
(
    select
        num as transactions_count
    from
    (
        select
            row_number() over (order by transaction_date) as num
        from Transactions
        union
        select 0
    ) r
    where num <= (select max(transactions_count) from cte)
)

select
    t.transactions_count,
    coalesce(a.visits_count, 0) as visits_count
from
(
    select
       transactions_count,
       count(user_id) as visits_count
    from cte c
    group by transactions_count
) a
right join transaction_num t on a.transactions_count = t.transactions_count
order by 1
```

### The second most recent activity

```
Table: UserActivity

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| username      | varchar |
| activity      | varchar |
| startDate     | Date    |
| endDate       | Date    |
+---------------+---------+
This table does not contain primary key.
This table contain information about the activity performed of each user in a period of time.
A person with username performed a activity from startDate to endDate.
```

Write an SQL query to show the second most recent activity of each user. If the user only has one activity, return that one. A user can't perform more than one activity at the same time. Return the result table in any order.

The query result format is in the following example:

```
UserActivity table:
+------------+--------------+-------------+-------------+
| username   | activity     | startDate   | endDate     |
+------------+--------------+-------------+-------------+
| Alice      | Travel       | 2020-02-12  | 2020-02-20  |
| Alice      | Dancing      | 2020-02-21  | 2020-02-23  |
| Alice      | Travel       | 2020-02-24  | 2020-02-28  |
| Bob        | Travel       | 2020-02-11  | 2020-02-18  |
+------------+--------------+-------------+-------------+

Result table:
+------------+--------------+-------------+-------------+
| username   | activity     | startDate   | endDate     |
+------------+--------------+-------------+-------------+
| Alice      | Dancing      | 2020-02-21  | 2020-02-23  |
| Bob        | Travel       | 2020-02-11  | 2020-02-18  |
+------------+--------------+-------------+-------------+

The most recent activity of Alice is Travel from 2020-02-24 to 2020-02-28, before that she was dancing from 2020-02-21 to 2020-02-23.
Bob only has one record, we just take that one.
```

#### Solution

```sql
--solution 1
with cte as
(
    select
        username,
        activity,
        startDate,
        endDate,
        row_number() over (partition by username order by startDate desc) as rn1,
        row_number() over (partition by username order by startDate) as rn2
    from UserActivity
)

select
    username,
    activity,
    startDate,
    endDate
from cte
where rn1 = 2 or (rn1 = 1 and rn2 = 1)

--Solution 2
with cte as
(
    select
        username,
        activity,
        startDate,
        endDate,
        row_number() over (partition by username order by startDate desc) as rn
    from UserActivity
)

select
    username,
    activity,
    startDate,
    endDate
from cte
where rn = 2
union
select
    username,
    activity,
    startDate,
    endDate
from cte
where username in
(
   select
        username
    from cte
    group by 1
    having max(rn) = 1
)
```

### Sales by Day of the Week

```
Table: Product

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| product_name  | varchar |
+---------------+---------+
product_id is the primary key for this table.
product_name is the name of the product.


Table: Sales

+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| product_id          | int     |
| period_start        | varchar |
| period_end          | date    |
| average_daily_sales | int     |
+---------------------+---------+
product_id is the primary key for this table.
period_start and period_end indicates the start and end date for sales period, both dates are inclusive.
The average_daily_sales column holds the average daily sales amount of the items for the period.
```

Write an SQL query to report the Total sales amount of each item for each year, with corresponding product name, product_id, product_name and report_year. Dates of the sales years are between 2018 to 2020. Return the result table ordered by product_id and report_year.

The query result format is in the following example:

```
Product table:
+------------+--------------+
| product_id | product_name |
+------------+--------------+
| 1          | LC Phone     |
| 2          | LC T-Shirt   |
| 3          | LC Keychain  |
+------------+--------------+

Sales table:
+------------+--------------+-------------+---------------------+
| product_id | period_start | period_end  | average_daily_sales |
+------------+--------------+-------------+---------------------+
| 1          | 2019-01-25   | 2019-02-28  | 100                 |
| 2          | 2018-12-01   | 2020-01-01  | 10                  |
| 3          | 2019-12-01   | 2020-01-31  | 1                   |
+------------+--------------+-------------+---------------------+

Result table:
+------------+--------------+-------------+--------------+
| product_id | product_name | report_year | total_amount |
+------------+--------------+-------------+--------------+
| 1          | LC Phone     |    2019     | 3500         |
| 2          | LC T-Shirt   |    2018     | 310          |
| 2          | LC T-Shirt   |    2019     | 3650         |
| 2          | LC T-Shirt   |    2020     | 10           |
| 3          | LC Keychain  |    2019     | 31           |
| 3          | LC Keychain  |    2020     | 31           |
+------------+--------------+-------------+--------------+
LC Phone was sold for the period of 2019-01-25 to 2019-02-28, and there are 35 days for this period. Total amount 35*100 = 3500.
LC T-shirt was sold for the period of 2018-12-01 to 2020-01-01, and there are 31, 365, 1 days for years 2018, 2019 and 2020 respectively.
LC Keychain was sold for the period of 2019-12-01 to 2020-01-31, and there are 31, 31 days for years 2019 and 2020 respectively.
```

#### Solutions

```sql
with cte as
(
    select
        i.item_category,
        Dayofweek(order_date) as day_of_week,
        quantity
    from Orders o
    right join Items i on o.item_id = i.item_id
)

select
    item_category as category,
    sum(case when day_of_week = 2 then quantity else 0 end) as Monday,
    sum(case when day_of_week = 3 then quantity else 0 end) as Tuesday,
    sum(case when day_of_week = 4 then quantity else 0 end) as Wednesday,
    sum(case when day_of_week = 5 then quantity else 0 end) as Thursday,
    sum(case when day_of_week = 6 then quantity else 0 end) as Friday,
    sum(case when day_of_week = 7 then quantity else 0 end) as Saturday,
    sum(case when day_of_week = 1 then quantity else 0 end) as Sunday
from cte
group by 1
order by 1
```

### Find the Quiet Students in All Exams

```
Table: Student

+---------------------+---------+
| Column Name         | Type    |
+---------------------+---------+
| student_id          | int     |
| student_name        | varchar |
+---------------------+---------+
student_id is the primary key for this table.
student_name is the name of the student.


Table: Exam

+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| exam_id       | int     |
| student_id    | int     |
| score         | int     |
+---------------+---------+
(exam_id, student_id) is the primary key for this table.
Student with student_id got score points in exam with id exam_id.

A "quite" student is the one who took at least one exam and didn't score neither the high score nor the low score.
```

Write an SQL query to report the students (student_id, student_name) being "quiet" in ALL exams. Don't return the student who has never taken any exam. Return the result table ordered by student_id.

The query result format is in the following example.

```
Student table:
+-------------+---------------+
| student_id  | student_name  |
+-------------+---------------+
| 1           | Daniel        |
| 2           | Jade          |
| 3           | Stella        |
| 4           | Jonathan      |
| 5           | Will          |
+-------------+---------------+

Exam table:
+------------+--------------+-----------+
| exam_id    | student_id   | score     |
+------------+--------------+-----------+
| 10         |     1        |    70     |
| 10         |     2        |    80     |
| 10         |     3        |    90     |
| 20         |     1        |    80     |
| 30         |     1        |    70     |
| 30         |     3        |    80     |
| 30         |     4        |    90     |
| 40         |     1        |    60     |
| 40         |     2        |    70     |
| 40         |     4        |    80     |
+------------+--------------+-----------+

Result table:
+-------------+---------------+
| student_id  | student_name  |
+-------------+---------------+
| 2           | Jade          |
+-------------+---------------+

For exam 1: Student 1 and 3 hold the lowest and high score respectively.
For exam 2: Student 1 hold both highest and lowest score.
For exam 3 and 4: Studnet 1 and 4 hold the lowest and high score respectively.
Student 2 and 5 have never got the highest or lowest in any of the exam.
Since student 5 is not taking any exam, he is excluded from the result.
So, we only return the information of Student 2.
```

#### Solutions

```sql
with non_quiet as
(
    select
        distinct
        student_id
    from
    (
        select
            exam_id,
            student_id,
            score,
            max(score) over (partition by exam_id) as max_score,
            min(score) over (partition by exam_id) as min_score
        from Exam
    ) a
    where score = max_score or score = min_score
)

select
    distinct
    e.student_id,
    s.student_name
from Exam e
left join Student s on e.student_id = s.student_id
where e.student_id not in
(
    select * from non_quiet
)
order by 1
```
