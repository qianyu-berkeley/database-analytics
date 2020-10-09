
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

### Calculate Median without using building function
The Employee table holds all employees. The employee table has three columns: Employee Id, Company Name, and Salary.

```bash
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

```bash
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

### Consecutivee Row Selection
X city built a new stadium, each day many people visit it and the stats are saved as these columns: id, visit_date, people

Please write a query to display the records which have 3 or more consecutive rows and the amount of people more than 100(inclusive).

```bash
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

```bash
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

```bash
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
