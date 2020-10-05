# Session Data Analysis Recipe

### Construct sessions from raw event log data without log-in / log-out events labels
* raw event log contain colums: `user_id`, `event_time`, `event_name` but there is not event name indicate user log-in and log-out
* Generate session end flag based on time gap
* Apply window function to create session_number and session_step

```sql
--If use idle for more than 10 mins, session ends
with session_label as
(
      select
          *,
          sum(end_of_session_flag) over (partition by user_id order by event_time rows unbounded preceding) as session_num
      from
      (
          select
              user_id,
              event_time,
              session_step_id,
              event_time - lag(event_time) over (partition by user_id order by event_time) as time_laps,
              case when time_laps > 10 then 1.0 else 0.0 end as end_of_session_flag
          from raw_sessions
      )
)

select
      user_id,
      event_time,
      time_laps,
      end_of_session_flag,
      session_num,
      row_number() over (partition by user_id, session_num order by event_time) as session_step
from session_label
```

### Cohort Engagement Analysis (Using Yammer data from `mode.com`)
* Measure user age meta data based on events
* Cohort user based on user age as active users

```sql
with user_meta as
(
    SELECT
        e.occurred_at,
        u.user_id,
        DATE_TRUNC('week',u.activated_at) AS activation_week,
        DATE_DIFF(day, u.activated_at, e.occurred_at) AS age_at_event,
        DATE_DIFF(day, u.activated_at, '2014-09-01'::TIMESTAMP) AS user_age
      FROM tutorial.yammer_users u
      JOIN tutorial.yammer_events e
        ON e.user_id = u.user_id
       AND e.event_type = 'engagement'
       AND e.event_name = 'login'
       AND e.occurred_at >= '2014-05-01'
       AND e.occurred_at < '2014-09-01'
     WHERE u.activated_at IS NOT NULL
),

user_cohort_bins as
(
    SELECT
        DATE_TRUNC('week',z.occurred_at) AS "week",
        AVG(z.age_at_event) AS "Average age during week",
        COUNT(DISTINCT CASE WHEN z.user_age > 70 THEN z.user_id ELSE NULL END) AS "10+ weeks",
        COUNT(DISTINCT CASE WHEN z.user_age < 70 AND z.user_age >= 63 THEN z.user_id ELSE NULL END) AS "9 weeks",
        COUNT(DISTINCT CASE WHEN z.user_age < 63 AND z.user_age >= 56 THEN z.user_id ELSE NULL END) AS "8 weeks",
        COUNT(DISTINCT CASE WHEN z.user_age < 56 AND z.user_age >= 49 THEN z.user_id ELSE NULL END) AS "7 weeks",
        COUNT(DISTINCT CASE WHEN z.user_age < 49 AND z.user_age >= 42 THEN z.user_id ELSE NULL END) AS "6 weeks",
        COUNT(DISTINCT CASE WHEN z.user_age < 42 AND z.user_age >= 35 THEN z.user_id ELSE NULL END) AS "5 weeks",
        COUNT(DISTINCT CASE WHEN z.user_age < 35 AND z.user_age >= 28 THEN z.user_id ELSE NULL END) AS "4 weeks",
        COUNT(DISTINCT CASE WHEN z.user_age < 28 AND z.user_age >= 21 THEN z.user_id ELSE NULL END) AS "3 weeks",
        COUNT(DISTINCT CASE WHEN z.user_age < 21 AND z.user_age >= 14 THEN z.user_id ELSE NULL END) AS "2 weeks",
        COUNT(DISTINCT CASE WHEN z.user_age < 14 AND z.user_age >= 7 THEN z.user_id ELSE NULL END) AS "1 week",
        COUNT(DISTINCT CASE WHEN z.user_age < 7 THEN z.user_id ELSE NULL END) AS "Less than a week"
    FROM user_meta z
    GROUP BY 1
    ORDER BY 1
)
```

### Email enagement analysis
* Collect weekly email engagement event counts using `CASE` statement and self-join
* check whether there is an open events with 5 mins of send, a ctr event within 5 mins of open
* We can also use window function `lead()`

```sql
with email_engagement_weekly as
(
    SELECT
        DATE_TRUNC('week',e1.occurred_at) AS week,
        COUNT(CASE WHEN e1.action = 'sent_weekly_digest' THEN e1.user_id ELSE NULL END) AS weekly_emails,
        COUNT(CASE WHEN e1.action = 'sent_weekly_digest' THEN e2.user_id ELSE NULL END) AS weekly_opens,
        COUNT(CASE WHEN e1.action = 'sent_weekly_digest' THEN e3.user_id ELSE NULL END) AS weekly_ctr,
        COUNT(CASE WHEN e1.action = 'sent_reengagement_email' THEN e1.user_id ELSE NULL END) AS retain_emails,
        COUNT(CASE WHEN e1.action = 'sent_reengagement_email' THEN e2.user_id ELSE NULL END) AS retain_opens,
        COUNT(CASE WHEN e1.action = 'sent_reengagement_email' THEN e3.user_id ELSE NULL END) AS retain_ctr
      FROM tutorial.yammer_emails e1
      LEFT JOIN tutorial.yammer_emails e2
        ON e2.occurred_at >= e1.occurred_at
       AND e2.occurred_at < e1.occurred_at + INTERVAL '5 MINUTE'
       AND e2.user_id = e1.user_id
       AND e2.action = 'email_open'
      LEFT JOIN tutorial.yammer_emails e3
        ON e3.occurred_at >= e2.occurred_at
       AND e3.occurred_at < e2.occurred_at + INTERVAL '5 MINUTE'
       AND e3.user_id = e2.user_id
       AND e3.action = 'email_clickthrough'
     WHERE e1.occurred_at >= '2014-06-01'
       AND e1.occurred_at < '2014-09-01'
       AND e1.action IN ('sent_weekly_digest','sent_reengagement_email')
     GROUP BY 1
)

SELECT
    week,
    weekly_opens/CASE WHEN weekly_emails = 0 THEN 1 ELSE weekly_emails END::FLOAT AS weekly_open_rate,
    weekly_ctr/CASE WHEN weekly_opens = 0 THEN 1 ELSE weekly_opens END::FLOAT AS weekly_ctr,
    retain_opens/CASE WHEN retain_emails = 0 THEN 1 ELSE retain_emails END::FLOAT AS retain_open_rate,
    retain_ctr/CASE WHEN retain_opens = 0 THEN 1 ELSE retain_opens END::FLOAT AS retain_ctr
FROM email_engagement_weekly a
ORDER BY 1
```

### Search Analysis

```sql
SELECT user_id,
        session,
        MIN(occurred_at) AS session_start,
        MAX(occurred_at) AS session_end
   FROM (
        SELECT bounds.*,
        		    CASE WHEN last_event >= INTERVAL '10 MINUTE' THEN id
        		         WHEN last_event IS NULL THEN id
        		         ELSE LAG(id,1) OVER (PARTITION BY user_id ORDER BY occurred_at) END AS session
          FROM (
               SELECT user_id,
                      event_type,
                      event_name,
                      occurred_at,
                      occurred_at - LAG(occurred_at,1) OVER (PARTITION BY user_id ORDER BY occurred_at) AS last_event,
                      LEAD(occurred_at,1) OVER (PARTITION BY user_id ORDER BY occurred_at) - occurred_at AS next_event,
                      ROW_NUMBER() OVER () AS id
                 FROM tutorial.yammer_events e
                WHERE e.event_type = 'engagement'
                ORDER BY user_id,occurred_at
               ) bounds
         WHERE last_event >= INTERVAL '10 MINUTE'
            OR next_event >= INTERVAL '10 MINUTE'
         	 OR last_event IS NULL
        	 	 OR next_event IS NULL
        ) final
GROUP BY 1,2
```
