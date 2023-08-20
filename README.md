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
-Total number of flights, that means one value for each. Hence, I have to use ```COUNT()```.
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


### <div style="text-align: justify"> Question 2: The delays could be caused by the type of aircraft and it's design. Look into this? How many flights, out of all flights, have been delayed per manufacturer and aircraft subtype? Make sure that all delayed flights, even if there is no aircraft assigned to the flight and include the capacity of for each aircraft where available. </div>
-We have to count the total number of delaed flights as per aircraft subtype and manufacturer, use ```COUNT()```.
~~~~sql
SELECT
        --baf.status,
        baa.ac_subtype,
        baa.manufacturer,
        bae.capacity,
        COUNT(baf.status) AS num_delayed_flights
FROM ba_flights AS baf
LEFT JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
LEFT JOIN ba_fuel_efficiency AS bae
ON baa.ac_subtype = bae.ac_subtype

WHERE baf.status = 'Completed' AND baf.delayed_flag = 'Y'

GROUP BY baa.ac_subtype, baa.manufacturer, bae.capacity
ORDER BY num_delayed_flights DESC;

~~~~

Output :
| ac_subtype | manufacturer | capacity | num_delayed_flights |
| ---------- | ------------ | -------- | ------------------- |
|            |              |          | 14                  |
| 73H        | Boeing       | 440      | 11                  |
| 772        | Boeing       | 220      | 7                   |
| 73W        | Boeing       | 189      | 6                   |
| E90        | Embraer      | 380      | 5                   |
| 789        | Boeing       | 550      | 3                   |
| E75        | Embraer      | 88       | 3                   |
| 332        | Airbus       | 406      | 3                   |
| 73J        | Boeing       | 330      | 2                   |
| 295        | Boeing       | 400      | 2                   |
