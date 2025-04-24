# Football Database System
### Project Overview
This project implements a relational database system for tracking football matches, teams, players, managers, stadiums, and performance metrics using MySQL. It includes schema transformations, analytical queries, and CTEs to support sports insights.
### Data Source
- [Download Here](https://drive.google.com/drive/folders/181xOhG99aFgsjaosum9w_ShcfTMOLe3Z)

### Tools Used
- Microsoft Excel (for data cleaning and transformation).
- MySQL for Data Analysis.

### Table Structure Summary
- Teams: Stores team data and stadium affiliation.
- Players: Stores player biodata and link to teams.
- Managers: Stores details of team managers.
- Matches: Stores game-specific data.
- Stadiums: Stores stadium infrastructure.
- Goals: Stores goal-related stats.
- Cities: Stores geographic location info.
These tables are connected primarily through foreign keys such as TEAM_ID, STADIUM_ID, and MATCH_ID, etc.
### Data Cleaning and Preparation
- The data consists of seven (7) different tables.
- On each table the dataset was duplicated, the duplicated data was removed by deleting them.
- The datasets were loaded on MySQL for data analysis and ensured each column on each table were of their suitable datatypes.
- Creating and usage of Database
### Creating database
A database named football was created using one of the DDL (Data Definition Language) command ‘CREATE’

```
CREATE 
  DATABASE 
    football;

-- The database to work with was specified using the USE command

USE 
 football;

```
### Data Modelling
After ensuring my data’s accuracy and consistency, I began the process of creating a data model for my tables. The relationship between tables was formed using one to many relationship.

![EER diagram](https://github.com/user-attachments/assets/a5033e74-464e-49e7-b351-f7a75029b727)

### Data Type Conversions
- Converting manager table DOB (date of birth) to date: This SQL query updates the DOB column in the manager table by converting the date format from string to a standard date format and alters the managers table by changing the DOB column datatype to DATE.

```
UPDATE manager 
SET 
    DOB = STR_TO_DATE(DOB, '%m/%d/%Y')
WHERE
     STR_TO_DATE(DOB, '%m/%d/%Y') IS NOT NULL;



ALTER TABLE manager
MODIFY DOB DATE;
```

- Converting matches table date_time to datetime datatype: This SQL query updates the date_time column in the matches table by converting the datetime format from string to a standard datetime format and alters the matches table by changing the date_time column datatype to DATETIME.

```
UPDATE matches
SET 
    DATE_TIME = STR_TO_DATE(DATE_TIME, '%m/%d/%Y %H:%i:%s')
WHERE
     STR_TO_DATE(DATE_TIME, '%m/%d/%Y %H:%i:%s') IS NOT NULL;

     
ALTER TABLE matches
MODIFY DATE_TIME DATETIME;
```

- Converting players table DOB to date datatype: This SQL query updates the DOB column in the players table by converting the date format from string to a standard date format and alters the players table by changing the DOB column datatype to DATE.

```
UPDATE players
SET 
 DOB = STR_TO_DATE(DOB, '%m/%d/%Y')
WHERE
 STR_TO_DATE(DOB, '%m/%d/%Y') IS NOT NULL;
 
 
ALTER TABLE players
MODIFY DOB DATE;
```
### Analytical Queries & Use Cases:
- Retrieve all the players under a specific manager.

```
SELECT 
	p.PLAYER_ID,
    p.first_name,
    p.last_name,
    p.position,
    p.height,
    p.weight,
    p.foot,
    m.FIRST_NAME,
    m.LAST_NAME
FROM
    players p
        INNER JOIN
    manager m ON m.TEAM_ID = p.TEAM_ID;
```
### Explanation
**Retrieved Columns:**

The query fetches the following columns from the players table:

1. PLAYER_ID

2. first_name

3. last_name

4. position

5. height

6. weight

7. foot

**Join Condition:**

The query joins the players table with the manager table on the TEAM_ID column, ensuring that players are matched with their respective team managers.

![Q1](https://github.com/user-attachments/assets/86ea0ac5-75c7-4461-bc42-efad541d1f64)

- Fetch all the matches that have been played in a specific country.
```
SELECT 
    mt.*, 
    c.country_name  
FROM
    matches mt
        JOIN
    stadiums s ON mt.STADIUM_ID = s.ID
        JOIN
    cities c ON c.CITY_NAME = s.CITY
ORDER BY c.country_name;
```
### Explanation
**The query fetches:**

1. All columns (*) from the matches table (mt).

2. The country_name column from the cities table ©.

**Join Conditions:**

The query joins three tables:

1. matches (mt) with stadiums (s) on the STADIUM_ID column.

2. stadiums (s) with cities © on the CITY_NAME column.

![Q2](https://github.com/user-attachments/assets/6c03b043-790e-4474-87a5-3240574e12ff)


- Retrieve all the teams that have won more than 3 matches in their home stadium.
```
SELECT 
    t.team_name, 
    COUNT(mt.match_id) AS Total_won_matches, 
    t.HOME_STADIUM_ID, 
    mt.STADIUM_ID 
FROM
    matches mt
        JOIN
    teams t ON t.HOME_STADIUM_ID = mt.STADIUM_ID 

WHERE
    mt.HOME_TEAM_SCORE > mt.AWAY_TEAM_SCORE

GROUP BY t.TEAM_NAME , t.HOME_STADIUM_ID , mt.STADIUM_ID
HAVING Total_won_matches > 3;
```
### Explanation
**Retrieved Columns**

**The query fetches:**

1. team_name from the teams table (t).

2. Total_won_matches: the count of matches won by each team at their home stadium.

3. HOME_STADIUM_ID from the teams table (t).

4. STADIUM_ID from the matches table (mt).

**Join Condition**

The query joins the matches table (mt) with the teams table (t) on the condition that the HOME_STADIUM_ID of the team matches the STADIUM_ID of the match.

**Filter Conditions**

The query filters the results to include only matches where:

1. The home team scored more than the away team (mt.HOME_TEAM_SCORE > mt.AWAY_TEAM_SCORE).

**Grouping and Having**

**The query:**

1. Groups the results by team_name, HOME_STADIUM_ID, and STADIUM_ID.

2. Applies a having clause to include only groups with more than 3 won matches (Total_won_matches > 3).

![Q3](https://github.com/user-attachments/assets/e3799d1a-d23a-414e-8e56-fc4cd5f0ff2f)

- Retrieve all the teams with foreign managers.
```
WITH 
 foreign_managers as( 
      SELECT 
       id,
       first_name,
       last_name,
       nationality,
       team_id
      FROM
       manager
      )

SELECT 
    fm.*,
    t.team_name,
    t.country
FROM
    foreign_managers fm
        JOIN
    teams t ON t.id = fm.team_id
WHERE
    fm.nationality <> t.COUNTRY;
```
### Explanation
**Common Table Expression (CTE)**

**The query creates a temporary result table, foreign_managers, which:**

**Retrieves columns from the manager table:**

1. id
2. first_name

3. last_name

4. nationality

5. team_id

**Final SELECT Statement**

**The final SELECT statement:**

Retrieves all columns (*) from the foreign_managers CTE (fm).

**Retrieves additional columns from the teams table (t):**

1. team_name
2. country

Joins the foreign_managers CTE with the teams table on the team_id column.

**Filter Condition**

The query filters the results to include only managers who are not from the same country as the team they manage:

fm.nationality <> t.country

![Q4](https://github.com/user-attachments/assets/d0a58159-397f-4552-b50c-30bfea328242)

- Fetch all the matches that were played in stadiums with a seating capacity greater than 60,000.
```
WITH over_60000_capacity as (
         SELECT 
        s.id, s.name, s.capacity
         FROM
          stadiums s
        WHERE
        s.capacity > 60000
       )
SELECT 
    O_60k_C.*,
    m.MATCH_ID,
    m.SEASON,
    m.HOME_TEAM_ID,
    m.AWAY_TEAM_ID
FROM
    over_60000_capacity O_60k_C
        JOIN
    matches m ON O_60k_C.id = m.STADIUM_ID
ORDER BY O_60k_C.capacity;
```
### Explanation
**Common Table Expression (CTE)**
**The query creates a temporary result table, over_60000_capacity, which:**

**Retrieves the following columns from the stadiums table (s):**

1. id

2. name

3. capacity

Filters the results to include only stadiums with a capacity greater than 60,000.

**Final SELECT Statement**

**The final SELECT statement:**

- Retrieves the following columns:

1. All columns (*) from the over_60000_capacity CTE (O_60k_C), which includes:

- id

- name

- capacity

2. Additional columns from the matches table (m):

4. MATCH_ID

5. SEASON

6. HOME_TEAM_ID

7. AWAY_TEAM_ID

Joins the over_60000_capacity CTE with the matches table on the STADIUM_ID column.

**Ordering**

The query orders the results by the stadium capacity (column 3) in ascending order.

![Q5](https://github.com/user-attachments/assets/07ea2ee5-188c-4535-984b-a0cdae7eacff)

- All goals made without an assist in 2020 by players having height greater than 165cm.
```
WITH 2020_goals as ( 
     SELECT 
      g.goal_id,
      g.match_id,
      g.pid,
      g.goal_desc,
      g.ASSIST,
      mt.DATE_TIME
     FROM
      goals g
      JOIN
       matches mt ON g.MATCH_ID = mt.MATCH_ID
     WHERE
      YEAR(mt.DATE_TIME) = 2020
      AND g.ASSIST = ''
                    )
SELECT 
    p.player_id,
    p.first_name,
    p.last_name,
    p.height,
    2020_goals.goal_id,
    2020_goals.date_time,
    2020_goals.assist
FROM
    2020_goals
        INNER JOIN
    players p ON 2020_goals.pid = p.PLAYER_ID
WHERE
    p.HEIGHT > 165;
```
### Explanation
**Common Table Expression (CTE)**

**The query creates a temporary result table, 2020_goals, which:**

**Retrieves the following columns:**
1. goal_id

2. match_id

3. pid

4. goal_desc

5. ASSIST

6. DATE_TIME (from the matches table)

Joins the goals table (g) with the matches table (mt) on the MATCH_ID column.

**Filters the results to include only:**

Goals scored in the year 2020 (YEAR(mt.DATE_TIME) = 2020).

Goals without assists (g.ASSIST = ‘’).

**Final SELECT Statement**

**The final SELECT statement:**
**Retrieves the following columns:**

1. player_id

2. first_name

3. last_name

4. height

5. goal_id

6. date_time

7. assist

Joins the 2020_goals CTE with the players table (p) on the PLAYER_ID column.

Filters the results to include only players with a height greater than 165 (p.HEIGHT > 165)

![Q6](https://github.com/user-attachments/assets/5aa59fdd-2390-4d86-8495-5b600f408e8d)

- All the England teams with a winning percentage less than 40% in home matches. The below query yielded no result, which means there is no England team with a winning percentage below 40%.
```
SELECT 
    m.home_team_id,
    t.id,
    t.country,
    concat(round(((sum(CASE
        WHEN mt.home_team_score > mt.away_team_score THEN 1
    END) / COUNT(mt.MATCH_ID)) * 100),2),'%') AS win_percentage
FROM
    teams t
        LEFT JOIN
    matches mt ON t.id = m.HOME_TEAM_ID
WHERE
    t.country = 'england'
GROUP BY m.HOME_TEAM_ID , t.id
HAVING win_percentage < 40;
```
### Explanation
**Retrieved Columns**
**The query fetches:**

1. mt.HOME_TEAM_ID

2. t.id

3. t.country

4. win_percentage: the percentage of home matches won by each team.

**Calculation**
**The query calculates the winning percentage using:**

- SUM(CASE WHEN mt.HOME_TEAM_SCORE > mt.AWAY_TEAM_SCORE THEN 1 END): counts the number of home matches won.

- COUNT(mt.MATCH_ID): counts the total number of home matches.

- The winning percentage is calculated by dividing the number of wins by the total number of matches and multiplying by 100.

**Filtering**

**The query filters the results to include only:**

- Teams from England (t.country = ‘england’).

- Teams with a winning percentage less than 40% (win_percentage < 40).

**Grouping**

**The query groups the results by:**

- mt.HOME_TEAM_ID

- t.id

![Q7](https://github.com/user-attachments/assets/b0567f22-b61e-46ea-8c17-7a146530f6bc)

- All stadiums that have hosted more than 5 matches with host team having a win percentage less than 60%.
```
SELECT 
    mt.STADIUM_ID,
    s.name,
    COUNT(match_id) AS total_matches,
    concat(round((SUM(CASE
        WHEN mt.HOME_TEAM_SCORE > mt.AWAY_TEAM_SCORE THEN 1
        ELSE 0
    END) * 100.0 / COUNT(mt.MATCH_ID)),2),'%') AS winpercent
FROM
    matches mt
        LEFT JOIN
    stadiums s ON s.id = mt.STADIUM_ID
GROUP BY mt.STADIUM_ID , s.name
having winpercent < 60;
```
### Explanation
**Retrieved Columns**

**The query fetches:**

1. STADIUM_ID

2. name (from the stadiums table)

3. total_matches: the total number of matches played at each stadium.

4. winpercent: the percentage of matches won by the home team at each stadium.

**Calculation**

**The query calculates the win percentage using:**

- SUM(CASE WHEN mt.HOME_TEAM_SCORE > mt.AWAY_TEAM_SCORE THEN 1 ELSE 0 END): counts the number of matches won by the home team.

- COUNT(mt.MATCH_ID): counts the total number of matches played.

- The win percentage is calculated by dividing the number of wins by the total number of matches and multiplying by 100.

**Grouping and Filtering**

**The query:**

- Groups the results by STADIUM_ID and name.

- Filters the results to include only stadiums where the home team win percentage is less than 60% (winpercent < 60).
  
![Q8](https://github.com/user-attachments/assets/801d43bd-bb4b-41c4-9da3-07b98dcffc20)

- The season with the greatest number of right-footed goals.
```
SELECT 
    mt.SEASON, 
    g.goal_desc, 
    COUNT(g.goal_desc) AS total_goals
FROM
    goals g
        INNER JOIN
    matches mt ON mt.MATCH_ID = g.MATCH_ID
WHERE
    g.goal_desc = 'right-footed shot'
GROUP BY mt.SEASON , g.goal_desc
ORDER BY total_goals DESC
LIMIT 1;
```
### Explanation
**Retrieved Columns**
**The query fetches:**

1. SEASON

2. goal_desc (which is always ‘right-footed shot’)

3. total_goals: the number of right-footed shot goals in each season.

**Filtering and Joining**

**The query:**

- Joins the goals table with the matches table on the MATCH_ID column.

- Filters the results to include only goals scored with a ‘right-footed shot’ (g.goal_desc = ‘right-footed shot’).

**Grouping and Ordering**

**The query:**

- Groups the results by SEASON and goal_desc.

- Orders the results by total_goals in descending order (most goals first).

- Limits the result to the top season with the most right-footed shot goals (LIMIT 1).

![Q9](https://github.com/user-attachments/assets/a5f167fd-5e20-4e09-822e-560c35b1d64e)

- The most successful manager of the UCL (2016–2022) based on wins.
```
SELECT 
    m.first_name,
    m.last_name,
    COUNT(CASE
        WHEN
            (mt.HOME_TEAM_ID = m.TEAM_ID
                AND mt.HOME_TEAM_SCORE > mt.AWAY_TEAM_SCORE)
                OR (mt.AWAY_TEAM_ID = m.TEAM_ID
                AND mt.AWAY_TEAM_SCORE > mt.HOME_TEAM_SCORE)
        THEN
            mt.MATCH_ID
    END) AS won_matches
FROM
    matches mt
        INNER JOIN
    manager m ON m.TEAM_ID IN (mt.HOME_TEAM_ID , mt.AWAY_TEAM_ID)
WHERE
    mt.DATE_TIME BETWEEN '2016-01-01' AND '2022-12-31'
GROUP BY m.first_name , m.last_name
ORDER BY won_matches DESC
LIMIT 1;
```
### Explanation

1. It joins the matches table with the manager table based on the team ID.

2. It filters the matches to only include those played between January 1, 2016, and December 31, 2022.

3. It counts the number of wins for each manager’s team, considering both home and away matches.

4. It groups the results by manager and orders them by the number of wins in descending order.

5. It returns the manager with the most wins.

![Q10](https://github.com/user-attachments/assets/756b68e7-98e5-4393-82f0-cccfa511290f)

### Conclusion

The queries are designed to extract insightful performance metrics and patterns from UCL data, focusing on player attributes, managerial influence, match outcomes, stadium characteristics, and team success. These insights can help identify trends such as managerial impact on team performance, effectiveness of stadiums as home grounds, the role of physical attributes like height in scoring, and the influence of playing foot on goal statistics. Ultimately, the dataset enables the identification of the most successful manager during the 2016–2022 UCL period, as well as deeper evaluations of teams’ and players’ competitive effectiveness.















