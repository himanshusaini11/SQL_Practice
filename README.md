# SQL Practice Project on Aireline Data.
It includes the SQL queries during my learning process.
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


### <div style="text-align: justify"> Question 2: The VP of operations suggested that the delays could be caused by the type of aircraft the passengers are boarding into. Some aircraft designs are easier to board than others. Could you look into this? How many flights, out of all flights, have been delayed per manufacturer and ac subtype? Make sure that all delayed flights, even if there is no aircraft assigned to the flight and include the capacity of for each aircraft where available. </div>
-We have to count the total number of delaed flights as per aircraft subtype and manufacturer, use ```COUNT()```.
~~~~sql
SELECT
        --baf.status,
        baa.ac_subtype,
        baa.manufacturer,
        bae.capacity,
        COUNT(baf.flight_id) AS num_delayed_flights
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


### Question 3: The head of international flights tends to be a little selfish and therefore its no surprise he wants to assess whether the international flights are not causing the delays. Could you analyse the number of delays (compared against non-delayed flights) only for the completed flights that has a capacity over 300 passengers? These airplanes tend to only be used for international flights.
~~~sql
SELECT
	CASE WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'Y' THEN 'Delayed'
      		WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'N' THEN 'Not Delayed'
	END AS flight_status,
	COUNT(baf.flight_id) AS num_flights
FROM ba_flights AS baf
LEFT JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
INNER JOIN ba_fuel_efficiency AS bae
ON baa.ac_subtype = bae.ac_subtype
WHERE bae.capacity > 300 AND baf.status LIKE 'Completed'
GROUP BY flight_status;
~~~~

Output :
| flight_status | num_flights |
| ------------- | ----------- |
| Not Delayed   | 131         |
| Delayed       | 26          |


### Question 4: Even though the query easily compares delayed versus not-delayed flights in question 3, we still don’t have an actual comparison against flights with a lower passenger capacity. Reuse the query of question 3, but instead of filtering for capacity over 300 passengers, create a column that segments aircraft below or equal to 300 and over 300. How do the delays in total number of flights compare between the two segments?
~~~~sql
SELECT
	CASE WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'Y' THEN 'Delayed'
      		WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'N' THEN 'Not Delayed'
	END AS flight_status,
	CASE WHEN bae.capacity <= 300 THEN 'Below or equal 300'
      	ELSE 'Above 300' END AS flight_segments,
	COUNT(baf.flight_id) AS num_flights
FROM ba_flights AS baf
LEFT JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
INNER JOIN ba_fuel_efficiency AS bae
ON baa.ac_subtype = bae.ac_subtype
WHERE baf.status LIKE 'Completed'
GROUP BY flight_status, flight_segments
ORDER BY flight_segments;
~~~~

Output :
| flight_status | flight_segments    | num_flights |
| ------------- | ------------------ | ----------- |
| Delayed       | Above 300          | 26          |
| Not Delayed   | Above 300          | 131         |
| Delayed       | Below or equal 300 | 16          |
| Not Delayed   | Below or equal 300 | 69          |


### Question 5: Your manager suggests to analyse whether specific locations are causing the delays. Are certain airports better organized than others? Let’s find out. What departure cities have experienced the highest number of delays in all the completed flights? Only include flights that operates with the manufacturers Boeing and Airbus.
~~~~sql
SELECT
	CASE WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'Y' THEN 'Delayed'
      		WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'N' THEN 'Not Delayed'
	END AS flights_status,
      	bar.departure_city,
      	COUNT(baf.flight_id) AS num_flights
FROM ba_flights AS baf
LEFT JOIN ba_flight_routes AS bar
ON baf.flight_number = bar.flight_number
INNER JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
WHERE baf.status = 'Completed' AND baf.delayed_flag = 'Y'
	AND (baa.manufacturer = 'Boeing' OR baa.manufacturer = 'Airbus')
GROUP BY bar.departure_city, baf.status, baf.delayed_flag
ORDER BY num_flights DESC;
~~~~

Output :
| flights_status | departure_city | num_flights |
| -------------- | -------------- | ----------- |
| Delayed        | London         | 11          |
| Delayed        | Krakow         | 2           |
| Delayed        | Nantes         | 2           |
| Delayed        | Dammam         | 2           |
| Delayed        | Basel          | 2           |
| Delayed        | Aberdeen       | 2           |
| Delayed        | Trondheim      | 2           |
| Delayed        | Manila         | 1           |
| Delayed        | Port of Spain  | 1           |
| Delayed        | Stockholm      | 1           |
| Delayed        | São Paulo      | 1           |
| Delayed        | Linköping      | 1           |
| Delayed        | Bergen         | 1           |
| Delayed        | Dubai          | 1           |
| Delayed        | Helsinki       | 1           |
| Delayed        | Kuala Lumpur   | 1           |
| Delayed        | Aarhus         | 1           |
| Delayed        | Lisbon         | 1           |


### Question 6: It looks like we have found a potential area where delays are happening more frequently. You remember that there are some flights did not have any aircraft assigned in the data at the moment. Because of that, you would like to check whether the number of delays increases even more if we include these flights with missing aircraft assignment. What is the total number of delays of all completed flights per manufacturer that departed from London?
~~~~sql
SELECT
	CASE WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'Y' THEN 'Delayed'
      		--WHEN baf.status LIKE 'Completed' AND baf.delayed_flag LIKE 'N' THEN 'Not Delayed'
      	END AS flights_status,
      	baa.manufacturer,
      	--bar.departure_city,
      	COUNT(baf.flight_id) AS num_flights
FROM ba_flights AS baf
LEFT JOIN ba_flight_routes AS bar
ON baf.flight_number = bar.flight_number
LEFT JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
WHERE baf.status = 'Completed' AND baf.delayed_flag = 'Y'
	AND bar.departure_city LIKE 'London'
GROUP BY baa.manufacturer, bar.departure_city, baf.status, baf.delayed_flag
ORDER BY num_flights DESC;
~~~~

Output :
| flights_status | manufacturer | num_flights |
| -------------- | ------------ | ----------- |
| Delayed        | Boeing       | 9           |
| Delayed        |              | 5           |
| Delayed        | Embraer      | 3           |
| Delayed        | Airbus       | 2           |


### Question 7: Create a list of flights, showing the flight ID, departure city, arrival city, manufacturer, and aircraft sub-type that will be used for each flight. Show the results for all flights that are available even if not all information is available for all flights.
~~~sql
SELECT
	baf.flight_id,
      	bar.departure_city,
      	bar.arrival_city,
      	baa.manufacturer,
      	baa.ac_subtype
FROM ba_flights AS baf
LEFT JOIN ba_flight_routes AS bar
ON baf.flight_number = bar.flight_number

LEFT JOIN ba_aircraft AS baa
ON baf.flight_id = baa.flight_id
LIMIT 10; -- To get the required answer remove the ```LIMIT```. I used just to show the output.
~~~

Output :
| flight_id | departure_city | arrival_city | manufacturer | ac_subtype |
| --------- | -------------- | ------------ | ------------ | ---------- |
| AAAA01    | Stockholm      | London       | Boeing       | 73H        |
| AAAA02    | London         | Riyadh       |              |            |
| AAAA03    | London         | Riyadh       |              |            |
| AAAA04    | Dammam         | London       |              |            |
| AAAA05    | London         | Dubai        |              |            |
| AAAA06    | Dubai          | London       |              |            |
| AAAA07    | Dubai          | London       | Airbus       | 332        |
| AAAA08    | London         | Kuwait City  | Airbus       | 332        |
| AAAA09    | London         | Kuwait City  | Airbus       | 332        |
| AAAA10    | Muscat         | Kuwait City  | Airbus       | 332        |


### Question 8: What is the maximum number of passengers that have been on every available aircraft (manufacturer and sub-type) for flights that have been completed? If the manufacturer and sub-type are not available for flights, we do not need to show the results of these flights.
~~~~sql
SELECT
      baa.manufacturer,
      baa.ac_subtype,
      MAX(baf.total_passengers) AS max_passengers
FROM ba_flights AS baf
INNER JOIN ba_aircraft AS baa
ON baa.flight_id = baf.flight_id
WHERE baf.status = 'Completed'
GROUP BY baa.manufacturer, baa.ac_subtype
ORDER BY max_passengers;
~~~~

Output :
| manufacturer | ac_subtype | max_passengers |
| ------------ | ---------- | -------------- |
| Boeing       | 73W        | 264            |
| Boeing       | 789        | 264            |
| Embraer      | E90        | 277            |
| Boeing       | 73H        | 326            |
| Boeing       | 772        | 373            |
| Airbus       | 332        | 373            |
| Boeing       | 295        | 373            |
| Embraer      | E75        | 376            |
| Boeing       | 73J        | 376            |


### Question 9: Since only some aircraft are capable of flying long distances overseas, we want to filter out the planes that only do shorter distances. What aircraft (manufacturer and sub-type) have completed flights of a distance of more than 7,000 km? If the manufacturer and sub-type are not available for flights, we do not need to show the results of these flights.
~~~~sql
SELECT
      DISTINCT baa.manufacturer,
      baa.ac_subtype,
      SUM(bar.distance_flown) AS distance_flown
FROM ba_flights AS baf
INNER JOIN ba_aircraft AS baa
ON baa.flight_id = baf.flight_id
INNER JOIN ba_flight_routes AS bar
ON bar.flight_number = baf.flight_number
WHERE baf.status = 'Completed'
	AND bar.distance_flown > 7000
GROUP BY baa.manufacturer, baa.ac_subtype;
~~~~

Output :
| manufacturer | ac_subtype | distance_flown |
| ------------ | ---------- | -------------- |
| Airbus       | 332        | 19550          |
| Boeing       | 295        | 17624          |
| Boeing       | 73H        | 18587          |
| Boeing       | 772        | 19102          |
| Boeing       | 789        | 28757          |
| Embraer      | E90        | 9775           |
