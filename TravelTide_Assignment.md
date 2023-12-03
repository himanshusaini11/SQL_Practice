## Question 1: Demographics and travel patterns
### (a) Which cross-section of age and gender travels the most.
~~~~sql
SELECT
	usr.user_id,
  usr.gender,
	EXTRACT(YEAR FROM AGE(CURRENT_DATE, usr.birthdate)) AS age,
  COUNT(s.flight_booked) AS num_times_travel
FROM users AS usr
LEFT JOIN sessions AS s
	ON s.user_id = usr.user_id
WHERE s.flight_booked = true
	AND s.cancellation = false
GROUP BY usr.user_id, usr.gender
ORDER BY num_times_travel DESC
LIMIT 10;
~~~~

Output: 
| user_id | gender | age | num_times_travel |
| ------- | ------ | --- | ---------------- |
| 54318   | F      | 50  | 11               |
| 50009   | F      | 39  | 11               |
| 29241   | F      | 50  | 11               |
| 144345  | F      | 38  | 11               |
| 26048   | F      | 41  | 11               |
| 12684   | F      | 39  | 10               |
| 44403   | F      | 53  | 10               |
| 1469    | F      | 48  | 10               |
| 807     | F      | 41  | 10               |
| 7030    | M      | 47  | 10               |

### (b) How does the travel behavior of customers married with children compare to childless single customers?
~~~~sql
