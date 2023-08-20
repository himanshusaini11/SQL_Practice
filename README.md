# SQL_Practice
It includes the queries during my learning process.
## Business case - Identifying business areas with the most flight delays

Database link: ```postgres://Test:bQNxVzJL4g6u@ep-noisy-flower-846766.us-east-2.aws.neon.tech/DA104.1``` \
It containd 4 tables:
1. ba_aircraft
2. ba_flight_routes
3. ba_filghts
4. ba_fuel_efficiency

### Question 1: What is the total number of flights that have been delayed versus not delayed for all completed flights?
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
Output :
| flight_status | num_delayed |
| ------------- | ----------- |
| Delayed       | 56          |
| Not Delayed   | 241         |
| Otherwise     | 715         |


### Question 2: The VP of operations suggested that the delays could be caused by the type of aircraft the passengers are boarding into. Some aircraft designs are easier to board than others. Could you look into this? How many flights, out of all flights, have been delayed per manufacturer and ac subtype? Make sure that all delayed flights, even if there is no aircraft assigned to the flight and include the capacity of for each aircraft where available.

### <div style="text-align: justify"> Question 2: The VP of operations suggested that the delays could be caused by the type of aircraft the passengers are boarding into. Some aircraft designs are easier to board than others. Could you look into this? How many flights, out of all flights, have been delayed per manufacturer and ac subtype? Make sure that all delayed flights, even if there is no aircraft assigned to the flight and include the capacity of for each aircraft where available. </div>
