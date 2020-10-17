# SQL Notes

### Query Clause Order

  ```sql
  SELECT
  FROM
  WHERE
  WINDOW_Function
  GROUP BY
  HAVING
  ORDER BY
  ```

### Column alias
- This is database technology dependent, in general
  - **Redshift** support lateral column alias
  - **MySQL** support column alias in `GROUP BY`, `ORDER BY`, or `HAVING` clauses, not in `WHERE` clause

### `Having` vs `WHERE`
- Where clause cannot be used with `group by`
- User `having` to filter aggregated columns
- `having` used after `group by` whereas `where` clause use before `group by` to filter

  ```sql
  select
    employee,
    sum(bonus)
  from emp_bonus
  group by employee
  having sum(bonus) > 1000;
  ```


### `Null` value
* By default `NULL` is larger than non-null values so if we order by `asc`, `null` will appear last, if `desc` null will apprear first
* We can use `NULLs First` or `NULLs Last` to set the orders


### round, ceiling, trunc or floor decimal
* round: rounding to
* ceiling: smallest integer > the number
* truncate: largest integer < the decimal

```sql
  SELECT
    round(rev, 0),
    ceiling(rev, 0)
    truncate(rev, 0)
  from employee
```

### `Exist` vs. `In`
- `EXISTS` conditions test for the existence of rows in a subquery, and return true if a subquery returns at least one row.
- `In` condition check within a list (if use sub-query, the sub-query needs to product a list)
  ```sql
  --EXISTS
  SELECT
    EnglishProductName 'Product'
  FROM DimProduct p
  WHERE EXISTS
  (
      SELECT * -- no data is returned, only a boolean true/false
      FROM DimProductSubcategory sc
      WHERE	p.ProductSubcategoryKey = sc.ProductSubcategoryKey
  	   AND	sc.EnglishProductSubcategoryName = 'Wheels'
  )

  SELECT fname, lname
  FROM Customers
  WHERE EXISTS
  (
      SELECT *
      FROM Orders
      WHERE Customers.customer_id = Orders.c_id
  );

  --IN
  SELECT
    EnglishProductName 'Product'
  FROM DimProduct p
  WHERE p.ProductSubcategoryKey IN
  (
      SELECT sc.ProductSubcategoryKey
      FROM DimProductSubcategory sc
      WHERE sc.EnglishProductSubcategoryName = 'Wheels'
  )

  select name, grade
  from highschooler
  where ID not in (select ID2 from likes) and
        ID not in (select ID1 from likes)
  order by grade, name
  ```


### Date / Timestamp Operations

- `between` `and` `>` `<` `<=` `<=` on dates

  ```sql
  SELECT *
  FROM FactInternetSales s
  INNER JOIN DimProduct p ON s.ProductKey = p.ProductKey
  WHERE	s.OrderDate >= '2013-01-01'
  AND	s.OrderDate <= '2013-12-31'

  --is the same
  --between ... an
  SELECT *
  FROM FactInternetSales s
  INNER JOIN DimProduct p ON s.ProductKey = p.ProductKey
  WHERE s.OrderDate BETWEEN '2013-01-01' AND '2013-12-31';
  ```

- `interval`

  ```sql
  SELECT
    companies.permalink,
    companies.founded_at_clean,
    companies.founded_at_clean::timestamp + INTERVAL '1 week' AS plus_one_week
  FROM tutorial.crunchbase_companies_clean_date companies
  WHERE founded_at_clean IS NOT NULL
  ```

- `now()`, `CURRENT_TIME`, `CURRENT_DATE`

  ```sql
  SELECT
    companies.permalink,
    companies.founded_at_clean,
    NOW() - companies.founded_at_clean::timestamp AS founded_time_ago
  FROM tutorial.crunchbase_companies_clean_date companies
  WHERE founded_at_clean IS NOT NULL

  SELECT
    CURRENT_DATE AS date,
    CURRENT_TIME AS time,
    CURRENT_TIMESTAMP AS timestamp,
    LOCALTIME AS localtime,
    LOCALTIMESTAMP AS localtimestamp,
    NOW() AS now
  ```

- `EXTRACT` only works on timestamp data type returns integer of date part
- `DATE_PART` function returns type in double

  ```sql
  SELECT cleaned_date,
         EXTRACT('year'   FROM cleaned_date) AS year,
         EXTRACT('month'  FROM cleaned_date) AS month,
         EXTRACT('day'    FROM cleaned_date) AS day,
         EXTRACT('hour'   FROM cleaned_date) AS hour,
         EXTRACT('minute' FROM cleaned_date) AS minute,
         EXTRACT('second' FROM cleaned_date) AS second,
         EXTRACT('decade' FROM cleaned_date) AS decade,
         EXTRACT('dow'    FROM cleaned_date) AS day_of_week
    FROM sf_crime_incidents_cleandate
  ```

- `DATE_TRUNC` round dates to the beginning of the period
  ```sql
  SELECT cleaned_date,
         DATE_TRUNC('year'   , cleaned_date) AS year,
         DATE_TRUNC('month'  , cleaned_date) AS month,
         DATE_TRUNC('week'   , cleaned_date) AS week,
         DATE_TRUNC('day'    , cleaned_date) AS day,
         DATE_TRUNC('hour'   , cleaned_date) AS hour,
         DATE_TRUNC('minute' , cleaned_date) AS minute,
         DATE_TRUNC('second' , cleaned_date) AS second,
         DATE_TRUNC('decade' , cleaned_date) AS decade
  FROM tutorial.sf_crime_incidents_cleandate
  ```

- `DATEDIFF` measure the time difference

    ```sql
    SELECT
      logs.created_at AS created_at,
      logs.user_id AS user_id,
      logs.event_id AS event_id,
      DATEDIFF(minute, LAG(logs.created_at) OVER
      (PARTITION BY logs.user_id, logs.event_id ORDER BY logs.created_at), logs.created_at) AS idle_time
    FROM events_log as logs
    ```

- `to_timestamp()` convert milli seconds to timestamp

### SELF JOIN

If a query can ben written using window function, use window function. Although Self-join is shorter and succinct, window function often has better performance as it divides and conque.

- Classic employee manager problem
  ```sql
  select
    a.Ename as employee,
    b.Ename as manager
  from employee_manager a
  join employee_manager b on a.manager_id = b.employee_id;
  ```

- An example of using self-join instead of use count and subquery
  ```sql
  -- without self join
  select
    a.director,
    b.title
  from
  (
    select
        director
    from movie
    group by director
    having count(*) > 1
  ) a
  join movie b
  on a.director = b.director
  order by a.director, b.title

  -- with self-join
  select
    a.director,
    b.title
  from movie a, movie b
  where a.director = b.director and a.title <> b.title
  order by a.director, b.title
  ```


### Window Functions

* Note: Window Functions outputs are not allowed in `WHERE` clause because it execute after the where clause, need to use subquery
* Functions: `AVG()`, `SUM()`,  `MAX()`, `MIN()`, `COUNT()`, `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE()`, etc
  * `AVG()`, `SUM()`,  `MAX()`, `MIN()`, `COUNT()`, `NITLE()`, `First_value()`, `Last_value()`: If order by clause is used, need to define frame clause

- Show each sales average for Group, Country, and Region all in one query
  ```sql
  SELECT DISTINCT
  	t.SalesTerritoryGroup,
    t.SalesTerritoryCountry,
    t.SalesTerritoryRegion,
    AVG(s.SalesAmount) OVER(PARTITION BY t.SalesTerritoryGroup ) as 'GroupAvgSales',
    AVG(s.SalesAmount) OVER(PARTITION BY t.SalesTerritoryCountry ) as 'CountryAvgSales',
    AVG(s.SalesAmount) OVER(PARTITION BY t.SalesTerritoryRegion ) as 'RegionAvgSales'
  FROM FactInternetSales s
  JOIN DimSalesTerritory t ON s.SalesTerritoryKey = t.SalesTerritoryKey
  WHERE YEAR(s.OrderDate) = 2013
  ORDER BY 1,2,3
  ```

 -`Rank()` rank the value within a partition window with holes. It will give the same rank for the same value and skip. e.g. if 2 values rank 4, then next rank is 6. `DENSE_RANK()` does not skip ranks (without holes). `ROW_NUMBER()` always give consecutive numbers.
 ```sql
 SELECT
    user_id,
    duration_seconds,
    event_time,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY event_time) AS row_num,
    RANK() OVER (PARTITION BY user_id ORDER BY event_time) AS rank_with_hole,
    DENSE_RANK() OVER (PARTITION BY user_id ORDER BY event_time) AS rank_without_hole
  FROM event_log
 WHERE event_time > '2019-01-01'
 ```

- Use `lag()`, `lead()` to calculate time difference
```sql
SELECT user_id,
       duration_seconds,
       duration_seconds - LAG(duration_seconds, 1) OVER
       (PARTITION BY start_terminal ORDER BY duration_seconds)
         AS difference
  FROM events_log
 WHERE event_time < '2019-01-01'
 ORDER BY user_id, duration_seconds
```

- PERCENTILE: `NTILE(#)` give percentile based on #. If the number of sample is much smaller than #, it will not work properly
```sql
SELECT duration_seconds,
       NTILE(100) over (order by duration_seconds) as percentile_dura
  FROM tutorial.dc_bikeshare_q1_2012
 WHERE start_time < '2012-01-08'
 order by 1 desc
```

- Frame Clause
  - For aggregate functions, the frame clause further refines the set of rows in a function's window when using ORDER BY. It enables you to include or exclude sets of rows within the ordered result. The frame clause consists of the ROWS keyword and associated specifiers.

  - The frame clause doesn't apply to `ranking` functions. Also, the frame clause isn't required when no ORDER BY clause is used in the OVER clause for an aggregate function. **If an ORDER BY clause is used for an aggregate function, an explicit frame clause is required.**

  - When no ORDER BY clause is specified, the implied frame is unbounded, equivalent to ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING.

### `COALESCE(exp1, exp2, ...)` return the first non-null expression and NVL() is an older function serves the same purpose
  ```sql
  SELECT
    user_id,
    (COALESCE(coupon_discount, 0)) + (COALESCE(gift_voucher_discount, 0)) AS combined_coupon_discount
  from orders
  ```

### CTE Example
- Use CTE to get employee hierarchy recursively

  ```sql
  WITH DirectReports (ManagerID, EmployeeID, Title, DeptID, Level)
  AS
  (
  -- Anchor member definition
      SELECT
        e.ParentEmployeeKey,
        e.EmployeeKey,
        e.Title,
        e.DepartmentName,
        0 AS Level
      FROM DimEmployee AS e
      WHERE e.ParentEmployeeKey IS NULL
      UNION ALL
  -- Recursive member definition
      SELECT
        e.ParentEmployeeKey,
        e.EmployeeKey,
        e.Title,
        e.DepartmentName,
        Level + 1
      FROM DimEmployee AS e
      INNER JOIN DirectReports AS d
      ON e.ParentEmployeeKey = d.EmployeeID
  )

  --Now you can show the hierachy levels
  SELECT
    ManagerID,
    EmployeeID,
    Title,
    Level
  FROM DirectReports

  --Or you can select a hierarchy or filter information
  SELECT
    ManagerID,
    EmployeeID,
    Title,
    DeptID,
    Level
  FROM DirectReports
  WHERE DeptID = 'Information Services' OR Level = 0
  ```
