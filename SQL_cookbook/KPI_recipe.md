# Recipe for common KPIs of online or ecommerce B2C businesses

## Business Health

### DAU, WAU, MAU

Data table is often a log with columns: `user_id`, `event_time`, `event_name`
optional where clause may be used depending on the event name column

```sql
-- total number
select
    event_time::date as day,
    count(distinct user_id) as DAU
from events_log
--where event_name = 'purchase' --or 'enagement' etc
group by 1
order by 1

-- WAU
select
  date_part('year', date(created_at)) as year,
  date_part('week', date(created_at)) as week,
  count(dstinct user_id)
from events_log
where event_name = 'purchase' --or 'engagement' etc
group by year, week;

-- MAU
select
  date_part('year', date(created_at)) as year,
  date_part('month', date(created_at)) as month,
  count(distinct user_id)
from events_log
where event_name = 'purchase' --or 'engagement' etc
group by year, month;
```

### DNU, WNU, MNU

Data table is often a log with columns: (`user_id`, `event_time`, `event_name`)
```sql
with user_starting_date as
(
    SELECT
        user_id,
        min(event_time::date) as start_date
    from events_log
    group by 1
)

select
    start_date,
    --date_part(w, start_date) as week
    --date_part(month, start_date) as month
    count(distinct user_id) as DNU
from user_starting_date
group by 1
order by 1
```

### Conversion rate
Data table is a time based long with columns: `user_id`, `event_time`, `event_name`
Event name defines the definiton of conversion

```sql
-- purchase conversion rate
select
    event_time::date as day,
    sum(case
        when event_name = 'purchase' then 1.0
        else 0.0 end)/sum(distinct user_id) as dau_conv_rate
from events_log
group by 1
order by 1

-- subscribtion rate
select
  sum(case when status = 'Subscribed' then 1.0 else 0.0 end) /
  count(distinct user_id) as converion_rate
from events_log

--subscription rate per month
select
  date_part(month, time) as month,
  sum(case when status = 'Subscribed' then 1.0 else 0.0 end) /
  count(distinct user_id) as converion_rate
from events_log
group by 1
```

### Statistics and Distributions

**Average time between trial and subscription**
* Data table is a time based long with columns: `user_id`, `event_time`, `event_name`
* event_name is either `trial` or `subscription`

```sql
--Use self join assume events has only trial and subscription
select
  avg(s.time - t.time) as average_duration
from table s, table t
where s.user_id = t.user_id
and s.status <> t.status
and s.time > t.time

--use window function
With trial_subscription as
(
    SELECT
        user_id,
        event_time,
        event_name,
        lead(event_name, 1) over (partition by user_id order by event_time) as next_event
    from events_log
)

SELECT
    avg(event_time - lag(event_time, 1) over (parition by user_id order by event_time)) as avg_time
from trial_subscription
where event = 'trial' and next_event = 'subscription'

--use join and sub-query
select
  avg(a.event_time - b.event_time) as average_duration
from
(
  select
    user_id
    event_time,
  from table
  where status = 'trial'
) a
join
(
  select
    user_id
    event_time,
  from table
  where status = 'subscribe'
) b on a.user_id = b.user_id
```

**Distribution of users engagement (in-session) time**
* Data table is a time based long with columns: `user_id`, `event_time`, `event_name`
* event_name is either `trial` or `subscription`

```sql

with session_length as
(
  select
    a.user_id,
    a.event_time,
    lead(a.event_time, 1) OVER (partition by a.user_id order by a.event_time) - a.event_time as session_length
  from
  (
    select
      user_id,
      event_name
      event_time,
    from events_log
    where (event_name = 'login' or event_name = 'logout')
  ) a
  where a.event_name = 'login'
),

select
  percentile,
  max(session_time) as session_length_percentile
from
(
  select
    session_time,
    ntile(100) over (order by session_length) as precentile
  from session_length
  order by 1 desc
)
group by 1
order by 1

--Alternatively and more accurately
select
  percentile_cont(0.10) within group (order by session_length) as pert_10,
  percentile_cont(0.20) within group (order by session_length) as pert_20,
  percentile_cont(0.30) within group (order by session_length) as pert_30,
  percentile_cont(0.40) within group (order by session_length) as pert_40,
  percentile_cont(0.50) within group (order by session_length) as pert_50,
  percentile_cont(0.60) within group (order by session_length) as pert_60,
  percentile_cont(0.70) within group (order by session_length) as pert_70,
  percentile_cont(0.80) within group (order by session_length) as pert_80,
  percentile_cont(0.90) within group (order by session_length) as pert_90
from session_length
```

### Finance Metrics

#### Moving average Sales (use 6 weeks example)
* `ROWS BETWEEN n PRECEDING AND CURRENT ROW`

```sql
SELECT
    weeknum,
    weeklysales,
    AVG(WeeklySales) OVER (ORDER BY WeekNum ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as AvgSales,
FROM
(
	SELECT
        DATE_PART(w, Order_Date) as weeknum
        SUM(SalesAmount) leeklysales,
	FROM orders s
	WHERE OrderDate between '2020-01-01' and '2020-03-31'
	GROUP BY 1
) AS s
GROUP BY 1, 2
ORDER BY 1 ASC
```

#### Running total and percentage of total sales
* Apply `SUM()` in window function

```sql
--running total
SELECT
    duration_seconds,
    SUM(duration_seconds) OVER (ORDER BY start_time) AS running_total
FROM logs

SELECT
    start_terminal,
    duration_seconds,
    SUM(duration_seconds) OVER (PARTITION BY start_terminal ORDER BY start_time) AS running_total
FROM log
WHERE start_time < '2019-01-01'

--percentage of total (of a window)
SELECT
    start_terminal,
    duration_seconds,
    SUM(duration_seconds) OVER (PARTITION BY start_terminal ORDER BY start_time) AS running_total,
    SUM(duration_seconds) OVER (PARTITION BY start_terminal) as start_terminal_sum
    (duration_seconds / (SUM(duration_seconds) OVER
     (PARTITION BY start_terminal))) * 100 as percent_total,
FROM logs
WHERE start_time < '2019-01-01'
```

#### Rolling total sales
* `ROWS UNBOUNDED PRECEDING`

```sql
SELECT
    salesyear,
    salesmonth,
    MonthlySales,
    SUM(MonthlySales) OVER (PARTITION BY salesyear ORDER BY salesMonth ROWS UNBOUNDED PRECEDING) as YTDSales,
FROM (
    SELECT
        date_part(year, order_date) as saleyear,
        date_part(month, order_date) as salemonth,
        SUM(SalesAmount) MonthlySales,
	FROM orders
	GROUP BY 1, 2
) AS s
GROUP BY 1, 2, 3
ORDER BY 1, 2 ASC
```
