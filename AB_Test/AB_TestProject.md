#### Understanding the GloBox Database

### Question 1: Can a user show up more than once in the activity table? Yes or no, and why?
~~~~sql
WITH usr_count AS (
  SELECT
    act.uid,
    COUNT(act.uid) AS usr_count
  FROM activity AS act
  GROUP BY 1
)

SELECT
	COUNT(usr_count.uid) AS num_usr_more_than_1_activity
FROM usr_count
WHERE usr_count.usr_count > 1;
~~~~

Output : 

| num_usr_more_than_1_activity |
| ---------------------------- |
| 139                          |


### Question 2: What are the start and end dates of the experiment?
~~~~sql
SELECT
	MIN(grp.join_dt) AS start_date,
  MAX(grp.join_dt) AS end_date
FROM groups AS grp;
~~~~

Output: 

| start_date | end_date   |
| ---------- | ---------- |
| 2023-01-25 | 2023-02-06 |


### Question 3: How many total users were in the experiment?
~~~~sql
SELECT
	COUNT(grp.uid) AS num_usr_in_experiment
FROM groups AS grp;
~~~~

Output:

| num_usr_in_experiment |
| --------------------- |
| 48943                 |


### Question 4: How many users were in the control and treatment groups?
~~~~sql
SELECT
	grp.group,
	COUNT(grp.uid) AS num_usr_in_experiment
FROM groups AS grp
GROUP BY 1;
~~~~

Output:

| group | num_usr_in_experiment |
| ----- | --------------------- |
| A     | 24343                 |
| B     | 24600                 |


### Question 5: What was the conversion rate of all users?
~~~~sql
WITH UserConversion AS (  
  SELECT
  	DISTINCT usr.id,
    CASE
      WHEN act.spent > 0 THEN 1
      ELSE 0
    END AS usr_conversion
  FROM users AS usr
  LEFT JOIN activity AS act
    ON act.uid = usr.id
  GROUP BY usr.id, act.spent
)

SELECT
  (SUM(uc.usr_conversion)::numeric/COUNT(uc.usr_conversion)::numeric)*100 AS usr_conversion_rate
FROM UserConversion AS uc;
~~~~

Output:

| usr_conversion_rate    |
| ---------------------- |
| 4.27844635596510226200 |


### Question 6: What is the user conversion rate for the control and treatment groups?
~~~~sql
WITH UserConversion AS (  
  SELECT
  	DISTINCT usr.id,
  	grp.group,
    CASE
      WHEN act.spent > 0 THEN 1
      ELSE 0
    END AS usr_conversion
  FROM users AS usr
  LEFT JOIN activity AS act
    ON act.uid = usr.id
  LEFT JOIN groups AS grp
  	ON grp.uid = usr.id
  --WHERE grp.group IS NULL
  GROUP BY 1,2, usr.id, act.spent
)

SELECT
	CASE
  	WHEN uc.group = 'A' THEN 'Control Group'
    ELSE 'Treatment Group'
  END AS usr_group,
  (SUM(uc.usr_conversion)::numeric/COUNT(uc.usr_conversion)::numeric)*100 AS usr_conversion_rate
FROM UserConversion AS uc
GROUP BY usr_group;
~~~~

Output :

| usr_group       | usr_conversion_rate    |
| --------------- | ---------------------- |
| Control Group   | 3.92309904284599268800 |
| Treatment Group | 4.63008130081300813000 |


### Question 7: What is the average amount spent per user for the control and treatment groups, including users who did not convert?
~~~~sql
WITH AvgSpent AS (
  SELECT
    usr.id,
    --act.uid,
    --grp.uid,
    grp.group,
    SUM(act.spent)::NUMERIC AS usr_spent
  FROM users AS usr
  LEFT JOIN activity AS act
    ON act.uid = usr.id
  LEFT JOIN groups AS grp
    ON grp.uid = usr.id
  --WHERE act.uid IS NOT NULL
  GROUP BY 1,2--,3,4
  ORDER BY 1
)

SELECT
	--avs.group,
  CASE
  	WHEN avs.group = 'A' THEN 'Control Group'
    WHEN avs.group = 'B' THEN 'Treatment Group'
    ELSE 'Unknown'
  END AS usr_group,
	SUM(avs.usr_spent)/COUNT(avs.id) AS avg_spent_per_user
FROM AvgSpent AS avs
GROUP BY 1;
~~~~

Output :

| usr_group       | avg_spent_per_user  |
| --------------- | ------------------- |
| Control Group   | 3.3745184679288412  |
| Treatment Group | 3.39086694588578326 |


### Question 8: Why does it matter to include users who did not convert when calculating the average amount spent per user?
~~~~sql
WITH AvgSpent AS (
  SELECT
    DISTINCT usr.id,
    --act.uid,
    --grp.uid,
    grp.group,
    SUM(act.spent)::NUMERIC AS usr_spent
  FROM users AS usr
  LEFT JOIN activity AS act
    ON act.uid = usr.id
  LEFT JOIN groups AS grp
    ON grp.uid = usr.id
  --WHERE act.uid IS NOT NULL
  GROUP BY 1,2--,3,4
  ORDER BY 1
),
UserConversion AS (  
  SELECT
  	DISTINCT usr.id,
  	grp.group,
    CASE
      WHEN act.spent > 0 THEN 1
      ELSE 0
    END AS usr_conversion
  FROM users AS usr
  LEFT JOIN activity AS act
    ON act.uid = usr.id
  LEFT JOIN groups AS grp
  	ON grp.uid = usr.id
  --WHERE grp.group IS NULL
  GROUP BY 1,2, usr.id, act.spent
)
SELECT
	--avs.group,
  CASE
  	WHEN avs.group = 'A' THEN 'Control Group'
    WHEN avs.group = 'B' THEN 'Treatment Group'
    ELSE 'Unknown'
  END AS usr_group,
	SUM(avs.usr_spent)/COUNT(avs.id) AS avg_spent_per_user,
  (SUM(uc.usr_conversion)::numeric/COUNT(uc.usr_conversion)::numeric)*100 AS usr_conversion_rate
FROM AvgSpent AS avs
LEFT JOIN UserConversion AS uc
	ON uc.id = avs.id
GROUP BY 1;
~~~~

Output :

| usr_group       | avg_spent_per_user  | usr_conversion_rate    |
| --------------- | ------------------- | ---------------------- |
| Control Group   | 3.3745184679288412  | 3.92309904284599268800 |
| Treatment Group | 3.39086694588578326 | 4.63008130081300813000 |


### Question 9: Why does it matter to include users who did not convert when calculating the average amount spent per user?
~~~~sql
WITH AvgSpent AS (
  SELECT
    DISTINCT usr.id,
    --act.uid,
    --grp.uid,
    grp.group,
    SUM(act.spent)::NUMERIC AS usr_spent
  FROM users AS usr
  LEFT JOIN activity AS act
    ON act.uid = usr.id
  LEFT JOIN groups AS grp
    ON grp.uid = usr.id
  --WHERE act.uid IS NOT NULL
  GROUP BY 1,2--,3,4
  ORDER BY 1
),
UserConversion AS (  
  SELECT
  	DISTINCT usr.id,
  	grp.group,
    CASE
      WHEN act.spent > 0 THEN 1
      ELSE 0
    END AS usr_conversion
  FROM users AS usr
  LEFT JOIN activity AS act
    ON act.uid = usr.id
  LEFT JOIN groups AS grp
  	ON grp.uid = usr.id
  --WHERE grp.group IS NULL
  GROUP BY 1,2, usr.id, act.spent
)
SELECT
	--avs.group,
  CASE
  	WHEN avs.group = 'A' THEN 'Control Group'
    WHEN avs.group = 'B' THEN 'Treatment Group'
    ELSE 'Unknown'
  END AS usr_group,
  COUNT(avs.id) AS num_users,
  SUM(uc.usr_conversion)::numeric AS num_users_converted,
  (SUM(uc.usr_conversion)::numeric/COUNT(uc.usr_conversion)::numeric)*100 AS usr_conversion_rate,
	SUM(avs.usr_spent)/COUNT(avs.id) AS avg_spent_per_user
FROM AvgSpent AS avs
LEFT JOIN UserConversion AS uc
	ON uc.id = avs.id
GROUP BY avs.group;
~~~~

Output :

| usr_group       | num_users | num_users_converted | usr_conversion_rate    | avg_spent_per_user  |
| --------------- | --------- | ------------------- | ---------------------- | ------------------- |
| Treatment Group | 24600     | 1139                | 4.63008130081300813000 | 3.39086694588578326 |
| Control Group   | 24343     | 955                 | 3.92309904284599268800 | 3.3745184679288412  |

--------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------


#### Extracting the Analysis Dataset

### Question : Write a SQL query that returns: the user ID, the user’s country, the user’s gender, the user’s device type, the user’s test group, whether or not they converted (spent > $0), and how much they spent in total ($0+).
~~~~sql
SELECT
	usr.id AS user_id,
  COALESCE(usr.country, 'Unknown') AS user_country,
  COALESCE(usr.gender, 'Unknown') AS user_gender,
  COALESCE(grp.device, 'Unknown') AS user_device_type,
  COALESCE(grp.group, 'Unknown') AS user_test_group,
  CASE
  	WHEN act.spent > 0 THEN 1
    ELSE 0
  END AS user_conv,
  SUM(COALESCE(act.spent, 0))::NUMERIC
FROM users AS usr
LEFT JOIN groups AS grp
	ON grp.uid = usr.id
LEFT JOIN activity AS act
	ON act.uid = usr.id
GROUP BY usr.id,grp.device,grp.group,act.spent
ORDER BY usr.id;
~~~~

Output :
| user_id | user_country | user_gender | user_device_type | user_test_group | user_conv | sum |
| ------- | ------------ | ----------- | ---------------- | --------------- | --------- | --- |
| 1000000 | CAN          | M           | I                | B               | 0         | 0   |
| 1000001 | BRA          | M           | A                | A               | 0         | 0   |
| 1000002 | FRA          | M           | A                | A               | 0         | 0   |
| 1000003 | BRA          | M           | I                | B               | 0         | 0   |
| 1000004 | DEU          | F           | A                | A               | 0         | 0   |
...
