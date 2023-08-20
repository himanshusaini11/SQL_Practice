# SQL_Practice
It includes the queries during my learning process.
# Business case - Identifying business areas with the most flight delays

Database link: ```<postgres://Test:bQNxVzJL4g6u@ep-noisy-flower-846766.us-east-2.aws.neon.tech/DA104.1>```

# Question #1: What is the total number of flights that have been delayed versus not delayed for all completed flights?
-Total number of flights, that means one value for each. Hence, I have to use COUNT().
~~~~sql
SELECT
        	CASE WHEN status LIKE 'Completed' AND delayed_flag LIKE 'Y' THEN 'Delayed'
          			WHEN status LIKE 'Completed' AND delayed_flag LIKE 'N' THEN 'Not Delayed'
                ELSE 'Otherwise' END AS flight_status,
        COUNT(flight_id) AS num_delayed
        --status,
        --delayed_flag
FROM ba_flights
--WHERE status LIKE 'Completed' AND delayed_flag LIKE 'Y';
GROUP BY flight_status
ORDER BY num_delayed;
~~~~
