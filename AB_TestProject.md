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


### Question 3: 
