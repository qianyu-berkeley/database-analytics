# Recipe for data transformation

### Create Pivot Table

#### `CASE` Statement: Create pivot table for numeric values

- User a subquery to collect data in long formation, use aggregate functions and case stategment to select desired column values as a wide column
  ```sql
  SELECT conference,
         SUM(players) as total_players
         SUM(CASE WHEN year = 'FR' THEN players ELSE NULL END) AS fr,
         SUM(CASE WHEN year = 'SO' THEN players ELSE NULL END) AS so,
         SUM(CASE WHEN year = 'JR' THEN players ELSE NULL END) AS jr,
         SUM(CASE WHEN year = 'SR' THEN players ELSE NULL END) AS sr
    FROM (
          SELECT teams.conference AS conference,
                 players.year,
                 COUNT(1) AS players
            FROM benn.college_football_players players
            JOIN benn.college_football_teams teams
              ON teams.school_name = players.school_name
           GROUP BY 1,2
         ) sub
   GROUP BY 1
   ORDER BY 2 DESC
  ```

#### Full outer join
- Create multiple tables with pivot columns then outer join them
```sql
  with doctors as
  (
    select
      name,
      row_number() over (order by name) as num
    from occupations
    where occupation = 'Doctor'
  ),

  professors as
  (
    select
      name,
      row_number() over (order by name) as num
    from occupations
    where occupation = 'Professor'
  ),

  singers as
  (
    select
      name,
      row_number() over (order by name) as num
    from occupations
    where occupation = 'Singer'
  ),

  actors as
  (
    select
      name,
      row_number() over (order by name) as num
    from occupations
    where occupation = 'Actor'
  )

  select
    doctors.name as doctor,
    professors.name as professor,
    singers.name as signer,
    actors.name as actor
  from doctors
  full outer join professors on professors.num = doctors.num
  full outer join singers on singers.num = professors.num
  full outer join actors on actors.num = singers.num
  ```
